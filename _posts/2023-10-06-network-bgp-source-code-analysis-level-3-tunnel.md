---
title: FRR BGP源码分析5 -- BGP 层3隧道
author: fnoobt
date: 2023-10-06 19:29:00 +0800
categories: [Network,路由协议]
tags: [network,bgp,frr]
---

本文，我们分析 BGP 和 层3隧道 相关的命令处理

```bash
router bgp 200
	no bgp default ipv4-unicast
	no bgp default show-hostname
	bgp router-id 2.2.2.2
	neighbor 3.3.3.3 remote-as 200
	neighbor 3.3.3.3 update-source 2.2.2.2
	neighbor 5.5.5.5 remote-as 100
	neighbor 5.5.5.5 ebgp-multihop 255
	neighbor 5.5.5.5 update-source 2.2.2.2
	!
	address-family ipv4 labeled-unicast
	  neighbor 3.3.3.3 activate
	exit-address-family
	!
	address-family ipv4 vpn
	  neighhor 5.5.5.5 activate
	exit-address-family
	!
	router bgp 200 vrf 1
	  !
	  address-family ipv4 unicast
	    redistribute ospf l00 metric l0
		redistribute ospf
		label vpn export auto
		rd vpn export 12.1.1.2:2
		rt vpn import 100:1
		rt vpn export 12.1.1.2:2
		export vpn
		import vpn
	  exit-address-family
	!
```
{: file='bgpd/bgp_route.c'}

## label export auto
配置label export auto 后，执行如下的命令

```c
DEFPY (af_rd_vpn_export,
       af_rd_vpn_export_cmd,
       "[no] rd vpn export ASN:NN_OR_IP-ADDRESS:NN$rd_str",
       NO_STR
       "Specify route distinguisher\n"
       "Between current address-family and vpn\n"
       "For routes leaked from current address-family to vpn\n"
       "Route Distinguisher (<as-number>:<number> | <ip-address>:<number>)\n")
{
	VTY_DECLVAR_CONTEXT(bgp, bgp);
	struct prefix_rd prd;
	int ret;
	afi_t afi;
	int idx = 0;
	bool yes = true;

	if (argv_find(argv, argc, "no", &idx))
		yes = false;

	if (yes) {
		ret = str2prefix_rd(rd_str, &prd);
		if (!ret) {
			vty_out(vty, "%% Malformed rd\n");
			return CMD_WARNING_CONFIG_FAILED;
		}
	}

	afi = vpn_policy_getafi(vty, bgp, false);
	if (afi == AFI_MAX)
		return CMD_WARNING_CONFIG_FAILED;

	/*
	 * pre-change: un-export vpn routes (vpn->vrf routes unaffected)
	 */
	vpn_leak_prechange(BGP_VPN_POLICY_DIR_TOVPN, afi,
			   bgp_get_default(), bgp);

	if (yes) {
		bgp->vpn_policy[afi].tovpn_rd_pretty =
			XSTRDUP(MTYPE_BGP, rd_str);
		bgp->vpn_policy[afi].tovpn_rd = prd;
		SET_FLAG(bgp->vpn_policy[afi].flags,
			 BGP_VPN_POLICY_TOVPN_RD_SET);
	} else {
		XFREE(MTYPE_BGP, bgp->vpn_policy[afi].tovpn_rd_pretty);
		UNSET_FLAG(bgp->vpn_policy[afi].flags,
			   BGP_VPN_POLICY_TOVPN_RD_SET);
	}

	/* post-change: re-export vpn routes */
	vpn_leak_postchange(BGP_VPN_POLICY_DIR_TOVPN, afi,
			    bgp_get_default(), bgp);

	return CMD_SUCCESS;
}
```
{: file='bgpd/bgp_vty.c'}

命令会给每个VRF 分配一个标签，然后下发到MPLS 的LFIB表里面，涉及MPLS表项下发到内核的流程，先大概给个流程，后续再补充。

> vpn_leak_postchange
>> vpn_leak_zebra_vrf_label_update
>>> zclient_send_vrf_label

zebra 处理

> zread_vrf_label
>> mpls_lsp_install

后续处理的逻辑和LDP 一样

## rd export
命令执行函数是：

```c
DEFPY (af_rd_vpn_export,
       af_rd_vpn_export_cmd,
       "[no] rd vpn export ASN:NN_OR_IP-ADDRESS:NN$rd_str",
       NO_STR
       "Specify route distinguisher\n"
       "Between current address-family and vpn\n"
       "For routes leaked from current address-family to vpn\n"
       "Route Distinguisher (<as-number>:<number> | <ip-address>:<number>)\n")
{
	VTY_DECLVAR_CONTEXT(bgp, bgp);
	struct prefix_rd prd;
	int ret;
	afi_t afi;
	int idx = 0;
	bool yes = true;
```
{: file='bgpd/bgp_vty.c'}

## Export/import vpn
命令执行函数是：

