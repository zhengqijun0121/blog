# ssh 使用和配置


## Ubuntu 安装 ssh 命令

```bash
apt-get install openssh-server
```

----

## ssh 使用和配置

### ssh 配置

1. 生成 ssh 密钥和公钥

```bash
ssh-keygen -t rsa
```

其中在 `~/.ssh` 文件夹下，`id_rsa` 是私钥，`id_rsa.pub` 是公钥。

2. 将公钥放在服务器上
3. 测试 ssh 连接

```bash
ssh -p port user@hostname
```

连接成功会出现 **Welcome to Gerrit Code Review** 字样！

### ssh 和 sshd

`/etc/ssh/ssh_config` 和 `/etc/ssh/sshd_config` 都是 `ssh` 的配置文件，区别在于 `ssh_config` 是客户端的配置文件，`sshd_config` 是服务端的配置文件。

`/etc/ssh/sshd_config` 部分文件内容

```
Port 22  # 默认端口号为 22
PermitRootLogin no  # 禁止 root 用户登录
X11Forwarding yes  # 允许 X11 图形化
```

### ssh 服务端控制命令

```bash
service sshd start    # 启动 sshd
service sshd stop     # 停止 sshd
service sshd restart  # 重启 sshd
```

### ssh config 配置文件

修改 `~/.ssh/config` 文件设置别名

```bash
Host Alias
    HostName ip[192.168.XXX.XXX]
    Port 22
    User zhengqijun
    IdentityFile ~/.ssh/id_rsa
```

直接输入 `ssh Alias` 命令就可以直接连接了！十分方便！

### ssh 免密登录

将 `ssh` 生成的公钥拷贝到目标机的 `~/.ssh/authorized_keys` 中，就可以实现免密登录了。`ssh` 连接连密码都省了，偷懒小能手！

```bash
ssh-copy-id user@hostname
```

----

## 传输文件

### scp 传输文件

`scp` 是基于 `ssh` 登陆进行安全的远程文件拷贝命令，所以可以通过网络传输文件。

```
scp local_file remote_username@remote_ip:remote_folder  # 传输本地文件到目标机
scp -r local_folder remote_username@remote_ip:remote_folder  # 传输本地目录到目标机

scp remote_username@remote_ip:remote_folder local_file  # 传输本地文件到目标机
scp -r remote_username@remote_ip:remote_folder local_folder  # 传输本地目录到目标机
```

----

## 跳板机

### ssh 通过跳板机直连原本访问不到的机器

`~/.ssh/config` 中配置如下：

```bash
Host gateway
  HostName 192.168.168.52
  User gateway
Host chej
  HostName 192.168.211.100
  Port 22
  User root
  ProxyJump gateway
```

通过命令直接登录，无需先到跳板机 `gateway` 。

```bash
ssh chej
```

----

## ssh 图形化连接

### X11 是什么

Linux 本身是没有图形化界面的，所谓的图形化界面系统只不过中 Linux 下的应用程序。这一点和 Windows 不一样。Windows 从 Windows 95 开始，图形界面就直接在系统内核中实现了，是操作系统不可或缺的一部分。Linux 的图形化界面，底层都是基于 X 协议。

X11 是 X 协议的某个版本,，应用程序通过 X 协议告诉服务器端需要显示什么图形，然后服务器端通过 X server 来显示。

但是在远程连接时，服务器是本地的机器，客户端是远程服务器上的程序。因为我们是想要在本地显示远程服务器上的应用结果。

### Mac 下的 X11

通过安装 [XQuartz](https://www.xquartz.org/)，Mac 就可以做一个 X11 server，这样在 Mac 上就能显示远程服务器的应用程序。当 XQuartz 在运行时，会显示图标。

通过 `xclock` 命令测试，如果一切顺利，应该在 Mac 上会弹出界面。

```
xclock
```

### VSCode 下的 Remote X11

能够做到远程执行命令显示图形界面并调试，还是很诱人的。

这个插件就可以在 vscode 终端中使用 X11 forwarding 了。

### Windows 下的 X11

Windows 下有一个终端 `MobaXterm`，自带 X server。


