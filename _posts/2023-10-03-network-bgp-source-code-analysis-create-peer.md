---
title: FRR BGP源码分析2 -- 创建对等体
author: fnoobt
date: 2023-10-03 19:29:00 +0800
categories: [Network,路由协议]
tags: [network,bgp,frr]
pin: true
---

本次分析 BGP 最简单的配置的代码实现，这样大家对BGP的框架会有进一步的熟悉：

```bash
router bgp 200
bgp router-id 2.2.2.2
 neighbor 3.3.3.3 remote-as 200
 neighbor 3.3.3.3 update-source 2.2.2.2
```

## router bgp XX
### router bgp执行函数

```c
/* "router bgp" commands. */
DEFUN_NOSH (router_bgp,
       router_bgp_cmd,
       "router bgp [ASNUM$instasn [<view|vrf> VIEWVRFNAME] [as-notation <dot|dot+|plain>]]",
       ROUTER_STR
       BGP_STR
       AS_STR
       BGP_INSTANCE_HELP_STR
       "Force the AS notation output\n"
       "use 'AA.BB' format for AS 4 byte values\n"
       "use 'AA.BB' format for all AS values\n"
       "use plain format for all AS values\n")
{
	int idx_asn = 2;
	int idx_view_vrf = 3;
	int idx_vrf = 4;
	int is_new_bgp = 0;
	int idx_asnotation = 3;
	int idx_asnotation_kind = 4;
	enum asnotation_mode asnotation = ASNOTATION_UNDEFINED;
	int ret;
	as_t as;
	struct bgp *bgp;
	const char *name = NULL;
	enum bgp_instance_type inst_type;
```
{: file='bgpd/bgp_vty.c'}

命令执行函数是 router_bgp_cmd ，主要完成以下几点：

1. 根据参数查找bgp 对象是否存在，第一个配置的`route bgp XX`将作为default，后续带vrf的将作为新的bgp对象
2. 如果bgp对象不存在，则需要重新创建一个bgp对象，`struct bgp *bgp`，所有的bgp对象存放在`struct bgp_master *bm`链表里面，此时BGP 最重要，贯穿全局的顶级结构体`struct bgp`闪亮登场，后续都会和这个结构体打交道，和bgp相关的几乎都在这里面，也是BGP最大的一个结构体。

### BGP结构体
这个数据结构包含了 BGP 实例的各种配置和状态信息，用于管理 BGP 路由器的行为

```c
/* BGP instance structure.  */
struct bgp {
	/* AS number of this BGP instance.  */
	as_t as;
	char *as_pretty;

	/* Name of this BGP instance.  */
	char *name;
	char *name_pretty;	/* printable "VRF|VIEW name|default" */

	/* Type of instance and VRF id. */
	enum bgp_instance_type inst_type;
	vrf_id_t vrf_id;

	/* Reference count to allow peer_delete to finish after bgp_delete */
	int lock;

	/* Self peer.  */
	struct peer *peer_self;

	/* BGP peer. */
	struct list *peer;
	struct hash *peerhash;

	/* BGP peer group.  */
	struct list *group;

	/* The maximum number of BGP dynamic neighbors that can be created */
	int dynamic_neighbors_limit;

	/* The current number of BGP dynamic neighbors */
	int dynamic_neighbors_count;
```
{: file='bgpd/bgpd.h'}

### 创建bgp对象
通过bgp_get函数创建或查找 BGP 实例，并确保与 Zebra 的正确集成：

