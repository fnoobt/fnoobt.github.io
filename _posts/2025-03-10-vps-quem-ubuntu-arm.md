---
title:  Ubuntu x86系统中通过QEMU安装Ubuntu ARM系统
author: fnoobt
date: 2025-03-10 14:17:00 +0800
categories: [VPS,服务搭建]
tags: [vps,linux,quem,ubuntu]
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
  -drive file=ubuntu.img,format=qcow2,if=virtio \
  -device virtio-scsi-device \
  -netdev user,id=net0 -device virtio-net-device,netdev=net0 \
  -nographic
```

​参数说明：
- `-M virt`：模拟ARM的“virt”通用平台，适用于 AArch64 架构。
- `-bios`：BIOS文件，指定ARM64 UEFI 固件路径
- `-cpu cortex-a57`：指定ARM处理器类型。
- `-netdev user`：配置用户模式网络（支持虚拟机联网）
- `-device virtio-scsi-device`：确保磁盘设备可被 UEFI 识别
- `-nographic`：文本模式安装（若需图形界面则替换为 -vnc :1）
- `-m 4096`：分配 4096 MB 内存给虚拟机。
- `-smp 4`：核心数4个。
  
> 注意：​必须创建 EFI 系统分区​（类型 EF00，建议 512MB），文件系统推荐使用 ext4 格式，引导安装位置`/dev/vda`（对应 virtio 设备）。
{: .prompt-info }  

根据安装向导完成Ubuntu ARM系统的安装，过程与物理机安装一致。安装完成后，移除`-cdrom`参数并重新启动即可进入已安装的系统。
```bash
qemu-system-aarch64 -m 4096 -cpu cortex-a57 -smp 4 \
  -M virt -bios QEMU_EFI.fd \
  -drive file=ubuntu.img,format=qcow2,if=virtio \
  -device virtio-scsi-device \
  -netdev user,id=net0,hostfwd=tcp::2222-:22 \
  -device virtio-net-device,netdev=net0 \
  -nographic
```

退出系统使用`Ctrl+A X`组合键，先同时按下Ctrl和A键，接着按X键。

## 常见问题解决
### ​启动时卡在 UEFI Shell
这是因为未检测到有效引导分区，在 UEFI Shell 中手动引导：
```bash
fs0:           # 进入第一个文件系统
\EFI\ubuntu\grubaa64.efi  # 执行引导文件
```

重新安装 GRUB，强制重新安装GRUB并生成正确的EFI文件路径。进入 Live 环境挂载镜像后执行
```bash
sudo grub-install --target=arm64-efi --efi-directory=/boot/efi
sudo update-grub
```

在虚拟机内运行`lsblk`检查磁盘分区，确保EFI分区（通常为vda1或vda2）已挂载到`/boot/efi`

若EFI分区未挂载，手动挂载并更新`/etc/fstab`：
```bash
sudo mount /dev/vda1 /boot/efi
echo "/dev/vda1 /boot/efi vfat defaults 0 1" | sudo tee -a /etc/fstab
```

### 网络无法连接
检查 `-netdev user` 参数是否正确映射端口
在虚拟机内执行 `dhclient eth0` 手动获取 IP

****

本文参考

> [QEMU创建arm64的ubuntu22.04虚拟机](https://blog.csdn.net/weixin_40837318/article/details/136737347)