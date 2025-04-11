---
title:  Windows 网络变化后 VMware 虚拟机无法联网
author: fnoobt
date: 2025-04-10 21:17:00 +0800
categories: [Windows,软件教程]
tags: [windows,vmware]
---

## 问题现象
Windows11下使用VMware 17 Pro安装了Ubuntu 22.04虚拟机。Windows开机先连接WiFi，再打开Ubuntu虚拟机时，Ubuntu可以正常联网。当Windows中途断开一段时间的WiFi之后，再次连接WiFi，Ubuntu无法联网，重新重启Windows系统才可以恢复联网。

## 问题分析
这种现象的核心原因在于，当Windows 11主机的WiFi断开再重新连接时，宿主机的网络状态发生了变化（比如可能获取了新的IP地址，或者网络接口状态重置）。VMware用于连接虚拟机和宿主机网络的虚拟网络服务（如NAT服务、DHCP服务、桥接服务）未能正确地感知到宿主机的网络变化，导致IP地址分配失败或虚拟网络路由表未更新，VMware自然无法继续为虚拟机提供网络通路。

1. **VMware NAT模式**: 如果你的虚拟机网络设置为NAT模式，VMware会在宿主机上运行一个NAT服务（VMnetNAT）和一个DHCP服务（VMnetDHCP）来为虚拟机分配私有IP地址并进行网络地址转换。当宿主机WiFi重连后，这个NAT服务可能没有成功地重新绑定到新的网络连接状态上，导致虚拟机无法通过它访问外部网络。
2. **VMware桥接模式**: 如果你的虚拟机网络设置为桥接模式（Bridged），它会尝试直接连接到你的物理网络（通过桥接宿主机的WiFi适配器）。当WiFi断开重连时，这个虚拟的“桥”可能与物理WiFi适配器的连接状态没有正确更新，导致虚拟机无法获取IP地址或与网络通信。
3. **VMware虚拟网络适配器**: VMware在Windows上安装了虚拟网络适配器（如VMnet1, VMnet8等）。宿主机的网络变化可能导致这些虚拟适配器与物理适配器之间的映射关系或者路由出现问题。

## 解决方法
### 一、​重启VMware网络服务（无需重启）
通过下面的批处理脚本重启Windows中的VMware网络服务。

```bat
net stop "VMware NAT Service"
net stop "VMware DHCP Service"
net start "VMware NAT Service"
net start "VMware DHCP Service"
```
{: file="重启VMware服务.bat" }

### 二、修复Ubuntu虚拟机网络配置​
1、重启Ubuntu网络服务

重启NetworkManager服务：

```bash
sudo systemctl restart NetworkManager
```

若仍无效，尝试清空网络状态文件：

```bash
sudo service network-manager stop
sudo rm /var/lib/NetworkManager/NetworkManager.state
sudo service network-manager start
```

2、检查IP地址分配

若未获取IP，尝试释放并重新请求DHCP租约：

```bash
# 释放当前IP          获取新IP
sudo dhclient -r && sudo dhclient
```

****

本文参考

> 1. [彻底解决VM ubuntu在虚拟机找不到网卡无法上网的问题](https://blog.csdn.net/linZinan_/article/details/135262830)
> 2. [VMware 虚拟机里连不上网的五种解决方案](https://cloud.tencent.com/developer/article/2103898)