```c
/* This command is valid only in a bgp vrf instance or the default instance */
DEFPY (bgp_imexport_vpn,
       bgp_imexport_vpn_cmd,
       "[no] <import|export>$direction_str vpn",
       NO_STR
       "Import routes to this address-family\n"
       "Export routes from this address-family\n"
       "to/from default instance VPN RIB\n")
{
	VTY_DECLVAR_CONTEXT(bgp, bgp);
	int previous_state;
	afi_t afi;
	safi_t safi;
	int idx = 0;
	bool yes = true;
	int flag;
	enum vpn_policy_direction dir;
```
{: file='bgpd/bgp_vty.c'}

## 层3VPN UPDATE消息的处理过程
层3VPN 的UPDATE消息和前面的路由更新报文的接受处理是一样的，从解析属性的地方开始区别，报文如下：

```
2019/12/26 09:32:44 BGP: : 20.20.20.20 rcvd UPDATE w/ attr: , origin ?, metric 100, extcommunity RT:12.1.1.2:2, path 200
2019/12/26 09:32:44 BGP: : 20.20.20.20 rcvd UPDATE wlen 0 attrlen 69 alen 0
2019/12/26 09:32:44 BGP: : 20.20.20.20 rcvd RD 12.1.1.2:2 1.1.1.1/32 label 154 IPv4 vpn
2019/12/26 09:32:44 BGP: : vpn_leak_to_vrf_update: start (path_vpn=0x5589ff3debf0)
2019/12/26 09:32:44 BGP: : vpn_leak_to_vrf_update_onevrf: skipping: import not set
2019/12/26 09:32:44 BGP: : vpn_leak_to_vrf_update_onevrf: updating 1.1.1.1/32 to vrf VRF 1
2019/12/26 09:32:44 BGP: : vpn_leak_to_vrf_update_onevrf: pfx 1.1.1.1/32: num_labels 1
2019/12/26 09:32:44 BGP: : leak_update: entry: leak-to=VRF 1, p=1.1.1.1/32, type=9, sub_type=0
2019/12/26 09:32:44 BGP: : leak_update: nexthop is valid (in vrf VRF default)
2019/12/26 09:32:44 BGP: : leak_update: ->VRF 1: 1.1.1.1/32: Added new route
2019/12/26 09:32:44 BGP: : group_announce_route_walkcb: afi=IPv4, safi=vpn, p=1.1.1.1/32
2019/12/26 09:32:44 BGP: : subgroup_process_announce_selected: p=1.1.1.1/32, selected=0x5589ff3debf0
2019/12/26 09:32:44 BGP: : bgp_zebra_announce: p=1.1.1.1/32, bgp_is_valid_label: 2
2019/12/26 09:32:44 BGP: : Tx route add VRF 7 1.1.1.1/32 metric 100 tag 0 count 1
2019/12/26 09:32:44 BGP: :   nhop [1]: 20.20.20.20 if 0 VRF 0 label 154
2019/12/26 09:32:44 BGP: : bgp_zebra_announce: 1.1.1.1/32: announcing to zebra (recursion set)
2019/12/26 09:32:44 BGP: : u11:s11 send UPDATE w/ attr: , origin ?, extcommunity RT:12.1.1.2:2, path 200
2019/12/26 09:32:44 BGP: : u11:s11 send MP_REACH for afi/safi 1/128
2019/12/26 09:32:44 BGP: : u11:s11 send UPDATE RD 12.1.1.2:2 1.1.1.1/32 label 154 IPv4 vpn
2019/12/26 09:32:44 BGP: : u11:s11 send UPDATE len 89 numpfx 1
2019/12/26 09:32:44 BGP: : u11:s11 20.20.20.20 send UPDATE w/ nexthop 10.10.10.10 and RD
```
### 属性解析
在属性解析函数 `bgp_attr_parse`里面，我们分析和层3隧道相关的属性解析。

```c
		case BGP_ATTR_CLUSTER_LIST:
			ret = bgp_attr_cluster_list(&attr_args);
			break;
		case BGP_ATTR_MP_REACH_NLRI:
			ret = bgp_mp_reach_parse(&attr_args, mp_update);
			break;
		case BGP_ATTR_MP_UNREACH_NLRI:
			ret = bgp_mp_unreach_parse(&attr_args, mp_withdraw);
			break;
		case BGP_ATTR_EXT_COMMUNITIES:
			ret = bgp_attr_ext_communities(&attr_args);
			break;
```
{: file='bgpd/bgp_attr.c -- bgp_attr_parse()'}

### 解析MP_REACH_NLRI

