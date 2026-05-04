# Manjaro 系统安装


一个简单、轻量级的发行版，试图保持简单。

<!--more-->

## Parallels 安装 arm64 版本 Manjaro

`Parallels` 不能直接安装 `Manjaro` 系统，需要通过 `Fedora` 镜像安装。

### `Fedora` 镜像文件下载

通过 `Fedora` 官网下载：

[官网下载链接](https://fedoraproject.org/zh-Hans/workstation/download/)

由于本地 `Parallels` 是 v17 版本，所以这里下载的是 v40 版本 `Fedora` 镜像，不是最新版本。

----

## `Manjaro` 系统安装

### 1. Parallels 新建虚拟机

选择 `Fedora-Workstation-Live-osb-40-1.14.aarch64.iso` 镜像启动。

### 2. 启动 Fedora 系统

启动系统，进入 `Fedora` 系统安装界面，选择 `Test this media 8 install Fedora 40`。

![](../posts/01_学习/25_Manjaro/img/001-01.png)

选择 `Not Now` 进入系统。

![](../posts/01_学习/25_Manjaro/img/001-02.png)

打开终端，方便下载 `Manjaro` 镜像文件。

![](../posts/01_学习/25_Manjaro/img/001-03.png)

在终端中执行以下命令下载 `Manjaro` 系统镜像文件。

```bash
wget -O - https://github.com/manjaro-arm/generic-efi-images/releases/download/23.02/Manjaro-ARM-minimal-generic-efi-23.02.img.xz | xzcat | sudo dd of=/dev/sda bs=4M
```

![](../posts/01_学习/25_Manjaro/img/001-04.png)

然后执行 `poweroff` 关闭虚拟机，关机之后断开镜像链接。

![](../posts/01_学习/25_Manjaro/img/001-05.png)

----

### 3. Manjaro 安装过程

开始安装 `Manjaro` 系统可以看到安装启动界面，回车下一步

![](../posts/01_学习/25_Manjaro/img/002-01.png)

键盘布局选择 `us`

![](../posts/01_学习/25_Manjaro/img/002-02.png)

然后输入用户名，这里输入 `${username}`

![](../posts/01_学习/25_Manjaro/img/002-03.png)

用户组这里选择默认，直接输入回车下一步

![](../posts/01_学习/25_Manjaro/img/002-04.png)

然后输入全名，这里输入 `${username}`

![](../posts/01_学习/25_Manjaro/img/002-05.png)

输入用户密码，这里输入 `${password}`

![](../posts/01_学习/25_Manjaro/img/002-06.png)

确认用户密码

![](../posts/01_学习/25_Manjaro/img/002-07.png)

输入 `root` 用户密码，这里输入 `${password}`

![](../posts/01_学习/25_Manjaro/img/002-08.png)

确认 `root` 用户密码

![](../posts/01_学习/25_Manjaro/img/002-09.png)

时区选择 `Asia/Shanghai`，然后 OK 下一步

![](../posts/01_学习/25_Manjaro/img/002-10.png)

系统语言环境选择 `en_US.UTF-8`，然后 OK 下一步

![](../posts/01_学习/25_Manjaro/img/002-11.png)

输入主机名，这里输入 `${hostname}`

![](../posts/01_学习/25_Manjaro/img/002-12.png)

然后确认刚刚输入的配置信息是否正确

![](../posts/01_学习/25_Manjaro/img/002-13.png)

然后系统配置生效，生效后会在 `5` 秒内重启系统，需要注意的是这里按 `Ctrl+C` 打断系统重启！

![](../posts/01_学习/25_Manjaro/img/002-14.png)

命令行输入配置 `boot` 启动命令：

```bash
sudo grub-install --efi-directory=/boot/efi
```

![](../posts/01_学习/25_Manjaro/img/002-15.png)

最后输入 `sudo reboot` 重启系统。

----

### 4. 启动 Manjaro 系统

重启之后，看到请输入用户就代表 `Manjaro` 系统安装成功。

`Manjaro` 系统更新中国源，选择清华源。

```bash
sudo pacman-mirrors -i -c China -m rank
```

安装好 `Manjaro` 系统之后，第一件事就是更新最新系统。

```bash
sudo pacman -Syu
```

----

### 5. 安装 gnome 桌面环境

打开终端执行安装命令

```bash
sudo pacman -S gnome
```

设置 `gdm` 服务自启动

```bash
sudo systemctl enable gdm.service --force
```

然后执行 `sudo reboot` 命令重启 `Manjaro` 系统。

----

