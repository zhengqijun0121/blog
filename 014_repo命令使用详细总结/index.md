# Repo 命令使用详细总结


# Repo 命令使用详细总结
Repo 是 Google 基于 Python 开发的**多 Git 仓库批量管理工具**，并非 Git 的替代品，而是对 Git 命令的上层封装，专为 AOSP（Android 开源项目）这类包含数百个独立 Git 仓库的超大型项目设计，核心通过 `manifest.xml` 清单文件统一管理所有子仓库的地址、分支、路径等配置，实现一键式批量同步、分支操作、代码提交流程。

## 一、核心基础
### 1.1 核心原理
Repo 的核心是**清单文件（manifest.xml）**，它就像项目的"导航图"，精准定义了每个子 Git 仓库的远程地址、本地存放路径、分支/提交哈希、仓库分组等规则。Repo 通过解析这份清单，自动批量执行底层 Git 命令，彻底告别逐个仓库手动操作的低效模式。

### 1.2 初始化后的 .repo 目录结构
执行 `repo init` 后，会在当前目录生成隐藏的 `.repo` 目录，这是 Repo 的工作核心，所有仓库数据和配置都存放于此：
```
.repo/
├── manifests/          # 清单仓库的本地Git库，存放所有manifest.xml文件
├── manifests.git/      # 清单仓库的裸仓库，manifests/.git的软链接目标
├── manifest.xml        # 当前生效的清单文件软链接，默认指向manifests/default.xml
├── repo/               # Repo工具自身的Python源码与执行脚本
├── projects/           # 所有子仓库的裸仓库，工作目录的.git均为指向这里的软链接
└── project-objects/    # 项目对象共享存储，用于减少磁盘占用，优化多仓库拉取
```
**关键说明**：工作目录中每个子仓库的 `.git` 都是软链接，实际数据均存放在 `.repo` 目录中，因此所有原生 Git 命令在子仓库中均可正常使用。

### 1.3 环境依赖与安装
#### 前置依赖
- Python 3.6+（推荐 3.8+）
- Git 1.7.2+（推荐 2.0+）
- 网络可访问对应清单仓库（国内建议配置镜像源）

#### 安装步骤（Linux/macOS/WSL）
1. 创建执行目录并配置环境变量
```bash
mkdir -p ~/bin
echo 'export PATH="$HOME/bin:$PATH"' >> ~/.bashrc
# 若使用zsh，执行 echo 'export PATH="$HOME/bin:$PATH"' >> ~/.zshrc
source ~/.bashrc
```

2. 下载 Repo 脚本并赋予执行权限
```bash
# 官方源
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
# 国内清华镜像源（推荐）
# curl https://mirrors.tuna.tsinghua.edu.cn/git-repo-downloads/repo > ~/bin/repo

chmod a+x ~/bin/repo
```

3. 验证安装
```bash
repo --version
```

## 二、核心命令全详解
Repo 命令通用格式：`repo <COMMAND> <OPTIONS> [project-list]`
其中 `project-list` 可指定仓库名/本地路径，不指定则默认作用于所有仓库；`.` 代表当前目录所在仓库。

### 2.1 帮助命令 `repo help`
**核心作用**：查看所有 Repo 命令列表，或指定命令的详细用法、参数说明
```bash
# 查看所有命令概览
repo help

# 查看指定命令的详细帮助（如init）
repo help init
# 等价于
repo init --help
```

### 2.2 初始化命令 `repo init`
**核心作用**：在当前目录初始化 Repo 环境，下载清单仓库，创建 `.repo` 工作目录，是所有 Repo 操作的第一步。
#### 基础用法
```bash
repo init -u <manifest-url> [OPTIONS]
```
#### 核心必选参数
| 参数 | 全称 | 说明 |
|------|------|------|
| `-u` | `--manifest-url` | 指定清单仓库（manifest）的远程URL，必填项 |

#### 高频可选参数
| 参数 | 全称 | 说明 |
|------|------|------|
| `-b` | `--manifest-branch` | 指定清单仓库的分支/修订版本，默认master |
| `-m` | `--manifest-name` | 指定使用的清单文件名，默认default.xml |
| `-g` | `--groups` | 指定要同步的仓库分组，多个分组用逗号分隔，默认拉取全部分组 |
| `--depth=1` | - | 浅克隆，仅拉取最新1个提交，大幅减少磁盘占用和拉取时间，适合仅编译场景 |
| `--no-repo-verify` | - | 跳过Repo脚本自身的签名校验，解决初始化校验失败问题 |
| `--repo-url` | - | 指定Repo工具自身的源码仓库地址，替换官方源，适配国内环境 |
| `--no-tags` | - | 不拉取远程标签，减少拉取数据量 |

