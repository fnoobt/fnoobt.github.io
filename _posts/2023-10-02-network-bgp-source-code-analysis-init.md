---
title: FRR BGP源码分析1 -- 初始化
author: fnoobt
date: 2023-10-02 19:29:00 +0800
categories: [Network,路由协议]
tags: [network,bgp,frr]
---

##  一、名词解释

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
RD 团体属性过滤器是将 VPN 中的 RD 属性作为匹配条件的过滤器，可在VPN 配置中利用 RD 属性区分路由时单独使用。  
VPN 实例通过路由标识符 RD 实现地址空间独立，区分使用相同地址空间的前缀。

#### AS路径属性（AS_PATH）
AS_PATH是记录前往目标网络需要经过AS号的列表。AS_PATH的功能有两个，一个是确保EBGP对等体之间传递无环，另一个是路由优选的策略。  
防环的原理是当AS_PATH中出现自己的区域号的时候，那么当前路由器拒收该路由。路由优选的原理是，如果一条路由经过AS_PATH区域号越少，那么该路由更优。

#### 对等体组（Peer Group）
Peer Group是一些具有某些相同属性的对等体的集合。当一个对等体加入对等体组中时，此对等体将获得与所在对等体组相同的配置。当对等体组的配置改变时，组内成员的配置也相应改变。在大型BGP网络中，对等体的数量会很多，其中很多对等体具有相同的策略，在配置时会重复使用一些命令，利用对等体组在很多情况下可以简化配置。

#### 联盟（Confederation）
Confederation是处理AS内部的IBGP网络连接激增的另一种方法，它将一个自治系统划分为若干个子自治系统，每个子自治系统内部的IBGP对等体建立全连接关系，子自治系统之间建立联盟内部EBGP连接关系。

## 二、源码结构
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
|-- bgp_btoa.c    #测试程序
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
|-- bgp_debug.c  #调试支持（vty 配置、数据包转储）
|-- bgp_debug.h
|-- bgp_dump.c     #MRT兼容的转储格式例程
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
|-- bgp_network.c   #打开和绑定套接字，查找接口地址
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
|-- bgp_routemap.c  #bgp路由图的实现（匹配和设置）
|-- bgp_routemap_nb.c
|-- bgp_routemap_nb.h
|-- bgp_routemap_nb_config.c
|-- bgp_rpki.c
|-- bgp_rpki.h
|-- bgp_script.c
|-- bgp_script.h
|-- bgp_snmp.c  #添加变量或调试SNMP
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
|-- bgp_vty.c   #协议范围的 vty 钩子
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
{: file='bgpd/'}

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

## 三、BGP初始化
初始化在`bgp_main.c`的main函数里开始，main函数里最重要是初始化
### 1.命令行参数的处理
通过 frr_opt_add 添加了一些命令行参数，例如设置BGP监听端口、指定监听地址、禁用内核路由、禁用Zebra通信等。

使用 frr_getopt 来解析命令行参数。

```c
frr_preinit(&bgpd_di, argc, argv);
	frr_opt_add(
		"p:l:SnZe:I:s:" DEPRECATED_OPTIONS, longopts,
		"  -p, --bgp_port     Set BGP listen port number (0 means do not listen).\n"
		"  -l, --listenon     Listen on specified address (implies -n)\n"
		"  -n, --no_kernel    Do not install route to kernel.\n"
		"  -Z, --no_zebra     Do not communicate with Zebra.\n"
		"  -S, --skip_runas   Skip capabilities checks, and changing user and group IDs.\n"
		"  -e, --ecmp         Specify ECMP to use.\n"
		"  -I, --int_num      Set instance number (label-manager)\n"
		"  -s, --socket_size  Set BGP peer socket send buffer size\n");

	/* Command line argument treatment. */
	while (1) {
		opt = frr_getopt(argc, argv, 0);
```
{: file='bgpd/bgp_main.c -- main()'}

### 2.事件驱动的初始化
初始化BGP的主控制结构，包括各种参数、列表的初始化，以及一些相关模块的初始化。

```c
	/* BGP master init. */
	bgp_master_init(frr_init(), buffer_size, addresses);
	bm->port = bgp_port;
```
{: file='bgpd/bgp_main.c -- main()'}

