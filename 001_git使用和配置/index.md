# git 使用和配置


# git 使用和配置

## Ubuntu 安装 git 命令

```bash
apt-get install git
```

## git 配置文件

`git` 配置文件有三个：

1. `/etc/gitconfig` 是系统级配置文件，使用 `git config --system` 命令进行修改。
2. `~/.gitconfig` 或 `~/.config/git/config` 是用户级配置文件，使用 `git config --global` 命令进行修改。
3. `local_dir/.git/config` 是仓库级配置文件，使用 `git config --local` 命令进行修改。

### 设置用户信息

```bash
$ git config --global user.name "username"
$ git config --global user.email username@example.com
```

### 设置 git 默认编辑器

```bash
git config --global core.editor emacs
```