#### 常用示例
```bash
# AOSP官方初始化示例
repo init -u https://android.googlesource.com/platform/manifest -b android-14.0.0_r29

# 国内镜像初始化（清华源）
repo init -u https://mirrors.tuna.tsinghua.edu.cn/aosp/platform/manifest -b android-14.0.0_r29 --depth=1

# 指定自定义清单文件
repo init -u git@xxx.com:xxx/manifest.git -b main -m my_project.xml
```

### 2.3 代码同步命令 `repo sync`
**核心作用**：根据清单文件，批量拉取/更新所有子仓库代码，是 Repo 最常用的核心命令。
#### 基础用法
```bash
repo sync [OPTIONS] [project-list]
```
#### 核心高频参数
| 参数 | 全称 | 说明 |
|------|------|------|
| `-j <num>` | - | 指定并行拉取的线程数，建议设置为CPU核心数的1-2倍（如-j8），大幅提升同步速度 |
| `-c` | `--current-branch` | 仅拉取清单中指定的当前分支，不拉取远程所有分支，减少数据量 |
| `-d` | `--detach` | 将指定仓库强制切回清单中指定的修订版本，丢弃本地分支变更，解决同步冲突 |
| `-f` | `--force-broken` | 即使某个仓库同步失败，也继续同步其他仓库 |
| `-q` | `--quiet` | 静默模式，抑制非必要的状态输出 |
| `-l` | `--local-only` | 仅本地更新，不执行远程fetch，适合本地多分支切换场景 |
| `-r` | `--rebase` | 同步时用rebase模式合并远程变更，默认行为 |

#### 常用示例
```bash
# 全量同步所有仓库（最常用）
repo sync -j8 -c

# 仅同步指定单个仓库
repo sync platform/frameworks/base

# 强制同步，丢弃本地变更，切回清单版本
repo sync -d platform/frameworks/base

# 静默同步，失败不中断
repo sync -j16 -c -q -f
```
#### 关键说明
- 首次执行 `repo sync` 等价于对所有仓库执行 `git clone`；非首次执行等价于 `git remote update && git rebase origin/目标分支`
- 若rebase出现冲突，需进入对应仓库，用原生Git命令（`git rebase --continue`/`git rebase --abort`）解决冲突后，重新执行同步

### 2.4 分支管理命令
Repo 提供了批量分支操作能力，替代逐个仓库执行Git分支命令的繁琐操作。

#### 2.4.1 创建开发分支 `repo start`
**核心作用**：基于清单中指定的基准版本，为指定/所有仓库创建并切换到新的开发分支，是开发前的必备操作（底层封装 `git checkout -b`）。
```bash
# 基础用法
repo start <分支名> [project-list]

# 为所有仓库创建名为my_feature的开发分支
repo start my_feature --all

# 仅为当前目录仓库创建分支
repo start my_feature .

# 仅为指定仓库创建分支
repo start my_feature platform/frameworks/base packages/apps/Settings
```

#### 2.4.2 切换分支 `repo checkout`
**核心作用**：批量切换指定/所有仓库到已存在的分支（底层封装 `git checkout`）
```bash
# 基础用法
repo checkout <分支名> [project-list]

# 所有仓库切换到main分支
repo checkout main

# 指定仓库切换到my_feature分支
repo checkout my_feature platform/frameworks/base
```

#### 2.4.3 删除分支 `repo abandon`
**核心作用**：强制删除指定/所有仓库的指定分支（底层封装 `git branch -D`）
```bash
# 基础用法
repo abandon <分支名> [project-list]

# 删除所有仓库的my_feature分支
repo abandon my_feature --all

# 删除指定仓库的test分支
repo abandon test platform/frameworks/base
```

#### 2.4.4 清理已合并分支 `repo prune`
**核心作用**：自动扫描并删除所有已合并到基准分支的无用分支，清理仓库冗余分支（底层封装 `git branch -d`）
```bash
# 基础用法
repo prune [project-list]

# 清理所有仓库的已合并分支
repo prune

# 清理指定仓库的已合并分支
repo prune platform/frameworks/base
```

### 2.5 代码变更查看命令
#### 2.5.1 查看仓库状态 `repo status`
**核心作用**：批量查看指定/所有仓库的工作区、暂存区状态，显示文件的修改、新增、删除、冲突等状态。
```bash
# 基础用法
repo status [project-list]

# 查看所有仓库的状态
repo status

# 查看当前目录仓库的状态
repo status .

# 查看指定仓库的状态
repo status platform/frameworks/base
```
**状态码说明**：输出为两列状态码，第一列大写字母表示暂存区与HEAD的差异，第二列小写字母表示工作区与暂存区的差异
| 状态码 | 含义 |
|--------|------|
| A/a | 新增文件 |
| M/m | 内容修改 |
| D/d | 文件删除 |
| R/r | 文件重命名 |
| C/c | 文件复制 |
| T/t | 文件权限模式变更 |
| U | 未合并的冲突（必须先解决） |
| - | 无变更 |

