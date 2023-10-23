---
title: EVE-NG 使用教程
author: fnoobt
date: 2023-10-23 15:29:00 +0800
categories: [Network,虚拟环境]
tags: [network,eve-ng]
---

EVE-NG（Emulated Virtual Environment - NextGeneration）下一代仿真虚拟环境，是一种基于软件定义网络（SDN）和网络功能处理（NFV）技术的网络模拟器和实验平台，可以模拟各大厂商（如华为、华三、思科等）的网络设备，路由器，交换机，防火墙等等，EVE-NG之所以可以模拟这么多不同厂商的网络设备是因为其三大组件：Dynamips，IOL，QEMU
- **Dynamips**是用于模拟思科路由器的，基于它的模拟器有小凡，以及大名鼎鼎的GNS3，但它只能模拟路由，交换基本上都不行
- **IOL**是将思科的ios运行在linux上，可以很好的支持交换功能，基于它的模拟器有web-iou
- **QEMU**是一套开源产品，是用纯软件实现的模拟器，几乎可以模拟任何硬件设备

****

本文参考

> 1. [EVE-NG详细安装使用指南](https://blog.csdn.net/balabalado/article/details/131867478)