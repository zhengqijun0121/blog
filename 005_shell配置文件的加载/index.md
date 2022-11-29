# Shell 配置文件的加载


## Shell 启动方式

### 启动方式

Shell 的启动方式有以下四种：

- 交互式登录
- 非交互式登录
- 交互式非登录
- 非交互式非登录

其中交互式是指需要使用者手动输入的方式，而非交互式指的是脚本方式运行，不需要使用者手动输入。登录指的是使用者需要输入用户名和密码的方式，需要特定用户才能执行，而非登录不需要输入用户名和密码，任何用户都能执行。

**比如：`GNOME Terminal` 默认是交互式非登录 `Shell`，`iTerm2 Terminal` 默认是交互式登录 `Shell`，`ssh` 连接也是交互式非登录 `Shell`，在终端上运行一般脚本是非交互式非登录 `Shell`，`ssh` 连接直接执行命令是非交互式登陆 `Shell` 等。**

### 区分方式

1. 交互式区分方式

- 根据 `$-` 变量区分，含有 `i` (interactive) 为交互式 `Shell`。

```bash
echo "$-"
himBHs
```

- 根据 `$PS1` 变量区分，不为空的是交互式 `Shell`。

```bash
echo "$PS1"
[\u@\h \W]\$
```

- 指定 `-c` 选项运行非交互式 `Shell`，指定 `-i` 选项运行交互式 `Shell`。

2. 登录式区分方式

- 根据 `$0` 变量区分，带 `-` 为登录 `Shell`。

```bash
echo "$0"
-bash
```

- 根据 `shopt login_shell`，`Bash` 独有。

```bash
login_shell     on
```

- 执行 `logout` 命令，只有登录 `Shell` 才能运行这条命令

```bash
logout
bash: logout: not login shell: use `exit'
```

- 指定 `-l` 或者 `--login` 选项运行登陆 `Shell`。

----

## Shell 配置文件的加载顺序

### Bash

![](../posts/03_学习/21_开发环境/img/005-1-Bash加载配置文件.png)

----

### Zsh

![](../posts/03_学习/21_开发环境/img/005-2-Zsh加载配置文件.png)

----

### Ash

![](../posts/03_学习/21_开发环境/img/005-3-Ash加载配置文件.png)

---


