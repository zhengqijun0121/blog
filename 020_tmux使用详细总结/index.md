# tmux 命令使用详细总结


# tmux 使用完全总结
tmux 是一款经典的**终端复用器（Terminal Multiplexer）**，核心能力是在单个终端窗口中创建、管理多个独立的终端会话，同时实现会话后台保活、窗口分屏、多任务并行处理，是远程SSH运维、服务器开发、多任务命令行操作的必备工具。

## 一、核心概念与架构
tmux 采用三级层级设计，所有操作都围绕这三个核心单元展开，理解层级关系是上手的关键：
| 层级 | 全称 | 核心说明 |
|------|------|----------|
| 会话 | Session | tmux 的最高层级，对应一个独立的任务集，**关闭终端/SSH断连后会话仍在后台运行**，可随时重连恢复。一个会话可包含多个窗口 |
| 窗口 | Window | 对应终端的一个全屏页面，类似浏览器的标签页，底部状态栏会显示窗口编号与名称。一个窗口可包含多个窗格 |
| 窗格 | Pane | 窗口内分割出的最小终端单元，可上下/左右拆分，实现单屏同时显示多个终端界面 |

### 核心优势
1.  **会话持久化**：SSH远程操作时，网络断开、终端关闭不会中断正在运行的程序，重连即可恢复
2.  **多屏分栏**：单窗口拆分多个窗格，无需打开多个终端，同时执行监控、编辑、运行等多任务
3.  **多会话管理**：按项目/场景创建独立会话，快速切换不同的工作环境
4.  **协作与共享**：支持多人同时接入同一个会话，实现结对编程、远程协助
5.  **高度可定制**：支持快捷键、状态栏、功能插件的深度自定义，适配个人工作流

## 二、安装教程
主流操作系统均已内置 tmux 软件源，直接通过包管理器安装即可：
```bash
# Ubuntu/Debian 系列
sudo apt update && sudo apt install -y tmux

# CentOS/RHEL/Rocky Linux 系列
sudo yum install -y tmux
# 或 dnf 版本
sudo dnf install -y tmux

# macOS（需先安装 Homebrew）
brew install tmux

# Arch Linux 系列
sudo pacman -S tmux
```

安装完成后，终端输入 `tmux -V` 可查看版本，确认安装成功。

## 三、基础核心操作
tmux 所有快捷键都基于**前缀键**触发，默认前缀键为 `Ctrl+b`。
> 关键操作规范：先同时按下 `Ctrl` 和 `b`，**完全松开两个按键**，再按下对应的功能键，不要同时按住前缀和功能键。

### 3.1 会话管理（最核心，持久化能力的核心）
会话操作分为「终端外命令行操作」和「tmux内快捷键操作」两类，前者用于未进入tmux时管理会话，后者用于tmux内快速操作。

| 操作场景 | 命令行操作（终端外执行） | 快捷键操作（前缀+按键） | 功能说明 |
|----------|----------------------------|--------------------------|----------|
| 新建会话 | `tmux` / `tmux new` | - | 创建无名称的新会话，自动按数字编号 |
| 新建命名会话 | `tmux new -s 会话名` | - | 创建带自定义名称的会话，便于识别和管理 |
| 后台新建会话 | `tmux new -s 会话名 -d` | - | 创建会话并直接放在后台运行，不立即进入 |
| 查看所有会话 | `tmux ls` / `tmux list-sessions` | `Ctrl+b s` | 列出所有运行中的会话，快捷键可交互式选择切换 |
| 接入/重连会话 | `tmux a -t 会话名/编号` / `tmux attach -t 会话名` | - | 进入指定的后台会话，恢复操作界面 |
| 分离会话（后台保活） | `tmux detach` | `Ctrl+b d` | 退出当前会话，会话和其中运行的程序继续在后台运行 |
| 重命名当前会话 | `tmux rename-session -t 旧名 新名` | `Ctrl+b $` | 修改会话名称，便于区分不同任务 |
| 切换会话 | `tmux switch -t 会话名` | `Ctrl+b ( / )` | 切换到上一个/下一个会话 |
| 关闭指定会话 | `tmux kill-session -t 会话名` | - | 彻底关闭指定会话，其中所有窗口/窗格和程序都会终止 |
| 关闭所有会话 | `tmux kill-server` | - | 彻底关闭tmux服务，所有会话全部终止 |

### 3.2 窗口管理（标签页式管理）
一个会话可创建多个窗口，每个窗口都是一个独立的全屏终端，底部状态栏会显示窗口列表。