```c
/* Called from VTY commands. */
int bgp_get(struct bgp **bgp_val, as_t *as, const char *name,
	    enum bgp_instance_type inst_type, const char *as_pretty,
	    enum asnotation_mode asnotation)
{
	struct bgp *bgp;
	struct vrf *vrf = NULL;
	int ret = 0;

	ret = bgp_lookup_by_as_name_type(bgp_val, as, name, inst_type);  //尝试通过自治系统号、名称和实例类型查找已有的 BGP 实例
	if (ret || *bgp_val)
		return ret;

	bgp = bgp_create(as, name, inst_type, as_pretty, asnotation);  //找不到时创建一个新的 BGP 实例

	/*
	 * view instances will never work inside of a vrf
	 * as such they must always be in the VRF_DEFAULT
	 * Also we must set this to something useful because
	 * of the vrf socket code needing an actual useful
	 * default value to send to the underlying OS.
	 *
	 * This code is currently ignoring vrf based
	 * code using the -Z option( and that is probably
	 * best addressed elsewhere in the code )
	 */
	if (inst_type == BGP_INSTANCE_TYPE_VIEW)
		bgp->vrf_id = VRF_DEFAULT;

	bgp_router_id_set(bgp, &bgp->router_id_zebra, true);
	bgp_address_init(bgp);
	bgp_tip_hash_init(bgp);
	bgp_scan_init(bgp);
	*bgp_val = bgp;

	bgp->t_rmap_def_originate_eval = NULL;

	/* If Default instance or VRF, link to the VRF structure, if present. */
	if (bgp->inst_type == BGP_INSTANCE_TYPE_DEFAULT
	    || bgp->inst_type == BGP_INSTANCE_TYPE_VRF) {
		vrf = bgp_vrf_lookup_by_instance_type(bgp);
		if (vrf)
			bgp_vrf_link(bgp, vrf);
	}
	/* BGP server socket already processed if BGP instance
	 * already part of the list
	 */
	/* 创建对应的socket，bgp是使用TCP 建立连接的，建立socker服务端，监听179端口，等待peer端的connect接入
	 * 其中会处理和VRF相关的socker创建的细节
	 */
	bgp_handle_socket(bgp, vrf, VRF_UNKNOWN, true);  
	listnode_add(bm->bgp, bgp);

	if (IS_BGP_INST_KNOWN_TO_ZEBRA(bgp)) {
		if (BGP_DEBUG(zebra, ZEBRA))
			zlog_debug("%s: Registering BGP instance %s to zebra",
				   __func__, bgp->name_pretty);
		bgp_zebra_instance_register(bgp);  //向 Zebra 注册 BGP 实例
	}

	return BGP_CREATED;
}
```
{: file='bgpd/bgpd.c'}

1. 尝试通过自治系统号、名称和实例类型查找已有的 BGP 实例。如果找到，则将其返回给调用者。
2. 如果没有找到现有的 BGP 实例，则通过bgp_create 创建一个新的 BGP 实例，并初始化里面很多的值和数据结构，其中最重要的peer对象
3. 检查实例类型是否为 BGP_INSTANCE_TYPE_VIEW（视图实例），如果是将vrf_id 设置为默认值 VRF_DEFAULT
4. 初始化 BGP 实例的路由器标识符和地址相关的数据结构
5. 将 BGP 实例添加到全局 BGP 实例列表中 `bm->bgp`
6. 如果 BGP 实例已经在 Zebra 中注册，向 Zebra 注册 BGP 实例
7. BGP 实例成功创建，返回 `BGP_CREATED`

## neighbor peer remote-as XXX
`neighbor peer remote-as` 命令就是配置一个对等体，peer是指对等体的地址（ipv4，ipv6地址），可以看到，bgp对等体之间是单播通信，与OSPF协议的组播是不一样的。

对于路由协议，不管是基于3层的还是2层的，都需要建立自己的寻路数据库，也就是通过邻居找到下一跳。创建对等体呢，就是给自己找邻居，不过呢，BGP这个人呢，更像一个干中介的，比如卖房的中介，他自己不建房子，只把建好的房源介绍给要买房的人，同时还维护这个房源信息库，及时更新已经卖掉的房子。

### neighbor peer remote-as执行函数
用于配置BGP对等体的命令，为指定的对等体设置远程AS号码

```c
DEFUN (neighbor_remote_as,
       neighbor_remote_as_cmd,
       "neighbor <A.B.C.D|X:X::X:X|WORD> remote-as <ASNUM|internal|external>",
       NEIGHBOR_STR
       NEIGHBOR_ADDR_STR2
       "Specify a BGP neighbor\n"
       AS_STR
       "Internal BGP peer\n"
       "External BGP peer\n")
{
	int idx_peer = 1;
	int idx_remote_as = 3;
	return peer_remote_as_vty(vty, argv[idx_peer]->arg,
				  argv[idx_remote_as]->arg);
}
```
{: file='bgpd/bgp_vty.c'}