#### 2.5.2 查看代码差异 `repo diff`
**核心作用**：批量查看指定/所有仓库的工作区与暂存区的代码差异（底层封装 `git diff`）
```bash
# 基础用法
repo diff [project-list]

# 查看所有仓库的未暂存修改
repo diff

# 查看当前仓库的修改差异
repo diff .

# 查看指定仓库的修改差异
repo diff platform/frameworks/base
```

#### 2.5.3 交互式暂存 `repo stage`
**核心作用**：交互式挑选文件修改并加入暂存区（底层封装 `git add --interactive`），适合精细化提交
```bash
# 基础用法
repo stage -i [project-list]

# 为当前仓库交互式暂存修改
repo stage -i .
```

### 2.6 批量执行命令 `repo forall`
**核心作用**：Repo 最强大的批量操作命令，可在指定/所有仓库中执行任意Shell命令，支持内置环境变量，实现高度自定义的批量操作。
#### 基础用法
```bash
repo forall [project-list] -c "<shell命令>" [OPTIONS]
```
#### 核心内置环境变量
| 变量名 | 说明 |
|--------|------|
| `REPO_PROJECT` | 当前仓库的唯一名称 |
| `REPO_PATH` | 当前仓库相对于根目录的本地路径 |
| `REPO_REMOTE` | 清单中定义的远程仓库名 |
| `REPO_LREV` | 清单中定义的本地跟踪分支名 |
| `REPO_RREV` | 清单中定义的远程修订版本号 |

#### 核心参数
| 参数 | 说明 |
|------|------|
| `-c` | 要执行的Shell命令，必填项，命令用引号包裹 |
| `-p` | 在每个仓库的命令输出前添加仓库标头，清晰区分输出归属 |
| `-v` | 显示命令执行时的stderr错误输出 |

#### 高频实用示例
```bash
# 批量为所有仓库执行git add + git commit
repo forall -p -c 'git add -A && git commit -m "feat: 统一功能更新"'

# 批量查看所有仓库的当前分支
repo forall -p -c 'git branch --show-current'

# 批量储藏所有仓库的未提交修改
repo forall -p -c 'git stash'

# 批量恢复储藏的修改
repo forall -p -c 'git stash pop'

# 批量拉取当前分支的远程最新变更
repo forall -p -c 'git pull origin $(git branch --show-current)'
```

### 2.7 代码审核相关命令
Repo 深度集成 Gerrit 代码审核系统，提供一键式提交、补丁下载能力，是团队协作的核心命令。

#### 2.7.1 上传审核 `repo upload`
**核心作用**：将本地仓库的提交批量上传到 Gerrit 审核系统，生成审核单（CL）。
```bash
# 基础用法
repo upload [OPTIONS] [project-list]
```
#### 核心参数
| 参数 | 全称 | 说明 |
|------|------|------|
| `--cbr` | `--current-branch` | 仅上传当前检出的分支，避免误上传其他分支 |
| `-t` | - | 自动将Gerrit的主题名设置为本地分支名 |
| `--topic=<主题名>` | - | 为上传的CL指定统一主题，方便关联多个提交 |
| `--replace` | - | 替换已上传的CL，生成新的补丁集（Patch Set），用于修改后重新提交 |
| `-d` | `--draft` | 上传为草稿CL，仅自己可见 |

#### 常用示例
```bash
# 上传当前仓库的当前分支修改
repo upload . --cbr

# 上传指定仓库的修改，指定主题
repo upload platform/frameworks/base --topic=my_feature

# 替换已上传的CL，重新提交修改
repo upload --replace platform/frameworks/base
```
#### 关键说明
- 上传前必须先执行 `repo start` 创建开发分支，否则会提示「无分支可上传」
- 每个提交会生成一个独立的Gerrit审核单，若需合并多个提交为一个，需先执行 `git rebase -i` 压缩提交

#### 2.7.2 下载补丁 `repo download`
**核心作用**：从Gerrit下载指定的审核补丁（CL）到本地仓库，用于代码评审、测试验证。
```bash
# 基础用法
repo download <目标仓库> <CL编号>

# 示例：下载platform/build仓库的23823号CL
repo download platform/build 23823
```
**注意**：执行 `repo sync` 会移除通过 `repo download` 下载的补丁，如需永久保留，需手动执行 `git cherry-pick` 合并提交。

