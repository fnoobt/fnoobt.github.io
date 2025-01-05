---
title: FRR BGP源码分析6 -- ZEBRA初始化
author: fnoobt
date: 2023-10-08 19:29:00 +0800
categories: [Network,路由协议]
tags: [network,bgp,frr]
---

zebra，翻译是斑马，它负责管理其他所有协议进程的路由信息的更新与交互，并负责与内核交换信息，整体的架构如下：

```
+------+  +------+  +------+  +------+  +------+  +------+  +------+
| bgpd |  | ripd |  | ospfd|  | ldpd |  | pbrd |  | pimd |  | ...  |
+------+  +------+  +------+  +------+  +------+  +------+  +------+
    |         |         |         |         |         |         |
+---v---------v---------v---------v---------v---------v---------v--+
|                                                                  |
|                              Zebra                               |
|                                                                  |
+------------------------------------------------------------------+
       |                         |                         |
       |                         |                         |
+------v-------+        +--------v---------+        +------v-------+
|              |        |                  |        |              |
| LINIX Kernel |        | Remote dataplane |        |    ......    |
|              |        |                  |        |              |
+--------------+        +------------------+        +--------------+
```

Zebra的初始化在`zebra/main.c`里面，查看main函数即可

## frr_init
`frr_init`创建zebra主进程的master数据结构，用来做事件驱动，我们可以看下`event_loop`的数据结构。

```c
/* Master of the theads. */
struct event_loop {
	char *name;

	struct event **read;
	struct event **write;
	struct event_timer_list_head timer;
	struct event_list_head event, ready, unuse;
	struct list *cancel_req;
	bool canceled;
	pthread_cond_t cancel_cond;
	struct hash *cpu_record;
	int io_pipe[2];
	int fd_limit;
	struct fd_handler handler;
	unsigned long alloc;
	long selectpoll_timeout;
	bool spin;
	bool handle_signals;
	pthread_mutex_t mtx;
	pthread_t owner;

	bool ready_run_loop;
	RUSAGE_T last_getrusage;
};
```
{: file='lib/frrevent.h'}

其整合了事件的可读、可写、定时器、信号的处理，后面有时间可以来学习下。
- `frr_pthread_init` 初始化所有的线程链表

## zebra_router_init
初始化和策略路由PBR ?? 相关的HASH

```c
void zebra_router_init(bool asic_offload, bool notify_on_ack)
{
	zrouter.sequence_num = 0;

	zrouter.allow_delete = false;

	zrouter.packets_to_process = ZEBRA_ZAPI_PACKETS_TO_PROCESS;

	zrouter.nhg_keep = ZEBRA_DEFAULT_NHG_KEEP_TIMER;

	zebra_vxlan_init();
	zebra_mlag_init();
	zebra_neigh_init();

	zrouter.rules_hash = hash_create_size(8, zebra_pbr_rules_hash_key,
					      zebra_pbr_rules_hash_equal,
					      "Rules Hash");

	zrouter.ipset_hash =
		hash_create_size(8, zebra_pbr_ipset_hash_key,
				 zebra_pbr_ipset_hash_equal, "IPset Hash");

	zrouter.ipset_entry_hash = hash_create_size(
		8, zebra_pbr_ipset_entry_hash_key,
		zebra_pbr_ipset_entry_hash_equal, "IPset Hash Entry");

	zrouter.iptable_hash = hash_create_size(8, zebra_pbr_iptable_hash_key,
						zebra_pbr_iptable_hash_equal,
						"IPtable Hash Entry");

	zrouter.nhgs =
		hash_create_size(8, zebra_nhg_hash_key, zebra_nhg_hash_equal,
				 "Zebra Router Nexthop Groups");
	zrouter.nhgs_id =
		hash_create_size(8, zebra_nhg_id_key, zebra_nhg_hash_id_equal,
				 "Zebra Router Nexthop Groups ID index");

	zrouter.rules_hash =
		hash_create_size(8, zebra_pbr_rules_hash_key,
				 zebra_pbr_rules_hash_equal, "Rules Hash");

	zrouter.qdisc_hash =
		hash_create_size(8, zebra_tc_qdisc_hash_key,
				 zebra_tc_qdisc_hash_equal, "TC (qdisc) Hash");
	zrouter.class_hash = hash_create_size(8, zebra_tc_class_hash_key,
					      zebra_tc_class_hash_equal,
					      "TC (classes) Hash");
	zrouter.filter_hash = hash_create_size(8, zebra_tc_filter_hash_key,
					       zebra_tc_filter_hash_equal,
					       "TC (filter) Hash");

	zrouter.asic_offloaded = asic_offload;
	zrouter.notify_on_ack = notify_on_ack;

	/*
	 * If you start using asic_notification_nexthop_control
	 * come talk to the FRR community about what you are doing
	 * We would like to know.
	 */
#if CONFDATE > 20251231
	CPP_NOTICE(
		"Remove zrouter.asic_notification_nexthop_control as that it's not being maintained or used");
#endif
	zrouter.asic_notification_nexthop_control = false;

#ifdef HAVE_SCRIPTING
	zebra_script_init();
#endif

	/* OS-specific init */
	kernel_router_init();
}
```
{: file='zebra/zebra_router.c'}

