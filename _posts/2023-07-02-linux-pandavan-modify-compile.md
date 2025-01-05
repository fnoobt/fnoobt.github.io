---
title: Padavan固件修改编译
author: fnoobt
date: 2023-07-02 19:29:00 +0800
categories: [Linux,Linux教程]
tags: [linux,padavan,编译]
---

Padavan固件是俄罗斯人Andy Padavan在bitbucket上发布的基于华硕路由器开源项目 asuswrt 修改而来的 rt-n56u 这套路由器系统的源码，所以国内就直接把这个路由器固件叫padavan了，俗称老毛子。

同梅林固件相似的是，老毛子固件也是由华硕官方固件修改而来，两者都在华硕官方固件的基础上，做了一定的功能加强。

与梅林固件不同的是，老毛子固件支持RT3883/MT7620/MT7621/MT7628 等系列 CPU 的路由器，对位的是中低端路由器产品；而梅林固件仅支持博通系列CPU，对位的是中高端系列路由器。

国内所说的老毛子固件，一般指的是在原版Padavan固件基础上，经过国内大神二次修改优化，更适合国内用户使用的版本。

Padavan固件特点：

- 硬件方面，适用于联发科 MTK 7620/7621/7628 CPU平台。
- 源自华硕官方固件，运行非常稳定。
- 对IPv6的支持相对完善。
- 支持硬件加速，转发效率高。
- 可集成众多功能强大的常用插件，比如去广告、SmartDNS、Adguard等。
- 可集成Shadowsocks/SSR/V2ray/Wireguard等常用科学上网插件。

文件结构 /trunk/ 下
```
├── configs  ---路由器型号和配置文件
│   ├── boards ---路由器适配文件目录，每个文件夹对应一款路由器
│   │   └── NEWIFI3 ---newifi3适配文件
│   │       ├── board.h
│   │       ├── board.mk
│   │       ├── kernel-3.4.x.config
│   │       └── libc.config -> ../uclibc-mipsel.config
│   └── templates
│       └── newifi_3.config ---newifi3配置，新增的插件编译选项开关可在此处追加
├── user ---用户自定义的配置文件和脚本
|   ├── Makefile ---控制所有插件的编译，和路由器配置联动达到插件可选化编译和打包
|   ├── frp
|   |   |-- Makefile ---控制单个插件的编译、打包文件产出，参照其他插件例子即可
|   |   └-- ...
|   ├── www  ---web页面文件，padavan中多数为asp
|   |   ├── dict  ---i18n实现，按需调整
|   |   │   └── CN.dict ---简体中文语言文件
|   |   ├── Makefile  ---新增插件若有相关页面文件，需要调整该文件
|   |   ├── state.js ---控制页面目录中的插件页面入口，新增插件需要调整该文件
|   |   └── n56u_ribbon_fixed
|   |       |-- PLUGIN_RELATED.asp ---插件页面相关的asp文件
|   |       └-- ...
|   ├── httpd ---构建http服务
|   |   ├── variables.c ---插件相关nvram持久化数据的变量结构体定义
|   |   ├── web_ex.c ---处理asp后端逻辑的核心文件，主要关注暴露出来的函数，如update_variables_ex
|   |   ├── common.h ---关注variable，为结构体属性的类型
|   |   └── ...
|   ├── shared ---数据保存
|   |   ├── defaults.c ---定义插件相关变量的默认值
|   |   └── ... 
|   ├── rc ---启动第三方应用和执行脚本,router control
|   |   ├── rc.c 
|   |   └── ... 
|   └── ...
└── tools ---用于构建和维护Padavan固件的工具和脚本
```

![PandavanFileStructure](/assets/img/commons/linux/router/pandavan-file-structure.png)
_Padavan web应用交互逻辑_

****

本文参考

> 1. [Padavan插件开发笔记](https://b.coz.moe/posts/tech/how-to-build-a-padavan-plugin/)
> 2. [Padavan源码分析](https://foxhome.top/2021/04/29/435.html)