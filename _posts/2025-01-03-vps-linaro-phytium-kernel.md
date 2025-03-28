---
title:  linaro 交叉编译 Phytinum 嵌入式内核
author: fnoobt
date: 2025-01-03 14:17:00 +0800
categories: [VPS,服务搭建]
tags: [vps,linux,linaro,qemu,phytinum,embedded,kernel,cross-compiles]
---

## 安装环境
构建机器是`x86_64-pc-linux-gnu`，系统`ubuntu22.04`

目标机器是`aarch64-linux-gnu`，系统`phytium-embedded-v3.0`

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

# 手动配置内核，默认配置修改 menuconfig 为 defconfig
make ARCH=arm64 CROSS_COMPILE=${CROSS_COMPILE} SYSROOT=${SYSROOT} menuconfig
## 启用Virtio块设备支持（Block devices）
Device Drivers ->
  Virtio drivers ->
    [*] PCI driver for virtio devices (CONFIG_VIRTIO_PCI=y)
## 启用根文件系统的支持
File systems ->
  [*] Ext4 (ext4) file system support (CONFIG_EXT4_FS=y)
## 启用伪文件系统支持
  Pseudo filesystems ->
    [*] /proc file system support (CONFIG_PROC_FS=y)
    [*] Tmpfs (virtual memory file system support) (CONFIG_TMPFS=y)

# 交叉编译内核，编译完成后，内核镜像位于 arch/arm64/boot/Image
make ARCH=arm64 CROSS_COMPILE=${CROSS_COMPILE} SYSROOT=${SYSROOT} -j8

# 如果目标设备需要设备树文件再编译，生成的 .dtb 文件通常位于 arch/arm64/boot/dts
make ARCH=arm64 CROSS_COMPILE=${CROSS_COMPILE} SYSROOT=${SYSROOT} dtbs

# 编译内核模块
make ARCH=arm64 CROSS_COMPILE=${CROSS_COMPILE} SYSROOT=${SYSROOT} modules

# 安装内核模块,模块会被安装到 INSTALL_MOD_PATH/lib/modules/$(kernel_version) 目录
make ARCH=arm64 CROSS_COMPILE=${CROSS_COMPILE} SYSROOT=${SYSROOT} INSTALL_MOD_PATH=./modules modules_install

# 拷贝 ./modules/lib/modules/ 到目标系统的 /lib/modules/ 下
# 拷贝 arch/arm64/boot/Image 到目标系统 /boot 下
# 拷贝 System.map 到目标系统 /boot 下
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
make ARCH=arm64 CROSS_COMPILE=${CROSS_COMPILE} menuconfig
## 启用静态编译支持，按空格选择后保存退出。
Settings  ---> 
  [*] Build static binary (no shared libs)
  [ ] SHA1: Use hardware accelerated instructions if possible
  [ ] SHA256: Use hardware accelerated instructions if possible
Networking Utilities  --->
  [ ] ip link set type can

# 编译 BusyBox
make ARCH=arm64 CROSS_COMPILE=${CROSS_COMPILE} -j8

# 创建根文件系统结构
mkdir -p ../rootfs/{bin,sbin,etc,proc,sys,usr/{bin,sbin},lib,tmp,home,root,mnt,dev}

# 安装 BusyBox，../rootfs 目录中包含了 BusyBox 安装的二进制文件和基础目录结构。
make ARCH=arm64 CROSS_COMPILE=${CROSS_COMPILE} CONFIG_PREFIX=../rootfs install

# 根文件系统的 dev 目录中创建设备节点
cd ../rootfs
sudo mknod -m 666 dev/null c 1 3
sudo mknod -m 600 dev/console c 5 1
sudo mknod -m 600 dev/ttyAMA0 c 4 64


# 创建启动脚本`/etc/init.d/rcS`
```bash
sudo mkdir -p etc/init.d
sudo tee etc/init.d/rcS > /dev/null <<EOF
#!/bin/sh
mount -t proc proc /proc
mount -t sysfs sysfs /sys
mount -o rw -t ext4 /dev/vda
exec /bin/sh
EOF

sudo chmod +x etc/init.d/rcS
```

# 创建 init 文件以设置默认入口：
```bash
sudo tee etc/inittab > /dev/null <<EOF
::sysinit:/etc/init.d/rcS
::askfirst:-/bin/sh
EOF
```