## zserv
zebra作为其他协议进程的服务端，通过建立socket和其它的进程建立通道来交互信息。

```c
void zserv_start(char *path)
{
	int ret;
	mode_t old_mask;
	struct sockaddr_storage sa;
	socklen_t sa_len;

	if (!frr_zclient_addr(&sa, &sa_len, path))
		/* should be caught in zebra main() */
		return;

	/* Set umask */
	old_mask = umask(0077);

	/* Make UNIX domain socket. */
	zsock = socket(sa.ss_family, SOCK_STREAM, 0);
	if (zsock < 0) {
		flog_err_sys(EC_LIB_SOCKET, "Can't create zserv socket: %s",
			     safe_strerror(errno));
		return;
	}

	if (sa.ss_family != AF_UNIX) {
		sockopt_reuseaddr(zsock);
		sockopt_reuseport(zsock);
	} else {
		struct sockaddr_un *suna = (struct sockaddr_un *)&sa;
		if (suna->sun_path[0])
			unlink(suna->sun_path);
	}

	setsockopt_so_recvbuf(zsock, 1048576);
	setsockopt_so_sendbuf(zsock, 1048576);

	frr_with_privs((sa.ss_family != AF_UNIX) ? &zserv_privs : NULL) {
		ret = bind(zsock, (struct sockaddr *)&sa, sa_len);
	}
	if (ret < 0) {
		flog_err_sys(EC_LIB_SOCKET, "Can't bind zserv socket on %s: %s",
			     path, safe_strerror(errno));
		close(zsock);
		zsock = -1;
		return;
	}

	ret = listen(zsock, 5);
	if (ret < 0) {
		flog_err_sys(EC_LIB_SOCKET,
			     "Can't listen to zserv socket %s: %s", path,
			     safe_strerror(errno));
		close(zsock);
		zsock = -1;
		return;
	}

	umask(old_mask);

	zserv_event(NULL, ZSERV_ACCEPT);
}

void zserv_event(struct zserv *client, enum zserv_event event)
{
	switch (event) {
	case ZSERV_ACCEPT:
		event_add_read(zrouter.master, zserv_accept, NULL, zsock, NULL);
		break;
	case ZSERV_PROCESS_MESSAGES:
		event_add_event(zrouter.master, zserv_process_messages, client,
				0, &client->t_process);
		break;
	case ZSERV_HANDLE_CLIENT_FAIL:
		event_add_event(zrouter.master, zserv_handle_client_fail,
				client, 0, &client->t_cleanup);
	}
}
```
{: file='zebra/zserv.c'}

`zserv_accept` 接受客户端的请求，并创建一个新的客户端，还会给每个客户端创建一个线程处理客户端的读、写请求。

```c
/*
 * Accept socket connection.
 */
static void zserv_accept(struct event *thread)
{
	int accept_sock;
	int client_sock;
	struct sockaddr_in client;
	socklen_t len;

	accept_sock = EVENT_FD(thread);

	/* Reregister myself. */
	zserv_event(NULL, ZSERV_ACCEPT);

	len = sizeof(struct sockaddr_in);
	client_sock = accept(accept_sock, (struct sockaddr *)&client, &len);

	if (client_sock < 0) {
		flog_err_sys(EC_LIB_SOCKET, "Can't accept zebra socket: %s",
			     safe_strerror(errno));
		return;
	}

	/* Make client socket non-blocking.  */
	set_nonblocking(client_sock);

	/* Create new zebra client. */
	zserv_client_create(client_sock);
}

```
{: file='zebra/zserv.c'}