```c
void bgp_master_init(struct event_loop *master, const int buffer_size,
		     struct list *addresses)
{
	qobj_init();  //初始化Q-Object系统，Q-Object是一种FRRouting中用于管理对象生命周期的机制。

	memset(&bgp_master, 0, sizeof(bgp_master));  //使用memset将bgp_master的内存清零，确保初始状态是空的。

	bm = &bgp_master;  //将bm指针指向全局的bgp_master结构。
	bm->bgp = list_new();  //创建一个链表用于存储BGP实例。
	bm->listen_sockets = list_new();  //创建一个链表用于存储监听套接字。
	bm->port = BGP_PORT_DEFAULT;
	bm->addresses = addresses;
	bm->master = master;
	bm->start_time = monotime(NULL);
	bm->t_rmap_update = NULL;
	bm->rmap_update_timer = RMAP_DEFAULT_UPDATE_TIMER;  //设置一些BGP更新和计时器的参数。
	bm->v_update_delay = BGP_UPDATE_DELAY_DEF;
	bm->v_establish_wait = BGP_UPDATE_DELAY_DEF;
	bm->terminating = false;
	bm->socket_buffer = buffer_size;  //设置BGP的套接字缓冲区大小。
	bm->wait_for_fib = false;
	bm->tcp_dscp = IPTOS_PREC_INTERNETCONTROL;
	bm->inq_limit = BM_DEFAULT_Q_LIMIT;
	bm->outq_limit = BM_DEFAULT_Q_LIMIT;

	bgp_mac_init();  //初始化BGP的MAC地址管理。
	/* init the rd id space.
	   assign 0th index in the bitfield,
	   so that we start with id 1
	 */
	bf_init(bm->rd_idspace, UINT16_MAX);  //初始化用于RD（路由区分符）分配的位图。
	bf_assign_zero_index(bm->rd_idspace);  //将0索引分配给RD空间，确保从ID 1开始分配。

	/* mpls label dynamic allocation pool */
	bgp_lp_init(bm->master, &bm->labelpool);  //初始化BGP的MPLS标签池。

	bgp_l3nhg_init();
	bgp_evpn_mh_init();
	QOBJ_REG(bm, bgp_master);  //注册BGP主控制结构为Q-Object。
}
```
{: file='bgpd/bgpd.c'}

### 3.VRF虚拟路由转发的初始化
调用 `bgp_vrf_init` 初始化VRF（虚拟路由转发），包括设置 VRF 操作的钩子函数、创建默认 VRF、处理 NETNS 情况等

```c
/* Initializations. */
	bgp_vrf_init();
```
{: file='bgpd/bgp_main.c -- main()'}

```c
static void bgp_vrf_init(void)
{
	vrf_init(bgp_vrf_new, bgp_vrf_enable, bgp_vrf_disable, bgp_vrf_delete);
}
```
{: file='bgpd/bgpd.c'}

### 4.BGP初始化
调用bgp_init初始化和配置BGP路由守护进程，包括各种BGP功能的初始化、模块的注册以及钩子函数的设置

```c
	/* BGP related initialization.  */
	bgp_init((unsigned short)instance);
```
{: file='bgpd/bgp_main.c -- main()'}

```c
void bgp_init(unsigned short instance)
{
	hook_register(bgp_config_end, peer_unshut_after_cfg);  //注册钩子函数，用于在BGP配置结束后执行peer_unshut_after_cfg

	/* allocates some vital data structures used by peer commands in
	 * vty_init */

	/* pre-init pthreads */
	bgp_pthreads_init();  //初始化BGP的pthread线程。

	/* Init zebra. */
	bgp_zebra_init(bm->master, instance);  //初始化Zebra，用于与路由守护进程之间的通信。

#ifdef ENABLE_BGP_VNC
	vnc_zebra_init(bm->master);
#endif

	/* BGP VTY commands installation.  */
	bgp_vty_init();  //初始化BGP的VTY（Virtual Terminal Interface虚拟终端接口）命令，用于交互式控制和配置。

	/* BGP inits. */
	bgp_attr_init();  //初始化BGP属性。
	bgp_debug_init();  //初始化BGP调试功能。
	bgp_community_alias_init();  //初始化BGP社区别名。
	bgp_dump_init();  //初始化BGP路由信息的Dump功能。
	bgp_route_init();  //初始化BGP路由。
	bgp_route_map_init();  //初始化BGP路由映射。
	bgp_scan_vty_init();  //初始化BGP扫描的VTY命令。
	bgp_mplsvpn_init();  //初始化BGP MPLS VPN。
#ifdef ENABLE_BGP_VNC
	rfapi_init();
#endif
	bgp_ethernetvpn_init();  //初始化BGP以太网VPN。
	bgp_flowspec_vty_init();  //初始化BGP流规格的VTY命令。

	/* Access list initialize. */
	access_list_init(); 
	access_list_add_hook(peer_distribute_update);
	access_list_delete_hook(peer_distribute_update);

	/* Filter list initialize. */
	bgp_filter_init();  //初始化BGP过滤器。
	as_list_add_hook(peer_aslist_add);    //为AS列表添加钩子函数，用于更新对等体信息。
	as_list_delete_hook(peer_aslist_del);  //为AS列表删除钩子函数，用于更新对等体信息。

	/* Prefix list initialize.*/
	prefix_list_init();  //初始化前缀列表。
	prefix_list_add_hook(peer_prefix_list_update);  //为前缀列表添加钩子函数，用于更新对等体信息。
	prefix_list_delete_hook(peer_prefix_list_update);  //为前缀列表删除钩子函数，用于更新对等体信息。
	/* Community list initialize. */
	bgp_clist = community_list_init();  //初始化BGP社区列表。

	/* BFD init */
	bgp_bfd_init(bm->master);  //初始化BFD，用于检测连接的状态。

	bgp_lp_vty_init();  //初始化BGP的标签分配策略。

	bgp_label_per_nexthop_init();  //初始化每个下一跳的BGP标签

	cmd_variable_handler_register(bgp_viewvrf_var_handlers);  //注册BGP的VTY命令处理程序。
}
```
{: file='bgpd/bgpd.c'}