```c
/* Multiprotocol reachability information parse. */
int bgp_mp_reach_parse(struct bgp_attr_parser_args *args,
		       struct bgp_nlri *mp_update)
{
	iana_afi_t pkt_afi;
	afi_t afi;
	iana_safi_t pkt_safi;
	safi_t safi;
	bgp_size_t nlri_len;
	size_t start;
	struct stream *s;
	struct peer *const peer = args->peer;
	struct attr *const attr = args->attr;
	const bgp_size_t length = args->length;

	/* Set end of packet. */
	s = BGP_INPUT(peer);
	start = stream_get_getp(s);

/* safe to read statically sized header? */
#define BGP_MP_REACH_MIN_SIZE 5
#define LEN_LEFT	(length - (stream_get_getp(s) - start))
	if ((length > STREAM_READABLE(s)) || (length < BGP_MP_REACH_MIN_SIZE)) {
		zlog_info("%s: %s sent invalid length, %lu, of MP_REACH_NLRI",
			  __func__, peer->host, (unsigned long)length);
		return BGP_ATTR_PARSE_ERROR_NOTIFYPLS;
	}

	/* Load AFI, SAFI. */
	pkt_afi = stream_getw(s);
	pkt_safi = stream_getc(s);
```
{: file='bgpd/bgp_attr.c'}

首先解析报文的afi和safi的值为1和128，然后转化为FRR 内部自己定义的afi和safi的值。

```c
	/* Load AFI, SAFI. */
	pkt_afi = stream_getw(s);
	pkt_safi = stream_getc(s);

	/* Convert AFI, SAFI to internal values, check. */
	if (bgp_map_afi_safi_iana2int(pkt_afi, pkt_safi, &afi, &safi)) {
		/* Log if AFI or SAFI is unrecognized. This is not an error
		 * unless
		 * the attribute is otherwise malformed.
		 */
		if (bgp_debug_update(peer, NULL, NULL, 0))
			zlog_debug(
				"%s sent unrecognizable AFI, %s or, SAFI, %s, of MP_REACH_NLRI",
				peer->host, iana_afi2str(pkt_afi),
				iana_safi2str(pkt_safi));
		return BGP_ATTR_PARSE_ERROR;
	}
```
{: file='bgpd/bgp_attr.c -- bgp_mp_reach_parse()'}

### 解析下一跳
获取下一跳的长度为12，然后获取RD(8字节)，但没有存储，只是跳过，在获取IP地址4字节，存放在attr下一跳nexthop里面。

```c
	/* Nexthop length check. */
	switch (attr->mp_nexthop_len) {
	case 0:
		if (safi != SAFI_FLOWSPEC) {
			zlog_info("%s: %s sent wrong next-hop length, %d, in MP_REACH_NLRI",
				  __func__, peer->host, attr->mp_nexthop_len);
			return BGP_ATTR_PARSE_ERROR_NOTIFYPLS;
		}
		break;
	case BGP_ATTR_NHLEN_VPNV4:
		stream_getl(s); /* RD high */
		stream_getl(s); /* RD low */
				/*
				 * NOTE: intentional fall through
				 * - for consistency in rx processing
				 *
				 * The following comment is to signal GCC this intention
				 * and suppress the warning
				 */
	/* FALLTHRU */
	case BGP_ATTR_NHLEN_IPV4:
		stream_get(&attr->mp_nexthop_global_in, s, IPV4_MAX_BYTELEN);
		/* Probably needed for RFC 2283 */
		if (attr->nexthop.s_addr == INADDR_ANY)
			memcpy(&attr->nexthop.s_addr,
			       &attr->mp_nexthop_global_in, IPV4_MAX_BYTELEN);
		break;
```
{: file='bgpd/bgp_attr.c -- bgp_mp_reach_parse()'}

```c
	mp_update->afi = afi;
	mp_update->safi = safi;
	mp_update->nlri = stream_pnt(s);
	mp_update->length = nlri_len;

	stream_forward_getp(s, nlri_len);

	attr->flag |= ATTR_FLAG_BIT(BGP_ATTR_MP_REACH_NLRI);
```
{: file='bgpd/bgp_attr.c -- bgp_mp_reach_parse()'}

最后存放在bgp_nlri的NLRI_MP_UPDATE里面，nlri保存的是MP_REACH_NLRI信息。

### 解析EXT_COMMUNITIES

