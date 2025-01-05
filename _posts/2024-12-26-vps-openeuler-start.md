---
title:  Ubuntu 22.04 构建OpenEuler Embedded
author: fnoobt
date: 2024-12-26 20:17:00 +0800
categories: [VPS,服务搭建]
tags: [vps,linux,ubuntu,openeuler]
---

## OpenEuler Embedded安装
openEuler Embedded是基于openEuler社区**面向嵌入式场景**的Linux版本。由于嵌入式系统应用受到如资源、功耗、多样性等因素的约束，面向服务器领域的Linux及相应的构建系统很难满足嵌入式场景的要求，因此业界广泛采用 **Yocto** 来定制化构建嵌入式Linux。openEuler Embedded当前也采用了Yocto进行构建，但实现了与openEuler其他版本代码同源。

> Yocto 项目是一个协作式开源项目，旨在为嵌入式Linux和物联网（IoT）设备开发提供一套可扩展、高性能且可靠的工具和方法。它并不直接提供一个预构建的Linux发行版，而是提供了创建定制化Linux系统所需的各种工具和资源，允许开发者根据具体需求构建适合特定硬件平台的Linux镜像。

### 构建 ARM64 QEMU 镜像
openEuler Embedded采用yocto构建，同时设计了基于Python的元工具 oebuild 以简化相对复杂的yocto构建流程。按照以下步骤可以快速构建出一个openEuler Embedded镜像。