这部分对最重要的内容进行初始化，包含：
#### 4.1 线程的初始化
```c
static void bgp_pthreads_init(void)
{
	/* 使用断言确保bgp_pth_io和bgp_pth_ka为NULL，即尚未被初始化。如果不满足条件，程序会中止执行并输出错误信息。 */
	assert(!bgp_pth_io);  
	assert(!bgp_pth_ka);

	struct frr_pthread_attr io = {
		.start = frr_pthread_attr_default.start,
		.stop = frr_pthread_attr_default.stop,
	};   //该结构体用于I/O线程。
	struct frr_pthread_attr ka = {
		.start = bgp_keepalives_start,
		.stop = bgp_keepalives_stop,
	};    //该结构体用于Keepalives线程。
	bgp_pth_io = frr_pthread_new(&io, "BGP I/O thread", "bgpd_io"); //创建一个I/O线程,线程名称"BGP I/O thread"，线程标识符"bgpd_io"
	bgp_pth_ka = frr_pthread_new(&ka, "BGP Keepalives thread", "bgpd_ka");  //创建Keepalives线程
}
```
{: file='bgpd/bgpd.c'}

FRR模拟了线程，BGP 的线程包含bgpd / bgpd_io / bgpd_ka，他们的主要任务：
- bgpd     --- 处理BGP的所有业务逻辑，处理函数frr_run
- bgpd_io  --- 收发包，然后把报文入队，唤醒bgpd继续处理,入口函数是fpt_run
- bgpd_ka  ---  处理BGP的keeplive，bgp_keepalives_start

#### 4.2 Zebra的初始化
初始化BGP与Zebra之间的通信机制。通过创建一个Zebra客户端，并设置了相应的回调函数和参数。这样的初始化允许BGP与Zebra进行交互，以便获取和传递路由信息。

```c
void bgp_zebra_init(struct event_loop *master, unsigned short instance)
{
	zclient_num_connects = 0;  //初始化连接数计数器为0，用于记录与Zebra的连接数。

	/* 设置Zebra的回调函数，这些回调函数用于处理接口（interface）的创建、启动、关闭和销毁。 */
	if_zapi_callbacks(bgp_ifp_create, bgp_ifp_up,
			  bgp_ifp_down, bgp_ifp_destroy);

	/* Set default values. */
	zclient = zclient_new(master, &zclient_options_default, bgp_handlers,
			      array_size(bgp_handlers));  //创建一个Zebra客户端
	zclient_init(zclient, ZEBRA_ROUTE_BGP, 0, &bgpd_privs);  //初始化Zebra客户端
	zclient->zebra_connected = bgp_zebra_connected;  //设置Zebra客户端的连接回调函数
	zclient->instance = instance;  //Zebra客户端的实例字段
}
```
{: file='bgpd/bgp_zebra.c'}

`zclient_new` 创建一个zclient客户端，`struct thread_master`填入bgp的master，这样客户端的回调函数就在bgp的进程上下文执行
```c
/* Initialize zebra client.  Argument redist_default is unwanted
   redistribute route type. */
void zclient_init(struct zclient *zclient, int redist_default,
		  unsigned short instance, struct zebra_privs_t *privs)
{
	int afi, i;

	/* Set -1 to the default socket value. */
	zclient->sock = -1;  //将Zebra客户端的套接字初始化为-1，表示默认的套接字值
	zclient->privs = privs;  //将Zebra客户端的权限设置为传入的privs结构体

	/* Clear redistribution flags. */
	for (afi = AFI_IP; afi < AFI_MAX; afi++)
		for (i = 0; i < ZEBRA_ROUTE_MAX; i++)
			zclient->redist[afi][i] = vrf_bitmap_init();  //用于表示重分发的各个类型。

	/* Set unwanted redistribute route.  bgpd does not need BGP route
	   redistribution. */
	zclient->redist_default = redist_default;  //设置Zebra客户端的默认路由类型，即不需要重分发的路由类型。
	zclient->instance = instance;  //设置Zebra客户端的实例
	/* Pending: make afi(s) an arg. */
	for (afi = AFI_IP; afi < AFI_MAX; afi++) {
		redist_add_instance(&zclient->mi_redist[afi][redist_default],
				    instance);

		/* Set default-information redistribute to zero. */
		zclient->default_information[afi] = vrf_bitmap_init();
	}

	if (zclient_debug)
		zlog_debug("scheduling zclient connection");

	zclient_event(ZCLIENT_SCHEDULE, zclient);  //调度Zebra客户端的连接事件，表示需要与Zebra建立连接
}
```
{: file='lib/zclient.c'}