`peer_remote_as_vty`处理CLI的命令:

```c
	/* If peer is peer group or interface peer, call proper function. */
	ret = str2sockunion(peer_str, &su);
	if (ret < 0) {
		struct peer *peer;

		/* Check if existing interface peer */
		peer = peer_lookup_by_conf_if(bgp, peer_str);

		ret = peer_remote_as(bgp, NULL, peer_str, &as, as_type, as_str);

		/* if not interface peer, check peer-group settings */
		if (ret < 0 && !peer) {
			ret = peer_group_remote_as(bgp, peer_str, &as, as_type,
						   as_str);
			if (ret < 0) {
				vty_out(vty,
					"%% Create the peer-group or interface first\n");
				return CMD_WARNING_CONFIG_FAILED;
			}
			return CMD_SUCCESS;
		}
	} else {
		if (peer_address_self_check(bgp, &su)) {
			vty_out(vty,
				"%% Can not configure the local system as neighbor\n");
			return CMD_WARNING_CONFIG_FAILED;
		}
		ret = peer_remote_as(bgp, &su, NULL, &as, as_type, as_str);
	}
```
{: file='bgpd/bgp_vty.c -- peer_remote_as_vty()'}

如果传给peer的值不是一个合法的地址，那么会被当做是一个`peer group/interface`名称来处理,如果是合法的，检查下地址是否是本地的，如果不是，`peer_remote_as` 则开始peer创建的奇妙之旅。

按照国际惯例，查找一下是不是已经为这个peer地址创建了peer，如果peer地址相同，as值不一样，就修改一下peer的as值，如果这个peer已经是某个group的成员，那么就不能成功创建对等体关系了。

### peer数据结构

```c
/* BGP neighbor structure. */
struct peer {
	/* BGP structure.  */
	struct bgp *bgp;

	/* reference count, primarily to allow bgp_process'ing of route_node's
	 * to be done after a struct peer is deleted.
	 *
	 * named 'lock' for hysterical reasons within Quagga.
	 */
	int lock;

	/* BGP peer group.  */
	struct peer_group *group;

	/* BGP peer_af structures, per configured AF on this peer */
	struct peer_af *peer_af_array[BGP_AF_MAX];

	/* Peer's remote AS number. */
	int as_type;
	as_t as;
	/* for vty as format */
	char *as_pretty;

	/* Peer's local AS number. */
	as_t local_as;
```
{: file='bgpd/bgpd.h'}

### 创建peer对象
如果上面的事情都没有发生，那么就可以创建一个新的对等体了，`peer_create`负责创我们的peer

```c
		if (conf_if)
			return BGP_ERR_NO_INTERFACE_CONFIG;

		/* If the peer is not part of our confederation, and its not an
		   iBGP peer then spoof the source AS */
		if (bgp_config_check(bgp, BGP_CONFIG_CONFEDERATION) &&
		    !bgp_confederation_peers_check(bgp, *as) && *as &&
		    bgp->as != *as)
			local_as = bgp->confed_id;
		else
			local_as = bgp->as;

		peer_create(su, conf_if, bgp, local_as, *as, as_type, NULL,
			    true, as_str);
	}
```
{: file='bgpd/bgpd.h -- peer_remote_as()'}

1. `peer_create`在BGP实例中创建一个新的对等体，并进行一系列的初始化和配置

```c
/*
 * Create new BGP peer.
 *
 * conf_if and su are mutually exclusive if configuring from the cli.
 * If we are handing a doppelganger, then we *must* pass in both
 * the original peer's su and conf_if, so that we can appropriately
 * track the bgp->peerhash( ie we don't want to remove the current
 * one from the config ).
 */
struct peer *peer_create(union sockunion *su, const char *conf_if,
			 struct bgp *bgp, as_t local_as, as_t remote_as,
			 int as_type, struct peer_group *group,
			 bool config_node, const char *as_str)
{
	int active;
	struct peer *peer;
	char buf[SU_ADDRSTRLEN];
	afi_t afi;
	safi_t safi;

	peer = peer_new(bgp);
```
{: file='bgpd/bgpd.h'}