| 操作场景 | 快捷键操作 | 功能说明 |
|----------|------------|----------|
| 新建窗口 | `Ctrl+b c` | 创建新窗口，自动按数字编号，状态栏会新增标签 |
| 重命名当前窗口 | `Ctrl+b ,` | 修改窗口名称，便于区分用途（如代码、日志、监控） |
| 切换到上一个窗口 | `Ctrl+b p` | 切换到编号更小的上一个窗口 |
| 切换到下一个窗口 | `Ctrl+b n` | 切换到编号更大的下一个窗口 |
| 切换到指定编号窗口 | `Ctrl+b 0-9` | 直接跳转到对应数字编号的窗口 |
| 切换到上一个活跃窗口 | `Ctrl+b Tab` | 快速在最近两个窗口之间来回切换 |
| 可视化选择窗口 | `Ctrl+b w` | 打开会话/窗口树状列表，方向键选择回车进入 |
| 关闭当前窗口 | `Ctrl+b &` | 关闭当前窗口，需输入 `y` 确认，窗口内所有窗格一并关闭 |

### 3.3 窗格管理（分屏核心）
将单个窗口拆分为多个独立的终端窗格，实现单屏多任务并行，是tmux最常用的功能之一。

| 操作场景 | 快捷键操作 | 功能说明 |
|----------|------------|----------|
| 上下水平分屏 | `Ctrl+b "` | 将当前窗格上下拆分为两个，新窗格在下方 |
| 左右垂直分屏 | `Ctrl+b %` | 将当前窗格左右拆分为两个，新窗格在右侧 |
| 切换窗格 | `Ctrl+b 方向键` / `Ctrl+b hjkl` | 按方向键切换到对应相邻窗格，hjkl为vi风格方向键（h左、j下、k上、l右） |
| 顺时针切换窗格 | `Ctrl+b o` | 按顺时针顺序循环切换当前窗口内的所有窗格 |
| 窗格全屏/还原 | `Ctrl+b z` | 将当前窗格临时全屏显示，再次按下恢复原布局 |
| 关闭当前窗格 | `Ctrl+b x` | 关闭当前选中的窗格，需输入 `y` 确认；窗格内无程序时也可直接 `Ctrl+d` 关闭 |
| 调整窗格大小 | `Ctrl+b Ctrl+方向键` | 按方向键微调窗格大小，长按可连续调整 |
| 循环切换布局 | `Ctrl+b 空格键` | 自动切换预设的窗格布局（平铺、主从、均等分等） |
| 显示窗格编号 | `Ctrl+b q` | 显示所有窗格的编号，按下对应数字可直接跳转到该窗格 |
| 交换窗格位置 | `Ctrl+b {` / `Ctrl+b }` | 将当前窗格与上一个/下一个窗格交换位置 |

### 3.4 复制粘贴与历史查看
tmux 有独立的复制模式，用于查看终端历史输出、复制文本，解决默认无法滚动查看历史的问题。

1.  **进入复制模式**：按下 `Ctrl+b [` 进入复制模式，此时可通过方向键、PageUp/PageDown 滚动查看终端历史输出。
2.  **复制操作（默认emacs模式）**：
    - 光标移动到复制起始位置，按下 `Ctrl+空格` 开始选中
    - 移动光标到结束位置，按下 `Alt+w` 复制选中内容到tmux缓冲区，自动退出复制模式
3.  **粘贴操作**：按下 `Ctrl+b ]`，将缓冲区中最新复制的内容粘贴到当前光标位置。
4.  **vi模式复制**：若配置了vi模式，进入复制模式后，按 `空格` 开始选中，按 `Enter` 完成复制，粘贴操作不变。

## 四、进阶用法与配置优化
### 4.1 核心配置文件
tmux 的所有配置都通过 `~/.tmux.conf` 文件实现，该文件不存在时可手动创建，修改配置后，需重启tmux服务，或在tmux内执行 `Ctrl+b :` 输入 `source-file ~/.tmux.conf` 重载配置生效。

