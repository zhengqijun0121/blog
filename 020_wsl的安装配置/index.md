# wsl 的安装配置


Windows Subsystem for Linux（简称 `WSL`）是一个在 `Windows` 上能够运行原生 `Linux` 二进制可执行文件（ELF格式）的兼容层。

<!--more-->

## `wsl` 的安装

### 启用适用于 Linux 的 Windows 子系统

需要先启用“适用于 Linux 的 Windows 子系统”可选功能，然后才能在 Windows 上安装 Linux 分发。使用管理员权限执行以下命令

```powershell
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
```

### 启用虚拟机功能

安装 `WSL2` 之前，必须启用“虚拟机平台”可选功能。使用管理员权限执行以下命令

```powershell
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

### 安装 `WSL2`

推荐使用 `Microsoft Store` 安装最新版 `wsl`，注意在 `Microsoft Store` 安装应用软件需要先登录用户。

```powershell
wsl --set-default-version 2
```

----

## `wsl` 安装 `Ubuntu`

### `Microsoft Store` 安装

推荐使用 `Microsoft Store` 安装 `Ubuntu`，注意在 `Microsoft Store` 安装应用软件需要先登录用户。默认情况下安装在C盘，这样会导致系统盘占用空间大，特别卡。所以接下来我们要把 `Ubuntu` 安装到非系统盘上。

```powershell
wsl install ubuntu-22.04
```

### 手动安装

```powershell
# 导出 Ubuntu-22.04 系统
wsl --export Ubuntu-22.04 D:\Software\WSL\Ubuntu-22.04.tar

# 删除 Ubuntu-22.04 系统
wsl --unregister Ubuntu-22.04

# 导入 Ubuntu-22.04 系统
wsl --import Ubuntu-22.04 D:\Software\WSL D:\Software\WSL\Ubuntu-22.04.tar
```

----

## `wsl` 的配置

### 启动 `systemd`

修改 `/etc/wsl.conf` 文件，启动 `systemd` 并修改默认用户。

```
[boot]
systemd=true

[user]
default=user
```

### 连接 `wsl` 端口

必须使用管理员权限执行以下命令

```powershell
# 查看转发端口
netsh interface portproxy show all

# 配置转发端口
netsh interface portproxy add v4tov4 listenport=8022 listenaddress=0.0.0.0 connectport=8022 connectaddress=192.168.2.1

# 配置端口通过防火墙
netsh advfirewall firewall add rule name=wsl_ssh_port dir=in action=allow protocol=tcp localport=8022

# 删除配置转发端口
netsh interface portproxy del v4tov4 listenport=8022 listenaddress=0.0.0.0

# 删除配置端口通过防火墙
netsh advfirewall firewall del rule name=wsl_ssh_port
```

### 通过命令行设置 `Windows` 网络

必须使用管理员权限执行以下命令

```powershell
# 查看电脑 ip 配置
netsh interface ip show config

# 设置以太网静态 ip
netsh interface ip set address "以太网" source=static addr=192.168.2.101 mask=255.255.255.0 gateway=192.168.2.1
netsh interface ip set address "以太网" static 192.168.2.101 255.255.255.0 192.168.2.1

# 设置以太网动态 ip
netsh interface ip set address "以太网" dhcp
```

----