### 制作根文件系统镜像
创建一个空的镜像文件，然后格式化为ext4，并挂载以复制文件进去，制作根文件系统镜像以供 QEMU 使用：
```bash
cd ..
# 创建一个大小为 128MB 的文件并格式化为 ext4
dd if=/dev/zero of=rootfs.img bs=1M count=128
mkfs.ext4 rootfs.img
# 挂载并复制文件
sudo mkdir -p /mnt/rootfs
sudo mount -o loop rootfs.img /mnt/rootfs
sudo cp -a rootfs/* /mnt/rootfs

# 卸载镜像
sudo umount /mnt/rootfs
sudo rmdir /mnt/rootfs
```

### 配置网络桥接
```bash
# 新增一个网桥，网桥名称可改，但后续需保持一致
sudo brctl addbr br0
# 网桥作为QEMU中linux的网关，需要配置IP地址（可改，但后续需保持一致）
sudo ifconfig br0 192.168.122.1/24
# 启动网桥
sudo ifconfig br0 up
# 将ip forward功能打开
sudo sysctl -w net.ipv4.ip_forward=1

# 配置iptables，ens33为宿主机的网卡名称
sudo iptables -A FORWARD -i br0 -o ens33 -j ACCEPT
sudo iptables -A FORWARD -o br0 -i ens33 -j ACCEPT
sudo iptables -t nat -A POSTROUTING -s 192.168.122.0/24 -j MASQUERADE # NAT地址转换
```

##　启动 QEMU 测试内核
### 启动命令
使用 QEMU 启动内核：
```bash
# 确保已经安装了QEMU
sudo apt install qemu-system-arm

# TAP 设备直连，网络隔离环境
sudo qemu-system-aarch64 -M virt -cpu cortex-a57 -m 1024 -nographic \
    -kernel phytium-linux-kernel-kernel-6.6_v3.0/arch/arm64/boot/Image \
    -append "root=/dev/vda rw console=ttyAMA0 init=/bin/sh" \
    -drive if=none,file=rootfs.img,format=raw,id=hd0 \
    -device virtio-blk-device,drive=hd0 \
    -netdev tap,id=net0,ifname=tap0,script=no,downscript=no \
    -device virtio-net-device,netdev=net0

# 网桥连接
qemu-system-aarch64 -M virt -cpu cortex-a57 -m 1024 -nographic \
    -kernel phytium-linux-kernel-kernel-6.6_v3.0/arch/arm64/boot/Image \
    -append "root=/dev/vda rw console=ttyAMA0 init=/bin/sh" \
    -drive if=none,file=rootfs.img,format=raw,id=hd0 \
    -device virtio-blk-device,drive=hd0 \
    -netdev bridge,id=net0,br=br0 \
    -device virtio-net-device,netdev=net0

# 目录共享
sudo qemu-system-aarch64 -M virt-4.0 -m 1G -cpu cortex-a57 -nographic \
    -kernel zImage \
    -initrd openeuler-image-qemu-aarch64-*.rootfs.cpio.gz \
    -netdev bridge,br=br_qemu,id=net0 -device virtio-net-pci,netdev=net0 \
    -device virtio-9p-device,fsdev=fs1,mount_tag=host \
    -fsdev local,security_model=passthrough,id=fs1,path=/home/protocol/project 
```

- `-M virt`：指定虚拟机模型为 virt，适用于 AArch64 架构。
- `-cpu cortex-a57`：指定虚拟 CPU 为 Cortex-A57，适用于 ARM 架构。
- `-m 1024`：分配 1024 MB 内存给虚拟机。
- `-nographic`：禁用图形输出，使用命令行界面输出。
- `-kernel`：加载指定的内核镜像。
- `-dtb`：指定设备树文件，用于描述硬件配置。
- `-append`：添加内核启动参数：
- `root=/dev/vda`：指定根文件系统为 /dev/vda，即您在 -drive 中指定的虚拟磁盘。
- `console=ttyAMA0`：通过串口控制台进行输出。
- `-drive`：指定虚拟磁盘（根文件系统镜像 rootfs.img）。
- `-initrd`：指定初始化 RAM 磁盘文件 rootfs.cpio.gz，包含启动所需的文件系统。如果根文件系统不是cpio（比如ext4）归档，则不需要。
- `-netdev` 和 `-device`：配置虚拟机的网络设备，使用 virtio-net 网络驱动。

#### 配置openEuler Embedded网卡
```bash
ip link set eth0 up
ip addr add 192.168.1.100/24 dev eth0
ip route add default via 192.168.1.1
```

****

本文参考

> 1. [linaro交叉编译工具链下载与使用笔记](https://blog.csdn.net/yjkhtddx/article/details/134676016)
> 2. [linux kernel mount rootfs失败问题](https://www.cnblogs.com/banshanjushi/p/17677402.html)