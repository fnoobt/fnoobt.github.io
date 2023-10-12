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
        ......
    }
```
{: file='bgpd/bgp_main.c'}

### 2.事件驱动的初始化
```c
	/* BGP master init. */
	bgp_master_init(frr_init(), buffer_size, addresses);
	bm->port = bgp_port;
```
{: file='bgpd/bgp_main.c'}

```c
void bgp_master_init(struct event_loop *master, const int buffer_size,
		     struct list *addresses)
{
	qobj_init();

	memset(&bgp_master, 0, sizeof(bgp_master));

	bm = &bgp_master;
	bm->bgp = list_new();
	bm->listen_sockets = list_new();
	bm->port = BGP_PORT_DEFAULT;
	bm->addresses = addresses;
	bm->master = master;
	bm->start_time = monotime(NULL);
	bm->t_rmap_update = NULL;
	bm->rmap_update_timer = RMAP_DEFAULT_UPDATE_TIMER;
	bm->v_update_delay = BGP_UPDATE_DELAY_DEF;
	bm->v_establish_wait = BGP_UPDATE_DELAY_DEF;
	bm->terminating = false;
	bm->socket_buffer = buffer_size;
	bm->wait_for_fib = false;
	bm->tcp_dscp = IPTOS_PREC_INTERNETCONTROL;
	bm->inq_limit = BM_DEFAULT_Q_LIMIT;
	bm->outq_limit = BM_DEFAULT_Q_LIMIT;

	bgp_mac_init();
	/* init the rd id space.
	   assign 0th index in the bitfield,
	   so that we start with id 1
	 */
	bf_init(bm->rd_idspace, UINT16_MAX);
	bf_assign_zero_index(bm->rd_idspace);

	/* mpls label dynamic allocation pool */
	bgp_lp_init(bm->master, &bm->labelpool);

	bgp_l3nhg_init();
	bgp_evpn_mh_init();
	QOBJ_REG(bm, bgp_master);
}
```
{: file='bgpd/bgpd.c'}

bgp_master 全局变量统管这一切，傲视天下。

### 3.VRF的初始化
```c
/* Initializations. */
	bgp_vrf_init();
```
{: file='bgpd/bgp_main.c'}

```c
static void bgp_vrf_init(void)
{
	vrf_init(bgp_vrf_new, bgp_vrf_enable, bgp_vrf_disable, bgp_vrf_delete);
}
```
{: file='bgpd/bgpd.c'}

VRF的初始化，FRR 里面VRF是在linux上创建的，整个流程的具体处理在vrf.c中

### 4.BGP初始化
```c
	/* BGP related initialization.  */
	bgp_init((unsigned short)instance);
