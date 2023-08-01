---
title: VPS性能测试脚本
author: fnoobt
date: 2021-07-17 17:12:00 +0800
categories: [VPS,脚本]
tags: [vps,脚本,linux]
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

## NetFlix解锁检测脚本

使用方法：
```bash
wget -O nf https://github.com/sjlleo/netflix-verify/releases/download/v3.1.0/nf_linux_amd64 && chmod +x nf && ./nf
```

测试结果如下：
```bash
**NetFlix 解锁检测小工具 v3.0 By @sjlleo**
[IPv4]
Netflix在您的出口IP所在的国家不提供服务

[IPv6]
您的出口IP完整解锁Netflix，支持非自制剧的观看
NF所识别的IP地域信息：美国
```

相关名词解释
- 不提供服务 - 所在的地区NF没开通，连自制剧也看不了
- 宽松版权 - 有些NF拍摄的影片不是特别注重版权，所以限制放的很开
- 解锁自制剧 - 代表可以看由NF自己拍摄的影片
- 解锁非自制剧 - 代表可以看NF买下的第三方版权影片
- 地域解锁 - NF在不同的地区可以看的片源都是不同的，有些影片只能在特定区观看

****

> [测速](https://www.vpsgo.com/vps-test-scripts.html)

