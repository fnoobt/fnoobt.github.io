---
title: BGP源码分析
author: fnoobt
date: 2023-10-06 19:29:00 +0800
categories: [Network,路由协议]
tags: [Network,BGP]
---

##  名词解释

#### BGP发言者（BGP Speaker）
发送BGP消息的路由器称为BGP发言者

#### 对等体（Peer）
相互交换消息的BGP发言者之间互称对等体

#### IBGP（Internal BGP）
当BGP运行于同一自治系统内部时，被称为IBGP

#### EBGP（External BGP）
当BGP运行于不同自治系统之间时，称为EBGP

#### 路由聚合（Routes Aggregation）
在大规模的网络中，BGP路由表十分庞大，使用路由聚合可以大大减小路由表的规模。路由聚合实际上是将多条路由合并的过程。这样BGP在向对等体通告路由时，可以只通告聚合后的路由，而不是将所有的具体路由都通告出去。

#### BGP路由衰减（Route Dampening）
BGP路由衰减是用来解决路由不稳定的问题。路由不稳定的主要表现形式是路由振荡（Route flaps），即路由表中的某条路由反复消失和重现。

BGP衰减使用惩罚值来衡量一条路由的稳定性，惩罚值越高则说明路由越不稳定。路由每发生一次振荡（路由从激活状态变为未激活状态），BGP便会给此路由增加一定的惩罚值。当惩罚值超过抑制阈值时，此路由被抑制，不加入到路由表中，也不再向其他BGP对等体发布更新报文。

#### 团体属性（Community）
BGP Community只是BGP路由可以携带的一种属性，是BGP中的一个可选可传递属性，所以可以选择为路由配置Community，也可以不配。如果配置，Community需要明确要求BGP路由器保留和传递该属性，否则邻居收不到路由的Community属性。

#### RD属性过滤器（Route Distinguisher-Filter）

#### AS路径属性（AS_PATH）

#### 对等体组（Peer Group）
Peer Group是一些具有某些相同属性的对等体的集合。当一个对等体加入对等体组中时，此对等体将获得与所在对等体组相同的配置。当对等体组的配置改变时，组内成员的配置也相应改变。在大型BGP网络中，对等体的数量会很多，其中很多对等体具有相同的策略，在配置时会重复使用一些命令，利用对等体组在很多情况下可以简化配置。

#### 联盟（Confederation）
Confederation是处理AS内部的IBGP网络连接激增的另一种方法，它将一个自治系统划分为若干个子自治系统，每个子自治系统内部的IBGP对等体建立全连接关系，子自治系统之间建立联盟内部EBGP连接关系。

