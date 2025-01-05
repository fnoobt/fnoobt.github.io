---
title: 小米6刷第三方REC之TWRP
author: fnoobt
date: 2020-09-08 17:21:00 +0800
categories: [Flash,手机]
tags: [flash,刷机,xiaomi,twrp]
---

## 准备阶段

测试例子
- adb/fastboot工具：Minimal ADB and Fastboot
- 刷之前的Miui为：线刷了Miui国际开发版8.3.1
- TWRP：官网上小米6的3.2.1-0版本
- 安装完成后刷入了基于原生Android开源项目的ROM包（基于android 8.1.0）

## 前提条件
- 手机已经解锁了BL（BootLoader）  
- 备份好手机资料，如通讯录、照片等  
- 电量充足，建议>80%

## 下载TWRP
到[TWRP设备列表](https://twrp.me/Devices/)中搜索`xiaomi`，进入找到[小米6机型](https://twrp.me/xiaomi/xiaomimi6.html)

在`Download Links`中选择一个区域进入下载页面，如[Primary (Americas)](https://dl.twrp.me/sagit/)

选择一个版本下载对应的img文件，并改名为`twrp.img`（可以不改，或改成其它名称如recovery.img），将文件复制到`adb/fastboot`工具的安装目录

## 刷入TWRP

1. 开启系统开发模式，打开USB调试开关
2. 用数据线连接手机和电脑，进入ADB文件夹，按住shift+鼠标右键打开命令行，依次输入如下命令

```bash
adb devices
adb reboot bootloader
fastboot flash recovery twrp.img
fastboot boot twrp.img # 此行命令可以进TWRP的rec（或者刷入TWRP后重启前按音量加减和电源键进手机rec模式）
fastboot reboot
```

>**注意：**如果在刷入TWRP完成后手机重启前，没有通过TWRP安装一些如第三方rom替换官方miui或者刷入SuperSU等root管理软件，那么手机重启后，TWRP会被覆盖，需要重新刷入
{: .prompt-tip }

****

本文参考

> 1. [Libin](https://zhuanlan.zhihu.com/p/34412300)
> 2. [TWRP for Mi6](https://twrp.me/xiaomi/xiaomimi6.html)
> 3. [Guide to Install TWRP Custom Recovery via Fastboot Mode](https://www.guidebeats.com/guide-install-twrp-custom-recovery-via-fastboot-mode/)