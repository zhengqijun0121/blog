# sshfs 挂载远程目录


`sshfs` 是连接到 SSH 服务器的网络文件系统客户端，方便本地操作远程文件。

<!--more-->

## Ubuntu 平台

### Ubuntu 安装 `sshfs`

```bash
sudo apt-get install sshfs
```

### Ubuntu 使用 `sshfs` 挂载目录

将 `user@hostname` 的 `/home/user` 远程目录挂载到 `/home/user/sshfs` 本地目录下，`/home/user/sshfs` 本地目录需要先创建，否则会执行失败。

```bash
sshfs user@hostname:/home/user/ /home/user/sshfs/
```

### Ubuntu 卸载挂载目录

```bash
sudo umount /home/user/sshfs
```

----

## Windows 平台

### Windows 安装 `sshfs`

1. [https://github.com/winfsp/winfsp](https://github.com/winfsp/winfsp)

2. [https://github.com/winfsp/sshfs-win](https://github.com/winfsp/sshfs-win)

### Windows 使用 `sshfs`

打开文件资源管理器，点击映射网络驱动器，输入远程目录即可。

```powershell
\\sshfs.r\user@hostname!port\path
```

### Windows 卸载挂载目录

打开文件资源管理器，点击断开连接。

### `sshfs-win` 命令说明

- `sshfs` 是相对于 `HOME` 目录
- `sshfs.r` 是相对于 `ROOT` 目录
- `sshfs.k` 是相对于 `HOME` 目录，并使用 ssh 密钥
- `sshfs.kr` 是相对于 `ROOT` 目录，并使用 ssh 密钥

----