```c
/*
 * Create a new client.
 *
 * This is called when a new connection is accept()'d on the ZAPI socket. It
 * initializes new client structure, notifies any subscribers of the connection
 * event and spawns the client's thread.
 *
 * sock
 *    client's socket file descriptor
 */
static struct zserv *zserv_client_create(int sock)
{
	struct zserv *client;
	size_t stream_size =
		MAX(ZEBRA_MAX_PACKET_SIZ, sizeof(struct zapi_route));
	int i;
	afi_t afi;

	client = XCALLOC(MTYPE_ZSERV_CLIENT, sizeof(struct zserv));

	/* Make client input/output buffer. */
	client->sock = sock;
	client->ibuf_fifo = stream_fifo_new();
	client->obuf_fifo = stream_fifo_new();
	client->ibuf_work = stream_new(stream_size);
	client->obuf_work = stream_new(stream_size);
	client->connect_time = monotime(NULL);
	pthread_mutex_init(&client->ibuf_mtx, NULL);
	pthread_mutex_init(&client->obuf_mtx, NULL);
	pthread_mutex_init(&client->stats_mtx, NULL);
	client->wb = buffer_new(0);
	TAILQ_INIT(&(client->gr_info_queue));

	/* Initialize flags */
	for (afi = AFI_IP; afi < AFI_MAX; afi++) {
		for (i = 0; i < ZEBRA_ROUTE_MAX; i++)
			client->redist[afi][i] = vrf_bitmap_init();
		client->redist_default[afi] = vrf_bitmap_init();
		client->ridinfo[afi] = vrf_bitmap_init();
		client->nhrp_neighinfo[afi] = vrf_bitmap_init();
	}

	/* Add this client to linked list. */
	frr_with_mutex (&client_mutex) {
		listnode_add(zrouter.client_list, client);
	}

	struct frr_pthread_attr zclient_pthr_attrs = {
		.start = frr_pthread_attr_default.start,
		.stop = frr_pthread_attr_default.stop
	};
	client->pthread =
		frr_pthread_new(&zclient_pthr_attrs, "Zebra API client thread",
				"zebra_apic");

	/* start read loop */
	zserv_client_event(client, ZSERV_CLIENT_READ);

	/* call callbacks */
	hook_call(zserv_client_connect, client);

	/* start pthread */
	frr_pthread_run(client->pthread, NULL);

	return client;
}
```
{: file='zebra/zserv.c'}

客户端比如bgp会调用`zclient_new`/`zclient_init`初始化客服端连接到zebra服务端，并发送关心的事件到zebra的服务端。

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

`bgp_zebra_connected` 是连接服务端成功后，向zebra注册各种事件的回调函数。

## rib_init

```c
/* Routing information base initialize. */
void rib_init(void)
{
	check_route_info();

	rib_queue_init();

	/* Init dataplane, and register for results */
	pthread_mutex_init(&dplane_mutex, NULL);
	dplane_ctx_q_init(&rib_dplane_q);
	zebra_dplane_init(rib_dplane_results);
}
```
{: file='zebra/zebra_rib.c'}

`rib_queue_init`初始化work queue相关的事情，ribq处理rib信息相关的，`meta_queue_new`会创建5个subq,每个队列是具有优先级的，也就是处理rib的消息是PQ的队列。

```c
/* initialise zebra rib work queue */
static void rib_queue_init(void)
{
	if (!(zrouter.ribq = work_queue_new(zrouter.master,
					    "route_node processing"))) {
		flog_err(EC_ZEBRA_WQ_NONEXISTENT,
			 "%s: could not initialise work queue!", __func__);
		return;
	}

	/* fill in the work queue spec */
	zrouter.ribq->spec.workfunc = &meta_queue_process;
	zrouter.ribq->spec.completion_func = NULL;
	/* XXX: TODO: These should be runtime configurable via vty */
	zrouter.ribq->spec.max_retries = 3;
	zrouter.ribq->spec.hold = ZEBRA_RIB_PROCESS_HOLD_TIME;
	zrouter.ribq->spec.retry = ZEBRA_RIB_PROCESS_RETRY_TIME;

	if (!(zrouter.mq = meta_queue_new())) {
		flog_err(EC_ZEBRA_WQ_NONEXISTENT,
			 "%s: could not initialise meta queue!", __func__);
		return;
	}
	return;
}
```
{: file='zebra/zebra_rib.c'}

```c
/* Create new meta queue.
   A destructor function doesn't seem to be necessary here.
 */
static struct meta_queue *meta_queue_new(void)
{
	struct meta_queue *new;
	unsigned i;

	new = XCALLOC(MTYPE_WORK_QUEUE, sizeof(struct meta_queue));

	for (i = 0; i < MQ_SIZE; i++) {
		new->subq[i] = list_new();
		assert(new->subq[i]);
	}

	return new;
}
```
{: file='zebra/zebra_rib.c'}

```c
/* meta-queue structure:
 * sub-queue 0: nexthop group objects
 * sub-queue 1: EVPN/VxLAN objects
 * sub-queue 2: Early Route Processing
 * sub-queue 3: Early Label Processing
 * sub-queue 4: connected
 * sub-queue 5: kernel
 * sub-queue 6: static
 * sub-queue 7: RIP, RIPng, OSPF, OSPF6, IS-IS, EIGRP, NHRP
 * sub-queue 8: iBGP, eBGP
 * sub-queue 9: any other origin (if any) typically those that
 *              don't generate routes
 */
#define MQ_SIZE 11
struct meta_queue {
	struct list *subq[MQ_SIZE];
	uint32_t size; /* sum of lengths of all subqueues */
};
```
{: file='zebra/rib.h'}

