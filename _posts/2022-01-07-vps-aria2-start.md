---
title: Aria2 安装（一键安装管理脚本）
author: fnoobt
date: 2022-01-07 15:32:00 +0800
categories: [VPS,服务搭建]
tags: [vps,aria2,ariang]
---

Aria2 是目前最强大的全能型下载工具，它支持 BT、磁力、HTTP、FTP 等下载协议，常用做离线下载的服务端。  
本教程安装 P3TERX 大佬开发的[Aria2增强版脚本](https://github.com/P3TERX/aria2.sh)，该脚本整合了 Aria2 完美配置，在安装 Aria2 的过程中会下载一套配置方案，这套方案包含了配置文件、附加功能脚本等文件，用于实现 Aria2 功能的增强和扩展，提升 Aria2 的下载速度与使用体验，解决 Aria2 在使用中遇到的 BT 下载无速度、文件残留占用磁盘空间、任务丢失、重复下载等问题。

## 系统要求

CentOS 6+ / Debian 6+ / Ubuntu 14.04+

## 架构支持

x86_64 / i386 / ARM64 / ARM32v7 / ARM32v6

## 使用说明

- 为了确保能正常使用，请先安装基础组件`wget`、`curl`、`ca-certificates`，以 Debian 为例子：
```bash
apt install wget curl ca-certificates
```

- 下载脚本
```bash
wget -N git.io/aria2.sh && chmod +x aria2.sh
```

- 运行脚本
```bash
./aria2.sh
```

- 在运行界面，选择你要执行的选项

```bash
 Aria2 一键安装管理脚本 增强版 [v2.7.4] by P3TERX.COM
 
  0. 升级脚本
 ———————————————————————
  1. 安装 Aria2
  2. 更新 Aria2
  3. 卸载 Aria2
 ———————————————————————
  4. 启动 Aria2
  5. 停止 Aria2
  6. 重启 Aria2
 ———————————————————————
  7. 修改 配置
  8. 查看 配置
  9. 查看 日志
 10. 清空 日志
 ———————————————————————
 11. 手动更新 BT-Tracker
 12. 自动更新 BT-Tracker
 ———————————————————————

 Aria2 状态: 已安装 | 已启动

 自动更新 BT-Tracker: 已开启

 请输入数字 [0-12]:
```

## 其他操作
启动：`/etc/init.d/aria2 start | service aria2 start`

停止：`/etc/init.d/aria2 stop | service aria2 stop`

重启：`/etc/init.d/aria2 restart | service aria2 restart`

查看状态：`/etc/init.d/aria2 status | service aria2 status`

配置文件路径：`/root/.aria2c/aria2.conf`{: .filepath} （配置文件有中文注释，若语言设置有问题会导致中文乱码）

默认下载目录：`/root/downloads`{: .filepath}

RPC 密钥：随机生成，可使用选项`7. 修改 配置`文件自定义

## 联动网盘离线下载

首先确保 Aria2 和 Rclone 已安装好

打开 Aria2 配置文件进行修改`/root/.aria2c/aria2.conf`{: .filepath}。找到“下载完成后执行的命令”，把`clean.sh`替换为`upload.sh`。
```conf
# 下载完成后执行的命令
on-download-complete=/root/.aria2c/upload.sh
```

打开附加功能脚本配置文件进行修改`/root/.aria2c/script.conf`{: .filepath}，有中文注释，按照自己的实际情况进行修改，第一次使用只建议修改网盘名称。
```conf
# 网盘名称(RCLONE 配置时填写的 name)
drive-name=GDrive
```

重启 Aria2 。脚本选项重启或者执行以下命令：
```bash
service aria2 restart
```

执行`upload.sh`脚本，提示success即代上传脚本能正常被调用，否则请检查与 RCLONE 有关的配置。
```bash
/root/.aria2c/upload.sh
```

更多信息可参见[Aria2 + Rclone 实现 OneDrive、Google Drive 等网盘离线下载](https://p3terx.com/archives/offline-download-of-onedrive-gdrive.html)