```c
/* Large Community attribute. */
static enum bgp_attr_parse_ret
bgp_attr_large_community(struct bgp_attr_parser_args *args)
{
	struct peer *const peer = args->peer;
	struct attr *const attr = args->attr;
	const bgp_size_t length = args->length;

	/*
	 * Large community follows new attribute format.
	 */
	if (length == 0) {
		bgp_attr_set_lcommunity(attr, NULL);
		/* Empty extcomm doesn't seem to be invalid per se */
		return bgp_attr_malformed(args, BGP_NOTIFY_UPDATE_OPT_ATTR_ERR,
					  args->total);
	}

	if (peer->discard_attrs[args->type] || peer->withdraw_attrs[args->type])
		goto large_community_ignore;

	bgp_attr_set_lcommunity(
		attr, lcommunity_parse(stream_pnt(peer->curr), length));
	/* XXX: fix ecommunity_parse to use stream API */
	stream_forward_getp(peer->curr, length);

	if (!bgp_attr_get_lcommunity(attr))
		return bgp_attr_malformed(args, BGP_NOTIFY_UPDATE_OPT_ATTR_ERR,
					  args->total);

	return BGP_ATTR_PARSE_PROCEED;

large_community_ignore:
	stream_forward_getp(peer->curr, length);

	return bgp_attr_ignore(peer, args->type);
}

/* Extended Community attribute. */
static enum bgp_attr_parse_ret
bgp_attr_ext_communities(struct bgp_attr_parser_args *args)
{
	struct peer *const peer = args->peer;
	struct attr *const attr = args->attr;
	const bgp_size_t length = args->length;
	uint8_t sticky = 0;
	bool proxy = false;
	struct ecommunity *ecomm;

	if (length == 0) {
		bgp_attr_set_ecommunity(attr, NULL);
		/* Empty extcomm doesn't seem to be invalid per se */
		return bgp_attr_malformed(args, BGP_NOTIFY_UPDATE_OPT_ATTR_ERR,
					  args->total);
	}

	ecomm = ecommunity_parse(
		stream_pnt(peer->curr), length,
		CHECK_FLAG(peer->flags,
			   PEER_FLAG_DISABLE_LINK_BW_ENCODING_IEEE));
	bgp_attr_set_ecommunity(attr, ecomm);
	/* XXX: fix ecommunity_parse to use stream API */
	stream_forward_getp(peer->curr, length);

	/* The Extended Community attribute SHALL be considered malformed if
	 * its length is not a non-zero multiple of 8.
	 */
	if (!bgp_attr_get_ecommunity(attr))
		return bgp_attr_malformed(args, BGP_NOTIFY_UPDATE_OPT_ATTR_ERR,
					  args->total);

	/* Extract DF election preference and  mobility sequence number */
	attr->df_pref = bgp_attr_df_pref_from_ec(attr, &attr->df_alg);

	/* Extract MAC mobility sequence number, if any. */
	attr->mm_seqnum = bgp_attr_mac_mobility_seqnum(attr, &sticky);
	attr->sticky = sticky;

	/* Check if this is a Gateway MAC-IP advertisement */
	attr->default_gw = bgp_attr_default_gw(attr);
```
{: file='bgpd/bgp_attr.c'}

### NLRI处理

```c
/**
 * Frontend for NLRI parsing, to fan-out to AFI/SAFI specific parsers.
 *
 * mp_withdraw, if set, is used to nullify attr structure on most of the
 * calling safi function and for evpn, passed as parameter
 */
int bgp_nlri_parse(struct peer *peer, struct attr *attr,
		   struct bgp_nlri *packet, bool mp_withdraw)
{
	switch (packet->safi) {
	case SAFI_UNICAST:
	case SAFI_MULTICAST:
		return bgp_nlri_parse_ip(peer, mp_withdraw ? NULL : attr,
					 packet);
	case SAFI_LABELED_UNICAST:
		return bgp_nlri_parse_label(peer, mp_withdraw ? NULL : attr,
					    packet);
	case SAFI_MPLS_VPN:
		return bgp_nlri_parse_vpn(peer, mp_withdraw ? NULL : attr,
					  packet);
	case SAFI_EVPN:
		return bgp_nlri_parse_evpn(peer, attr, packet, mp_withdraw);
	case SAFI_FLOWSPEC:
		return bgp_nlri_parse_flowspec(peer, attr, packet, mp_withdraw);
	}
	return BGP_NLRI_PARSE_ERROR;
}
```
{: file='bgpd/bgp_packet.c'}

bgp_nlri_parse 函数里面由于前面解析的 safi 是 SAFI_MPLS_VPN ，所以NLRI的处理会是 bgp_nlri_parse_vpn 函数，主要处理过程如下，可以结合上面的报文一起看代码处理过程：
1. 清理的前缀结构
2. 获取前缀长度
3. 针对数据包数据\IP地址存储\地址族的最大位长度进行健全性检查
4. 将标签复制到前缀
5. 将路由标识符复制到 RD
6. 解码RD类型
7. 排除标签和RD
8. 数据包长度一致性检查