```
{: file='bgpd/bgp_main.c'}

```c
void bgp_init(unsigned short instance)
{
	hook_register(bgp_config_end, peer_unshut_after_cfg);

	/* allocates some vital data structures used by peer commands in
	 * vty_init */

	/* pre-init pthreads */
	bgp_pthreads_init();

	/* Init zebra. */
	bgp_zebra_init(bm->master, instance);

#ifdef ENABLE_BGP_VNC
	vnc_zebra_init(bm->master);
#endif

	/* BGP VTY commands installation.  */
	bgp_vty_init();

	/* BGP inits. */
	bgp_attr_init();
	bgp_debug_init();
	bgp_community_alias_init();
	bgp_dump_init();
	bgp_route_init();
	bgp_route_map_init();
	bgp_scan_vty_init();
	bgp_mplsvpn_init();
#ifdef ENABLE_BGP_VNC
	rfapi_init();
#endif
	bgp_ethernetvpn_init();
	bgp_flowspec_vty_init();

	/* Access list initialize. */
	access_list_init();
	access_list_add_hook(peer_distribute_update);
	access_list_delete_hook(peer_distribute_update);

	/* Filter list initialize. */
	bgp_filter_init();
	as_list_add_hook(peer_aslist_add);
	as_list_delete_hook(peer_aslist_del);

	/* Prefix list initialize.*/
	prefix_list_init();
	prefix_list_add_hook(peer_prefix_list_update);
	prefix_list_delete_hook(peer_prefix_list_update);

	/* Community list initialize. */
	bgp_clist = community_list_init();

	/* BFD init */
	bgp_bfd_init(bm->master);

	bgp_lp_vty_init();

	bgp_label_per_nexthop_init();

	cmd_variable_handler_register(bgp_viewvrf_var_handlers);
}
```
{: file='bgpd/bgpd.c'}

这部分对最重要的内容进行初始化，包含：
#### 4.1 线程的初始化
```c
static void bgp_pthreads_init(void)
{
	assert(!bgp_pth_io);
	assert(!bgp_pth_ka);

	struct frr_pthread_attr io = {
		.start = frr_pthread_attr_default.start,
		.stop = frr_pthread_attr_default.stop,
	};
	struct frr_pthread_attr ka = {
		.start = bgp_keepalives_start,
		.stop = bgp_keepalives_stop,
	};
	bgp_pth_io = frr_pthread_new(&io, "BGP I/O thread", "bgpd_io");
	bgp_pth_ka = frr_pthread_new(&ka, "BGP Keepalives thread", "bgpd_ka");
}
```
{: file='bgpd/bgpd.c'}

FRR模拟了线程，BGP 的线程包含bgpd / bgpd_io / bgpd_ka，他们的主要任务：
- bgpd     --- 处理BGP的所有业务逻辑，处理函数frr_run
- bgpd_io  --- 收发包，然后把报文入队，唤醒bgpd继续处理,入口函数是fpt_run
- bgpd_ka  ---  处理BGP的keeplive，bgp_keepalives_start

#### 4.2 Zebra的初始化
```c
void bgp_zebra_init(struct event_loop *master, unsigned short instance)
{
	zclient_num_connects = 0;

	if_zapi_callbacks(bgp_ifp_create, bgp_ifp_up,
			  bgp_ifp_down, bgp_ifp_destroy);

	/* Set default values. */
	zclient = zclient_new(master, &zclient_options_default, bgp_handlers,
			      array_size(bgp_handlers));
	zclient_init(zclient, ZEBRA_ROUTE_BGP, 0, &bgpd_privs);
	zclient->zebra_connected = bgp_zebra_connected;
	zclient->instance = instance;
}
```
{: file='bgpd/bgp_zebra.c'}

zclient_new 创建一个zclient客户端，struct thread_master填入bgp的master，这样客户端的回调函数就在bgp的进程上下文执行
```c
/* Initialize zebra client.  Argument redist_default is unwanted
   redistribute route type. */
void zclient_init(struct zclient *zclient, int redist_default,
		  unsigned short instance, struct zebra_privs_t *privs)
{
	int afi, i;

	/* Set -1 to the default socket value. */
	zclient->sock = -1;
	zclient->privs = privs;

	/* Clear redistribution flags. */
	for (afi = AFI_IP; afi < AFI_MAX; afi++)
		for (i = 0; i < ZEBRA_ROUTE_MAX; i++)
			zclient->redist[afi][i] = vrf_bitmap_init();

	/* Set unwanted redistribute route.  bgpd does not need BGP route
	   redistribution. */
	zclient->redist_default = redist_default;
	zclient->instance = instance;
	/* Pending: make afi(s) an arg. */
	for (afi = AFI_IP; afi < AFI_MAX; afi++) {
		redist_add_instance(&zclient->mi_redist[afi][redist_default],
				    instance);

		/* Set default-information redistribute to zero. */
		zclient->default_information[afi] = vrf_bitmap_init();
	}

	if (zclient_debug)
		zlog_debug("scheduling zclient connection");

	zclient_event(ZCLIENT_SCHEDULE, zclient);
}
```
{: file='lib/zclient.c'}

zclient_init 初始化客户端相关的数据后，会调用 zclient_event ,添加一个事件，回调函数是 zclient_connect ，初始化完成后，后续会和zebra进程建立连接。

```c
static void zclient_event(enum zclient_event event, struct zclient *zclient)
{
	switch (event) {
	case ZCLIENT_SCHEDULE:
		event_add_event(zclient->master, zclient_connect, zclient, 0,
				&zclient->t_connect);
		break;
	case ZCLIENT_CONNECT:
		if (zclient_debug)
			zlog_debug(
				"zclient connect failures: %d schedule interval is now %d",
				zclient->fail, zclient->fail < 3 ? 10 : 60);
		event_add_timer(zclient->master, zclient_connect, zclient,
				zclient->fail < 3 ? 10 : 60,
				&zclient->t_connect);
		break;
	case ZCLIENT_READ:
		zclient->t_read = NULL;
		event_add_read(zclient->master, zclient_read, zclient,
			       zclient->sock, &zclient->t_read);
		break;
	}
}
```
{: file='lib/zclient.c'}

然后会填充各种bgp关心的事件的回调函数，在bgp_zebra_connected回调函数里面（客户端连接zebra成功后会调用），会注册各种BGP 感兴趣的事件，如router-id, interfaces, redistributed routes。

#### 4.3 命令行的初始化
```c
void bgp_vty_init(void)
{
.......
}
```
{: file='bgpd/bgp_vty.c'}


```c
void bgp_debug_init(void)
{
.......
}
```
{: file='bgpd/bgp_debug.c'}

```c
void bgp_dump_init(void)
{
	memset(&bgp_dump_all, 0, sizeof(bgp_dump_all));
	memset(&bgp_dump_updates, 0, sizeof(bgp_dump_updates));
	memset(&bgp_dump_routes, 0, sizeof(bgp_dump_routes));

	bgp_dump_obuf =
		stream_new(BGP_MAX_PACKET_SIZE + BGP_MAX_PACKET_SIZE_OVERFLOW);

	install_node(&bgp_dump_node);

	install_element(CONFIG_NODE, &dump_bgp_all_cmd);
	install_element(CONFIG_NODE, &no_dump_bgp_all_cmd);

	hook_register(bgp_packet_dump, bgp_dump_packet);
	hook_register(peer_status_changed, bgp_dump_state);
}
```
{: file='bgpd/bgp_dump.c'}

```c
void bgp_scan_vty_init(void)
{
	install_element(VIEW_NODE, &show_ip_bgp_nexthop_cmd);
	install_element(VIEW_NODE, &show_ip_bgp_import_check_cmd);
	install_element(VIEW_NODE, &show_ip_bgp_instance_all_nexthop_cmd);
}
```
{: file='bgpd/bgp_nexthop.c'}

准备注册CLI

#### 4.4 属性相关的初始化
```c
/* Initialization of attribute. */
void bgp_attr_init(void)
{
	aspath_init();
	attrhash_init();
	community_init();
	ecommunity_init();
	lcommunity_init();
	cluster_init();
	transit_init();
	encap_init();
	srv6_init();
}
```
{: file='bgpd/bgp_attr.c'}

##### 4.4.1 aspath_init
```c
/* AS path hash initialize. */
void aspath_init(void)
{
	ashash = hash_create_size(32768, aspath_key_make, aspath_cmp,
				  "BGP AS Path");
}
```
{: file='bgpd/bgp_aspath.c'}

aspath_init 初始化aspath的hash存储，hash头ashash，所有的aspath用全局的hash存放，相关的数据结构如下：

```c
/* AS_PATH segment data in abstracted form, no limit is placed on length */
struct assegment {
	struct assegment *next;
	as_t *as;
	unsigned short length;
	uint8_t type;
};