`zclient_init` 初始化客户端相关的数据后，会调用 `zclient_event` ，根据不同的Zebra客户端事件类型，执行相应的操作，如建立连接、重新尝试连接或处理读取数据。这是一种基于事件的异步处理机制。添加一个事件，回调函数是 `zclient_connect` ，初始化完成后，后续会和zebra进程建立连接。

```c
static void zclient_event(enum zclient_event event, struct zclient *zclient)
{
	switch (event) {
	case ZCLIENT_SCHEDULE:  //调度连接事件
		event_add_event(zclient->master, zclient_connect, zclient, 0,
				&zclient->t_connect);
		break;
	case ZCLIENT_CONNECT:  //连接事件
		if (zclient_debug)
			zlog_debug(
				"zclient connect failures: %d schedule interval is now %d",
				zclient->fail, zclient->fail < 3 ? 10 : 60);
		event_add_timer(zclient->master, zclient_connect, zclient,
				zclient->fail < 3 ? 10 : 60,
				&zclient->t_connect);
		break;
	case ZCLIENT_READ:  //读取事件
		zclient->t_read = NULL;
		event_add_read(zclient->master, zclient_read, zclient,
			       zclient->sock, &zclient->t_read);
		break;
	}
}
```
{: file='lib/zclient.c'}

然后会填充各种bgp关心的事件的回调函数，在`bgp_zebra_connected`回调函数里面（客户端连接zebra成功后会调用），会注册各种BGP 感兴趣的事件，如router-id, interfaces, redistributed routes。

#### 4.3 命令行的初始化
vty 初始化函数在 vty 中注册和安装与BGP路由器配置相关的各种命令和选项，以便通过命令行界面进行BGP的配置和管理

```c
void bgp_vty_init(void)
{
	cmd_variable_handler_register(bgp_var_neighbor);
	cmd_variable_handler_register(bgp_var_peergroup);

	cmd_init_config_callbacks(bgp_config_start, bgp_config_end);
```
{: file='bgpd/bgp_vty.c'}

调试初始化函数进行注册和安装各种BGP调试命令，以便在调试过程中查看和分析BGP路由器的内部状态和行为

```c
void bgp_debug_init(void)
{
	install_node(&debug_node);

	install_element(ENABLE_NODE, &show_debugging_bgp_cmd);

	install_element(ENABLE_NODE, &debug_bgp_as4_cmd);
	install_element(CONFIG_NODE, &debug_bgp_as4_cmd);
	install_element(ENABLE_NODE, &debug_bgp_as4_segment_cmd);
	install_element(CONFIG_NODE, &debug_bgp_as4_segment_cmd);
```
{: file='bgpd/bgp_debug.c'}

初始化BGP路由器的报文转储功能，使得路由器可以在发送或接收BGP报文时将其内容进行记录或输出，以便进行调试和分析

```c
void bgp_dump_init(void)
{
	memset(&bgp_dump_all, 0, sizeof(bgp_dump_all));
	memset(&bgp_dump_updates, 0, sizeof(bgp_dump_updates));
	memset(&bgp_dump_routes, 0, sizeof(bgp_dump_routes));

	bgp_dump_obuf =
		stream_new(BGP_MAX_PACKET_SIZE + BGP_MAX_PACKET_SIZE_OVERFLOW);

	install_node(&bgp_dump_node);  //注册报文转储节点

	install_element(CONFIG_NODE, &dump_bgp_all_cmd);
	install_element(CONFIG_NODE, &no_dump_bgp_all_cmd);

	hook_register(bgp_packet_dump, bgp_dump_packet);  //钩子在每个BGP报文发送或接收时调用，用于进行报文的转储。
	hook_register(peer_status_changed, bgp_dump_state);  //钩子在BGP对等体状态发生变化时调用，用于更新转储状态。
}
```
{: file='bgpd/bgp_dump.c'}

初始化vty扫描功能，通过vty界面提供了对BGP路由表和导入检查的查看功能，使得用户可以在虚拟终端界面上获取相关的BGP信息