```c
int bgp_nlri_parse_vpn(struct peer *peer, struct attr *attr,
		       struct bgp_nlri *packet)
{
	struct prefix p;
	uint8_t psize = 0;
	uint8_t prefixlen;
	uint16_t type;
	struct rd_as rd_as;
	struct rd_ip rd_ip;
	struct prefix_rd prd = {0};
	mpls_label_t label = {0};
	afi_t afi;
	safi_t safi;
	bool addpath_capable;
	uint32_t addpath_id;
	int ret = 0;

	/* Make prefix_rd */
	prd.family = AF_UNSPEC;
	prd.prefixlen = 64;

	struct stream *data = stream_new(packet->length);
	stream_put(data, packet->nlri, packet->length);
	afi = packet->afi;
	safi = packet->safi;
	addpath_id = 0;

	addpath_capable = bgp_addpath_encode_rx(peer, afi, safi);

#define VPN_PREFIXLEN_MIN_BYTES (3 + 8) /* label + RD */
	while (STREAM_READABLE(data) > 0) {
		/* Clear prefix structure. */
		memset(&p, 0, sizeof(p));

		if (addpath_capable) {
			STREAM_GET(&addpath_id, data, BGP_ADDPATH_ID_LEN);
			addpath_id = ntohl(addpath_id);
		}

		if (STREAM_READABLE(data) < 1) {
			flog_err(
				EC_BGP_UPDATE_RCV,
				"%s [Error] Update packet error / VPN (truncated NLRI of size %u; no prefix length)",
				peer->host, packet->length);
			ret = BGP_NLRI_PARSE_ERROR_PACKET_LENGTH;
			goto done;
		}

		/* Fetch prefix length. */
		STREAM_GETC(data, prefixlen);
		p.family = afi2family(packet->afi);
		psize = PSIZE(prefixlen);

		if (prefixlen < VPN_PREFIXLEN_MIN_BYTES * 8) {
			flog_err(
				EC_BGP_UPDATE_RCV,
				"%s [Error] Update packet error / VPN (prefix length %d less than VPN min length)",
				peer->host, prefixlen);
			ret = BGP_NLRI_PARSE_ERROR_PREFIX_LENGTH;
			goto done;
		}

		/* sanity check against packet data */
		if (STREAM_READABLE(data) < psize) {
			flog_err(
				EC_BGP_UPDATE_RCV,
				"%s [Error] Update packet error / VPN (prefix length %d exceeds packet size %u)",
				peer->host, prefixlen, packet->length);
			ret = BGP_NLRI_PARSE_ERROR_PACKET_OVERFLOW;
			goto done;
		}

		/* sanity check against storage for the IP address portion */
		if ((psize - VPN_PREFIXLEN_MIN_BYTES) > (ssize_t)sizeof(p.u)) {
			flog_err(
				EC_BGP_UPDATE_RCV,
				"%s [Error] Update packet error / VPN (psize %d exceeds storage size %zu)",
				peer->host,
				prefixlen - VPN_PREFIXLEN_MIN_BYTES * 8,
				sizeof(p.u));
			ret = BGP_NLRI_PARSE_ERROR_PACKET_LENGTH;
			goto done;
		}

		/* Sanity check against max bitlen of the address family */
		if ((psize - VPN_PREFIXLEN_MIN_BYTES) > prefix_blen(&p)) {
			flog_err(
				EC_BGP_UPDATE_RCV,
				"%s [Error] Update packet error / VPN (psize %d exceeds family (%u) max byte len %u)",
				peer->host,
				prefixlen - VPN_PREFIXLEN_MIN_BYTES * 8,
				p.family, prefix_blen(&p));
			ret = BGP_NLRI_PARSE_ERROR_PACKET_LENGTH;
			goto done;
		}

		/* Copy label to prefix. */
		if (STREAM_READABLE(data) < BGP_LABEL_BYTES) {
			flog_err(
				EC_BGP_UPDATE_RCV,
				"%s [Error] Update packet error / VPN (truncated NLRI of size %u; no label)",
				peer->host, packet->length);
			ret = BGP_NLRI_PARSE_ERROR_PACKET_LENGTH;
			goto done;
		}

		STREAM_GET(&label, data, BGP_LABEL_BYTES);
		bgp_set_valid_label(&label);

		/* Copy routing distinguisher to rd. */
		if (STREAM_READABLE(data) < 8) {
			flog_err(
				EC_BGP_UPDATE_RCV,
				"%s [Error] Update packet error / VPN (truncated NLRI of size %u; no RD)",
				peer->host, packet->length);
			ret = BGP_NLRI_PARSE_ERROR_PACKET_LENGTH;
			goto done;
		}
		STREAM_GET(&prd.val, data, 8);

		/* Decode RD type. */
		type = decode_rd_type(prd.val);

		switch (type) {
		case RD_TYPE_AS:
			decode_rd_as(&prd.val[2], &rd_as);
			break;

		case RD_TYPE_AS4:
			decode_rd_as4(&prd.val[2], &rd_as);
			break;

		case RD_TYPE_IP:
			decode_rd_ip(&prd.val[2], &rd_ip);
			break;

#ifdef ENABLE_BGP_VNC
		case RD_TYPE_VNC_ETH:
			break;
#endif

		default:
			flog_err(EC_BGP_UPDATE_RCV, "Unknown RD type %d", type);
			break; /* just report */
		}

		/* exclude label & RD */
		p.prefixlen = prefixlen - VPN_PREFIXLEN_MIN_BYTES * 8;
		STREAM_GET(p.u.val, data, psize - VPN_PREFIXLEN_MIN_BYTES);

		if (attr) {
			bgp_update(peer, &p, addpath_id, attr, packet->afi,
				   SAFI_MPLS_VPN, ZEBRA_ROUTE_BGP,
				   BGP_ROUTE_NORMAL, &prd, &label, 1, 0, NULL);
		} else {
			bgp_withdraw(peer, &p, addpath_id, packet->afi,
				     SAFI_MPLS_VPN, ZEBRA_ROUTE_BGP,
				     BGP_ROUTE_NORMAL, &prd, &label, 1, NULL);
		}
	}
	/* Packet length consistency check. */
	if (STREAM_READABLE(data) != 0) {
		flog_err(
			EC_BGP_UPDATE_RCV,
			"%s [Error] Update packet error / VPN (%zu data remaining after parsing)",
			peer->host, STREAM_READABLE(data));
		return BGP_NLRI_PARSE_ERROR_PACKET_LENGTH;
	}

	goto done;

stream_failure:
	flog_err(
		EC_BGP_UPDATE_RCV,
		"%s [Error] Update packet error / VPN (NLRI of size %u - length error)",
		peer->host, packet->length);
	ret = BGP_NLRI_PARSE_ERROR_PACKET_LENGTH;

done:
	stream_free(data);
	return ret;

#undef VPN_PREFIXLEN_MIN_BYTES
}
```
{: file='bgpd/bgp_mplsvpn.c'}