`peer_new`主要实现了下面两个功能
- 创建一个新的peer结构，并给里面的 status 赋值为 Idle，初始化大部分数据结构，获取BGP服务的端口号（默认端口号179）。
- 初始化peer的`ibuf`和`obuf`缓存区来收发报文

```c
/* Allocate new peer object, implicitely locked.  */
struct peer *peer_new(struct bgp *bgp)
{
	afi_t afi;
	safi_t safi;
	struct peer *peer;
	struct servent *sp;

	/* bgp argument is absolutely required */
	assert(bgp);

	/* Allocate new peer. */
	peer = XCALLOC(MTYPE_BGP_PEER, sizeof(struct peer));

	/* Set default value. */
	peer->fd = -1;  //文件描述符初始化
	peer->v_start = BGP_INIT_START_TIMER;  //初始化启动计时器
	peer->v_connect = bgp->default_connect_retry;  //初始化连接重试计时器
	peer->status = Idle;
	peer->ostatus = Idle;
	peer->cur_event = peer->last_event = peer->last_major_event = 0;
	peer->bgp = bgp_lock(bgp);
	peer = peer_lock(peer); /* initial reference */
	peer->local_role = ROLE_UNDEFINED;
	peer->remote_role = ROLE_UNDEFINED;
	peer->password = NULL;
	peer->max_packet_size = BGP_STANDARD_MESSAGE_MAX_PACKET_SIZE;

	/* Set default flags. */
	FOREACH_AFI_SAFI (afi, safi) {
		SET_FLAG(peer->af_flags[afi][safi], PEER_FLAG_SEND_COMMUNITY);
		SET_FLAG(peer->af_flags[afi][safi],
			 PEER_FLAG_SEND_EXT_COMMUNITY);
		SET_FLAG(peer->af_flags[afi][safi],
			 PEER_FLAG_SEND_LARGE_COMMUNITY);

		SET_FLAG(peer->af_flags_invert[afi][safi],
			 PEER_FLAG_SEND_COMMUNITY);
		SET_FLAG(peer->af_flags_invert[afi][safi],
			 PEER_FLAG_SEND_EXT_COMMUNITY);
		SET_FLAG(peer->af_flags_invert[afi][safi],
			 PEER_FLAG_SEND_LARGE_COMMUNITY);
		peer->addpath_type[afi][safi] = BGP_ADDPATH_NONE;
		peer->soo[afi][safi] = NULL;
	}

	/* set nexthop-unchanged for l2vpn evpn by default */
	SET_FLAG(peer->af_flags[AFI_L2VPN][SAFI_EVPN],
		 PEER_FLAG_NEXTHOP_UNCHANGED);

	SET_FLAG(peer->sflags, PEER_STATUS_CAPABILITY_OPEN);

	/* Initialize per peer bgp GR FSM */
	bgp_peer_gr_init(peer);

	/* Create buffers.  */
	peer->ibuf = stream_fifo_new();  //初始化对等体的输入缓冲区
	peer->obuf = stream_fifo_new();  //初始化对等体的输出缓冲区
	pthread_mutex_init(&peer->io_mtx, NULL);

	peer->ibuf_work =
		ringbuf_new(BGP_MAX_PACKET_SIZE + BGP_MAX_PACKET_SIZE/2);

	/* Get service port number.  */
	sp = getservbyname("bgp", "tcp"); 
	peer->port = (sp == NULL) ? BGP_PORT_DEFAULT : ntohs(sp->s_port);

	QOBJ_REG(peer, peer);
	return peer;
}
```
{: file='bgpd/bgpd.c'}

2. `peer_create`把 peer 加入 `bgp->peer` 的链表，以及 peerhash 的HASH表里面

```c
	peer = peer_lock(peer); /* bgp peer list reference */
	peer->group = group;
	listnode_add_sort(bgp->peer, peer);
```
{: file='bgpd/bgpd.c -- peer_create()'}

3. `peer_create`把peer加入对应的地址族里面