## 三、标准开发工作流
完整的 Repo 日常开发流程如下，适配团队协作+Gerrit审核模式：
1. **初始化环境**
```bash
mkdir my_project && cd my_project
repo init -u <manifest-url> -b <分支>
```

2. **同步代码**
```bash
repo sync -j8 -c
```

3. **创建开发分支**
```bash
# 全仓库创建
repo start my_feature --all
# 仅修改的仓库创建
repo start my_feature platform/frameworks/base packages/apps/Settings
```

4. **代码开发**
进入对应仓库，进行代码修改，使用原生Git命令执行add、commit操作
```bash
cd platform/frameworks/base
# 代码修改后
git add -A
git commit -m "feat: 新增xxx功能"
```

5. **查看修改与状态**
```bash
repo status
repo diff
```

6. **上传到Gerrit审核**
```bash
repo upload . --cbr --topic=my_feature
```

7. **审核修改后重新提交**
```bash
# 修改代码后，更新提交（不要新建提交）
git commit --amend
# 替换已上传的CL
repo upload --replace . --cbr
```

8. **开发完成，清理分支**
```bash
# 代码合并后，清理已合并分支
repo prune
```

## 四、进阶用法与最佳实践
### 4.1 国内环境优化
1. **镜像源配置**：清单仓库、Repo源码均替换为国内镜像（清华、中科大等），解决拉取失败、速度慢问题
2. **代理配置**：若需访问官方源，可配置HTTP/HTTPS代理
```bash
export HTTP_PROXY=http://proxy-ip:port
export HTTPS_PROXY=http://proxy-ip:port
```
3. **浅克隆优化**：仅编译场景使用 `--depth=1` 初始化，大幅减少磁盘占用和拉取时间

### 4.2 同步优化
1. **并行数设置**：`-j` 参数建议设置为CPU核心数的1.5倍，避免过高导致IO/网络瓶颈
2. **增量同步**：日常开发仅同步修改的仓库，无需全量同步 `repo sync <仓库名>`
3. **强制同步**：出现分支混乱、冲突时，使用 `repo sync -d` 强制切回清单基准版本

### 4.3 分支管理最佳实践
1. **必须用 `repo start` 创建分支**：不要直接用 `git checkout -b` 创建分支，否则Repo无法跟踪分支，会导致upload失败
2. **单功能单分支**：每个功能/修复创建独立的主题分支，避免多个功能混在一个分支，影响审核和合并
3. **定期清理**：开发完成后，及时用 `repo prune` 清理已合并分支，避免分支冗余

### 4.4 批量操作规范
1. 复杂批量操作先用单个仓库测试，再全量执行，避免误操作影响所有仓库
2. 批量提交前，先通过 `repo forall -p -c 'git status'` 检查所有仓库状态
3. 批量执行Git命令时，优先使用Repo内置的环境变量，保证命令通用性

## 五、常见问题与解决方案
### 1. repo init 失败，提示无法连接到官方源
- 解决方案：替换为国内镜像源，或配置代理；添加 `--no-repo-verify` 跳过签名校验
- 示例：`repo init -u https://mirrors.tuna.tsinghua.edu.cn/aosp/platform/manifest --no-repo-verify`

### 2. repo sync 过程中某个仓库拉取失败，中断同步
- 解决方案：添加 `-f` 参数跳过失败仓库，继续同步其他仓库；单独同步失败的仓库
- 示例：`repo sync -f -j8`，之后单独执行 `repo sync 失败的仓库名`

### 3. repo upload 提示 "no branches ready for upload"
- 解决方案：未通过 `repo start` 创建跟踪分支，需恢复提交并重新创建分支
```bash
# 先保存当前提交
git commit -a -m "save changes"
# 基于当前提交创建开发分支
repo start my_feature .
# 重新上传
repo upload . --cbr
```

### 4. repo sync 时出现 rebase 冲突
- 解决方案：进入冲突仓库，解决冲突后继续同步，或丢弃本地变更强制同步
```bash
# 方法1：解决冲突后继续
cd 冲突仓库路径
git add 冲突文件
git rebase --continue
# 回到根目录重新执行 repo sync

# 方法2：丢弃本地变更，强制切回基准版本
repo sync -d 冲突仓库名
```

### 5. 提示 repo: 未找到命令
- 解决方案：检查 `~/bin` 是否加入PATH环境变量，重新执行source命令；检查repo脚本是否有可执行权限
```bash
chmod a+x ~/bin/repo
source ~/.bashrc
```