以下是生产级常用优化配置，可直接复制使用：
```conf
# ==============================================
# 基础环境配置
# ==============================================
# 设置终端为256色，解决颜色显示异常
set -g default-terminal "screen-256color"
# 增大历史缓冲区行数，默认2000行，改为5万行，方便查看历史输出
set -g history-limit 50000
# 窗口/窗格编号从1开始，默认从0开始，更符合键盘操作直觉
set -g base-index 1
setw -g pane-base-index 1
# 关闭窗口自动重命名，避免自定义的窗口名被程序覆盖
setw -g automatic-rename off
# 新建窗格时继承当前窗格的工作路径，无需重复cd
bind _ split-window -v -c "#{pane_current_path}"
bind - split-window -h -c "#{pane_current_path}"

# ==============================================
# 快捷键优化
# ==============================================
# 可选：将前缀键改为 Ctrl+a，比 Ctrl+b 更符合左手操作习惯，与screen兼容
# unbind C-b
# set -g prefix C-a
# bind C-a send-prefix

# 优化分屏快捷键，替换默认的"和%，更直观
unbind '"'
unbind %
bind - split-window -v   # 按前缀+- 上下水平分屏
bind | split-window -h   # 按前缀+| 左右垂直分屏

# 快速重载配置，按前缀+r 直接重载配置文件，无需手动source
bind r source-file ~/.tmux.conf \; display "配置重载成功！"

# ==============================================
# 鼠标支持（tmux 2.1+ 版本）
# ==============================================
# 开启鼠标全功能支持：点击切换窗格、拖动调整窗格大小、滚轮滚动历史、拖选复制
set-option -g mouse on
# 解决macOS终端复制选中内容自动退出复制模式的问题
set -g mouse-copy-command "pbcopy"
# 复制时不自动跳转到终端末尾
set -g mouse-wheel-scrolls-in-copy-mode on

# ==============================================
# 状态栏美化与信息定制
# ==============================================
# 状态栏主题样式
set -g status-style bg=black,fg=white
# 左侧显示：会话名 + 窗口列表
set -g status-left "#[fg=green,bold] #S #[default]"
set -g status-left-length 20
# 右侧显示：主机名 + 时间
set -g status-right "#[fg=yellow] #H #[fg=cyan] %Y-%m-%d %H:%M:%S #[default]"
set -g status-right-length 60
# 窗口标签样式：当前窗口高亮
setw -g window-status-current-style fg=brightred,bold
# 窗格边框样式：当前激活窗格高亮
set -g pane-border-style fg=green
set -g pane-active-border-style fg=brightred,bold

# ==============================================
# 插件管理器 TPM 配置（必须放在配置文件最底部）
# ==============================================
set -g @plugin 'tmux-plugins/tpm'
# 常用插件，按需添加
set -g @plugin 'tmux-plugins/tmux-resurrect'  # 会话保存与恢复，重启服务器也能恢复
set -g @plugin 'tmux-plugins/tmux-continuum'  # 自动定时保存会话，搭配resurrect使用
set -g @plugin 'tmux-plugins/tmux-yank'        # 优化复制粘贴，支持系统剪贴板

# 初始化 TPM 插件管理器
run '~/.tmux/plugins/tpm/tpm'
```

### 4.2 插件管理与常用插件
tmux 可通过 TPM（Tmux Plugin Manager）插件管理器快速扩展功能，安装与使用步骤如下：

1.  **安装 TPM**
```bash
git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm
```

2.  **配置插件**：在 `~/.tmux.conf` 中添加插件配置（如上文中的配置示例），保存后重载配置。

3.  **插件操作**
    - 安装插件：在tmux内按下 `Ctrl+b I`（大写i），TPM会自动下载安装配置中添加的所有插件
    - 更新插件：按下 `Ctrl+b U`，自动更新所有已安装插件
    - 卸载插件：删除配置文件中对应的插件行，重载配置后按下 `Ctrl+b Alt+u`，自动卸载未配置的插件

#### 高频实用插件
| 插件名 | 核心功能 |
|---------|----------|
| tmux-resurrect | 手动保存/恢复会话布局和运行状态，服务器重启、tmux服务关闭后仍可恢复 |
| tmux-continuum | 搭配resurrect使用，自动定时保存会话，开机自动恢复，无需手动操作 |
| tmux-yank | 优化复制粘贴，支持将tmux缓冲区内容同步到系统剪贴板，跨终端复制 |
| tmux-sensible | tmux 基础通用优化配置，兼容不同版本，解决各类默认配置的坑 |
| tmux-power | 美化状态栏，提供更丰富的系统信息显示（CPU、内存、网速等） |

### 4.3 其他进阶技巧
#### 1. 多窗格同步操作
需要在多个窗格中执行相同命令时（如批量操作多台服务器），可开启同步输入：
1.  按下 `Ctrl+b :` 进入命令模式
2.  输入 `setw synchronize-panes on` 回车，开启同步，当前窗口所有窗格会同步接收键盘输入
3.  关闭同步：命令模式输入 `setw synchronize-panes off`

#### 2. 会话共享（多人协作）
tmux 支持多个用户同时接入同一个会话，实现结对编程、远程协助，核心步骤如下：
```bash
# 1. 会话创建者，新建一个共享会话
tmux new -s shared_session

# 2. 其他用户在同一台服务器上，直接接入该会话
tmux attach -t shared_session
```
> 注意：所有接入用户拥有完全相同的操作权限，可同时输入和操作。