```c
void bgp_scan_vty_init(void)
{
	install_element(VIEW_NODE, &show_ip_bgp_nexthop_cmd);  //用于显示BGP路由表中的下一跳信息
	install_element(VIEW_NODE, &show_ip_bgp_import_check_cmd);  //用于显示BGP导入检查的信息
	install_element(VIEW_NODE, &show_ip_bgp_instance_all_nexthop_cmd);  //用于显示BGP实例中所有路由表的下一跳信息
}
```
{: file='bgpd/bgp_nexthop.c'}

准备注册CLI

#### 4.4 属性相关的初始化
在BGP路由器启动时，会预先初始化这些属性，以确保在运行时可以有效地处理和管理相关的BGP属性

```c
/* Initialization of attribute. */
void bgp_attr_init(void)
{
	aspath_init();  //初始化AS路径相关的数据结构
	attrhash_init();  //初始化AS路径相关的数据结构
	community_init();  //初始化Community属性相关的数据结构
	ecommunity_init();  //初始化Extended Community属性相关的数据结构
	lcommunity_init();  //初始化Large Community属性相关的数据结构
	cluster_init();  //初始化Cluster属性相关的数据结构
	transit_init();  //初始化Transit属性相关的数据结构
	encap_init();  //初始化封装（Encapsulation）属性相关的数据结构
	srv6_init();  //初始化SRv6（Segment Routing over IPv6）属性相关的数据结构
}
```
{: file='bgpd/bgp_attr.c'}

##### 4.4.1 aspath_init
初始化AS Path的hash存储，AS Path 哈希表键生成函数 `aspath_key_make` ，AS Path 比较函数 `aspath_cmp`

```c
/* AS path hash initialize. */
void aspath_init(void)
{
	ashash = hash_create_size(32768, aspath_key_make, aspath_cmp,
				  "BGP AS Path");
}
```
{: file='bgpd/bgp_aspath.c'}

所有的AS Path用全局ashash存放，相关的数据结构如下：

```c
/* AS_PATH segment data in abstracted form, no limit is placed on length */
struct assegment {
	struct assegment *next;  //指向下一个 AS Path 段的指针，因为 AS Path 可以由多个段组成
	as_t *as;  //指向 AS 的指针
	unsigned short length;  //AS Path 段的长度
	uint8_t type;  //AS Path 段的类型
};

/* AS path may be include some AsSegments.  */
struct aspath {
	/* Reference count to this aspath.  */
	unsigned long refcnt;  //对此 AS Path 的引用计数。用于跟踪有多少个对象正在引用此 AS Path，以便在不再需要时正确释放内存。

	/* segment data */
	struct assegment *segments;  //指向 AS Path 的第一个段的指针。

	/* AS path as a json object */
	json_object *json;  //表示 AS Path 的 JSON 对象。可能在某些上下文中用于序列化和反序列化 AS Path。

	/* String expression of AS path.  This string is used by vty output
	   and AS path regular expression match.  */
	char *str;  //AS Path 的字符串表达形式，用于 vty 输出和 AS Path 正则表达式匹配。
	unsigned short str_len;  //字符串表达形式的长度。

	/* AS notation used by string expression of AS path */
	enum asnotation_mode asnotation;  //AS Path 字符串表达形式所使用的 AS 表示法（AS 表示法的模式）
};
```
{: file='bgpd/bgp_aspath.h'}

##### 4.4.2 attrhash_init
初始化属性的hash存储，hash头attrhash，所有的属性用全局的hash存放，数据结构：

```c
static void attrhash_init(void)
{
	attrhash =
		hash_create(attrhash_key_make, attrhash_cmp, "BGP Attributes");
}
```
{: file='bgpd/bgp_attr.c'}

attr是BGP的核心属性结构体，数据结构较大，包含了BGP的所有属性，AS_PATH，community,med, origin, local_pref, med等等，数据结构：

```c
/* BGP core attribute structure. */
struct attr {
	/* AS Path structure */
	struct aspath *aspath;

	/* Community structure */
	struct community *community;

	/* Reference count of this attribute. */
	unsigned long refcnt;

	/* Flag of attribute is set or not. */
	uint64_t flag;

	/* Apart from in6_addr, the remaining static attributes */
	struct in_addr nexthop;
	uint32_t med;
	uint32_t local_pref;
	ifindex_t nh_ifindex;

	/* Path origin attribute */
	uint8_t origin;
```
{: file='bgpd/bgp_attr.h'}

##### 4.4.3 community_init
初始化community的HASH，全局comhash存储，数据结构：

```c
/* Communities attribute.  */
struct community {
	/* Reference count of communities value.  */
	unsigned long refcnt;  // 引用计数

	/* Communities value size.  */
	int size;

	/* Communities value.  */
	uint32_t *val;

	/* Communities as a json object */
	json_object *json;

	/* String of community attribute.  This sring is used by vty output
	   and expanded community-list for regular expression match.  */
	char *str;  //// Community 属性的字符串表示
};
```
{: file='bgpd/bgp_community.h'}