## 源码结构
BGP源码目录结构
```bash
|-- Makefile      #编译文件
|-- bgp_addpath.c
|-- bgp_addpath.h
|-- bgp_addpath_types.h
|-- bgp_advertise.c   #定义一系列函数，主要用于将具有相同Update信息的BPG相邻结点封装加入到同一个队列或从队列中取出，进行发送。
|-- bgp_advertise.h   #定义BGP的发送方式和相邻结点。
|-- bgp_aspath.c    #主要进行AS路径管理操作。
|-- bgp_aspath.h    #主要用于进行AS path的相关定义，并以线性表方式存储
|-- bgp_attr.c
|-- bgp_attr.h    #主要进行BGP 类型的定义
|-- bgp_attr_evpn.c
|-- bgp_attr_evpn.h
|-- bgp_bfd.c
|-- bgp_bfd.h
|-- bgp_bmp.c
|-- bgp_bmp.h
|-- bgp_btoa.c
|-- bgp_clist.c   #主要实现了BGP团体属性的列表和拓展属性列表
|-- bgp_clist.h   #主要定义了BGP团体属性的列表，以线性表的方面存储。
|-- bgp_community.c   #实现了团体属性的相关函数，其中包括简单增加团体属性，删除团体属性，并根据as path属性值进行综合排序。
|-- bgp_community.h   #定义了团体属性的相关函数
|-- bgp_community_alias.c
|-- bgp_community_alias.h
|-- bgp_conditional_adv.c
|-- bgp_conditional_adv.h
|-- bgp_damp.c
|-- bgp_damp.h   #定义了路由衰落惩罚状态的数据以及惩罚控制的信息
|-- bgp_debug.c
|-- bgp_debug.h
|-- bgp_dump.c
|-- bgp_dump.h
|-- bgp_ecommunity.c   #实现了BGP扩展社区属性，可以实现向扩展社区属性中添加信息，以及扩展属性的复制、合并等操作
|-- bgp_ecommunity.h   #定义了BGP扩展社区属性
|-- bgp_encap_tlv.c
|-- bgp_encap_tlv.h
|-- bgp_encap_types.h
|-- bgp_errors.c
|-- bgp_errors.h
|-- bgp_evpn.c
|-- bgp_evpn.h
|-- bgp_evpn_mh.c
|-- bgp_evpn_mh.h
|-- bgp_evpn_private.h
|-- bgp_evpn_vty.c
|-- bgp_evpn_vty.h
|-- bgp_filter.c   #实现了AS path过滤器操作，其中包含AS过滤器的增、删、改、查操作
|-- bgp_filter.h   #定义了AS path过滤器，其中包含过滤器的两种状态——拒绝（deny）和接受（permit）
|-- bgp_flowspec.c
|-- bgp_flowspec.h
|-- bgp_flowspec_private.h
|-- bgp_flowspec_util.c
|-- bgp_flowspec_util.h
|-- bgp_flowspec_vty.c
|-- bgp_fsm.c   #实现BGP的三种类型状态功能。
|-- bgp_fsm.h   #定义了BGP的一系列有限状态
|-- bgp_io.c
|-- bgp_io.h
|-- bgp_keepalives.c
|-- bgp_keepalives.h
|-- bgp_label.c
|-- bgp_label.h
|-- bgp_labelpool.c
|-- bgp_labelpool.h
|-- bgp_lcommunity.c
|-- bgp_lcommunity.h
|-- bgp_mac.c
|-- bgp_mac.h
|-- bgp_main.c   #bgpd代码的主程序，用于事件的开始，执行一系列的初始化，启动bgp的有限状态的转化。通过命令行不同参数的输入，执行一系列不同状态的参数设置与对应事件，启动master主线程以及解析配置文件等。
|-- bgp_memory.c
|-- bgp_memory.h
|-- bgp_mpath.c
|-- bgp_mpath.h
|-- bgp_mplsvpn.c   #主要用于解析MPLS-VPN的相关操作
|-- bgp_mplsvpn.h   #主要用于定义MPLS-VPN的属性值
|-- bgp_mplsvpn_snmp.c
|-- bgp_mplsvpn_snmp.h
|-- bgp_network.c
|-- bgp_network.h   #定义了BPG网络的相关头部信息
|-- bgp_nexthop.c
|-- bgp_nexthop.h
|-- bgp_nht.c
|-- bgp_nht.h
|-- bgp_open.c   #发送open报文和进行消息处理，使数据能进行交互
|-- bgp_open.h   #主要用于处理BGP发送的open信息，其中定义了需要发送报文的标准头部信息
|-- bgp_packet.c   #实现BGP管理数据的程序，主要包含增加分组、释放分组，检查连接等操作
|-- bgp_packet.h   #定义BGP用于数据管理的头部信息，定义了bgp的read、write、keepalive、open、notify、update、receive等报文的发送方式。
|-- bgp_pbr.c
|-- bgp_pbr.h
|-- bgp_rd.c
|-- bgp_rd.h
|-- bgp_regex.c   #实现AS的正则表达式匹配与转化。
|-- bgp_regex.h   #定义了AS中有关正则表达式匹配的相关函数
|-- bgp_route.c   #实现的静态路由的相关操作，以及过滤器方法和许多路由相关方法。
|-- bgp_route.h   #定义了bgp路由的基本信息库
|-- bgp_routemap.c
|-- bgp_routemap_nb.c
|-- bgp_routemap_nb.h
|-- bgp_routemap_nb_config.c
|-- bgp_rpki.c
|-- bgp_rpki.h
|-- bgp_script.c
|-- bgp_script.h
|-- bgp_snmp.c
|-- bgp_snmp.h
|-- bgp_snmp_bgp4.c
|-- bgp_snmp_bgp4.h
|-- bgp_snmp_bgp4v2.c
|-- bgp_snmp_bgp4v2.h
|-- bgp_table.c   #实现了bgp的路由表的简单操作
|-- bgp_table.h   #定义了bgp的路由向量表
|-- bgp_trace.c
|-- bgp_trace.h
|-- bgp_updgrp.c
|-- bgp_updgrp.h
|-- bgp_updgrp_adv.c
|-- bgp_updgrp_packet.c
|-- bgp_vnc_types.h
|-- bgp_vpn.c
|-- bgp_vpn.h
|-- bgp_vty.c
|-- bgp_vty.h
|-- bgp_zebra.c   #主要处理Zebra的接口事件，重新分配路由。
|-- bgp_zebra.h   #定义了zebra的连接、初始化以及重新分配的相关操作函数。
|-- bgpd.c   #主要进行BGP实例的初始化工作
|-- bgpd.h   #定义主要的BGP发送信息。
|-- rfapi
|   |-- bgp_rfapi_cfg.c
|   |-- bgp_rfapi_cfg.h
|   |-- rfapi.c
|   |-- rfapi.h
|   |-- rfapi_ap.c
|   |-- rfapi_ap.h
|   |-- rfapi_backend.h
|   |-- rfapi_descriptor_rfp_utils.c
|   |-- rfapi_descriptor_rfp_utils.h
|   |-- rfapi_encap_tlv.c
|   |-- rfapi_encap_tlv.h
|   |-- rfapi_import.c
|   |-- rfapi_import.h
|   |-- rfapi_monitor.c
|   |-- rfapi_monitor.h
|   |-- rfapi_nve_addr.c
|   |-- rfapi_nve_addr.h
|   |-- rfapi_private.h
|   |-- rfapi_rib.c
|   |-- rfapi_rib.h
|   |-- rfapi_vty.c
|   |-- rfapi_vty.h
|   |-- vnc_debug.c
|   |-- vnc_debug.h
|   |-- vnc_export_bgp.c
|   |-- vnc_export_bgp.h
|   |-- vnc_export_bgp_p.h
|   |-- vnc_export_table.c
|   |-- vnc_export_table.h
|   |-- vnc_import_bgp.c
|   |-- vnc_import_bgp.h
|   |-- vnc_import_bgp_p.h
|   |-- vnc_zebra.c
|   `-- vnc_zebra.h
|-- rfp-example
|   |-- librfp
|   |   |-- Makefile
|   |   |-- rfp.h
|   |   |-- rfp_example.c
|   |   |-- rfp_internal.h
|   |   `-- subdir.am
|   `-- rfptest
|       |-- Makefile
|       |-- rfptest.c
|       |-- rfptest.h
|       `-- subdir.am
`-- subdir.am
```

#### bgpd.h
定义主要的BGP发送信息

Peer：指相互交换消息的BGP发言者，其中包含BGP的实例化，AS编号，本地AS编号，路由表，当前状态，详细信息（文件描述符，TTL，最小跳数，端口号），工作队列等。

BGP_notify: 出错时通知信息的格式，其中包含用于指定错误类型的差错码code，显示错误类型的详细信息的差错子码subcode，发现错误的原因数据data，以及信息长度length

Peer_goup: 具有某些相同属性的对等体的集合，实现相互交换报文。定义了BGP列表，邻居结点Peer列表

此外还定义了一些BGP相应事件的状态的标识符。

#### bgpd.c
主要进行BGP实例的初始化工作，初始化BGP对应的命令行,Zebra，相关属性，访问列表，前缀表，社团属性，启动BGP_master主线程，此外还进行BPG相关操作如BGP的分配，配置以及停止等。

#### bgp_advertise.h
定义BGP的发送方式和相邻结点

BGP_advertise：定义了先进先出的发送方式，相同属性的发送链表，前缀表信息，BGP信息。

BGP_adj_out: 定义了BGP的邻接关系，其中包含对等体peer，发送属性，以及发送链表。

#### bgp_aspath.h:
主要用于进行AS path的相关定义，并以线性表方式存储。

Assegment: 定义了AS Path的相关数据，其中包含A实例，AS path报文长度，报文内容种类以及线性表中指向下一个segment的next指针。

Aspath: 定义存放as path的结构体，包含AS路径的字符串表达式和AS号。

#### bgp_aspath.c
主要进行AS路径管理操作。其中包括根据报文解析并创建segments，路径解析，将数据写入As path的操作，合并AS操作，以及链表的新建结点操作、复制操作、释放操作、加入新结点等简单操作。

#### bgp_clist.h:
主要定义了BGP团体属性的列表，以线性表的方面存储。

community_entry: 定义BGP团体属性的每个个体项目，其中包括指向前/后的团体属性指针以及其他的属性。

#### bgp_clist.c
主要实现了BGP团体属性的列表和拓展属性列表，其中包含团体属性列表中的增加结点、删除结点、清空结点的简单操作以及团体属性的正则匹配。

#### bgp_community.h:
定义了团体属性的相关函数。

community：定义了团体的相关属性，如用于输出和扩展团体列表进行正则匹配的字符串，以及团体属性的值。

#### bgp_fsm.c:
实现BGP的三种类型状态功能。状态一是线程函数。状态二是事件函数。状态三是关于FSM函数。 bgp_timer_set 函数用于设置计时器，通过当前bgp的不同状态（Idle、Connect、Active、OpenSent、OpenConfirm、Established）执行相应的事件，bgp_start_time函数用于启动计时器，并将启动事件放入主线程中。

#### bgp_regex.c
实现AS的正则表达式匹配与，其中通过转化函数，将字符“_”根据相应情况转化成字符“(”、“^”、“|”、“[”、“{”、“}”、“]”、“$”、“)”。

#### bgp_route.h
定义了bgp路由的基本信息库，其中包含路由的类型（RIP、OSPF、BGP），BGP路由状态（NORMAL、STATIC、AGGTEGATE、REDISTRIBUTE）以及一系列bgp的信息初始化和解析函数。

#### bgp_table.h
定义了bgp的路由向量表，其中包含bgp_node类，通过存放BGP路由的关键数据，用于分派、查找、实际操作，适用于协议处理不多的情况。

#### bgp_table.c
实现了bgp的路由表的简单操作，其中包括bgp_node结点的初始化、创建、销毁和加入，以及路由表在线程中的加锁（lock）、解锁（unlock）、初始化与结束等操作。

****

本文参考

> 1. [BGP官方文档](https://docs.frrouting.org/en/latest/bgp.html)
> 2. [SONiC BGP源码分析](https://vlambda.com/wz_7i9lzpN4MSE.html)