/* AS path may be include some AsSegments.  */
struct aspath {
	/* Reference count to this aspath.  */
	unsigned long refcnt;

	/* segment data */
	struct assegment *segments;

	/* AS path as a json object */
	json_object *json;

	/* String expression of AS path.  This string is used by vty output
	   and AS path regular expression match.  */
	char *str;
	unsigned short str_len;

	/* AS notation used by string expression of AS path */
	enum asnotation_mode asnotation;
};
```
{: file='bgpd/bgp_aspath.h'}

##### 4.4.2 attrhash_init
attrhash_init 初始化属性的hash存储，hash头attrhash，所有的属性用全局的hash存放，数据结构：
```c
static void attrhash_init(void)
{
	attrhash =
		hash_create(attrhash_key_make, attrhash_cmp, "BGP Attributes");
}
```
{: file='bgpd/bgp_attr.c'}

attr数据结构较大，包含了BGP的所有属性，AS_PATH，community,med, origin, local_pref, med等等，数据结构：
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
    .
    .
    .
}
```
{: file='bgpd/bgp_attr.h'}

##### 4.4.3 community_init
community_init 初始化community的HASH，全局comhash存储，数据结构：
```c
/* Communities attribute.  */
struct community {
	/* Reference count of communities value.  */
	unsigned long refcnt;

	/* Communities value size.  */
	int size;

	/* Communities value.  */
	uint32_t *val;

	/* Communities as a json object */
	json_object *json;

	/* String of community attribute.  This sring is used by vty output
	   and expanded community-list for regular expression match.  */
	char *str;
};
```
{: file='bgpd/bgp_community.h'}

##### 4.4.4 ecommunity_init
ecommunity_init初始化ecommunity的HASH ，全局ecomhash存储，数据结构：
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
lcommunity_init初始化lcommunity的HASH ，全局lcomhash存储，数据结构：
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
cluster_init 初始化 cluster 路由反射器的HASH，全局变量cluster_hash，数据结构：
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
transit_init初始化传输的属性的HASH，全局变量transit_hash，数据结构：
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
encap_init 初始化BGP Encap Hash，数据结构：
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
transit_init初始化传输的属性的HASH，全局变量transit_hash，数据结构：
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
srv6_init初始化srv6的HASH，数据结构：
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
    .
    .
    .
}
```
{: file='bgpd/bgp_route.c'}

bgp_table初始化函数
```c
/*
 * bgp_table_init
 */
