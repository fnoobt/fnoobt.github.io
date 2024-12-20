---
title: ZXR10 1800路由器配置
author: fnoobt
date: 2019-05-21 11:49:00 +0800
categories: [Network,路由配置]
tags: [network,网络,zte,zxr10]
---

ZXR10 1809&1800系列路由器简单配置，功能很多，这里只配置两个协议，主要是为了能让它实现简单路由上网。

首先用超级终端进入路由器配置，这里值得注意的是载波率为115200并非默认的9600

配置代码
```
ZXR10>en     
ZXR10>zxr10   // 密码
ZXR10#config terminal   // 进入全局配置模式
ZXR10(config)#webenable    // 使能网页后台页面
ZXR10(config)#ip local pool dhcp 192.168.10.2 192.168.10.254 255.255.255.0 // DHCP地址域名称及地址范围
ZXR10(config)#ip dhcp enable     // 启用路由器DHCP功能
ZXR10(config)#ip dhcp server dns 211.142.211.124 111.8.14.18 // 设置DHCP的域名服务器
ZXR10(config)#interface fei_1/2    // 开始内网口配置
ZXR10(config-if)#porttype l3       // 设置端口属性为三层
ZXR10(config-if)#ip address 192.168.10.1 255.255.255.0  // 设备内网口接口地址
ZXR10(config-if)#ip dhcp mode server // 设置内网口启用DHCP
ZXR10(config-if)#ip dhcp server gateway 192.168.10.1 // 设置内网口DHCP网关
ZXR10(config-if)#peer default ip pool dhcp  // 设置内网口DHCP所用地址域
ZXR10(config-if)#exit
ZXR10(config)#ip dhcp server update arp
ZXR10(config)#ip local pool conflict-ip 10
ZXR10(config)#ip dhcp server leasetime 90

ZXR10(config)#ip nat start     // 启用路由器NAT功能
ZXR10(config)#interface fei_1/2
ZXR10(config-if)#ip nat inside   // 设置内网口NAT属性
ZXR10(config-if)#exit

ZXR10(config)#interface fei_1/1  // 外网口
ZXR10(config-if)#porttype l3     // 设置端口属性为三层
ZXR10(config-if)#ip address 211.142.218.162 255.255.255.252 // 设置外网口接口地址，及业务地址
ZXR10(config-if)#ip nat outside  // 设置外网口NAT属性
ZXR10(config-if)#exit

ZXR10(config)#acl standard number 10   // NAT转换规则
ZXR10(config-std-acl)#permit 192.168.10.0 0.0.0.255 // 设置NAT转换的私网地址段
ZXR10(config-std-acl)#exit

ZXR10(config)#ip nat pool outpool 211.142.218.162 211.142.218.162 prefix-length 30 // 设备NAT转换后的公网地址，也即业务地址
ZXR10(config)#ip nat inside source list 10 pool outpool overload // 匹配NAT转换规则
ZXR10(config)#ip route 0.0.0.0 0.0.0.0 211.142.218.161 // 设置路由器默认路由
ZXR10(config)#exit
ZXR10#write   // 保存配置
```

配置说明：
- `ip nat pool start_ip end_ip prefix-length NAT转换池` 其中的开始IP和结束IP均为外网IP非私网IP
- `ip local pool pool_name start_ip end_ip subnet_mask` 配置dhcp地址池 私网IP池
- `portytype l3` 设置为端口为三层，OSI的第三层即网络层，l2为数据链路层即是做交换机交换数据用
- `ip local pool conflict-ip 10` IP冲突时间
- `ip dhcp server leasetime 90` IP租期时间
- `ip dhcp server update arp` 更新arp
- `stand acl number  acl_number` 标准访问控制列表，1-99为标准控制列表，100-199为扩展控制列
- `permit 192.168.10.0 0.0.0.255` 允许NAT地址转换的IP地址的规则 ，后面的为子网掩码，逆向子网掩码
- `ip nat inside/ouside` 分别为地址转换的内网和地址转换的外网
- `ip route 0.0.0.0 0.0.0.0 gateway` 所有的数据向网关发送
- `ip nat inside source list acl_number pool nat_poolname overload` NAT转换

****

本文参考

> 1. [ZXR10 1800路由器配置](https://wenku.baidu.com/view/2db13e6e93c69ec3d5bbfd0a79563c1ec5dad729.html?_wkts_=1690290012826&bdQuery=ZXR10+1800%E8%B7%AF%E7%94%B1%E5%99%A8%E9%85%8D%E7%BD%AE)