bgp_update 前面分析的时候，大概过程已经分析过了，下面会分析下，前面忽略的L3VPN相关的处理过程。
- `has_valid_label`处理会变成1，因为前面传入了解析出来的label值

```c
	/* TODO: Check to see if we can get rid of "is_valid_label" */
	if (afi == AFI_L2VPN && safi == SAFI_EVPN)
		has_valid_label = (num_labels > 0) ? 1 : 0;
	else
		has_valid_label = bgp_is_valid_label(label);

	if (has_valid_label)
		assert(label != NULL);
```
{: file='bgpd/bgp_route.c -- bgp_update'}

- 在构建`struct bgp_path_info`的时候，如果label有值，还需要构建`struct bgp_path_info_extra`，然后把label值存放在info_extra里面

```c
/* Ancillary information to struct bgp_path_info,
 * used for uncommonly used data (aggregation, MPLS, etc.)
 * and lazily allocated to save memory.
 */
struct bgp_path_info_extra {
	/* Pointer to dampening structure.  */
	struct bgp_damp_info *damp_info;

	/** List of aggregations that suppress this path. */
	struct list *aggr_suppressors;

	/* Nexthop reachability check.  */
	uint32_t igpmetric;

	/* MPLS label(s) - VNI(s) for EVPN-VxLAN  */
	mpls_label_t label[BGP_MAX_LABELS];
	uint32_t num_labels;
```
{: file='bgpd/bgp_route.h'}


```c
	/* Make new BGP info. */
	new = info_make(type, sub_type, 0, peer, attr_new, dest);

	/* Update MPLS label */
	if (has_valid_label) {
		extra = bgp_path_info_extra_get(new);
		if (extra->label != label) {
			memcpy(&extra->label, label,
			       num_labels * sizeof(mpls_label_t));
			extra->num_labels = num_labels;
		}
		if (!(afi == AFI_L2VPN && safi == SAFI_EVPN))
			bgp_set_valid_label(&extra->label[0]);
	}
```
{: file='bgpd/bgp_route.c -- bgp_update()'}

- 把路由信息导入到VRF的BGP里面

```c
	if (SAFI_MPLS_VPN == safi
	    && bgp->inst_type == BGP_INSTANCE_TYPE_DEFAULT) {
		vpn_leak_to_vrf_update(bgp, new, &bgp_static->prd);
	}
```
{: file='bgpd/bgp_route.c -- bgp_static_update_safi()'}

`vpn_leak_to_vrf_update` 函数处理有点长，主体处理的是把vpn peer收到的路由信息，导入到VRF的BGP里面

```c
bool vpn_leak_to_vrf_update(struct bgp *from_bgp,
			    struct bgp_path_info *path_vpn,
			    struct prefix_rd *prd)
{
	struct listnode *mnode, *mnnode;
	struct bgp *bgp;
	bool leak_success = false;

	int debug = BGP_DEBUG(vpn, VPN_LEAK_TO_VRF);

	if (debug)
		zlog_debug("%s: start (path_vpn=%p)", __func__, path_vpn);

	/* Loop over VRFs */
	for (ALL_LIST_ELEMENTS(bm->bgp, mnode, mnnode, bgp)) {

		if (!path_vpn->extra
		    || path_vpn->extra->bgp_orig != bgp) { /* no loop */
			leak_success |= vpn_leak_to_vrf_update_onevrf(
				bgp, from_bgp, path_vpn, prd);
		}
	}
	return leak_success;
}
```
{: file='bgpd/bgp_mplsvpn.c'}


