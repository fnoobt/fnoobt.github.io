---
title:  linaro 交叉编译 Phytinum 嵌入式内核
author: fnoobt
date: 2025-01-03 14:17:00 +0800
categories: [VPS,服务搭建]
tags: [vps,linux,linaro,phytinum,embedded,kernel,cross-compiles]
---

## 安装环境
构建机器是`x86_64-pc-linux-gnu`，系统ubuntu22.04

目标机器是`aarch64-linux-gnu`，系统phytium-embedded-v3.0

## 下载工具链
通过[linaro官方](https://releases.linaro.org/components/toolchain/binaries/)下载对应的linaro版本

本文使用[linaro6.3.1版本](https://releases.linaro.org/components/toolchain/binaries/6.3-2017.02/aarch64-linux-gnu/)

1. `gcc-linaro-6.3.1-2017.02-x86_64_aarch64-linux-gnu.tar.xz`:  
这是 Linaro 提供的 GCC 交叉编译器工具链，支持从 x86_64 架构生成 aarch64 (ARM64) 架构的二进制文件。包含 `gcc`、`g++` 等工具和链接器 `ld`。
2. `runtime-gcc-linaro-6.3.1-2017.02-aarch64-linux-gnu.tar.xz`:  
包含运行时库（如 C 库、标准 C++ 库等），这些库需要在目标平台（aarch64-linux-gnu）上存在，以便执行由上述 GCC 工具链编译的应用程序。
3. `sysroot-glibc-linaro-2.23-2017.02-aarch64-linux-gnu.tar.xz`:  
包含了一个完整的 ARM64 系统根文件系统（sysroot），其中包括头文件和库文件，用于交叉编译时链接和和查找库文件。包含 `glibc 2.23` 版本的动态库和头文件。Glibc 是 GNU C 库的一部分，它提供了 C 语言标准库的实现，是 Linux 系统中的一个重要组成部分。
4. `*.asc`:  
对应文件的签名文件，用于验证文件的完整性和真实性。验证方法`gpg --verify *.asc`。

通常，在交叉编译环境中，交叉编译器用于生成二进制可执行文件，而 runtime 和 sysroot 文件则提供了在目标平台上运行这些生成的可执行文件所需的运行时支持和库。确保这些文件版本与您的项目和目标平台的要求相匹配。

## 解压工具链
```bash
# 解压交叉编译工具链文件
sudo tar -xvf gcc-linaro-6.3.1-2017.02-x86_64_aarch64-linux-gnu.tar.xz -C /opt/linaro

# 解压 sysroot 文件
sudo tar -xvf sysroot-glibc-linaro-2.23-2017.02-aarch64-linux-gnu.tar.xz -C /opt/linaro
```

## 设置环境变量
```bash
# 为工具链配置 CROSS_COMPILE
export CROSS_COMPILE=/opt/linaro/gcc-linaro-6.3.1-2017.02-x86_64_aarch64-linux-gnu/bin/aarch64-linux-gnu-

# 为交叉编译配置 sysroot
export SYSROOT=/opt/linaro/sysroot-glibc-linaro-2.23-2017.02-aarch64-linux-gnu
```

## 编译飞腾内核
```bash
# 下载飞腾内核源码
wget https://gitee.com/phytium_embedded/phytium-linux-kernel/archive/refs/tags/kernel-6.6_v3.0.tar.gz

tar -xzf phytium-linux-kernel-kernel-6.6_v3.0.tar.gz
cd phytium-linux-kernel-kernel-6.6_v3.0
 
# 配置内核，手动调整配置使用 menuconfig
make ARCH=arm64 CROSS_COMPILE=${CROSS_COMPILE} SYSROOT=${SYSROOT} defconfig

# 交叉编译内核，编译完成后，内核镜像通常位于arch/aarch64/boot/Image
make ARCH=arm64 CROSS_COMPILE=${CROSS_COMPILE} SYSROOT=${SYSROOT} -j8

# 编译内核模块，生成的内核模块将放置在 lib/modules/$(kernel_version)
make ARCH=arm64 CROSS_COMPILE=${CROSS_COMPILE} SYSROOT=${SYSROOT} modules

# 安装内核模块,模块会被安装到 ${SYSROOT}/lib/modules/$(kernel_version) 目录
make ARCH=arm64 CROSS_COMPILE=${CROSS_COMPILE} SYSROOT=${SYSROOT} INSTALL_MOD_PATH=${SYSROOT} modules_install

# 拷贝/lib/modules/ 到目标系统的/lib/modules/下
# 拷贝 arch/ arm64/boot/zImage 到目标系统/boot下
# 拷贝System.map 到目标系统/boot下
```

## QEMU运行
### 创建根文件系统
为了在QEMU中运行内核，需要一个根文件系统。可以使用BusyBox快速创建一个简单的根文件系统：
```bash
# 安装BusyBox
wget https://busybox.net/downloads/busybox-1.37.0.tar.bz2
tar -xvf busybox-1.37.0.tar.bz2
cd busybox-1.37.0

# 配置 BusyBox
make ARCH=arm64 CROSS_COMPILE=${CROSS_COMPILE} SYSROOT=${SYSROOT} defconfig

# 启用静态编译支持：进入 Settings -> Build static binary (no shared libs)，按空格选择后保存退出。
# make ARCH=arm64 CROSS_COMPILE=${CROSS_COMPILE} menuconfig

# 编译 BusyBox
make ARCH=arm64 CROSS_COMPILE=${CROSS_COMPILE} SYSROOT=${SYSROOT} -j4

# 创建根文件系统结构
mkdir -p ../rootfs/{bin,sbin,etc,proc,sys,usr/{bin,sbin},lib,tmp,home,root,mnt,dev}

# 安装 BusyBox，../rootfs 目录中包含了 BusyBox 安装的二进制文件和基础目录结构。
make ARCH=arm64 CROSS_COMPILE=${CROSS_COMPILE} SYSROOT=${SYSROOT} INSTALL_MOD_PATH=${SYSROOT} CONFIG_PREFIX=../rootfs install
# 设置设备节点
mknod dev/console c 5 1
mknod dev/null c 1 3

# 挂载 sysroot
sudo cp -a $SYSROOT/* ../rootfs/
```

建简单的启动脚本`/etc/init.d/rcS`
```bash
sudo tee etc/init.d/rcS > /dev/null <<EOF
#!/bin/sh
mount -t proc proc /proc
mount -t sysfs sysfs /sys
/bin/sh
EOF

sudo chmod +x etc/init.d/rcS
```

创建 init 文件以设置默认入口：
```bash
sudo tee etc/inittab > /dev/null <<EOF
::sysinit:/etc/init.d/rcS
::askfirst:/bin/sh
::ctrlaltdel:/bin/umount -a
EOF
```

### 制作根文件系统镜像
首先，创建一个空的镜像文件，然后格式化为ext4，并挂载以复制文件进去，制作根文件系统镜像以供 QEMU 使用：
```bash
# 创建一个大小为 64MB 的文件并格式化为 ext4
dd if=/dev/zero of=rootfs.img bs=1M count=64
mkfs.ext4 rootfs.img
# 挂载并复制文件
mkdir -p mnt
sudo mount rootfs.img mnt
sudo cp -a rootfs/* mnt/
sudo umount mnt
```

##　启动 QEMU 测试内核
### 启动命令
使用 QEMU 启动内核：
```bash
# 确保已经安装了QEMU
sudo apt install qemu-system-arm

qemu-system-aarch64 -M virt \
    -cpu cortex-a57 -m 2048 -nographic \
    -kernel arch/arm64/boot/Image \
    -dtb arch/arm64/boot/dts/phytium/phytium.dtb \
    -append "root=/dev/vda console=ttyAMA0" \
    -drive if=none,file=rootfs.img,format=raw,id=hd0 \
    -device virtio-blk-device,drive=hd0 \
    -netdev user,id=net0 -device virtio-net-device,netdev=net0
```

`-kernel`指定了内核镜像，`-append`提供了内核命令行参数，`-drive`指定了根文件系统的镜像文件，`-netdev`和`-device`设置了网络设备。调整这些选项以匹配你的实际情况，`-dtb` 指定设备树文件路径，`-initrd` 指定初始化RAM磁盘（根文件系统），`-nographic` 表示不使用图形界面，输出到终端。

****

本文参考

> 1. [linaro交叉编译工具链下载与使用笔记](https://blog.csdn.net/yjkhtddx/article/details/134676016)