##### 4.4.4 ecommunity_init
初始化ecommunity的HASH ，全局ecomhash存储，数据结构：

```c
/* Extended Communities attribute.  */
struct ecommunity {
	/* Reference counter.  */
	unsigned long refcnt;

	/* Size of Each Unit of Extended Communities attribute.
	 * to differentiate between IPv6 ext comm and ext comm
	 */
	uint8_t unit_size;

	/* Size of Extended Communities attribute.  */
	uint32_t size;

	/* Extended Communities value.  */
	uint8_t *val;

	/* Human readable format string.  */
	char *str;

	/* Disable IEEE floating-point encoding for extended community */
	bool disable_ieee_floating;
};
```
{: file='bgpd/bgp_ecommunity.h'}

##### 4.4.5 lcommunity_init
初始化lcommunity的HASH ，全局lcomhash存储，数据结构：

```c
/* Large Communities attribute.  */
struct lcommunity {
	/* Reference counter.  */
	unsigned long refcnt;

	/* Size of Extended Communities attribute.  */
	int size;

	/* Large Communities value.  */
	uint8_t *val;

	/* Large Communities as a json object */
	json_object *json;

	/* Human readable format string.  */
	char *str;
};
```
{: file='bgpd/bgp_lcommunity.h'}

##### 4.4.6 cluster_init
初始化 cluster 路由反射器的HASH，全局变量cluster_hash，数据结构：

```c
/* Router Reflector related structure. */
struct cluster_list {
	unsigned long refcnt;
	int length;
	struct in_addr *list;
};
```
{: file='bgpd/bgp_attr.h'}

##### 4.4.7 transit_init
初始化传输的属性的HASH，全局变量transit_hash，数据结构：

```c
/* Unknown transit attribute. */
struct transit {
	unsigned long refcnt;
	int length;
	uint8_t *val;
};
```
{: file='bgpd/bgp_attr.h'}

##### 4.4.8 encap_init
初始化BGP Encap Hash，数据结构：

```c
/* PMSI tunnel types (RFC 6514) */

struct bgp_attr_encap_subtlv {
	struct bgp_attr_encap_subtlv *next; /* for chaining */
	/* Reference count of this attribute. */
	unsigned long refcnt;
	uint16_t type;
	uint16_t length;
	uint8_t value[0]; /* will be extended */
};
```
{: file='bgpd/bgp_attr.h'}

##### 4.4.7 transit_init
初始化传输的属性的HASH，全局变量transit_hash，数据结构：

```c
/* Unknown transit attribute. */
struct transit {
	unsigned long refcnt;
	int length;
	uint8_t *val;
};
```
{: file='bgpd/bgp_attr.h'}

##### 4.4.8 srv6_init
初始化srv6的HASH，数据结构：

```c
/*
 * Prefix-SID type-4
 * SRv6-VPN-SID-TLV
 * draft-dawra-idr-srv6-vpn-04
 */
struct bgp_attr_srv6_vpn {
	unsigned long refcnt;
	uint8_t sid_flags;
	struct in6_addr sid;
};

/*
 * Prefix-SID type-5
 * SRv6-L3VPN-Service-TLV
 * draft-dawra-idr-srv6-vpn-05
 */
struct bgp_attr_srv6_l3vpn {
	unsigned long refcnt;
	uint8_t sid_flags;
	uint16_t endpoint_behavior;
	struct in6_addr sid;
	uint8_t loc_block_len;
	uint8_t loc_node_len;
	uint8_t func_len;
	uint8_t arg_len;
	uint8_t transposition_len;
	uint8_t transposition_offset;
};
```
{: file='bgpd/bgp_attr.h'}

#### 4.5 路由表的初始化
初始化 BGP 路由表的各个组件，同时安装相关的命令，以便用户可以配置和管理 BGP 路由

```c
void bgp_route_init(void)
{
	afi_t afi;
	safi_t safi;

	/* Init BGP distance table. */
	FOREACH_AFI_SAFI (afi, safi)
		bgp_distance_table[afi][safi] = bgp_table_init(NULL, afi, safi);

	/* IPv4 BGP commands. */
	install_element(BGP_NODE, &bgp_table_map_cmd);
	install_element(BGP_NODE, &bgp_network_cmd);
	install_element(BGP_NODE, &no_bgp_table_map_cmd);

	install_element(BGP_NODE, &aggregate_addressv4_cmd);
```
{: file='bgpd/bgp_route.c'}

初始化 BGP 路由表，`bgp_table_init`接受 BGP 实例（bgp）、网络类型（afi）和子网络类型（safi）作为参数，并返回一个指向初始化后的 BGP 路由表的指针