> 当前 **仅支持在x86_64位的Linux环境** 下使用 oebuild 进行构建，且需在 **普通用户** 下进行 oebuild 的安装运行。更多关于 oebuild 的介绍请参阅 [oebuild 介绍](https://embedded.pages.openeuler.org/master/oebuild/userguide/install/index.html#oebuild-install) 章节。
> openEuler Embedded 的 CI 会归档最新的构建镜像。若希望快速获取可用的镜像，请访问 dailybuild 地址_121.36.84.172/dailybuild/EBS-openEuler-Mainline_，在 dailybuild/EBS-openEuler-Mainline/openeuler-xxxx-xx-xx/embedded_img 中可以下载镜像。
{: .prompt-info }

#### 1. 安装必要的主机包
需要在构建主机上安装必要的主机包，包括oebuild及其运行依赖：
```bash
# 安装必要的软件包
$ sudo apt-get install git python3 python3-pip docker docker.io
# 安装oebuild，如果同时安装了pyhotn2.X和3.X，可能需要使用pip3 install oebuild
$ sudo pip install oebuild

# 配置docker环境
$ sudo usermod -a -G docker $(whoami)
$ sudo systemctl daemon-reload && sudo systemctl restart docker
$ sudo chmod o+rw /var/run/docker.sock
```

#### 2. 初始化oebuild构建环境

> 在开始构建前，请确保构建主机满足以下条件：
> - 至少有 50G 以上的空闲磁盘空间，建议预留尽可能多的空间，有助于运行多个镜像构建，通过重复构建提升效率。
> - 至少有 8G 内存，建议使用内存、CPU数量更多的机器，增加构建速度。
{: .prompt-info }

运行 oebuild 完成初始化工作，包括创建工作目录、拉取构建容器等，之后的构建都需要在 `<work_dir>` 下进行：
```bash
# <work_dir> 为要创建的工作目录
$ oebuild init <work_dir>

# 切换到工作目录
$ cd <work_dir>

# 拉取构建容器、yocto-meta-openeuler 项目代码
$ oebuild update
```

oebuild 的工作目录与 openEuler Embedded 版本是相关联的，构建不同版本的 openEuler Embedded ，需要创建相对应的工作目录。
- `oebuild init [-u URL] [-b BRANCH] [DIRECTORY]`oebuild初始化的完整命令
 + `-u <URL>`：yocot-meta-openeuler 仓库地址，默认为 [yocto-meta-openeuler](https://gitee.com/openeuler/yocto-meta-openeuler)。
 + `-b <BRANCH>`：yocot-meta-openeuler 版本分支，默认为 [master](https://gitee.com/openeuler/yocto-meta-openeuler/tree/master)。 如需构建其它版本，直接使用该参数指定对应分支，例如：-b openEuler-22.03-LTS-SP2。
 + `<DIRECTORY>`：需要创建的工作目录。

#### 3. 开始构建
继续执行以下命令进行 `ARM64 QEMU` 镜像的构建，`build_arm64`为该镜像的构建目录：
```bash
# 所有的构建工作都需要在 oebuild 工作目录下进行
$ cd <work_dir>

# 为 openeuler-image-qemu-arm64 镜像创建配置文件 compile.yaml
$ oebuild generate -p qemu-aarch64 -d build_arm64

# 切换到包含 compile.yaml 的编译空间目录，如 build/build_arm64/
$ cd build/build_arm64/

# 根据提示进入 build_arm64 构建目录，并开始构建，该步骤耗时很久
$ oebuild bitbake openeuler-image
```

除了使用上述命令进行配置文件生成之外，还可以使用`oebuild generate`命令进入到菜单选择界面进行对应配置填写和选择，此菜单选项可以替代上述命令中的 `oebuild generate -p qemu-aarch64 -d build_arm64` ，选择保存之后继续执行上述命令中的bitbake及后续命令即可。

运行`oebuild generate`后，会弹出如下的界面:
```bash
    *** this is choose build target ***
    choice build target (OS)  --->
    *** this is choose OS platform ***
    choice platform (qemu-aarch64)  --->
    *** this is choose OS features ***
[ ] wayland (NEW)
[ ] hmi (NEW)
[ ] dsoftbus (NEW)
[ ] debug (NEW)
[ ] isulad (NEW)
[ ] x11 (NEW)
[ ] xen (NEW)
[ ] mcs (NEW)
[ ] systemd (NEW)
[ ] obmc (NEW)
[ ] zvm (NEW)
[ ] ros (NEW)
[ ] busybox (NEW)
[ ] musl (NEW)
[ ] kernel6 (NEW)
[ ] rt (NEW)
[ ] kubeedge (NEW)
[ ] clang (NEW)
[ ] opengl (NEW)
    *** this is common config ***
    choice build in docker or host (docker)  --->
[ ] no_fetch (this will set openeuler_fetch to disable) (NEW)
[ ] no_layer (this will disable layer repo update when startting bitbake environment) (NEW)
(None) sstate_mirrors (this param is for SSTATE_MIRRORS) (NEW)
(None) sstate_dir (this param is for SSTATE_DIR) (NEW)
(None) toolchain_dir (this param is for external gcc toolchain dir, if you want use your own toolchain) (NEW)
(None) llvm_toolchain_dir (this param is for external llvm toolchain dir, if you want use your own toolchain) (NEW)
(None) datetime (NEW)
(None) cache_src_dir     (this param is cache src directory) (NEW)
(None) directory     (this param is build directory) (NEW)
```

1. 此时在`choice build target`中选择OS表示构建OS镜像，
2. 在`choice platform`选择需要构建的单板，图中示例为qemu-aarch64
3. 在`os features`下选择需要构建的特性
4. 在`directory`输入编译目录名，至此按q，退出，按y确认即可生成编译目录

#### 4. 运行镜像
完成构建后，在构建目录下的 `output/<构建时间戳>` 目录下可以看到如下文件：
- `zImage`: 内核镜像，基于openEuler社区Linux 5.10内核构建；
- `openeuler-image-qemu-xxx.cpio.gz`: 标准根文件系统镜像， 进行了必要安全加固，增加了audit、cracklib、OpenSSH、Linux PAM、shadow、iSulad容器等所支持的软件包；
- `openeuler-image-qemu-aarch64-xxx.iso`: iso格式的镜像，可用于制作U盘启动盘；
- `vmlinux`: 对应的vmlinux镜像，可用于内核调试。

在主机上通过以下命令安装QEMU:
```bash
sudo apt-get install qemu-system-arm
```

之后，通过以下命令启动镜像：
```bash
#进入目录
cd output/<构建时间戳>

sudo qemu-system-aarch64 -M virt-4.0 -m 1G -cpu cortex-a57 -nographic \
    -kernel zImage \
    -initrd openeuler-image-qemu-aarch64-*.rootfs.cpio.gz \
    -netdev bridge,br=br_qemu,id=net0 -device virtio-net-pci,netdev=net0 \
    -device virtio-9p-device,fsdev=fs1,mount_tag=host \
    -fsdev local,security_model=passthrough,id=fs1,path=/home/protocol/project 
```

QEMU运行成功并登录后，将会呈现openEuler Embedded的Shell。

如果想关闭当前镜像，可以使用’<Ctrl-A>+X’直接退出，或者在初始用户登录完成后，通过以下命令关闭:
```bash
poweroff
```

QEMU就会退出并回到启动时的目录。

> 由于标准根文件系统镜像进行了安全加固，因此第一次启动时，需要为登录用户名root设置密码，且密码强度有相应要求，需要 **数字、字母、特殊字符组合最少8位**，例如openEuler@2023
{: .prompt-info }

## QEMU使用

```bash
# ubuntu安装QEMU
sudo apt-get install qemu-system-arm

# qemu执行命令
qemu-system-aarch64 -M virt-4.0 -m 1G -cpu cortex-a57 -nographic \
    -kernel zImage \
    -initrd openeuler-image-qemu-aarch64-*.rootfs.cpio.gz \
    -netdev bridge,br=br_qemu,id=net0 -device virtio-net-pci,netdev=net0 \
    -device virtio-9p-device,fsdev=fs1,mount_tag=host \
    -fsdev local,security_model=passthrough,id=fs1,path=/home/protocol/project 
```

### 使能互联网
```bash
出栈
虚拟机网卡（eth0）-> 虚拟机网卡（vnet0）->网桥（br）---- iptable forward -----宿主机网卡（enp1s0）->发出
入栈
虚拟机网卡（eth0）<- 虚拟机网卡（vnet0）<-网桥（br）---- iptable forward -----宿主机网卡（enp1s0）<-收到
```

#### 宿主机中
```bash
# 新增一个网桥，网桥名称可改，但后续需保持一致
sudo brctl addbr br_qemu
# 网桥作为QEMU中openEuler Embedded的网关，需要配置IP地址（可改，但后续需保持一致）
sudo ifconfig br_qemu 192.168.122.1/24
# 启动网桥
sudo ifconfig br_qemu up
# 将ip forward功能打开
sudo sysctl -w net.ipv4.ip_forward=1

# 配置iptables，ens33为宿主机的网卡名称
sudo iptables -A FORWARD -i br_qemu -o ens33 -j ACCEPT
sudo iptables -A FORWARD -o br_qemu -i ens33 -j ACCEPT
sudo iptables -t nat -A POSTROUTING -s 192.168.122.0/24 -j MASQUERADE # NAT地址转换
```

启动QEMU时添加netdev
```bash
sudo qemu-system-aarch64 -M virt-4.0 -m 1G -cpu cortex-a57 -nographic \
    -kernel zImage \
    -initrd openeuler-image-qemu-aarch64-*.rootfs.cpio.gz \
    -netdev bridge,br=br_qemu,id=net0 -device virtio-net-pci,netdev=net0
```

其中， `-netdev` 选项中 `br` 参数为宿主机上新建的网桥名称。 `-device` 选项中 `netdev` 参数为 `-netdev` 指定的 `id`， 指定了设备应该连接到的后端。

#### 配置openEuler Embedded网卡
```bash
ifconfig eth0 192.168.122.11 netmask 255.255.255.0
ip route add default via 192.168.122.1 dev eth0
```

其中，192.168.122.11可以更改为任意的未被占用的局域网地址。但是，需要保证与宿主机的网桥在同一个网段。 ip route 添加默认网关的作用是，当openEuler Embedded需要访问互联网时，将数据包转发到宿主机的网桥上，再由宿主机的网桥转发到互联网。

#### 配置DNS
```bash
# 修改DNS配置文件
vi /etc/resolv.conf
# 添加DNS服务器地址
nameserver 114.114.114.114
```

## 共享文件系统
过共享文件系统，可以使得运行 QEMU 仿真器的宿主机和 openEuler Embedded 共享文件，这样在宿主机上交叉编译的程序，拷贝到共享目录中，即可在 openEuler Embedded 上运行。注意 QEMU 必须支持 virtfs，即配置了 `--enable-virtfs`。
假设将宿主机的 `/tmp/project ` 目录作为共享目录，使能共享文件系统功能的操作指导如下：

### 启动QEMU时添加fsdev，运行如下命令：
```bash
sudo qemu-system-aarch64 -M virt-4.0 -m 1G -cpu cortex-a57 -nographic \
    -kernel zImage \
    -initrd openeuler-image-qemu-aarch64-*.rootfs.cpio.gz \
    -device virtio-9p-device,fsdev=fs1,mount_tag=host \
    -fsdev local,security_model=passthrough,id=fs1,path=/tmp/project 
```

### 映射文件系统
在 openEuler Embedded 启动并登录之后，需要运行如下命令，映射(mount)共享文件系统：
```bash
mkdir project
mount -t 9p -o trans=virtio,version=9p2000.L host ~/project
```

## 安装dnf
```bash
# 下载dnf
wget https://repo.openeuler.openatom.cn/openEuler-24.03-LTS/everything/aarch64/Packages/dnf-4.16.2-3.oe2403.noarch.rpm

# 将 .rpm 文件转换为 .cpio 格式，解压到当前目录
rpm2cpio dnf-4.16.2-3.oe2403.noarch.rpm | cpio -idmv

rm -rf ./lib/ld-linux-aarch64.so.1
rm -rf ./lib64/libc.so.6

cp -r etc/* /etc/
cp -r lib64/* /lib64/
cp -r run/* /run/
cp -r sbin/* /sbin/
cp -r var/* /var/
cp -r usr/* /usr/

# 将 /bin/sh 符号链接到 /bin/bash
ln -sf /bin/bash /bin/sh

# 导入 GPG 公钥
rpm --import https://repo.openeuler.openatom.cn/openEuler-24.03-LTS/everything/aarch64/RPM-GPG-KEY-openEuler

# 手动安装 rpm 包
rpm -ivh rpm-*.rpm

# 安装 dnf
rpm -ivh dnf-*.rpm

# 查找链接对应的库，在已安装的系统上，先用find查找所在的文件路径
find / -name libgcrypt.so.20
# 再用rpm -qf查找对应的库
rpm -qf /usr/lib64/libgcrypt.so.20
```

****

本文参考

> 1. [openEuler Embedded快速上手](https://embedded.pages.openeuler.org/master/getting_started/index.html)
> 2. [构建OS镜像使用指导](https://embedded.pages.openeuler.org/master/oebuild/userguide/build/index.html)