#### 3. 自动化会话脚本
可通过shell脚本一键创建预设好窗口、窗格、命令的会话，无需每次手动配置，示例如下：
```bash
#!/bin/bash
# 文件名：dev_env.sh，一键创建开发环境会话
SESSION_NAME="dev_env"

# 检查会话是否已存在，不存在则创建
tmux has-session -t $SESSION_NAME 2>/dev/null
if [ $? != 0 ]; then
    # 新建会话，后台运行，第一个窗口命名为code
    tmux new-session -d -s $SESSION_NAME -n code
    # 第一个窗口执行进入项目目录，打开vim
    tmux send-keys -t $SESSION_NAME:1 "cd ~/project && vim" C-m

    # 新建第二个窗口，命名为server，启动开发服务
    tmux new-window -t $SESSION_NAME:2 -n server
    tmux send-keys -t $SESSION_NAME:2 "cd ~/project && npm run dev" C-m

    # 新建第三个窗口，命名为logs，实时查看日志
    tmux new-window -t $SESSION_NAME:3 -n logs
    tmux send-keys -t $SESSION_NAME:3 "tail -f ~/project/logs/app.log" C-m

    # 新建第四个窗口，命名为monitor，打开系统监控
    tmux new-window -t $SESSION_NAME:4 -n monitor
    tmux send-keys -t $SESSION_NAME:4 "htop" C-m

    # 默认选中第一个窗口
    tmux select-window -t $SESSION_NAME:1
fi

# 接入会话
tmux attach -t $SESSION_NAME
```
使用方式：
```bash
chmod +x dev_env.sh
./dev_env.sh
```
执行后会自动创建预设好的4个窗口，直接进入开发环境。

## 五、常见问题与解决方案
### 1. SSH断开后，重连找不到会话
- 原因：重连时使用了不同的用户，或tmux服务被终止
- 解决方案：
  1.  用创建会话时的相同用户登录服务器，执行 `tmux ls` 查看会话
  2.  若提示 `no server running on /tmp/tmux-xxx/default`，说明会话已被关闭，无法恢复
  3.  避免会话被关闭：不要用 `exit` 退出tmux，要用 `Ctrl+b d` 分离会话后台保活

### 2. 无法用鼠标滚轮滚动查看历史输出
- 原因：默认未开启鼠标支持，滚轮事件被tmux拦截
- 解决方案：
  1.  临时解决：按下 `Ctrl+b [` 进入复制模式，即可用滚轮/方向键滚动
  2.  永久解决：在 `~/.tmux.conf` 中添加 `set -g mouse on`，重载配置后生效

### 3. 配置文件修改后不生效
- 解决方案：
  1.  进入tmux后，按下 `Ctrl+b :`，输入 `source-file ~/.tmux.conf` 回车，手动重载配置
  2.  若仍不生效，检查配置文件语法是否正确，或关闭所有会话 `tmux kill-server` 后重启tmux
  3.  确保配置文件路径为 `~/.tmux.conf`，而非其他目录

### 4. 终端内颜色显示异常、vim主题错乱
- 原因：终端类型配置错误，未开启256色支持
- 解决方案：在 `~/.tmux.conf` 中添加 `set -g default-terminal "screen-256color"`，重载配置后生效

### 5. 中文显示乱码
- 解决方案：
  1.  确保系统locale配置为UTF-8，执行 `export LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8`
  2.  在 `~/.tmux.conf` 中添加：
  ```conf
  set -g utf8 on
  set-window-option -g utf8 on
  ```

## 六、常用快捷键速查表
| 分类 | 快捷键 | 核心功能 |
|------|--------|----------|
| 前缀键 | `Ctrl+b` | 所有快捷键的触发前缀 |
| 会话 | `Ctrl+b d` | 分离会话，后台保活 |
| 会话 | `Ctrl+b s` | 可视化查看/切换所有会话 |
| 会话 | `Ctrl+b $` | 重命名当前会话 |
| 窗口 | `Ctrl+b c` | 新建窗口 |
| 窗口 | `Ctrl+b ,` | 重命名当前窗口 |
| 窗口 | `Ctrl+b n/p` | 切换到下一个/上一个窗口 |
| 窗口 | `Ctrl+b 0-9` | 跳转到指定编号窗口 |
| 窗口 | `Ctrl+b &` | 关闭当前窗口 |
| 窗格 | `Ctrl+b "` | 上下水平分屏 |
| 窗格 | `Ctrl+b %` | 左右垂直分屏 |
| 窗格 | `Ctrl+b 方向键/hjkl` | 切换窗格 |
| 窗格 | `Ctrl+b z` | 窗格全屏/还原 |
| 窗格 | `Ctrl+b x` | 关闭当前窗格 |
| 窗格 | `Ctrl+b 空格键` | 循环切换窗格布局 |
| 复制粘贴 | `Ctrl+b [` | 进入复制模式/滚动历史 |
| 复制粘贴 | `Ctrl+b ]` | 粘贴缓冲区内容 |
| 配置 | `Ctrl+b :` | 进入tmux命令模式 |
| 插件 | `Ctrl+b I` | 安装TPM插件 |