```c
static bool vpn_leak_to_vrf_update_onevrf(struct bgp *to_bgp,   /* to */
					  struct bgp *from_bgp, /* from */
					  struct bgp_path_info *path_vpn,
					  struct prefix_rd *prd)
{
	const struct prefix *p = bgp_dest_get_prefix(path_vpn->net);
	afi_t afi = family2afi(p->family);

	struct attr static_attr = {0};
	struct attr *new_attr = NULL;
	struct bgp_dest *bn;
	safi_t safi = SAFI_UNICAST;
	const char *debugmsg;
	struct prefix nexthop_orig;
	mpls_label_t *pLabels = NULL;
	uint32_t num_labels = 0;
	int nexthop_self_flag = 1;
	struct bgp_path_info *bpi_ultimate = NULL;
	int origin_local = 0;
	struct bgp *src_vrf;
	struct interface *ifp;
	char rd_buf[RD_ADDRSTRLEN];
	int debug = BGP_DEBUG(vpn, VPN_LEAK_TO_VRF);

	if (!vpn_leak_from_vpn_active(to_bgp, afi, &debugmsg)) {
		if (debug)
			zlog_debug(
				"%s: from vpn (%s) to vrf (%s), skipping: %s",
				__func__, from_bgp->name_pretty,
				to_bgp->name_pretty, debugmsg);
		return false;
	}
```
{: file='bgpd/bgp_mplsvpn.c'}

### 发送端ospf vrf的路由导入BGP处理：
Zebra:

```bash
ubuntu# 2019/12/26 13:05:00 ZEBRA: : netlink_parse_info: netlink-listen (NS 0) type RTM_NEWNEIGH(28), len=76, seq=0, pid=0

2019/12/26 13:05:00 ZEBRA: :    Neighbor Entry received is not on a VLAN or a BRIDGE, ignoring

2019/12/26 13:05:00 ZEBRA: : netlink_parse_info: netlink-listen (NS 0) type RTM_NEWNEIGH(28), len=76, seq=0, pid=0

2019/12/26 13:05:00 ZEBRA: :    Neighbor Entry received is not on a VLAN or a BRIDGE, ignoring

2019/12/26 13:05:08 ZEBRA: : zebra message[ZEBRA_ROUTE_ADD:7:45] comes from socket [42]

2019/12/26 13:05:08 ZEBRA: : Read 1 packets from client: ospf

2019/12/26 13:05:08 ZEBRA: : rib_add_multipath: 7:1.1.1.1/32: Inserting route rn 0x56275b8132b0, re 0x56275b812140 (ospf) existing (nil)

2019/12/26 13:05:08 ZEBRA: : rib_add_multipath: dumping RE entry 0x56275b812140 for 1.1.1.1/32 vrf 7

2019/12/26 13:05:08 ZEBRA: : 1.1.1.1/32: uptime == 118014, type == 6, instance == 0, table == 1

2019/12/26 13:05:08 ZEBRA: : 1.1.1.1/32: metric == 100, mtu == 0, distance == 110, flags == 0, status == 0

2019/12/26 13:05:08 ZEBRA: : 1.1.1.1/32: nexthop_num == 1, nexthop_active_num == 0

2019/12/26 13:05:08 ZEBRA: : 1.1.1.1/32: NH 12.1.1.1[5] vrf 1(7) with flags

2019/12/26 13:05:08 ZEBRA: : 1.1.1.1/32: dump complete

2019/12/26 13:05:08 ZEBRA: : rib_link: 7:1.1.1.1/32: rn 0x56275b8132b0 adding dest

2019/12/26 13:05:08 ZEBRA: : rib_meta_queue_add: 7:1.1.1.1/32: queued rn 0x56275b8132b0 into sub-queue 2

2019/12/26 13:05:08 ZEBRA: : 7:1.1.1.1/32: Processing rn 0x56275b8132b0

2019/12/26 13:05:08 ZEBRA: : 7:1.1.1.1/32: Examine re 0x56275b812140 (ospf) status 2 flags 0 dist 110 metric 100

2019/12/26 13:05:08 ZEBRA: : 7:1.1.1.1/32: After processing: old_selected 0x0 new_selected 0x56275b812140 old_fib 0x0 new_fib 0x56275b812140

2019/12/26 13:05:08 ZEBRA: : 7:1.1.1.1/32: Adding route rn 0x56275b8132b0, re 0x56275b812140 (ospf)

2019/12/26 13:05:08 ZEBRA: : netlink_route_multipath(): RTM_NEWROUTE 1.1.1.1/32 vrf 7(1)

2019/12/26 13:05:08 ZEBRA: : netlink_route_multipath() (single-path): nexthop via 12.1.1.1  if 5(7)

2019/12/26 13:05:08 ZEBRA: : netlink_talk: netlink-dp (NS 0) type RTM_NEWROUTE(24), len=60 seq=115 flags 0x501

2019/12/26 13:05:08 ZEBRA: : 7:1.1.1.1/32: rn 0x56275b8132b0 dequeued from sub-queue 2

2019/12/26 13:05:08 ZEBRA: : update_from_ctx: 7:1.1.1.1/32: SELECTED

2019/12/26 13:05:08 ZEBRA: : 7:1.1.1.1/32 update_from_ctx(): no fib nhg

2019/12/26 13:05:08 ZEBRA: : 7:1.1.1.1/32 update_from_ctx(): rib nhg matched, changed 'true'

2019/12/26 13:05:08 ZEBRA: : 7:1.1.1.1/32: Redist update re 0x56275b812140 (ospf), old 0x0 (None)

2019/12/26 13:05:08 ZEBRA: : zsend_redistribute_route: ZEBRA_REDISTRIBUTE_ROUTE_ADD to client bgp: type ospf, vrf_id 7, p 1.1.1.1/32

2019/12/26 13:05:08 ZEBRA: : Not Notifying Owner: 6 about prefix 1.1.1.1/32(1) 2 vrf: 7
```

