---
title: FRR BGP源码分析2 -- 创建对等体
author: fnoobt
date: 2023-10-03 19:29:00 +0800
categories: [Network,路由协议]
tags: [network,bgp,frr]
---

本次继续分析BGP最简单的配置的代码实现，这样大家对BGP的框架会有进一步的熟悉：
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
	.
	.
	.
}
```
{: file='bgpd/bgp_vty.c'}

命令执行函数是router_bgp_cmd，主要完成以下几点：

1. 根据参数查找bgp 对象是否存在，第一个配置的route bgp XX将作为default，后续带vrf的将作为新的bgp对象
2. 如果bgp对象不存在，则需要重新创建一个bgp对象，struct bgp *bgp，所有的bgp对象存放在struct bgp_master *bm链表里面，此时BGP 最重要，贯穿全局的顶级结构体struct bgp闪亮登场，后续都会和这个结构体打交道，和bgp相关的几乎都在这里面，也是BGP最大的一个结构体。

### BGP结构体
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
	.
	.
	.
}
```
{: file='bgpd/bgpd.h'}

### 创建bgp对象
bgp_get函数创建bgp对象：
```c
/* Called from VTY commands. */
int bgp_get(struct bgp **bgp_val, as_t *as, const char *name,
	    enum bgp_instance_type inst_type, const char *as_pretty,
	    enum asnotation_mode asnotation)
{
	struct bgp *bgp;
	struct vrf *vrf = NULL;
	int ret = 0;

	ret = bgp_lookup_by_as_name_type(bgp_val, as, name, inst_type);
	if (ret || *bgp_val)
		return ret;

	bgp = bgp_create(as, name, inst_type, as_pretty, asnotation);

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
	bgp_handle_socket(bgp, vrf, VRF_UNKNOWN, true);
	listnode_add(bm->bgp, bgp);

	if (IS_BGP_INST_KNOWN_TO_ZEBRA(bgp)) {
		if (BGP_DEBUG(zebra, ZEBRA))
			zlog_debug("%s: Registering BGP instance %s to zebra",
				   __func__, bgp->name_pretty);
		bgp_zebra_instance_register(bgp);
	}

	return BGP_CREATED;
}
```
{: file='bgpd/bgpd.c'}

1. bgp_create -- 会创建新的struct bgp结构，并初始化里面很多的值和数据结构，其中最重要的peer对象
2. bgp_router_id_set -- 处理router_id
3. bgp_address_init -- 建立 bgp address的HASH
4. bgp_tip_hash_init
5. bgp_scan_init
6. 处理VRF相关的事情
7. bgp_handle_socket -- 创建对应的socket,bgp是使用TCP 建立连接的，建立socker服务端，监听179端口，等待peer端的connect接入，其中会处理和VRF相关的socker创建的细节
8. listnode_add(bm->bgp, bgp) -- 加入全局的bgp链表

## neighbor peer remote-as XXX
`neighbor peer remote-as` 命令就是配置一个对等体，peer是指对等体的地址（ipv4，ipv6地址），可以看到，bgp对等体之间是单播通信，与OSPF协议的组播是不一样的。

对于路由协议，不管是基于3层的还是2层的，都需要建立自己的寻路数据库，也就是通过邻居找到下一跳，你要走的远，你就得认识更多邻居，以及邻居的邻居，好比一句老话，在家靠父母，出门靠朋友，朋友多路好走，就这么一个道理。

那么话又说回来了，创建对等体呢，就是给自己找邻居，找朋友，不过呢，BGP这个人呢，更像一个干中介的，比如卖房的中介，他自己不建房子，只把建好的房源介绍给要买房的人，同时还维护这个房源信息库，及时更新已经卖掉的房子。
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
{: file='bgpd/bgp_vty.c'}

如果传给peer的值不是一个合法的地址，那么会被当做是一个`peer group/interface`名称来处理,如果是合法的，检查下地址是否是本地的，如果不是，`peer_remote_as` 则开始peer创建的奇妙之旅。

按照国际惯例，查找一下是不是已经为这个peer地址创建了peer，如果peer地址相同，as值不一样，就修改一下peer的as值，如果这个peer已经是某个group的成员，那么就不能成功创建对等体关系了。

如果上面的事情都没有发生，那么就可以创建一个新的对等体了，`peer_create`负责创我们的peer
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

- 创建一个新的peer结构，并给里面的status赋值为Idle，端口号 `BGP_PORT_DEFAULT = 179`，初始化大部分的数据结构。
- 初始化peer的`ibuf`和`obuf`来收发报文
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
	peer->fd = -1;
	peer->v_start = BGP_INIT_START_TIMER;
	peer->v_connect = bgp->default_connect_retry;
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
	peer->ibuf = stream_fifo_new();
	peer->obuf = stream_fifo_new();
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
	if (conf_if) {
		peer->conf_if = XSTRDUP(MTYPE_PEER_CONF_IF, conf_if);
		if (su)
			peer->su = *su;
		else
			bgp_peer_conf_if_to_su_update(peer);
		XFREE(MTYPE_BGP_PEER_HOST, peer->host);
		peer->host = XSTRDUP(MTYPE_BGP_PEER_HOST, conf_if);
	} else if (su) {
		peer->su = *su;
		sockunion2str(su, buf, SU_ADDRSTRLEN);
		XFREE(MTYPE_BGP_PEER_HOST, peer->host);
		peer->host = XSTRDUP(MTYPE_BGP_PEER_HOST, buf);
	}
	peer->local_as = local_as;
	peer->as = remote_as;
	/* internal and external values do not use as_pretty */
	if (as_str && asn_str2asn(as_str, NULL))
		peer->as_pretty = XSTRDUP(MTYPE_BGP, as_str);
	peer->as_type = as_type;
	peer->local_id = bgp->router_id;
	peer->v_holdtime = bgp->default_holdtime;
	peer->v_keepalive = bgp->default_keepalive;
	peer->v_routeadv = (peer_sort(peer) == BGP_PEER_IBGP)
				   ? BGP_DEFAULT_IBGP_ROUTEADV
				   : BGP_DEFAULT_EBGP_ROUTEADV;
	if (bgp_config_inprocess())
		peer->shut_during_cfg = true;

	peer = peer_lock(peer); /* bgp peer list reference */
	peer->group = group;
	listnode_add_sort(bgp->peer, peer);
```
{: file='bgpd/bgpd.c'}

- peer会加入 bgp -> peer 的链表，以及peerhash的HASH表里面
- 在把peer加入对应的地址族里面

```c
struct peer *peer_create(union sockunion *su, const char *conf_if,
			 struct bgp *bgp, as_t local_as, as_t remote_as,
			 int as_type, struct peer_group *group,
			 bool config_node, const char *as_str)
{
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
{: file='bgpd/bgpd.c'}

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


- 最后，将这个peer加入到定时器任务中：
```c
	/* Set up peer's events and timers. */
	else if (!active && peer_active(peer))
		bgp_timer_set(peer);
```
{: file='bgpd/bgpd.c'}


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