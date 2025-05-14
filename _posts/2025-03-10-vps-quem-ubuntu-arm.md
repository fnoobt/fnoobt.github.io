---
title:  Ubuntu x86系统中通过QEMU安装Ubuntu ARM系统
author: fnoobt
date: 2025-03-10 14:17:00 +0800
categories: [VPS,服务搭建]
tags: [vps,linux,quem,ubuntu,arm]
---

## 环境准备
### 1、安装QEMU及相关工具
在Ubuntu x86虚拟机中执行以下命令安装QEMU和ARM架构支持包：
```bash
sudo apt update
sudo apt install qemu qemu-system-arm qemu-efi qemu-user-static
```

需要确保包含`qemu-system-arm`（模拟ARM系统）和`qemu-efi`（支持UEFI启动）

### 2、下载ARM版Ubuntu镜像
从[官网](https://cdimage.ubuntu.com/releases/)获取ARM架构的Ubuntu镜像，或通过以下命令下载：
```bash
wget https://cdimage.ubuntu.com/releases/24.04/release/ubuntu-24.04-live-server-arm64.iso
```
### 3、准备UEFI固件文件
QEMU需要加载UEFI固件以启动ARM虚拟机，下载地址：
```bash
wget https://releases.linaro.org/components/kernel/uefi-linaro/16.02/release/qemu64/QEMU_EFI.fd
```

## 创建虚拟磁盘
### 生成初始img文件
使用 qemu-img 创建动态分配的 QCOW2 格式镜像（支持动态分配空间）
```bash
qemu-img create -f qcow2 ubuntu.img 20G

# 查看镜像信息
qemu-img info ubuntu.img
```

## 启动QEMU安装ARM系统
​启动安装命令
执行以下命令启动QEMU虚拟机，加载ISO镜像并开始安装：
```bash
qemu-system-aarch64 -m 4096 -cpu cortex-a57 -smp 4 \
  -M virt -bios QEMU_EFI.fd \
  -drive file=ubuntu-24.04-live-server-arm64.iso,media=cdrom,format=raw \
  -drive file=ubuntu.img,format=qcow2,if=none,id=hd0 \
  -device virtio-blk-pci,drive=hd0 \
  -netdev user,id=net0 \
  -device virtio-net-pci,netdev=net0 \
  -nographic
```

​参数说明：
- `-m 4096`：分配 4096 MB 内存给虚拟机。
- `-cpu cortex-a57`：指定ARM处理器类型。
- `-smp 4`：分配CPU核心数4个。
- `-M virt`：模拟ARM的`virt`通用平台，适用于 AArch64 架构。
- `-bios`：BIOS文件，指定ARM64 UEFI 固件路径。
- `-drive media=cdrom`：定义 CD-ROM 或 DVD 驱动器，加载安装 ISO。
- `-drive if=none,id=hd0`：定义磁盘，通过`id`命名。
- `-device virtio-blk-pci`：定义 virtio-blk 设备，并连接到指定ID的驱动器。
- `-netdev user`：定义用户模式网络，`user` 模式模拟一个 NAT 网络，通过`id`命名。
- `-device virtio-net-pci`：定义一个标准的 `virtio-net-pci` 网络设备，连接到 `net0` 网络后端。
- `-nographic`：文本模式安装（若需图形界面则替换为 `-vnc :1`）
  
> 注意：​必须创建 EFI 系统分区​（类型 EF00，建议 512MB），文件系统推荐使用 ext4 格式，引导安装位置`/dev/vda`（对应 virtio 设备）。
{: .prompt-info }  

根据安装向导完成Ubuntu ARM系统的安装，过程与物理机安装一致。安装完成后，移除`-cdrom`参数并重新启动即可进入已安装的系统。
```bash
qemu-system-aarch64 -m 4096 -cpu cortex-a57 -smp 4 \
  -M virt -bios QEMU_EFI.fd \
  -drive file=ubuntu.img,format=qcow2,if=none,id=hd0 \
  -device virtio-blk-pci,drive=hd0 \
  -netdev user,id=net0,hostfwd=tcp::2222-:22 \
  -device virtio-net-pci,netdev=net0 \
  -nographic
```

参数说明：
- `-netdev user,id=net0,hostfwd=tcp::2222-:22`：定义网络后端，`hostfwd`将宿主机上的端口流量转发到虚拟机内部的端口。

退出系统使用`Ctrl+A X`组合键，先同时按下Ctrl和A键，接着按X键。

## 常见问题解决
### ​启动时卡在 UEFI Shell
这是因为未检测到有效引导分区，在 UEFI Shell 中手动引导：
```bash
fs0:           # 进入第一个文件系统
\EFI\ubuntu\grubaa64.efi  # 执行引导文件
```

强制重新安装GRUB并生成正确的EFI文件路径，进入系统后执行
```bash
sudo grub-install --target=arm64-efi --efi-directory=/boot/efi --removable
sudo update-grub
```

在虚拟机内运行`lsblk`检查磁盘分区，确保EFI分区（通常为vda1或vda2）已挂载到`/boot/efi`

若EFI分区未挂载，手动挂载并更新`/etc/fstab`：
```bash
sudo mount /dev/vda1 /boot/efi
echo "/dev/vda1 /boot/efi vfat defaults 0 1" | sudo tee -a /etc/fstab
```

### 网络无法连接
首先检查 `-netdev user` 参数是否正确映射端口，其次在虚拟机内执行 `dhclient eth0` 手动获取 IP。

当没有网络端口时，可以尝试将`-drive file=ubuntu.img,format=qcow2,if=none,id=hd0`替换为`-drive file=ubuntu.img,format=qcow2,if=virtio`，并删除`-device virtio-blk-pci,drive=hd0`，使用 `if=virtio` 作为一个快捷方式来定义一个使用 Virtio 块设备的驱动器。QEMU 会自动为这个驱动器创建一个 virtio-blk 设备并连接它。

### 传输文件
当 QEMU 网络模式为用户模式 (User Mode)，也称为 SLiRP，QEMU 在宿主机上模拟了一个简单的 NAT 网络。虚拟机的 IP 地址（例如 10.0.2.15）是在这个由 QEMU 模拟的私有网络内部的。宿主机不会在这个 10.0.2.0/24 网段拥有 IP 地址，因此您无法直接从宿主机通过 10.0.2.15 这个 IP 地址访问虚拟机，可以通过使用 SSH 和端口转发实现文件传输。

1、 从宿主机向虚拟机传输文件：

```bash
# 上传文件到虚拟机
scp -P 2222 /path/to/your/local/file username@localhost:/path/on/vm/

# 从虚拟机下载文件
scp -P 2222 username@localhost:/path/on/vm/file /path/to/your/local/
```

2、 从虚拟机向宿主机传输文件：

虚拟机可以直接访问宿主机。宿主机的 IP 地址通常是 10.0.2.2（QEMU 模拟的网关）。
```bash
scp /path/on/vm/file yourhostusername@10.0.2.2:/path/on/host/
```

如果要传递文件夹，可以使用`scp` 的`-r`参数，递归传输目录及子文件。

****

本文参考

> [QEMU创建arm64的ubuntu22.04虚拟机](https://blog.csdn.net/weixin_40837318/article/details/136737347)