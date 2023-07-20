---
title: VPS性能测试脚本
author: fnoobt
date: 2022-07-17 00:34:00 +0800
categories: [VPS,脚本]
tags: [VPS,脚本]
img_path: '/assets/img/commons/vps'
---

## bensh.sh

bensh.sh测试的内容包括VPS系统基本信息、全球各知名数据中心的测试点的速度（支持IPv6）以及IO 测试。

使用方法：

```bash
wget -qO- bench.sh | bash
```

测试结果如下：

![Bench](bench.png)

测试结果图来自：[SuperBench.sh](https://www.vpsgo.com/url/aHR0cHM6Ly93d3cub2xka2luZy5uZXQvMzUwLmh0bWw=)

## ZBench

ZBench 将bench.sh和SuperBench.sh这两个脚本结合在一起，然后加入 Ping 以及 路由测试 功能。

中文版使用方法：

```bash
wget -N --no-check-certificate https://raw.githubusercontent.com/FunctionClub/ZBench/master/ZBench-CN.sh && bash ZBench-CN.sh
```

英文版使用方法：

```bash
wget -N --no-check-certificate https://raw.githubusercontent.com/FunctionClub/ZBench/master/ZBench-CN.sh && bash ZBench-CN.sh
```

测试结果如下：

![ZBench](ZBench.png)

测试结果图来自：[ZBench](https://www.vpsgo.com/url/aHR0cHM6Ly9naXRodWIuY29tL0Z1bmN0aW9uQ2x1Yi9aQmVuY2g=)

## 91yuntest

91yuntest支持IO测试、宽带测试、SpeedTest国内节点测试、世界各地下载速度测试、路由测试、回程路由测试、全国Ping测试、国外Ping测试、UnixBench跑分测试：

使用方法：

```bash
wget -N --no-check-certificate https://raw.githubusercontent.com/91yun/91yuntest/master/test.sh && bash test.sh -i "io,bandwidth,chinabw,download,traceroute,backtraceroute,allping"
```

****

[原文出处](https://www.vpsgo.com/vps-test-scripts.html)