`zebra_dplane_init` 是初始化data plane数据平面的信息，并会初始化linux kernel数据平面处理的函数，也就是会做一个适配层，隔离FRR和DP面，减少耦合。

`zebra_dplane_start`会Start the dataplane pthread，处理数据下发到DP 数据平面的消息。(main函数中调用)

```c
/*
 * Start the dataplane pthread. This step needs to be run later than the
 * 'init' step, in case zebra has fork-ed.
 */
void zebra_dplane_start(void)
{
	struct dplane_zns_info *zi;
	struct zebra_dplane_provider *prov;
	struct frr_pthread_attr pattr = {
		.start = frr_pthread_attr_default.start,
		.stop = frr_pthread_attr_default.stop
	};

	/* Start dataplane pthread */

	zdplane_info.dg_pthread = frr_pthread_new(&pattr, "Zebra dplane thread",
						  "zebra_dplane");

	zdplane_info.dg_master = zdplane_info.dg_pthread->master;

	zdplane_info.dg_run = true;

	/* Enqueue an initial event for the dataplane pthread */
	event_add_event(zdplane_info.dg_master, dplane_thread_loop, NULL, 0,
			&zdplane_info.dg_t_update);

	/* Enqueue requests and reads if necessary */
	frr_each (zns_info_list, &zdplane_info.dg_zns_list, zi) {
#if defined(HAVE_NETLINK)
		event_add_read(zdplane_info.dg_master, dplane_incoming_read, zi,
			       zi->info.sock, &zi->t_read);
		dplane_kernel_info_request(zi);
#endif
	}

	/* Call start callbacks for registered providers */

	DPLANE_LOCK();
	prov = dplane_prov_list_first(&zdplane_info.dg_providers);
	DPLANE_UNLOCK();

	while (prov) {

		if (prov->dp_start)
			(prov->dp_start)(prov);

		/* Locate next provider */
		prov = dplane_prov_list_next(&zdplane_info.dg_providers, prov);
	}

	frr_pthread_run(zdplane_info.dg_pthread, NULL);
}
```
{: file='zebra/zebra_dplance.c'}

## zebra_mpls_init
初始化MPLS相关的信息，主要功能有：

```c
/*
 * Global MPLS initialization.
 */
void zebra_mpls_init(void)
{
	mpls_enabled = false;
	mpls_pw_reach_strict = false;

	if (mpls_kernel_init() < 0) {
		flog_warn(EC_ZEBRA_MPLS_SUPPORT_DISABLED,
			  "Disabling MPLS support (no kernel support)");
		return;
	}

	zebra_mpls_turned_on();
}
```
{: file='zebra/zebra_mpls.c'}

1. 内核是否支持MPLS

```c
int mpls_kernel_init(void)
{
	struct stat st;

	/*
	 * Check if the MPLS module is loaded in the kernel.
	 */
	if (stat("/proc/sys/net/mpls", &st) != 0)
		return -1;

	return 0;
};
```
{: file='zebra/zebra_mpls_netlink.c'}

2. Zebra处理MPLS 信息的work queue

```c
/*
 * Initialize work queue for processing changed LSPs.
 */
static void mpls_processq_init(void)
{
	zrouter.lsp_process_q = work_queue_new(zrouter.master, "LSP processing");

	zrouter.lsp_process_q->spec.workfunc = &lsp_process;
	zrouter.lsp_process_q->spec.del_item_data = &lsp_processq_del;
	zrouter.lsp_process_q->spec.completion_func = &lsp_processq_complete;
	zrouter.lsp_process_q->spec.max_retries = 0;
	zrouter.lsp_process_q->spec.hold = 10;
}
```
{: file='zebra/zebra_mpls.c'}

最后 `frr_run` ，zebra 主线程跑起来，全部初始化完成，其它的初始化点，后续在继续补充 ！！！！！！！

****

本文参考

> 1. [BGP官方文档](https://docs.frrouting.org/en/latest/bgp.html)
> 2. [FRR BGP 协议分析](https://blog.csdn.net/armlinuxww/article/details/103313633?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522169700705016800225544715%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=169700705016800225544715&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-1-103313633-null-null.nonecase&utm_term=FRR%20BGP%20%E5%8D%8F%E8%AE%AE%E5%88%86%E6%9E%9011&spm=1018.2226.3001.4450)