```c
/*
 * bgp_table_init
 */
struct bgp_table *bgp_table_init(struct bgp *bgp, afi_t afi, safi_t safi)
{
	struct bgp_table *rt;

	rt = XCALLOC(MTYPE_BGP_TABLE, sizeof(struct bgp_table));

	rt->route_table = route_table_init_with_delegate(&bgp_table_delegate);  //初始化一个路由表

	/*
	 * Set up back pointer to bgp_table.
	 */
	route_table_set_info(rt->route_table, rt);

	/*
	 * pointer to bgp instance allows working back from bgp_path_info to bgp
	 */
	rt->bgp = bgp;

	bgp_table_lock(rt);
	rt->afi = afi;
	rt->safi = safi;

	return rt;
}
```
{: file='bgpd/bgp_table.c'}

bgp_table数据结构可以对 BGP 路由表进行灵活的管理和操作，并且与 BGP 协议的不同特性和网络类型相关联：

```c
struct bgp_table {
	/* table belongs to this instance */
	struct bgp *bgp;  //路由表所属的 BGP 实例

	/* afi/safi of this table */
	afi_t afi;    //网络类型（IPV4, IPV6）
	safi_t safi;  //子网络类型（SAFI_UNICAST,SAFI_MULTICAST,SAFI_RESERVED_3,SAFI_MPLS_VPN,SAFI_ENCAP）

	int lock;   //引用计数

	/* soft_reconfig_table in progress */
	bool soft_reconfig_init;  //软重配置是否已经初始化
	struct event *soft_reconfig_thread;  //软重配置线程

	/* list of peers on which soft_reconfig_table has to run */
	struct list *soft_reconfig_peers;  //列表，包含需要运行软重配置的对等体（peers）

	struct route_table *route_table;  //存放路由表项的集合，与zebra使用的是同一个数据结构
	uint64_t version;  //路由表的版本号，用于跟踪路由表的更改
};
```
{: file='bgpd/bgp_table.h'}

#### 4.6 初始化路由映射
初始化路由映射，初始化和安装匹配条件和设置，以便在 BGP 路由映射中使用

```c
/* Initialization of route map. */
void bgp_route_map_init(void)
{
	route_map_init();  //初始化路由映射系统

	route_map_add_hook(bgp_route_map_add);
	route_map_delete_hook(bgp_route_map_delete);
	route_map_event_hook(bgp_route_map_event);

	route_map_match_interface_hook(generic_match_add);
	route_map_no_match_interface_hook(generic_match_delete);
```
{: file='bgpd/bgp_routemap.c'}

为路由映射系统进行必要的初始化，包括创建散列表、设置调试标志、安装命令等

```c
/* Initialization of route map vector. */
void route_map_init(void)
{
	int i;

	route_map_master_hash =
		hash_create_size(8, route_map_hash_key_make, route_map_hash_cmp,
				 "Route Map Master Hash");
```
{: file='lib/routemap.c'}

route_map数据结构提供了管理和追踪路由映射对象的信息所需的各种属性

```c
/* Route map list structure. */
struct route_map {
	/* Name of route map. */
	char *name;

	/* Route map's rule. */
	struct route_map_index *head;
	struct route_map_index *tail;

	/* Make linked list. */
	struct route_map *next;
	struct route_map *prev;

	/* Maintain update info */
	bool to_be_processed; /* True if modification isn't acted on yet */
	bool deleted;         /* If 1, then this node will be deleted */
	bool optimization_disabled;

	/* How many times have we applied this route-map */
	uint64_t applied;
	uint64_t applied_clear;

	/* Counter to track active usage of this route-map */
	uint16_t use_count;

	/* Tables to maintain IPv4 and IPv6 prefixes from
	 * the prefix-list match clause.
	 */
	struct route_table *ipv4_prefix_table;
	struct route_table *ipv6_prefix_table;

	QOBJ_FIELDS;
};
```
{: file='lib/routemap.h'}

#### 4.7 mplsvpn
MPLS VPN的初始化安装了一系列与BGP VPN相关的命令，包括IPv4和IPv6 VPN网络的配置命令、显示命令等