Bgp:

```bash
ubuntu# 2019/12/26 13:05:08 BGP: : vpn_leak_from_vrf_update: from vrf VRF 1

2019/12/26 13:05:08 BGP: : vpn_leak_from_vrf_update: post merge static_attr.ecommunity{12.1.1.2:2}

2019/12/26 13:05:08 BGP: : vpn_leak_from_vrf_update: new_attr->ecommunity{12.1.1.2:2}

2019/12/26 13:05:08 BGP: : leak_update: entry: leak-to=VRF default, p=1.1.1.1/32, type=6, sub_type=3

2019/12/26 13:05:08 BGP: : leak_update: nexthop is valid (in vrf VRF 1)

2019/12/26 13:05:08 BGP: : leak_update: ->VRF default: 1.1.1.1/32: Added new route

2019/12/26 13:05:08 BGP: : vpn_leak_to_vrf_update: start (path_vpn=0x557deca2fe90)

2019/12/26 13:05:08 BGP: : vpn_leak_to_vrf_update_onevrf: skipping: import not set

2019/12/26 13:05:08 BGP: : Rx route ADD VRF 7 ospf[0] 1.1.1.1/32 nexthop 12.1.1.1 (type 3 if 5) metric 100 tag 0

2019/12/26 13:05:08 BGP: : group_announce_route_walkcb: afi=IPv4, safi=vpn, p=1.1.1.1/32

2019/12/26 13:05:08 BGP: : subgroup_process_announce_selected: p=1.1.1.1/32, selected=0x557deca2fe90

2019/12/26 13:05:08 BGP: : u4:s4 send UPDATE w/ attr: nexthop 0.0.0.0, metric 100, extcommunity RT:12.1.1.2:2, originator 2.2.2.2, path

2019/12/26 13:05:08 BGP: : u4:s4 send MP_REACH for afi/safi 1/128

2019/12/26 13:05:08 BGP: : u4:s4 send UPDATE RD 12.1.1.2:2 1.1.1.1/32 label 154 IPv4 vpn

2019/12/26 13:05:08 BGP: : u4:s4 send UPDATE len 92 numpfx 1

2019/12/26 13:05:08 BGP: : u4:s4 10.10.10.10 send UPDATE w/ nexthop 20.20.20.20 and RD

2019/12/26 13:05:08 BGP: : 10.10.10.10 rcvd UPDATE w/ attr: , origin ?, extcommunity RT:12.1.1.2:2, path 100 200

2019/12/26 13:05:08 BGP: : 10.10.10.10 rcvd UPDATE wlen 0 attrlen 66 alen 0

2019/12/26 13:05:08 BGP: : 10.10.10.10 rcvd UPDATE about RD 12.1.1.2:2 1.1.1.1/32 label 154 IPv4 vpn -- DENIED due to: as-path contains our own AS;
```

Bgp端的函数`subgroup_update_packet`构造的update报文发送

```c
/* Make BGP update packet.  */
struct bpacket *subgroup_update_packet(struct update_subgroup *subgrp)
{
	struct bpacket_attr_vec_arr vecarr;
	struct bpacket *pkt;
	struct peer *peer;
	struct stream *s;
	struct stream *snlri;
	struct stream *packet;
	struct bgp_adj_out *adj;
	struct bgp_advertise *adv;
	struct bgp_dest *dest = NULL;
	struct bgp_path_info *path = NULL;
	bgp_size_t total_attr_len = 0;
	unsigned long attrlen_pos = 0;
	size_t mpattrlen_pos = 0;
	size_t mpattr_pos = 0;
	afi_t afi;
	safi_t safi;
	int space_remaining = 0;
	int space_needed = 0;
	char send_attr_str[BUFSIZ];
	int send_attr_printed = 0;
	int num_pfx = 0;
	bool addpath_capable = false;
	int addpath_overhead = 0;
	uint32_t addpath_tx_id = 0;
	struct prefix_rd *prd = NULL;
	mpls_label_t label = MPLS_INVALID_LABEL, *label_pnt = NULL;
	uint32_t num_labels = 0;
```
{: file='bgpd/bgp_updgrp_packet.c'}

发送的时候如果报文里面nexthop属性为0的话，bpacket_reformat_for_peer 这个函数会改nexthop的IP地址

****

本文参考

> 1. [BGP官方文档](https://docs.frrouting.org/en/latest/bgp.html)
> 2. [FRR BGP 协议分析](https://blog.csdn.net/armlinuxww/article/details/103264932)