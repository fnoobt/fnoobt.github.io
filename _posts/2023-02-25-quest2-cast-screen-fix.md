---
title: Quest2 开机卡透视和投屏修复
author: fnoobt
date: 2023-02-25 16:31:00 +0800
categories: [VR,VR教程]
tags: [vr,quest2]
---

因为 Quest2 开机会自动检测网络连接，导致国内会出现开机卡透视，此外投屏时默认的联网判断是访问谷歌或者Facebook，因为国内这俩都不能正常访问，所以会提示网络受限无法投屏

## 开机卡透视

这是由于开机连接了网络但是无法连接 Quest2 服务器，需要在Quest2中关闭wifi自动连接，如果无法进入操作页面试试下面两个方法

### 1.更换 wifi

离开原来 wifi 覆盖区域，或者关闭路由器让 Quest2 无法连接wifi

### 2.用 adb 关闭开机自动连wifi

```bash
adb root
adb shell svc wifi disable
```

## 投屏修复

用 adb 完全关闭网络受限检测
```bash
adb shell settings put global captive_portal_mode 0 
```

**或者**设置光临为连接小米（MIUI）的服务器检测
```bash
adb shell settings put global captive_portal_https_url http://connect.rom.miui.com/generate_204
```

重新在头显中关闭再打开WIFI，此时wifi状态会变成已连接，就可以进行投屏了

****

本文参考

> 1. [QUEST2卡透视、开机卡安全区域的解决方法](https://www.bilibili.com/read/cv16865650)
> 2. [Quest2 国内投屏修复-不用魔法](https://www.bilibili.com/read/cv18921201)