```c
/* If 'bgp default <afi>-<safi>' is configured, then activate the
	 * neighbor for the corresponding address family. IPv4 Unicast is
	 * the only address family enabled by default without expliict
	 * configuration.
	 */
	FOREA	CH_AFI_SAFI (afi, safi) {
		if (bgp->default_af[afi][safi]) {
			peer->afc[afi][safi] = 1;
			peer_af_create(peer, afi, safi);
		}
	}
```
{: file='bgpd/bgpd.c -- peer_create()'}

`peer_af_create`为BGP对等体创建并初始化特定地址族的数据结构

```c
struct peer_af *peer_af_create(struct peer *peer, afi_t afi, safi_t safi)
{
	struct peer_af *af;
	int afid;
	struct bgp *bgp;

	if (!peer)
		return NULL;

	afid = afindex(afi, safi);  //函数获取给定AFI和SAFI的对应索引
	if (afid >= BGP_AF_MAX)
		return NULL;

	bgp = peer->bgp;
	assert(peer->peer_af_array[afid] == NULL);

	/* Allocate new peer af */
	af = XCALLOC(MTYPE_BGP_PEER_AF, sizeof(struct peer_af));

	peer->peer_af_array[afid] = af;
	af->afi = afi;
	af->safi = safi;
	af->afid = afid;
	af->peer = peer;
	bgp->af_peer_count[afi][safi]++;

	return af;
}
```
{: file='bgpd/bgpd.c'}

`peer_af` 的数据结构：

```c
struct peer_af {
	/* back pointer to the peer */
	struct peer *peer;

	/* which subgroup the peer_af belongs to */
	struct update_subgroup *subgroup;

	/* for being part of an update subgroup's peer list */
	LIST_ENTRY(peer_af) subgrp_train;

	/* for being part of a packet's peer list */
	LIST_ENTRY(peer_af) pkt_train;

	struct bpacket *next_pkt_to_send;

	/*
	 * Trigger timer for bgp_announce_route().
	 */
	struct event *t_announce_route;

	afi_t afi;
	safi_t safi;
	int afid;
};
```
{: file='bgpd/bgpd.h'}

4. 最后，将这个peer加入到定时器任务中：

```c
	/* Set up peer's events and timers. */
	else if (!active && peer_active(peer))
		bgp_timer_set(peer);
```
{: file='bgpd/bgpd.c -- peer_create()'}

`bgp_timer_set`设置BGP对等体状态机中各种定时器，根据对等体的当前状态执行相应的操作，包括启动、连接、保持时间等定时器的开启和关闭

```c
/* Hook function called after bgp event is occered.  And vty's
   neighbor command invoke this function after making neighbor
   structure. */
void bgp_timer_set(struct peer *peer)
{
	afi_t afi;
	safi_t safi;

	switch (peer->status) {
	case Idle:
		/* First entry point of peer's finite state machine.  In Idle
		   status start timer is on unless peer is shutdown or peer is
		   inactive.  All other timer must be turned off */
		if (BGP_PEER_START_SUPPRESSED(peer) || !peer_active(peer)
		    || peer->bgp->vrf_id == VRF_UNKNOWN) {
			EVENT_OFF(peer->t_start);
		} else {
			BGP_TIMER_ON(peer->t_start, bgp_start_timer,
				     peer->v_start);
		}
		EVENT_OFF(peer->t_connect);
		EVENT_OFF(peer->t_holdtime);
		bgp_keepalives_off(peer);
		EVENT_OFF(peer->t_routeadv);
		EVENT_OFF(peer->t_delayopen);
		break;
```
{: file='bgpd/bgp_fsm.c'}

开启start timer定时器，定时器到期后，开始peer的状态机协商，最简单的配置已经分析完成，后面开始BGP 状态机的协商和分析。

****

本文参考

> 1. [BGP官方文档](https://docs.frrouting.org/en/latest/bgp.html)
> 2. [FRR BGP 协议分析](https://blog.csdn.net/armlinuxww/article/details/103127726?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522169685963516800188578633%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=169685963516800188578633&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-4-103127726-null-null.nonecase&utm_term=BGP%E5%8D%8F%E8%AE%AE%E5%88%86%E6%9E%90&spm=1018.2226.3001.4450)