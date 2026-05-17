# qemu 启动 arm64 版本 archlinux


<!--more-->

# qemu 启动 arm64 版本 archlinux

## 下载 archlinux 镜像和 Linux 内核

1. 下载 archlinux 镜像

[ArchLinuxARM-aarch64-latest.tar.gz](https://mirrors.aliyun.com/archlinuxarm/os/ArchLinuxARM-aarch64-latest.tar.gz)

命令行下载

```bash
# 使用 wget 命令下载
wget https://mirrors.aliyun.com/archlinuxarm/os/ArchLinuxARM-aarch64-latest.tar.gz
# 使用 curl 命令下载
curl -# -O https://mirrors.aliyun.com/archlinuxarm/os/ArchLinuxARM-aarch64-latest.tar.gz
```

2. 下载 Linux 内核

这里使用的是 v5.15.99 版本 Linux 内核。

[linux-5.15.99.tar.xz](https://mirrors.aliyun.com/linux-kernel/v5.x/linux-5.15.99.tar.xz)

命令行下载

```bash
# 使用 wget 命令下载
wget https://mirrors.aliyun.com/linux-kernel/v5.x/linux-5.15.99.tar.xz
# 使用 curl 命令下载
curl -# -O https://mirrors.aliyun.com/linux-kernel/v5.x/linux-5.15.99.tar.xz
```

## 编译 Linux 内核

需要使用 arm64 架构的编译工具链。如果是 x86_64 平台，则需要安装 arm64 架构的交叉编译工具链。这里是 aarch64 平台，直接使用 gcc 编译即可。

```bash
# 使用默认配置 .config
make defconfig
# 使用图形化配置 .config
make menuconfig
# 编译内核
make -j$(nproc)
```

这里使用默认 `.config` 配置文件就可以，不用修改。编译完成之后可以看到 `arch/arm64/boot/Image` 文件。

## 制作根文件系统镜像

1. 首先创建一个 4G 的文件，用于制作根文件系统镜像。

```bash
# 使用 dd 命令创建文件
sudo dd if=/dev/zero of=rootfs.img bs=1M count=4096
# 或者使用 qemu-img 命令创建文件
qemu-img create -f raw rootfs.img 4G
```

2. 使用 `mkfs.ext4` 命令创建 ext4 文件系统。

```bash
mkfs.ext4 rootfs.img
```

3. 使用 `mount` 命令挂载根文件系统镜像。

```bash
mkdir rootfs
sudo mount -t ext4 rootfs.img rootfs
```

4. 使用 `bsdtar` 命令将 archlinux 镜像解压到根文件系统镜像中。

```bash
sudo bsdtar -xpf ArchLinuxARM-aarch64-latest.tar.gz -C rootfs
```

5. 添加 `etc/inittab` 文件，用于启动内核。

```bash
echo "::askfirst:-/bin/bash" | sudo tee rootfs/etc/inittab > /dev/null
```

6. 使用 `umount` 命令卸载根文件系统镜像。

```bash
sudo umount rootfs
```

或者直接使用脚本创建根文件系统镜像，qemu-archlinux.sh 脚本内容如下：

```bash
#!/bin/bash

if [[ ! -f $(pwd)/ArchLinuxARM-aarch64-latest.tar.gz ]]; then
    wget https://mirrors.aliyun.com/archlinuxarm/os/ArchLinuxARM-aarch64-latest.tar.gz
fi

if [[ -d rootfs ]]; then
    sudo rm -rf rootfs
    rm -rf rootfs.img start.sh stop.sh
fi

mkdir rootfs
qemu-img create rootfs.img 4G
mkfs.ext4 rootfs.img
sudo mount -t ext4 rootfs.img rootfs
sudo bsdtar -xpf ArchLinuxARM-aarch64-latest.tar.gz -C rootfs/
echo "::askfirst:-/bin/bash" | sudo tee rootfs/etc/inittab > /dev/null
sudo umount rootfs

echo "sudo qemu-system-aarch64 -M virt -cpu cortex-a72 -smp 2 -m 2G -nographic -kernel \$(pwd)/Image -drive file=\$(pwd)/rootfs.img,format=raw -append \"root=/dev/vda rw console=ttyAMA0 \"" > start.sh

echo "sudo kill -9 \$(pidof qemu-system-aarch64)" > stop.sh

chmod a+x start.sh stop.sh

echo "rootfs.img creation completed, please execute start.sh to run!"
```

现在当前目录下应该有以下文件：

```bash
ArchLinuxARM-aarch64-latest.tar.gz  Image  qemu-archlinux.sh  rootfs  rootfs.img  start.sh  stop.sh
```

## 使用 qemu 启动 archlinux

使用命令行启动

```bash
sudo qemu-system-aarch64 -M virt -cpu cortex-a72 -smp 2 -m 2G -nographic -kernel Image -drive file=rootfs.img,format=raw -append "root=/dev/vda rw console=ttyAMA0"
```

命令行参数解析：
- `-M virt`: 使用 virt 架构
- `-cpu cortex-a72`: 使用 cortex-a72 架构
- `-smp 2`: 使用 2 个 CPU 核心
- `-m 2G`: 使用 2G 内存
- `-nographic`: 不显示图形界面
- `-kernel Image`: 使用 Image 内核
- `-drive file=rootfs.img,format=raw`: 使用 rootfs.img 根文件系统镜像
- `-append "root=/dev/vda rw console=ttyAMA0"`: 启动参数，root=/dev/vda 表示根文件系统在 /dev/vda 设备中，rw 表示读写模式，console=ttyAMA0 表示输出到 ttyAMA0 设备

或者使用脚本启动

```bash
./start.sh
```

当启动成功之后，会显示 archlinux 的登陆界面。

```bash
Welcome to Arch Linux ARM!

Arch Linux ARM 5.15.99 (ttyAMA0)

alarm login:
```

archlinux 默认有 root 用户和 alarm 用户。

```bash
# root
User: root
Password: root
# alarm
User: alarm
Password: alarm
```

----