struct bgp_table *bgp_table_init(struct bgp *bgp, afi_t afi, safi_t safi)
{
	struct bgp_table *rt;

	rt = XCALLOC(MTYPE_BGP_TABLE, sizeof(struct bgp_table));

	rt->route_table = route_table_init_with_delegate(&bgp_table_delegate);

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

bgp_table数据结构：
```c
struct bgp_table {
	/* table belongs to this instance */
	struct bgp *bgp;

	/* afi/safi of this table */
	afi_t afi;    //网络类型（IPV4, IPV6）
	safi_t safi;  //子网络类型（SAFI_UNICAST,SAFI_MULTICAST,SAFI_RESERVED_3,SAFI_MPLS_VPN,SAFI_ENCAP）

	int lock;   //引用计数

	/* soft_reconfig_table in progress */
	bool soft_reconfig_init;
	struct event *soft_reconfig_thread;

	/* list of peers on which soft_reconfig_table has to run */
	struct list *soft_reconfig_peers;

	struct route_table *route_table;  //路由表项的集合，与zebra使用的是同一个数据结构，存放路由表项
	uint64_t version;
};
```
{: file='bgpd/bgp_table.h'}

#### 4.6 初始化路由图
初始化路由图，并增加hook的回调函数
```c
/* Initialization of route map. */
void bgp_route_map_init(void)
{
	route_map_init();

	route_map_add_hook(bgp_route_map_add);
	route_map_delete_hook(bgp_route_map_delete);
	route_map_event_hook(bgp_route_map_event);

	route_map_match_interface_hook(generic_match_add);
	route_map_no_match_interface_hook(generic_match_delete);
    .
    .
    .
}
```
{: file='bgpd/bgp_routemap.c'}

route_map 初始化
```c
/* Initialization of route map vector. */
void route_map_init(void)
{
	int i;

	route_map_master_hash =
		hash_create_size(8, route_map_hash_key_make, route_map_hash_cmp,
				 "Route Map Master Hash");
    .
    .
    .
}
```
{: file='lib/routemap.c'}

route_map数据结构
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
MPLS VPN的初始化全是CLI的初始化
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
bgp_ethernetvpn_init函数，CLI的初始化

#### 4.9 FLOWSPEC
bgp_flowspec_vty_init函数，CLI的初始化

#### 4.10 Access list
CLI的初始化
```c
	/* Access list initialize. */
	access_list_init();
	access_list_add_hook(peer_distribute_update);
	access_list_delete_hook(peer_distribute_update);
```
{: file='bgpd/bgpd.c'}

#### 4.11 Prefix list
CLI的初始化
```c
	/* Filter list initialize. */
	bgp_filter_init();
	as_list_add_hook(peer_aslist_add);
	as_list_delete_hook(peer_aslist_del);
```
{: file='bgpd/bgpd.c'}

#### 4.12 Community list
CLI的初始化
```c
	/* Community list initialize. */
	bgp_clist = community_list_init();
```
{: file='bgpd/bgpd.c'}

#### 4.13 BFD init
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

### 5.配置frr
```c
void frr_config_fork(void)
{
	hook_call(frr_late_init, master);

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
		frr_daemonize();

	frr_is_after_fork = true;

	if (!di->pid_file)
		di->pid_file = pidfile_default;
	pid_output(di->pid_file);
	zlog_tls_buffer_init();
}
```
{: file='lib/libfrr.c'}

### 6.创建线程
```c
void bgp_pthreads_run(void)
{
	frr_pthread_run(bgp_pth_io, NULL);
	frr_pthread_run(bgp_pth_ka, NULL);

	/* Wait until threads are ready. */
	frr_pthread_wait_running(bgp_pth_io);
	frr_pthread_wait_running(bgp_pth_ka);
}
```
{: file='bgpd/bgpd.c'}

Bgp_pthread_run创建对应的线程，并运行起来。

然后bgpd在frr_run中开始死循环，处理各种事件。

**初始化完成！！！！！！！！！**

****

本文参考

> 1. [BGP官方文档](https://docs.frrouting.org/en/latest/bgp.html)
> 2. [SONiC BGP源码分析](https://vlambda.com/wz_7i9lzpN4MSE.html)
> 3. [FRR BGP 协议分析](https://blog.csdn.net/weixin_39094034/article/details/115220101)