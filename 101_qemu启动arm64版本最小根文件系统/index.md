# qemu 启动 arm64 版本最小根文件系统


<!--more-->

# qemu 启动 arm64 版本最小根文件系统

## 下载 Linux 内核源码和 Busybox 源码

1. 下载 Linux 内核源码

这里使用的是 v5.15.99 版本 Linux 内核。

[linux-5.15.99.tar.xz](https://mirrors.aliyun.com/linux-kernel/v5.x/linux-5.15.99.tar.xz)

命令行下载

```bash
# 使用 wget 命令下载
wget https://mirrors.aliyun.com/linux-kernel/v5.x/linux-5.15.99.tar.xz
# 使用 curl 命令下载
curl -# -O https://mirrors.aliyun.com/linux-kernel/v5.x/linux-5.15.99.tar.xz
```

2. 下载 Busybox 源码

这里使用的是 v1.36.1 稳定版本 Busybox。

[busybox-1.36.1.tar.bz2](https://busybox.net/downloads/busybox-1.36.1.tar.bz2)

命令行下载

```bash
# 使用 wget 命令下载
wget https://busybox.net/downloads/busybox-1.36.1.tar.bz2
# 使用 curl 命令下载
curl -# -O https://busybox.net/downloads/busybox-1.36.1.tar.bz2
```

## 编译 Linux 内核源码和 Busybox 源码

1. 编译 Linux 内核源码

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

2. 编译 Busybox 源码

```bash
# 使用默认配置 .config
make defconfig
# 使用图形化配置 .config
make menuconfig
# 编译 Busybox
make -j$(nproc)
# 安装 Busybox
make install
```

安装完成之后，可以看到 `_install` 文件夹，里面有 Busybox 的可执行文件。

## 制作 ext4 格式最小根文件系统

1. 创建一个 4G 的文件，用于制作最小根文件系统。

```bash
# 使用 dd 命令创建文件
sudo dd if=/dev/zero of=rootfs.img bs=1M count=4096
# 或者使用 qemu-img 命令创建文件
qemu-img create -f raw rootfs.img 4G
```

2. 使用 `mkfs.ext4` 命令创建 ext4 文件系统。

这里使用的是 `EXT4` 格式的根文件系统，所以格式化为 `ext4` 格式。

```bash
mkfs.ext4 rootfs.img
```

3. 使用 `mount` 命令挂载根文件系统镜像。

```bash
mkdir rootfs
sudo mount -t ext4 rootfs.img rootfs
```

4. 创建必要系统文件

```bash
mkdir -p rootfs/{dev,etc/init.d,home,lib,mnt,proc,root,sys,tmp,usr}
```

5. 使用 `cp` 命令将 Busybox 的安装文件复制到根文件系统中。

```bash
cp -rf _install/* rootfs/
```

5. 添加系统配置文件

5.1 添加 `etc/inittab` 文件，用于启动内核。

```bash
::sysinit:/etc/init.d/rcS
::respawn:-/bin/sh
::restart:/sbin/init
::ctrlaltdel:/sbin/reboot
```

5.2 添加 `etc/init.d/rcS` 文件，用于启动 Busybox。

```bash
#!/bin/sh

/bin/mount -a
/sbin/mdev -s

ifconfig eth0 192.168.1.121
```

5.3 添加 `etc/profile` 文件，用于设置环境变量。

```bash
#!/bin/sh

export USER=root
export HOSTNAME=arm64
export HOME=root
export PS1="[$USER@$HOSTNAME \w]\# "
export PATH=/bin:/sbin:/usr/bin:/usr/sbin
export LD_LIBRARY_PATH=/lib:/usr/lib
```

5.4 添加 `etc/fstab` 文件，用于挂载文件系统。

```bash
# <file system> <mount point>   <type>      <options>   <dump>  <pass>
proc            /proc           proc        defaults    0       0
sysfs           /sys            sysfs       defaults    0       0
devtmpfs        /dev            devtmpfs    defaults    0       0
tmpfs           /tmp            tmpfs       defaults    0       0
```

制作完成之后，可以使用 `chroot` 命令进入根文件系统验证。

```bash
sudo chroot rootfs/ /bin/sh
```

如果进入成功，则说明制作成功。接下来卸载根文件系统镜像即可！

6. 使用 `umount` 命令卸载根文件系统镜像。

```bash
sudo umount rootfs
```

或者直接使用脚本创建最小根文件系统，qemu-arm64.sh 脚本内容如下：

```bash
#!/bin/bash

if [[ ! -f $(pwd)/busybox-1.36.1.tar.bz2 ]]; then
    wget https://busybox.net/downloads/busybox-1.36.1.tar.bz2
    tar -xf busybox-1.36.1.tar.bz2
fi

if [[ ! -f $(pwd)/linux-5.15.99.tar.xz ]]; then
    wget https://mirrors.aliyun.com/linux-kernel/v5.x/linux-5.15.99.tar.xz
    tar -xf linux-5.15.99.tar.xz
fi

if [[ -f $(pwd)/rootfs.img ]]; then
    rm -rf rootfs.img
fi

mkdir rootfs
qemu-img create -f raw rootfs.img 4G
mkfs.ext4 rootfs.img
sudo mount -t ext4 rootfs.img rootfs
sudo mkdir -p rootfs/{dev,etc/init.d,home,lib,mnt,proc,root,sys,tmp,usr}
sudo cp -rf busybox-1.36.1/_install/* rootfs/

# etc/inittab
echo "::sysinit:/etc/init.d/rcS" | sudo tee rootfs/etc/inittab > /dev/null
echo "::respawn:-/bin/sh" | sudo tee -a rootfs/etc/inittab > /dev/null
echo "::restart:/sbin/init" | sudo tee -a rootfs/etc/inittab > /dev/null
echo "::ctrlaltdel:/sbin/reboot" | sudo tee -a rootfs/etc/inittab > /dev/null

# etc/init.d/rcS
echo "#!/bin/sh" | sudo tee rootfs/etc/init.d/rcS > /dev/null
echo "" | sudo tee -a rootfs/etc/init.d/rcS > /dev/null
echo "/bin/mount -a" | sudo tee -a rootfs/etc/init.d/rcS > /dev/null
echo "/sbin/mdev -s" | sudo tee -a rootfs/etc/init.d/rcS > /dev/null
echo "" | sudo tee -a rootfs/etc/init.d/rcS > /dev/null
echo "ifconfig eth0 192.168.1.121" | sudo tee -a rootfs/etc/init.d/rcS > /dev/null

# etc/profile

echo "#!/bin/sh" | sudo tee rootfs/etc/profile > /dev/null
echo "" | sudo tee -a rootfs/etc/profile > /dev/null
echo "export USER=root" | sudo tee -a rootfs/etc/profile > /dev/null
echo "export HOSTNAME=arm64" | sudo tee -a rootfs/etc/profile > /dev/null
echo "export HOME=root" | sudo tee -a rootfs/etc/profile > /dev/null
echo "export PS1=\"[\$USER@\$HOSTNAME \w]\# \"" | sudo tee -a rootfs/etc/profile > /dev/null
echo "export PATH=/bin:/sbin:/usr/bin:/usr/sbin" | sudo tee -a rootfs/etc/profile > /dev/null
echo "export LD_LIBRARY_PATH=/lib:/usr/lib" | sudo tee -a rootfs/etc/profile > /dev/null

# etc/fstab
echo "# <file system> <mount point>   <type>      <options>   <dump>  <pass>" | sudo tee rootfs/etc/fstab > /dev/null
echo "proc            /proc           proc        defaults    0       0" | sudo tee -a rootfs/etc/fstab > /dev/null
echo "sysfs           /sys            sysfs       defaults    0       0" | sudo tee -a rootfs/etc/fstab > /dev/null
echo "devtmpfs        /dev            devtmpfs    defaults    0       0" | sudo tee -a rootfs/etc/fstab > /dev/null
echo "tmpfs           /tmp            tmpfs       defaults    0       0" | sudo tee -a rootfs/etc/fstab > /dev/null

sudo umount rootfs

echo "sudo qemu-system-aarch64 -M virt -cpu cortex-a72 -smp 2 -m 2G -nographic -kernel \$(pwd)/Image -drive file=\$(pwd)/rootfs.img,format=raw -append \"root=/dev/vda rw console=ttyAMA0 \"" > start.sh

echo "sudo kill -9 \$(pidof qemu-system-aarch64)" > stop.sh

chmod a+x start.sh stop.sh

echo "rootfs.img creation completed, please execute start.sh to run!"
```

## 使用 qemu 启动最小根文件系统

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

当启动成功之后，会显示 Linux 启动界面。

```bash
[    0.509666] VFIO - User Level meta-driver version: 0.3
[    0.513444] ehci_hcd: USB 2.0 'Enhanced' Host Controller (EHCI) Driver
[    0.513611] ehci-pci: EHCI PCI platform driver
[    0.513829] ehci-platform: EHCI generic platform driver
[root@arm64 /]#
```

-----

## 设置共享目录

`qemu-system-aarch64` 命令启动时添加参数 `-fsdev local,security_model=passthrough,id=fsdev0,path=$(pwd)/shared -device virtio-9p-pci,id=fs0,fsdev=fsdev0,mount_tag=hostshare`，将当前目录下的 shared 目录作为共享目录。

启动完整命令如下：

```bash
qemu-system-aarch64 -M virt -cpu cortex-a72 -smp 2 -m 2G -nographic \
    -kernel Image -drive file=rootfs.img,format=raw \
    -append "root=/dev/vda rw console=ttyAMA0" \
    -fsdev local,security_model=passthrough,id=fsdev0,path=$(pwd)/shared \
    -device virtio-9p-pci,id=fs0,fsdev=fsdev0,mount_tag=host
```

Linux 启动成功之后，在 Linux 环境内执行挂载命令。

```bash
mkdir -p /mnt/shared
mount -t 9p -o trans=virtio,version=9p2000.L hostshare /mnt/shared
```

然后就可以在 Linux 环境内访问共享目录了。

-----

## 制作 initramfs 格式根文件系统


