# Mac M1 使用 VMWare 安装 ArchLinux 系统


一个简单、轻量级的发行版，试图保持简单。

<!--more-->

## 安装准备

### Mac 安装 VMWare Fusion

通过 [VMWare 官网](https://www.vmware.com/products/desktop-hypervisor/workstation-and-fusion) 下载 `VMWare Fusion` 软件，这里下载的是 26H1 版本。

### 下载 archboot.iso

通过 [archboot 官网](https://release.archboot.net/aarch64/latest/iso/) 下载 `archboot.iso`。这里下载的是 archboot-2026.05.02-02.29-7.0.3-1-aarch64-ARCH-local-aarch64.iso 版本。

----

## `ArchLinux` 系统安装

### 1. VMWare 新建虚拟机

`VMWare` 软件点击新建虚拟机

![](../posts/01_学习/24_ArchLinux/img/002-01.png)

选择刚刚下载的 ISO 文件

![](../posts/01_学习/24_ArchLinux/img/002-02.png)

然后选择 Linux 系统 6.x 版本内核

![](../posts/01_学习/24_ArchLinux/img/002-03.png)

最后确认完成

![](../posts/01_学习/24_ArchLinux/img/002-04.png)

最后设置系统的 CPU 和内存大小，选择 2 核 4G 内存，硬盘这里选择 30G。

![](../posts/01_学习/24_ArchLinux/img/002-05.png)

-----

### 2. 安装启动过程

开始安装 `ArchLinux` 系统可以看到安装启动界面，选择 `Launch UEFI Archboot - Arch Linux aarch64` 下一步

![](../posts/01_学习/24_ArchLinux/img/002-06.png)

启动完成后可以看到 Archboot 界面，按回车键下一步

![](../posts/01_学习/24_ArchLinux/img/002-07.png)

**下面就开始正式配置安装 `ArchLinux`**

这里 Locale 选择 `en_US English`

![](../posts/01_学习/24_ArchLinux/img/002-08.png)

由于下载的是离线镜像，这里使用离线模式安装

![](../posts/01_学习/24_ArchLinux/img/002-09.png)

这里 Timezone Region 选择 `Asia`

![](../posts/01_学习/24_ArchLinux/img/002-10.png)

Timezone 选择 `Shanghai`

![](../posts/01_学习/24_ArchLinux/img/002-11.png)

Date 按回车下一步

![](../posts/01_学习/24_ArchLinux/img/002-12.png)

Time 按回车下一步

![](../posts/01_学习/24_ArchLinux/img/002-13.png)

这里 Launcher Menu 选择 `1 Launch Archboot Setup`

![](../posts/01_学习/24_ArchLinux/img/002-14.png)

Setup Menu 选择 `1 Prepare Storage Device`

![](../posts/01_学习/24_ArchLinux/img/002-15.png)

Prepare Storage Device 选择 `1 Quick Setup (erases the ENTIRE storage device)`

![](../posts/01_学习/24_ArchLinux/img/002-16.png)

Storage Device 选择 `/dev/nvme0n1`

![](../posts/01_学习/24_ArchLinux/img/002-17.png)

Device Name Scheme 选择 `PARTUUID`

![](../posts/01_学习/24_ArchLinux/img/002-18.png)

EFI System Partition 选择 `/boot SINGLE`

![](../posts/01_学习/24_ArchLinux/img/002-19.png)

EFI System Partition (ESP) in MiB 选择默认 `512`

![](../posts/01_学习/24_ArchLinux/img/002-20.png)

Swap in MiB 选择 `4096`

![](../posts/01_学习/24_ArchLinux/img/002-21.png)

Filesystem / and /home 选择 `ext4`

![](../posts/01_学习/24_ArchLinux/img/002-22.png)

Confirmation 选择 `Yes`

![](../posts/01_学习/24_ArchLinux/img/002-23.png)

/ in MiB 这里选择 `0`，/home 不单独分配空间

![](../posts/01_学习/24_ArchLinux/img/002-24.png)

Confirmation 选择 `Yes`

![](../posts/01_学习/24_ArchLinux/img/002-25.png)

确认擦除 /dev/nvme0n1 选择 `Yes`

![](../posts/01_学习/24_ArchLinux/img/002-26.png)

Setup Menu 选择 `2 Install Packages`

![](../posts/01_学习/24_ArchLinux/img/002-27.png)

Summary 选择 `Yes`

![](../posts/01_学习/24_ArchLinux/img/002-28.png)

Setup Menu 选择 `3 Configure System`

![](../posts/01_学习/24_ArchLinux/img/002-29.png)

New root Password 输入 root 用户密码

![](../posts/01_学习/24_ArchLinux/img/002-30.png)

Retype root Password 再次输入密码

![](../posts/01_学习/24_ArchLinux/img/002-31.png)

Text Editor 选择 `Neovim`

![](../posts/01_学习/24_ArchLinux/img/002-32.png)

MKINITCPIO EARLY USERSPACE 选择 `SYSTEMD`

![](../posts/01_学习/24_ArchLinux/img/002-33.png)

System Configuration 选择 `/etc/hostname`

![](../posts/01_学习/24_ArchLinux/img/002-34.png)

修改主机名为 `archlinux`

![](../posts/01_学习/24_ArchLinux/img/002-35.png)

System Configuration 选择 `/etc/pacman.d/mirrorlist`

![](../posts/01_学习/24_ArchLinux/img/002-36.png)

在文件第一行添加清华源 `Server = http://mirrors.tuna.tsinghua.edu.cn/archlinuxarm/$repo/$arch`

![](../posts/01_学习/24_ArchLinux/img/002-37.png)

System Configuration 选择 `Back`

![](../posts/01_学习/24_ArchLinux/img/002-38.png)

Setup Menu 选择 `4 Install Bootloader`

![](../posts/01_学习/24_ArchLinux/img/002-39.png)

AA64 UEFI Bootloader 选择 `GRUB_UEFI`

![](../posts/01_学习/24_ArchLinux/img/002-40.png)

确认 GRUB 配置文件选择 `OK`

![](../posts/01_学习/24_ArchLinux/img/002-41.png)

保存 grub.cfg 文件退出

![](../posts/01_学习/24_ArchLinux/img/002-42.png)

Setup Menu 选择 `5 Exit`

![](../posts/01_学习/24_ArchLinux/img/002-43.png)

Exit Menu 选择 `3 Poweroff System`

![](../posts/01_学习/24_ArchLinux/img/002-44.png)

设置修改 CD/DVD 为 Autodetect

![](../posts/01_学习/24_ArchLinux/img/002-45.png)

启动虚拟机，可以看到登陆画面

![](../posts/01_学习/24_ArchLinux/img/002-46.png)

-----

### 3. 更新系统

由于是安装的离线版，安装的不是最新内核，这里执行 `Pacman -Syu` 命令更新下系统

![](../posts/01_学习/24_ArchLinux/img/002-47.png)

再次重启后，系统已更新，安装完成！

![](../posts/01_学习/24_ArchLinux/img/002-48.png)

----