```c
void bgp_mplsvpn_init(void)
{
	install_element(BGP_VPNV4_NODE, &vpnv4_network_cmd);
	install_element(BGP_VPNV4_NODE, &vpnv4_network_route_map_cmd);
	install_element(BGP_VPNV4_NODE, &no_vpnv4_network_cmd);

	install_element(BGP_VPNV6_NODE, &vpnv6_network_cmd);
	install_element(BGP_VPNV6_NODE, &no_vpnv6_network_cmd);

	install_element(VIEW_NODE, &show_bgp_ip_vpn_all_rd_cmd);
	install_element(VIEW_NODE, &show_bgp_ip_vpn_rd_cmd);
#ifdef KEEP_OLD_VPN_COMMANDS
	install_element(VIEW_NODE, &show_ip_bgp_vpn_rd_cmd);
	install_element(VIEW_NODE, &show_ip_bgp_vpn_all_cmd);
	install_element(VIEW_NODE, &show_ip_bgp_vpn_all_tags_cmd);
	install_element(VIEW_NODE, &show_ip_bgp_vpn_rd_tags_cmd);
	install_element(VIEW_NODE, &show_ip_bgp_vpn_all_neighbor_routes_cmd);
	install_element(VIEW_NODE, &show_ip_bgp_vpn_rd_neighbor_routes_cmd);
	install_element(VIEW_NODE,
			&show_ip_bgp_vpn_all_neighbor_advertised_routes_cmd);
	install_element(VIEW_NODE,
			&show_ip_bgp_vpn_rd_neighbor_advertised_routes_cmd);
#endif /* KEEP_OLD_VPN_COMMANDS */
}
```
{: file='bgpd/mplsvpn.c'}

#### 4.8 EVPN
`bgp_ethernetvpn_init`函数，CLI的初始化

#### 4.9 FLOWSPEC
`bgp_flowspec_vty_init`函数，CLI的初始化

#### 4.10 Access list
CLI的初始化

```c
	/* Access list initialize. */
	access_list_init();
	access_list_add_hook(peer_distribute_update);
	access_list_delete_hook(peer_distribute_update);
```
{: file='bgpd/bgpd.c -- bgp_init()'}

#### 4.11 Prefix list
CLI的初始化

```c
	/* Filter list initialize. */
	bgp_filter_init();
	as_list_add_hook(peer_aslist_add);
	as_list_delete_hook(peer_aslist_del);
```
{: file='bgpd/bgpd.c -- bgp_init()'}

#### 4.12 Community list
CLI的初始化

```c
	/* Community list initialize. */
	bgp_clist = community_list_init();
```
{: file='bgpd/bgpd.c -- bgp_init()''}

#### 4.13 BFD init
初始化配置和管理BFD功能，以实现双向转发检测

```c
void bgp_bfd_init(struct event_loop *tm)
{
	/* Initialize BFD client functions */
	bfd_protocol_integration_init(zclient, tm);

	/* "neighbor bfd" commands. */
	install_element(BGP_NODE, &neighbor_bfd_cmd);
	install_element(BGP_NODE, &neighbor_bfd_param_cmd);
	install_element(BGP_NODE, &neighbor_bfd_check_controlplane_failure_cmd);
	install_element(BGP_NODE, &no_neighbor_bfd_cmd);

#if HAVE_BFDD > 0
	install_element(BGP_NODE, &neighbor_bfd_profile_cmd);
	install_element(BGP_NODE, &no_neighbor_bfd_profile_cmd);
#endif /* HAVE_BFDD */
}
```
{: file='bgpd/bgp_bfd.c'}

### 5.配置项解析
使用 `frr_config_fork` 和 `bgp_gr_apply_running_config` 来处理配置项，fork进程以及应用运行时配置。

```c
void frr_config_fork(void)
{
	hook_call(frr_late_init, master);  //通过hook执行FRR的后期初始化

	if (!(di->flags & FRR_NO_SPLIT_CONFIG)) {
		/* Don't start execution if we are in dry-run mode */
		if (di->dryrun) {
			frr_config_read_in(NULL);
			exit(0);
		}
 
		event_add_event(master, frr_config_read_in, NULL, 0,
				&di->read_in);
	}

	if (di->daemon_mode || di->terminal)
		frr_daemonize();  //守护进程

	frr_is_after_fork = true;

	if (!di->pid_file)
		di->pid_file = pidfile_default;
	pid_output(di->pid_file);
	zlog_tls_buffer_init();  //初始化日志系统的 TLS 缓冲区
}
```
{: file='lib/libfrr.c'}

### 6.创建线程
调用`bgp_pthread_run`创建BGP相关的线程，并运行起来。

```c
void bgp_pthreads_run(void)
{
	frr_pthread_run(bgp_pth_io, NULL);  //启动 BGP I/O 线程
	frr_pthread_run(bgp_pth_ka, NULL);  //启动 BGP Keepalive 线程

	/* Wait until threads are ready. */
	frr_pthread_wait_running(bgp_pth_io);  //等待线程就绪
	frr_pthread_wait_running(bgp_pth_ka);
}
```
{: file='bgpd/bgpd.c'}

使用`frr_run`来运行主程序，处理各种事件。

**初始化完成！！！！！！！！！**

****

本文参考

> 1. [BGP官方文档](https://docs.frrouting.org/en/latest/bgp.html)
> 2. [SONiC BGP源码分析](https://vlambda.com/wz_7i9lzpN4MSE.html)
> 3. [FRR BGP 协议分析](https://blog.csdn.net/weixin_39094034/article/details/115220101)