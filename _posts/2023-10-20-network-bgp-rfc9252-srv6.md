---
title: RFC 9252：基于 IPv6 分段路由 (SRv6) 的 BGP 覆盖服务
author: fnoobt
date: 2023-10-20 19:29:00 +0800
categories: [Network,路由协议]
tags: [network,bgp]
---

## 摘要

本文定义了基于SRv6的BGP服务的过程和消息，包括3层虚拟专用网络（L3VPN）、以太网VPN（EVPN）和互联网服务。它建立在“BGP/MPLS IP虚拟专用网络（VPNs）”（RFC 4364）和“基于BGP的MPLS以太网VPN”（RFC 7432）的基础上。

## 本备忘录的状态

本文是Internet Standards Track文档。

本文是互联网工程任务组（IETF）的成果。它代表了IETF社区的共识。它已经接受了公共审查，并已经由互联网工程指导组（IESG）批准发表。有关互联网标准的更多信息，请参阅RFC 7841的第2节。

关于本文档的当前状态、任何勘误以及如何提供反馈的信息，请访问<https://www.rfc-editor.org/info/rfc9252>。

## 版权声明

版权所有（c）2022 IETF Trust 和文档作者。 版权所有。

本文件受本文件发布之日生效的 [BCP 78](https://datatracker.ietf.org/doc/html/bcp78) 和 [IETF 信托基金有关 IETF 文件的法律规定](http://trustee.ietf.org/license-info) 的约束。请仔细阅读这些文件，因为它们描述了您与本文件相关的权利和限制。从本文档中提取的代码组件必须包括信托法律条款第 4.e 节中所述的简化 BSD 许可证文本，并且不提供简化 BSD 许可证中所述的保证。

## 1. 简介

SRv6 (Segment Routing over IPv6 Services)指的是在 IPv6 数据平面上实例化的分段路由 [RFC8402]。

BGP 用于通告特定服务的前缀从出口提供商边缘 (PE) 到入口 PE 节点的可达性。

基于SRv6的BGP业务是指以BGP为控制平面、SRv6为数据平面的三层和二层叠加业务。 本文档定义了基于 SRv6 的 BGP 服务的流程和消息，包括 L3VPN、EVPN 和 Internet 服务。 它基于[RFC4364：BGP/MPLS IP 虚拟专用网络 (VPN)](https://datatracker.ietf.org/doc/html/rfc4364) 和[RFC7432: 基于 BGP MPLS 的以太网 VPN](https://datatracker.ietf.org/doc/html/rfc7432) 的基础上。

SRv6 SID (Segment Identifier) 是指 SRv6 分段标识符，如 [RFC8402] 中定义。

SRv6 服务 SID 是指与广告 PE 路由器上特定于服务的 SRv6 端点行为之一关联的 SRv6 SID，例如（但不限于）End.DT（在虚拟路由和转发 (VRF) 表中查找） 或 L3VPN 服务情况下的 End.DX（交叉连接到下一跳）行为，如 [RFC8986] 中定义。 本文档描述了PE之间现有的BGP消息如何携带SRv6服务SID来互连PE并形成VPN。

为了向 SRv6 服务提供尽力而为的连接，出口 PE 使用 BGP 覆盖服务路由发送 SRv6 服务 SID 信号。 入口 PE 将有效负载封装在外部 IPv6 标头中，其中目标地址是出口 PE 提供的 SRv6 服务 SID。 PE 之间的底层仅需要支持普通 IPv6 转发 [RFC8200]。

为了结合从入口 PE 到出口 PE 的底层服务级别协议 (SLA) 提供 SRv6 服务，出口 PE 使用颜色扩展社区 [RFC9012] 对覆盖服务路由进行着色，以引导这些路由的流量，如 [段路由策略] 第 8 节。 入口 PE 将有效负载数据包封装在外部 IPv6 标头中，其中包含与相关 SLA 关联的 SR 策略段列表以及与使用分段路由标头 (Segment Routing Header， SRH) [RFC8754] 的路由关联的 SRv6 服务 SID。 SRv6 SID 属于 SRH 段列表一部分的底层节点必须支持 SRv6 数据平面。

### 1.1. 要求语言

本文档中的关键词“必须”、“不得”、“需要”、“应”、“不应”、“应该”、“不应该”、“推荐”、“不推荐”、“可以”和“可选”当且仅当它们以全部大写形式出现时按照 应按照 BCP 14 [RFC2119] [RFC8174] 中的描述进行解释，如此处所示。("MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL")

本文档不是协议规范，本文档中的关键词是为了清晰和强调需求语言而使用的。

## 2. SRv6 Services TLV
本文档扩展了 BGP Prefix-SID 属性 [RFC8669] 的使用，以携带 SRv6 SID 及其与本节中后面列出的 BGP 地址族相关的信息。

SRv6 Services TLV (Segment Routing over IPv6 Services Type-Length-Value，基于 IPv6 服务的分段路由类型-长度-值) 被定义为 BGP Prefix-SID 属性的两个新 TLV，以实现 L3 和 L2 服务的 SRv6 SIDs 信令。

SRv6 L3 Services TLV：
: 该 TLV 编码了 SRv6 基础 L3 服务的 Service SID 信息。 它对应于通过 L3 服务路由接收时 MPLS 标签提供的等效功能，如 [RFC4364]、[RFC4659]、[RFC8950] 和 [RFC9136] 中所定义。 可以编码的一些 SRv6 端点行为包括但不限于 End.DX4、End.DT4、End.DX6、End.DT6 和 End.DT46。

SRv6 L2 Services TLV：
: 该 TLV 编码了 SRv6 基础 L2 服务的 Service SID 信息。 它对应于 L2 服务的以太网 VPN (EVPN) 路由类型的 MPLS 标签提供的等效功能，如 [RFC7432] 中所定义。 可以编码的一些 SRv6 端点行为包括但不限于 End.DX2、End.DX2V、End.DT2U 和 End.DT2M。

当出口 PE 通过 SRv6 数据平面启用 BGP 服务时，它会附加到多协议 BGP (MP-BGP) 网络层可达性信息 (Network Layer Reachability Information，NLRI)中的 BGP Prefix-SID 属性内发送包含在 SRv6 Services TLV(s) 中的一个或多个 SRv6 Services SIDs 信号。此处适用于 [RFC4760]、[RFC4659]、[RFC8950]、[RFC7432]、[RFC4364] 和 [RFC9136] 中描述的情形，详见第 5 节和第 6 节中。

本文不涵盖对 SRv6 的 BGP 多播 VPN (MVPN) 服务 [RFC6513] 的支持。

下面是 BGP Prefix-SID 属性中 SRv6 Services TLVs 的编码示例：

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|   TLV Type    |         TLV Length            |   RESERVED    |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|   SRv6 Service Sub-TLVs                                      //
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```
_图1: SRv6 Service TLVs_

TLV Type （1 个八位字节）：
: 该字段用于标识TLV的类型，其值从 IANA 的“BGP Prefix-SID TLV Types”子注册表中获得。 对于 SRv6 L3 Service TLV，其值设置为5。 对于 SRv6 L2 Service TLV，其值设置为 6。

TLV Length （2 个八位字节）：
: 该字段指定 TLV 值的总长度，以字节为单位。

RESERVED（1 个八位字节）：
: 该字段为保留字段，发送方必须将其设置为0，接收方应该忽略。

SRv6 Service Sub-TLVs （可变长度）：
: 该字段包含 SRv6 服务相关信息，并被编码为 Sub-TLV 的无序列表，其格式如下所述。  
- Sub-TLV Type（1个八位字节）： 用于标识Sub-TLV类型的字段。
- Sub-TLV Length（2个八位字节）： 指定Sub-TLV值的总长度，以字节为单位。
- Sub-TLV Value（可变长度）： 包含具体的SRv6服务信息。

当 BGP 设备在接收到一个或多个具有 BGP Prefix-SID 属性的SRv6 Service TLV 路由时，BGP 发言者在向其他对等体通告该路由时遵循以下规则：

- 如果 BGP 下一跳在通告期间未更改，则应进一步传播 SRv6 Service TLV，包括任何无法识别的 Sub-TLV 和 Sub-Sub-TLV 类型。 此外，TLV、Sub-TLV 或 Sub-Sub-TLV 中的所有保留字段必须按原样传播。
- 如果 BGP 下一跳发生更改，则应使用本地分配的 SRv6 SID 信息更新 TLV、Sub-TLV 和 Sub-Sub-TLV。 必须删除任何收到的 Sub-TLV 和无法识别的 Sub-Sub-TLV。

## 3. SRv6 Service Sub-TLVs
单个 SRv6 Service Sub-TLVs 的格式如下所示：

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| SRv6 Service  |    SRv6 Service               | SRv6 Service //
| Sub-TLV       |    Sub-TLV                    | Sub-TLV      //
| Type          |    Length                     | Value        //
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```
_图2: SRv6 Service Sub-TLVs_

SRv6 Service Sub-TLV Type（1 个八位字节）：
: 该字段标识SRv6服务信息的类型。 其值从 IANA 的“SRv6 Service Sub-TLV 类型”子注册表中分配。

SRv6 Service Sub-TLV Length（2 个八位字节）：
: 该字段指定子 TLV 值字段的总长度（以字节为单位）。

SRv6 Service Sub-TLV Value（可变长度）：
: 该字段包含特定于 Sub-TLV 类型的数据。 除了固定长度数据之外，它还包含编码为一组 SRv6 Service Data Sub-Sub-TLVs 的 SRv6 服务的其他属性，其格式在下面第 3.2 节中描述。

### 3.1. SRv6 SID Information Sub-TLV
SRv6 Service Sub-TLV 类型 1 分配给 SRv6 SID Information Sub-TLV。 该 Sub-TLV 包含一个单独的 SRv6 SID 及其属性。 其编码如下所示：

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| SRv6 Service  |    SRv6 Service               |               |
| Sub-TLV       |    Sub-TLV                    |               |
| Type=1        |    Length                     |  RESERVED1    |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  SRv6 SID Value (16 octets)                                  //
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Svc SID Flags |   SRv6 Endpoint Behavior      |   RESERVED2   |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  SRv6 Service Data Sub-Sub-TLVs                              //
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```
_图3: SRv6 SID Information Sub-TLV_

SRv6 Service Sub-TLV Type（1 个八位字节）：
: 该字段设置为 1 以表示 SRv6 SID 信息子 TLV。

SRv6 Service Sub-TLV Length（2 个八位字节）：
: 该字段包含 Sub-TLV 值字段的总长度，以字节为单位。

RESERVED 1（1 个八位字节）：
: 保留字段，该字段必须由发送方设置为 0，并被接收方忽略。

SRv6 SID 值（16 个八位字节）：
: 该字段对 SRv6 SID 进行编码，如 [RFC8986](https://www.rfc-editor.org/rfc/rfc9252.html#RFC8986) 中定义。

SRv6 Service SID 标志（1 个八位字节）：
: 该字段对 SRv6 服务 SID 标志进行编码——当前尚未定义。 发送方必须将其设置为 0，并且接收方必须忽略任何未知标志。

SRv6 端点行为（2 个八位字节）：
: 该字段对与 SRv6 SID 关联的 SRv6 端点行为代码点值进行编码。 使用的代码点来自 IANA 的“Segment Routing”注册表下的 “SRv6 Endpoint Behaviours”子注册表，该注册表由 [RFC8986] 引入。 当通告路由器希望抽象其本地实例化的 SRv6 SID 的实际行为时，可以使用不透明的 SRv6 端点行为（即值 0xFFFF）。

RESERVED2（1 个八位字节）：
: 该字段必须由发送方设置为 0，并被接收方忽略。

SRv6 Service Data Sub-Sub-TLV 值（可变长度）：
: 该字段用于通告SRv6 SID的属性。 它被编码为一组 SRv6 服务数据子子 TLV。

SRv6 SID 的 SRv6 端点行为的选择完全取决于通告的发起者。 虽然第 5 节和第 6 节列出了特定路由通告通常预期使用的 SRv6 端点行为，但其他 SRv6 端点行为（例如，将来可能引入的新行为）的接收不被视为错误。 接收方不得将无法识别的 SRv6 端点行为视为无效，除非涉及使用参数的行为（有关参数验证的详细信息，请参阅第 3.2.1 节）。 当收到意外行为时，实现可以记录速率受限的警告。

当存在多个 SRv6 SID 信息Sub-TLV 时，入口 PE 应使用来自子 TLV 的第一个实例的 SRv6 SID。 实现可以提供本地策略来覆盖此选择。

### 3.2. SRv6 Service Data Sub-Sub-TLVs
SRv6 Service Data Sub-Sub-TLV 的格式如下所示：

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Service Data |  Sub-Sub-TLV Length               |Sub-Sub TLV //
| Sub-Sub-TLV  |                                   |  Value     //
| Type         |                                   |            //
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

_图4: SRv6 Service Data Sub-Sub-TLVs_

SRv6 Service Data Sub-Sub-TLV Type（1 个八位字节）：
: 该字段标识 Sub-Sub-TLV 的类型。 其值从 IANA 的“SRv6 Service Data Sub-Sub-TLV Types”子注册表中获得。

SRv6 Service Data Sub-Sub-TLV Length（2 个八位字节）：
: 该字段指定 Sub-Sub-TLV 值字段的总长度（以八位字节为单位）。

SRv6 Service Data Sub-Sub-TLV Value（变量）：
: 该字段包含特定于 Sub-Sub-TLV 类型的数据。

### 3.2.1. SRv6 SID Structure Sub-Sub-TLV
SRv6 SID Structure Sub-Sub-TLV 类型 1 被分配给 SRv6 SID Structure Sub-Sub-TLV。 SRv6 SID Structure Sub-Sub-TLV 用于通告 SRv6 SID 各个部分的长度，如 [RFC8986] 中定义。 术语“Locator Block”和“Locator Node”分别对应于 [RFC8986] 第 3.1 节中定义的 SRv6 定位器的 B 部分和 N 部分。 它作为 SRv6 SID Information  Sub-TLV 中的 Sub-Sub-TLV 进行传递。

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| SRv6 Service  |    SRv6 Service               | Locator Block |
| Data Sub-Sub  |    Data Sub-Sub-TLV           | Length        |
| -TLV Type=1   |    Length                     |               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Locator Node  | Function      | Argument      | Transposition |
| Length        | Length        | Length        | Length        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Transposition |
| Offset        |
+-+-+-+-+-+-+-+-+
```

_图5: SRv6 SID Structure Sub-Sub-TLV_

SRv6 Service Data Sub-Sub-TLV 类型（1 个八位字节）：
: 该字段设置为 1 以表示 SRv6 SID Structure Sub-Sub-TLV。

SRv6 Service Data Sub-Sub-TLV 长度（2 个八位字节）：
: 该字段包含 6 个八位字节的总长度。

Locator Block长度（1 个八位字节）：
: 该字段包含 SRv6 SID 定位器块的长度（以位为单位）。

Locator Node长度（1 个八位字节）：
: 该字段包含 SRv6 SID 定位节点的长度（以位为单位）。

Function 长度（1 个八位字节）：
: 该字段包含 SRv6 SID Function 的长度（以位为单位）。

Argument 长度（1 个八位字节）：
: 该字段包含 SRv6 SID 参数的长度（以位为单位）。

Transpose 长度（1 个八位字节）：
: 转置长度，该字段是已转置（或移位）到 MPLS 标签字段中的 SID 部分的大小（以位为单位）。

Transpose 偏移（1 个八位字节）：
: 转置偏移量，该字段是已转置（或移位）到 MPLS 标签字段中的 SID 部分的偏移位置（以位为单位）。

第 4 节描述了通过调换 SRv6 SID 值的可变部分并在现有 MPLS 标签字段中携带该可变部分来发送 SRv6 Service SID 的机制，以在 BGP 更新消息中更有效地打包这些服务前缀 NLRI。 当 SRv6 Service SID 被分成多个部分时，SRv6 SID Structure Sub-Sub-TLV 包含适当的长度字段，以使接收器能够准确地将 SID 组合在一起。

转置偏移量指示比特位置，转置长度指示从 SRv6 SID 值中取出并在 MPLS 标签字段中编码的比特数。 SID 值中已移出的位必须设置为 0。

转置长度 为 0 表示没有进行转置，并且整个 SRv6 SID 值都编码在 SID Information Sub-TLV 中。 在这种情况下，转置偏移必须设置为 0。

MPLS 标签字段的大小限制从 SRv6 SID 值转入其中的位数。 例如，在[RFC4364]和[RFC8277]中MPLS标签字段的大小为20比特，在[RFC7432]中MPLS标签字段的大小为24比特。

根据 [RFC8986] 中的定义，定位器块长度 (Locator Block Length，LBL)、定位器节点长度 (Locator Node Length，LNL)、函数长度 (Function Length，FL) 和参数长度 (Argument Length，AL) 字段的总和必须小于或等于 128 且大于转置偏移量和转置长度之和。

例如，假设定位器块和定位器节点部分的总和为 64。对于整个 Function 部分大小为16位的SRv6 SID，转置偏移设置为 64，转置长度设置为 16. 而对于 FL 为 24 位且仅低序 20 位进行转置的 SRv6 SID（例如，由于 MPLS 标签字段大小的限制），转置偏移设置为 68，转置长度设置为 20。

不支持此规范的 BGP 发言者可能会在接收到基于 SRv6 的 BGP 服务路由更新时，将 MPLS 标签字段中编码的 SRv6 SID 部分误解为基于 MPLS 的服务的 MPLS 标签值。 支持此规范的实现必须提供一种机制来控制基于每个邻居和每个服务的基于 SRv6 的 BGP 服务路由的通告。 部署设计和实施选项的详细信息不在本文档的范围内。

对于特定的 SRv6 端点行为的 SID（例如 End.DT2M），参数可能是通用的； 因此，对于参数不适用的 SID，AL 必须设置为 0。 接收方无法验证其未知的 SRv6 端点行为参数的适用性，因此必须忽略具有未知 SRv6 端点行为的参数（由非零 AL 指示）的 SRv6 SID。 对于与已知 SRv6 端点行为对应的 SID，接收方必须验证 AL 与特定 SRv6 端点行为定义的一致性。

## 4. 编码 SRv6 SID 信息

BGP 服务前缀的 SRv6 Service SID 包含在 BGP Prefix-SID 属性的 SRv6 Service TLV 中。

对于某些类型的 BGP 服务，例如使用每 VRF SID 分配的 L3VPN（即 End.DT4 或 End.DT6 行为），相同的 SID 在多个 NLRI 之间共享，从而提供高效的打包。 然而，对于某些其他类型的 BGP 服务，例如需要按 PW SID 分配的 EVPN 虚拟专线服务 (VPWS)（即 End.DX2 行为），每个 NLRI 将拥有自己唯一的 SID，从而导致效率低下的打包。

为了实现高效打包，本文档允许以下两种方式之一：
1. 在 SRv6 Service TLV 中将 SRv6 Service SID 作为一个整体进行编码
2. 在 SRv6 Service TLV 中仅对 SRv6 SID 的公共部分（例如，定位器）进行编码，并将变量部分（例如，Function或Argument部分）编码在特定于该服务编码的现有标签字段中。

第二种编码形式称为转置方案，其中 SRv6 SID Structure Sub-Sub-TLV 描述了 SRv6 SID 各部分的大小，并且还指示了变量部分的偏移量及其在 SRv6 SID 值中的长度。 对于允许使用转置方案的特定服务编码，建议使用转置方案，如第 5 节和第 6 节中进一步所述的一样。

例如，对于 6.1.2 节中进一步描述的 EVPN VPWS 服务前缀，SRv6 SID 的 Function 部分编码在 NLRI 的 MPLS 标签字段中，SRv6 Services TLV 中的 SID 值仅携带了 SRv6 SID Structure Sub-Sub-TLV 的 Locator 部分。 SRv6 SID Structure Sub-Sub-TLV 定义了Locator Block，Locator Node和 Function 部分的长度（Arguments不适用于 End.DX2 行为）。 Transposition Offset 表示比特位置，Transposition Length 表示从 SID 中取出并放入标签字段的比特数。

在另一个例子中，对于第6.1.1节中进一步描述的每个以太网段（ES）路由的EVPN以太网自动发现（A-D），仅需要用信号发送SID的Argument部分。 SRv6 SID 的此Argument部分可以在 ESI Label扩展团体的以太网段标识符 (ESI) 标签字段中转置，并且 SRv6 Service TLV 中的 SID 值设置为 0，同时包含 SRv6 SID Structure Sub-Sub-TLV。 SRv6 SID Structure Sub-Sub-TLV 定义了Locator Node、Locator Node、Locator Node和Argument部分的长度。 将Argument部分SID值移动到标签字段的转置方案中的偏移量和长度在SID Structure TLV的转置偏移量和长度中设置。 然后，接收路由器能够将整个 SRv6 Service SID 组合在一起（例如，对于EVPN路由类型3值接收的 End.DT2M 行为），放入每个 ES 路由的以太网 A-D 的 ESI Label字段中接收的标签值中，该值位于SRv6 SID中的正确转置偏移和长度中。

## 5. 基于 SRv6 的 BGP L3 服务
BGP 出口节点（出口 PE）通告一组可达前缀。 标准 BGP 更新传播方案 [RFC4271] 可以利用路由反射器 [RFC4456]，用于传播这些前缀。 BGP 入口节点（入口 PE）接收这些通告，并可以将前缀添加到适当 VRF 中的 RIB 中。

支持基于 SRv6 的 L3 服务的出口 PE 通告覆盖服务前缀以及包含在 BGP Prefix-SID 属性内的 SRv6 L3 服务 TLV 中的服务 SID。 该 TLV 有两个目的：首先，它表明出口 PE 支持 SRv6 覆盖，接收该路由的 BGP 入口 PE 必须执行 IPv6 封装并在需要时插入 SRH [RFC8754]； 其次，它指示要在封装中使用的 Service SID 的值。

因此，只有在出口PE处，用信号发送的服务SID才具有本地意义，其中可以在每个客户边缘（CE）或每个VRF的基础上对其进行分配或配置。 实际上，SID 可以将交叉连接编码到特定地址族表 (End.DT) 或下一跳/接口 (End.DX)，如 [RFC8986] 中所定义。

SRv6 Service SID 应在出口 PE 的自治系统 (AS) 内是可路由的（请参阅 [RFC8986] 的第 3.3 节），并具有双重目的：提供入口 PE 和出口 PE 之间的可达性，同时还对 SRv6 端点行为进行编码。

当 SRv6 服务的转向基于到出口 PE 的最短路径转发（例如，尽力而为或 IGP 灵活算法 [IGP-FLEX-ALGO]）时，入口 PE 将 IPv4 或 IPv6 客户数据包封装在外部 IPv6 标头中（使用 [RFC8986] 中指定的 H.Encaps 或 H.Encaps.Red 风格），其中目标地址是与相关 BGP 路由更新关联的 SRv6 Service SID。 因此，在考虑将接收到的用于 BGP 最佳路径计算之前，入口 PE 必须对 SRv6 Service SID 执行可解析性检查。 可解析性按照 [RFC4271] 进行评估。 如果可以通过多个转发表访问 SRv6 SID，则使用本地策略来确定使用哪个表。 如果入口 PE 具有允许备用转向机制到达出口 PE 的本地策略，则可以忽略 SRv6 Service SID 可解析性的结果（例如，当通过 IGP 灵活算法提供时）。 此类转向机制的详细信息不在本文档的讨论范围之内。

对于 SRv6 核心上的服务，出口 PE 将 BGP 下一跳设置为其 IPv6 地址之一。 这样的地址可以被分配 SRv6 服务 SID 的 SRv6 定位器覆盖。 根据现有的BGP程序，BGP 下一跳用于跟踪出口 PE 的可达性。

当入口 PE 接收到的 BGP 路由使用颜色扩展社区进行着色并且有效的 SRv6 策略可用时，将执行服务流的转向，如 [SEGMENT-ROUTING-POLICY] 第 8 节中的所述。 当入口 PE 确定（借助 SRv6 SID Structure 的帮助）Service SID 与 SR 策略段列表中（出口 PE 的）最后一个 SRv6 SID 属于同一 SRv6 定位器时，可以在在导向服务流时排除该最后一个 SRv6 SID。 例如，与 SID 列表 \< S1, S2, S3 > 关联的 SRv6 策略的有效段列表将为 \< S1, S2, S3-Service-SID >。

### 5.1. 基于 SRv6 核心的 IPv4 VPN
SRv6 核心上的 MP_REACH_NLRI 根据 [RFC8950] 中定义的 IPv6 核心上的 IPv4 VPN 单播进行编码。

IPv4-VPN NLRI 的标签字段按照 [RFC8277] 中的规定进行编码，当使用编码转置方案（第 4 节 Transposition Schem）时，其中20 位标签值设置为 SRv6 SID 的全部或部分Function部分 ; 否则，它被设置为隐式 NULL。 使用转置方案时，转置长度必须小于或等于 20 且小于或等于 FL。

SRv6 服务 SID 被编码为 SRv6 L3 服务 TLV 的一部分。 SRv6 端点行为应该是以下之一：End.DX4、End.DT4 或 End.DT46。

### 5.2. 基于 SRv6 核心的 IPv6 VPN
SRv6 核心上的 MP_REACH_NLRI 根据 IPv6 核心上的 IPv6 VPN 进行编码，如 [RFC4659] 中所定义。

IPv6-VPN NLRI 的标签字段按照 [RFC8277] 中的规定进行编码，当使用编码转置方案（第 4 节）时，其中20 位标签值设置为 SRv6 SID 的全部或部分Function部分; 否则，它被设置为隐式 NULL。 使用转置方案时，转置长度必须小于或等于 20 且小于或等于 FL。

SRv6 服务 SID 被编码为 SRv6 L3 服务 TLV 的一部分。 SRv6 端点行为应该是以下之一：End.DX6、End.DT6 或 End.DT46。

### 5.3. Global IPv4 over SRv6 Core
SRv6 核心上的 MP_REACH_NLRI 根据 IPv6核心上的 IPv4 进行编码，如 [RFC8950] 中所定义。

SRv6 Service SID 被编码为 SRv6 L3 Service TLV 的一部分。 SRv6 端点行为应该是以下之一：End.DX4、End.DT4 或 End.DT46。

### 5.4. Global IPv6 over SRv6 Core
SRv6 核心上的 MP_REACH_NLRI 的编码遵循 [RFC2545] 。

SRv6 Service SID 被编码为 SRv6 L3 Service TLV 的一部分。 SRv6 端点行为应该是以下之一：End.DX6、End.DT6 或 End.DT46。

## 6. 基于 SRv6 的BGP以太网VPN（EVPN）
[RFC7432] 提供了构建 EVPN 覆盖网络的可扩展方法。 它主要关注基于 MPLS 的 EVPN，并且[RFC8365] 将其扩展到基于 IP 的 EVPN 覆盖。 [RFC7432]定义了路由类型1、2和3，它们携带前缀和MPLS标签字段； 标签字段对于 EVPN 流量的 MPLS 封装有特定用途。 [RFC9136] 中定义了携带 MPLS 标签信息（以及封装信息）的 EVPN 的路由类型 5。 路由类型 6、7 和 8 在 [RFC9251] 中定义。

- 以太网自动发现 (A-D) 路由（路由类型 1）
- MAC/IP 通告路由（路由类型 2）
- 包容性多播以太网标记路由（路由类型 3）
- 以太网段路由（路由类型 4）
- IP 前缀路由（路由类型 5）
- 选择性多播以太网标记路由（路由类型 6）
- 多播成员报告同步路由（路由类型 7）
- 多播离开同步路由（路由类型 8）

其他 EVPN 路由类型的规范不在本文档的讨论范围内。

为了支持基于 SRv6 的 EVPN 覆盖，将使用路由类型 1、2、3 和 5 通告一个或多个 SRv6 Service SID。每种路由类型的 SRv6 Service SID 在 BGP Prefix-SID属性中的 SRv6 L3/L2 Service TLV 中进行通告。 SRv6 Service SID 的信令有两个目的 -- 首先，它表明 BGP 出口设备支持 SRv6 覆盖，并且接收此路由的 BGP 入口设备必须执行 IPv6 封装并在需要时插入 SRH [RFC8754]； 其次，它指示封装中要使用的 Service SID 的值。

SRv6 Service SID 应在出口 PE 的 AS 内是可路由的（请参阅 [RFC8986] 的第 3.3 节），并具有双重目的：提供入口 PE 和出口 PE 之间的可达性，同时还对 SRv6 端点行为进行编码。

当 SRv6 Service的转向基于到出口 PE 的最短路径转发（例如，尽力而为或 IGP 灵活算法 [IGP-FLEX-ALGO]）时，入口 PE 将客户第 2 层以太网数据包封装在外部 IPv6 标头中（使用 [RFC8986] 中指定的 H.Encaps.L2 或 H.Encaps.L2.Red 风格），其中目标地址是与相关 BGP 路由更新关联的 SRv6 服务 SID。 因此，入口 PE 必须在考虑接收到用于 BGP 最佳路径计算的前缀之前对 SRv6 Service SID 执行可解析性检查。 可解析性按照 [RFC4271] 进行评估。 如果可以通过多个转发表访问 SRv6 SID，则使用本地策略来确定使用哪个表。 如果入口 PE 具有允许备用转向机制到达出口 PE 的本地策略，则可以忽略 SRv6 服务 SID 可解析性的结果（例如，当通过 IGP 灵活算法提供时）。 此类转向机制的详细信息不在本文档的讨论范围内。

对于 SRv6 核心上的服务，出口 PE 将 BGP 下一跳设置为其 IPv6 地址之一。 这样的地址可以被分配 SRv6 服务 SID 的 SRv6 定位器覆盖。 BGP 下一跳用于基于现有 BGP 过程跟踪出口 PE 的可达性。

当入口 PE 接收到的 BGP 路由使用颜色扩展社区进行着色并且有效的 SRv6 策略可用时，服务流的转向将按照 [SEGMENT-ROUTING-POLICY] 第 8 节中的描述执行。 当入口 PE 确定（借助 SRv6 SID 结构的帮助）服务 SID 与 SR 策略段列表中（出口 PE 的）最后一个 SRv6 SID 属于同一 SRv6 定位器时，在转向服务流时，它可以排除该最后一个 SRv6 SID 。 例如，与 SID 列表 /< S1, S2, S3> 相关联的 SRv6 策略的有效段列表将为 /< S1, S2, S3-Service-SID>。

### 6.1. 基于 SRv6 核心的以太网自动发现路由
以太网自动发现(Auto-Discovery， A-D)路由是 [RFC7432] 中定义的路由类型 1，可用于实现水平分割过滤、快速收敛和混叠。 EVPN 路由类型 1也用于EVPN-VPWS以及EVPN-灵活交叉，主要用于通告点到点的业务ID。

提醒一下，EVPN 路由类型 1 的编码如下：

```
                +-----------------------------------------+
                |  RD (8 octets)                          |
                +-----------------------------------------+
                |  Ethernet Segment Identifier (10 octets)|
                +-----------------------------------------+
                |  Ethernet Tag ID (4 octets)             |
                +-----------------------------------------+
                |  MPLS label (3 octets)                  |
                +-----------------------------------------+
```

_图6: EVPN Route Type 1_

#### 6.1.1. 每个 ES 路由的以太网自动发现
SRv6 核心上的每 ES 路由 NLRI 编码的以太网 A-D 遵循 [RFC7432]。

当ESI过滤方法与编码转置（第4节）方案一起使用时，ESI标签扩展社区的24位ESI标签字段携带SRv6 SID的全部或部分Argument部分； 否则，它的高位 20 位将被设置为隐式 NULL（即 0x000030）。 在任何情况下，该值均设置为 24 位。 当使用转置方案时，转置长度必须小于或等于 24 且小于或等于 AL。

BGP Prefix-SID 属性内的 SRv6 L2 Service TLV 中包含的Service SID 与 A-D 路由一起通告。 SRv6 端点行为应该是 End.DT2M。 当使用 ESI 过滤方法时，Service SID 用于指示适用的 End.DT2M 行为的 Arg.FE2 SID 参数 [RFC8986]。 当使用本地偏置方法 [RFC8365] 时，Service SID 的值可以为 0。

#### 6.1.2. 每个 EVI 路由的以太网自动发现
SRv6 核心上的每个 EVPN 实例 (EVI) 路由 NLRI 编码的以太网 A-D 与 [RFC7432] 和 [RFC8214] 中描述的类似，但有以下更改：

MPLS Label：
: 当使用转置方案编码（第 4 节）时，24 位字段携带 SRv6 SID 的全部或部分Function部分； 否则，它的高位 20 位被设置为隐式 NULL（Implicit NULL，即 0x000030）。 在任何情况下，该值均设置为 24 位。 使用转置方案时，转置长度必须小于或等于 24 并且小于或等于 FL。

BGP Prefix-SID 属性内的 SRv6 L2 Service TLV 中包含的Service SID 与 A-D 路由一起通告。 SRv6 端点行为应该是以下之一：End.DX2、End.DX2V 或 End.DT2U。

### 6.2. 基于 SRv6 核心的 MAC/IP 通告路由
EVPN 路由类型 2 用于通过 MP-BGP 向给定 EVPN 实例中的所有其他 PE 通告单播流量介质访问控制 (Media Access Control， MAC) + IP 地址可达性。

提醒一下，EVPN 路由类型 2 的编码如下：

```
                +-----------------------------------------+
                |  RD (8 octets)                          |
                +-----------------------------------------+
                |  Ethernet Segment Identifier (10 octets)|
                +-----------------------------------------+
                |  Ethernet Tag ID (4 octets)             |
                +-----------------------------------------+
                |  MAC Address Length (1 octet)           |
                +-----------------------------------------+
                |  MAC Address (6 octets)                 |
                +-----------------------------------------+
                |  IP Address Length (1 octet)            |
                +-----------------------------------------+
                |  IP Address (0, 4, or 16 octets)        |
                +-----------------------------------------+
                |  MPLS Label1 (3 octets)                 |
                +-----------------------------------------+
                |  MPLS Label2 (0 or 3 octets)            |
                +-----------------------------------------+
```

_图7: EVPN Route Type 2_

SRv6 核心上的 NLRI 编码与 [RFC7432] 中描述的类似，但有以下更改：

MPLS Label 1：
: 这与 SRv6 L2 Service TLV 关联。 当使用编码转置方案（第 4 节）时，该 24 位字段携带 SRv6 SID 的全部或部分 Function 部分； 否则，它的高位 20 位被设置为隐式 NULL（即 0x000030）。 在任何情况下，该值均设置为 24 位。 使用转置方案时，转置长度必须小于或等于 24 且小于或等于 FL。

MPLS Label 2：
: 这与 SRv6 L3 Service TLV 关联。 当使用编码转置方案（第 4 节）时，该 24 位字段携带 SRv6 SID 的全部或部分 Function 部分； 否则，它的高位 20 位被设置为隐式 NULL（即 0x000030）。 在任何情况下，该值均设置为 24 位。 使用转置方案时，转置长度必须小于或等于 24 且小于或等于 FL。

在 BGP Prefix-SID 属性中的 SRv6 L2 Service TLV 和 SRv6 L3 Service TLV（可选的） 中封装的Service SID 与 MAC/IP 通告路由一起通告。

下面描述的是不同类型的路由类型 2 通告。

#### 6.2.1. 仅带有 MAC 的 MAC/IP 通告路由
MPLS Label 1：
: 这与 SRv6 L2 Service TLV 关联。 当使用编码转置方案（第 4 节）时，该 24 位字段携带 SRv6 SID 的全部或部分Function部分； 否则，它的高位 20 位被设置为隐式 NULL（即 0x000030）。 在任何情况下，该值均设置为 24 位。 使用转置方案时，转置长度必须小于或等于 24 且小于或等于 FL。

BGP Prefix-SID 属性内的 SRv6 L2 Service TLV 中包含的Service SID 与路由一起通告。 SRv6 端点行为应该是以下之一：End.DX2 或 End.DT2U。

#### 6.2.2. 带有 MAC/IP 的 MAC+IP 通告路由
MPLS Label 1：
: 这与 SRv6 L2 Service TLV 关联。 当使用编码转置方案（第 4 节）时，该 24 位字段携带 SRv6 SID 的全部或部分Function部分； 否则，它的高位 20 位被设置为隐式 NULL（即 0x000030）。 在任何情况下，该值均设置为 24 位。 使用转置方案时，转置长度必须小于或等于 24 且小于或等于 FL。

MPLS Label 2：
: 这与 SRv6 L3 Service TLV 关联。 当使用编码转置方案（第 4 节）时，该 24 位字段携带 SRv6 SID 的全部或部分Function部分； 否则，它的高位 20 位被设置为隐式 NULL（即 0x000030）。 无论哪种情况，该值均设置为 24 位。 使用转置方案时，转置长度必须小于或等于 24 且小于或等于 FL。

BGP Prefix-SID 属性内的 SRv6 L2 Service TLV 中封装的 L2 Service SID 与该路由一起通告。 此外，BGP Prefix-SID 属性内的 SRv6 L3 Service TLV 中封装的 L3 Service SID 也可以与该路由一起通告。 SRv6 端点行为应该是以下之一：对于 L2 Service SID，为 End.DX2 或 End.DT2U，对于 L3 Service SID，为 End.DT46、End.DT4、End.DT6、End.DX4 或 End.DX6 。

### 6.3. 基于 SRv6 核心的包容性多播以太网标记路由
EVPN 路由类型 3 用于通过 MP-BGP 向给定 EVPN 实例中的所有其他 PE 通告多播流量可达性信息。

提醒一下，EVPN 路由类型 3 的编码如下：

```
               +---------------------------------------+
               |  RD (8 octets)                        |
               +---------------------------------------+
               |  Ethernet Tag ID (4 octets)           |
               +---------------------------------------+
               |  IP Address Length (1 octet)          |
               +---------------------------------------+
               |  Originating Router's IP Address      |
               |          (4 or 16 octets)             |
               +---------------------------------------+
```

_图8: EVPN Route Type 3_

SRv6 核心上的 NLRI 编码与 [RFC7432] 中描述的类似。

P-多播服务接口 (P-Multicast Service Interface, PMSI) 隧道属性 [RFC6514] 用于标识用于发送广播、未知单播或多播 (BUM) 流量的提供商隧道 (P-tunnel)。 PMSI 隧道属性的格式在 SRv6 核心上编码如下：

```
               +---------------------------------------+
               |  Flag (1 octet)                       |
               +---------------------------------------+
               |  Tunnel Type (1 octet)                |
               +---------------------------------------+
               |  MPLS label (3 octets)                 |
               +---------------------------------------+
               |  Tunnel Identifier (variable)         |
               +---------------------------------------+
```

_图9：PMSI 隧道属性_

Flag：
: 根据 [RFC7432] 的定义，该字段的值为 0。

Tunnel Type：
: 该字段是根据 [RFC6514] 定义的。

MPLS Label：
: 当使用入口复制并使用编码转置方案（第 4 节）时，此 24 位字段携带 SRv6 SID 的全部或部分Function部分； 否则，它按照[RFC6514]中的定义进行设置。 使用转置方案时，转置长度必须小于或等于 24 且小于或等于 FL。

Tunnel Identifier：
: 隧道标识符，该字段为出口PE的IP地址。

BGP Prefix-SID 属性内的 SRv6 L2 Service TLV 中包含的Service SID 与路由一起通告。 SRv6 端点行为应该是 End.DT2M。

- 当基于 ESI 的过滤用于多宿主或以太网树 (E-Tree) 过程时，与 EVPN 路由类型 1 一起携带的Service SID 的 ESI 过滤参数（[RFC8986] 中引入的 Arg.FE2 表示法）应通过执行按位逻辑或运算在入口PE上创建单个SID，与远程 PE 通告的路由类型3的适用 End.DT2M SID 合并。 [RFC7432] 中描述了多宿主的水平分割、基于 ESI 的过滤机制的详细信息。 [RFC8317] 中提供了 EVPN E-Tree 服务中源自叶子的 BUM 流量过滤机制的详细信息。
- 当“局部偏置（local-bias）”用作多宿主水平分割方法时，ESI 过滤参数不应与入口 PE 上相应的 End.DT2M SID 合并。 有关局部偏置过程的详细信息在[RFC8365]中描述。

使用多播树作为 P 隧道不在本文档的讨论范围内。

### 6.4. 基于 SRv6 核心的以太网段路由
提醒一下，以太网网段路由（即 EVPN 路由类型 4）编码如下：

```
               +---------------------------------------+
               |  RD (8 octets)                        |
               +---------------------------------------+
               |  Ethernet Tag ID (4 octets)           |
               +---------------------------------------+
               |  IP Address Length (1 octet)          |
               +---------------------------------------+
               |  Originating Router's IP Address      |
               |          (4 or 16 octets)             |
               +---------------------------------------+
```

_图10: EVPN Route Type 4_

SRv6 核心上的 NLRI 编码与 [RFC7432] 中描述的类似。

BGP Prefix-SID 属性内的 SRv6 类似于 TLV 不会随此路由一起通告。 路由的处理没有改变 -- 它仍然如[RFC7432]中所描述的那样。

### 6.5. 基于 SRv6 核心的 IP 前缀路由
EVPN 路由类型 5 用于通过 MP-BGP 向给定 EVPN 实例中的所有其他 PE 通告 IP 地址可达性。 IP 地址可以包括主机 IP 前缀或任何特定子网。

提醒一下，EVPN 路由类型 5 的编码如下：

```
               +-----------------------------------------+
               |  RD (8 octets)                          |
               +-----------------------------------------+
               |  Ethernet Segment Identifier (10 octets)|
               +-----------------------------------------+
               |  Ethernet Tag ID (4 octets)             |
               +-----------------------------------------+
               |  IP Prefix Length (1 octet)             |
               +-----------------------------------------+
               |  IP Prefix (4 or 16 octets)             |
               +-----------------------------------------+
               |  GW IP Address (4 or 16 octets)         |
               +-----------------------------------------+
               |  MPLS Label (3 octets)                  |
               +-----------------------------------------+
```
_图11: EVPN Route Type 5_

SRv6 核心上的 NLRI 编码与 [RFC9136] 中描述的类似，但有以下更改：

MPLS Label：
: 当使用编码转置方案（第 4 节）时，该 24 位字段携带 SRv6 SID 的全部或部分 Function 部分； 否则，它的高位 20 位被设置为隐式 NULL（即 0x000030）。 无论哪种情况，该值均设置为 24 位。 使用转置方案时，转置长度必须小于或等于 24 且小于或等于 FL。

SRv6 Service SID 被编码为 SRv6 L3 Service TLV 的一部分。 SRv6 端点行为应该是以下之一：End.DT4、End.DT6、End.DT46、End.DX4 或 End.DX6。

### 6.6. 基于 SRv6 核心的 EVPN 多播路由（路由类型 6、7 和 8）
这些路由不需要随 SRv6 Service TLV 一起通告。 与EVPN路由类型4类似，BGP下一跳等于出口PE的IPv6地址。

## 7. 错误处理
如果在处理 SRv6 Service TLV 时遇到任何错误，应记录错误的详细信息以供进一步分析。

如果遇到多个 SRv6 L3 Service TLV 的实例，则必须忽略除第一个实例之外的所有实例。

如果遇到多个 SRv6 L2 Service TLV 的实例，则必须忽略除第一个实例之外的所有实例。

在以下情况下，SRv6 Service TLV 被视为格式错误：

- TLV Length 小于 1。
- TLV Length 与BGP Prefix-SID 属性长度不一致。
- 至少其中一个 Sub-TLV 组成部分格式错误。

在以下情况下，SRv6 Service Sub-TLV 被视为格式错误：

- Sub-TLV 长度与封装的 SRv6 Service Sub-TLV 的长度不一致。

在以下情况下，SRv6 SID Information Sub-TLV 被视为格式错误：

- Sub-TLV 长度小于 21。
- Sub-TLV 长度与封装的 SRv6 Service TLV 的长度不一致。
- 至少其中一个组成的 Sub-Sub-TLV 格式错误。

在以下情况下，SRv6 Service Data Sub-Sub-TLV 被视为格式错误：

- Sub-Sub-TLV 长度与封装的 SRv6 service Sub-TLV 长度不一致。

任何 TLV、Sub-TLV 或 Sub-Sub-TLV 都不被视为格式错误，因为其类型无法识别。

任何 TLV、Sub-TLV 或 Sub-Sub-TLV 不会因为其值字段的任何语义验证失败而被视为格式错误。

SRv6 覆盖服务需要Service SID 进行转发。 当 BGP Prefix-SID 属性中至少存在一个格式错误的 SRv6 service TLV 时，必须执行视为撤回（treat-as-withdraw）操作 [RFC7606]。

当 SID Structure Sub-Sub-TLV 转置长度大于标签字段的位数时，或者不满足第 3.2.1 节中规定的 Sub-Sub-TLV 字段的任何条件，SRv6 SID Information Sub-TLV 中的 SRv6 SID 值无效。 当 Sub-Sub-TLV 与转置方案不适用的路由一起通告时（例如，对于没有标签字段的全局 IPv6 服务 [RFC2545]），转置偏移和长度必须为 0。 在为相应前缀选择最佳路径期间，具有任何此类 Prefix-SID 属性但没有任何有效 SRv6 SID 信息的路径必须被视为不合格。

## 8. IANA 注意事项

### 8.1. BGP Prefix-SID TLV 类型注册表
本文档介绍了 BGP Prefix-SID 属性的两种新 TLV 类型。 IANA 已在“BGP Prefix-SID TLV Types”子注册表中分配了类型值，如下所示：

| Value |         Type        |   参考   |
|:-----:|:-------------------:|:---------:|
|   4   |      Deprecated     |  RFC 9252 |
|   5   | SRv6 L3 Service TLV |  RFC 9252 |
|   6   | SRv6 L2 Service TLV |  RFC 9252 |

值 4 以前对应于 SRv6-VPN SID TLV，该值在本文档的早期草案版本中进行了规定，并由本规范的早期实现使用。 它已被弃用并由 SRv6 L3 Service和 SRv6 L2 Service TLV 取代。

### 8.2. SRv6 Service Sub-TLV 类型注册表
IANA 已在“边界网关协议 (BGP) 参数”注册表下创建并维护一个名为“SRv6 Service Sub-TLV Types”的新子注册表。 根据 [RFC8126]，该子注册表的注册程序如下表所示。

|  范围   |         注册程序     |
|:-------:|:-----------------------:|
|  1-127  |       IETF Review       |
| 128-254 | First Come First Served |
|   255   |       IETF Review       |

IANA 已按如下方式填充此子注册表。 请注意，SRv6 SID Information Sub-TLV 在本文档中定义：

| Value |             Type             |    参考   |
|:-----:|:----------------------------:|:---------:|
|   0   |           Reserved           |  RFC 9252 |
|   1   | SRv6 SID Information Sub-TLV |  RFC 9252 |
|  255  |           Reserved           |  RFC 9252 |

### 8.3. SRv6 Service Data Sub-Sub-TLV 类型注册表
IANA 已在“边界网关协议 (BGP) 参数”注册表下创建并维护一个名为“SRv6 Service Data Sub-Sub-TLV Types”的新子注册表。 该子注册表的注册程序如下所示。

|  范围   |            注册程序     |
|:-------:|:-----------------------:|
| 1-127   | IETF Review             |
| 128-254 | First Come First Served |
| 255     | IETF Review             |

本文档中定义了以下 Sub-Sub-TLV 类型：

| Value |              Type              |   参考   |
|:-----:|:------------------------------:|:---------:|
|   0   |            Reserved            |  RFC 9252 |
|   1   | SRv6 SID Structure Sub-Sub-TLV |  RFC 9252 |
|  255  |            Reserved            |  RFC 9252 |

### 8.4. BGP SRv6 Service SID 标志注册表
IANA 已在“边界网关协议 (BGP) 参数”注册表下创建并维护一个名为“BGP SRv6 Service SID Flags”的新子注册表。 该子注册表的注册程序是 IETF Review，目前所有8位标志的位置均未分配。

### 8.5. SAFI 值注册表
IANA 还在“后续地址族标识符 (Subsequent Address Family Identifiers，SAFI) 参数”注册表下的“SAFI Values”子注册表中值 128（“MPLS 标记的 VPN 地址”）添加了对本文档的引用。

## 9. 安全注意事项
本文档规定了用于 SRv6 服务信令的 BGP 协议的扩展。 这些规范利用现有的 BGP 协议机制来发送各种类型的服务。 它还构建在 SR 架构（更具体地说，SRv6）的现有元素之上。 因此，本节主要提供了这些现有规范的安全注意事项（作为提醒），同时还涵盖了本文档新引入的规范的某些较新的安全方面。

### 9.1. 与 BGP 会话相关的注意事项
应用与 BGP 会话身份验证相关的技术，以保护 BGP 对等体之间的消息安全，如 BGP 规范 [RFC4271] 和 BGP 安全分析 [RFC4272] 中所述。 关于使用 TCP 身份验证选项来保护 BGP 会话的讨论可在 [RFC5925] 中找到，而 [RFC6952] 包括对 BGP 密钥和身份验证问题的分析。 本文档不介绍任何其他 BGP 会话安全注意事项。

### 9.2. 与 BGP 服务相关的注意事项
本文档没有引入新服务或 BGP NLRI 类型，而是扩展了 SRv6 现有服务或 BGP NLRI 类型的信令。 因此，各个 BGP 服务的安全注意事项（例如 BGP IPv4 over IPv6 NH [RFC8950]、BGP IPv6 L3VPN [RFC4659]、BGP IPv6 [RFC2545]、BGP EVPN [RFC7432] 和 IP EVPN [RFC9136]）适用于它们各自的文档。[RFC8669]讨论了防止携带 SR 信息的 BGP Prefix-SID 属性在 SR 域之外泄漏的机制。

提醒一下，一些 BGP 服务（即用于其信令的 AFI/SAFI）最初是针对一种封装机制引入的，后来又扩展到其他封装机制，例如，EVPN MPLS [RFC7432] 被扩展为虚拟可扩展局域网（Virtual eXtensible Local Area Network， VXLAN）封装和使用通用路由封装的网络虚拟化 (Network Virtualization Using Generic Routing Encapsulation， NVGRE)  [RFC8365]。 [RFC9012] 允许使用各种 IP 封装机制以及不同的 BGP SAFI 来实现各自的服务。 用于防止封装信息（在 BGP 属性中携带）泄漏并防止从提供商的内部地址空间（特别是 SRv6 块，如 [RFC8986] 中讨论）向外部对等方（或向内部对等方）通告前缀的现有的过滤机制也适用于 SRv6的情况。

特定于SRv6的是，上述 BGP 过滤机制中的错误配置或错误可能会导致向外部对等方或其他未经授权的实体暴露信息，例如 SRv6 Service SID。 然而，现有 SRv6 数据平面安全机制可以缓解利用此信息或通过将数据包注入网络（例如，使用 VPN 服务的客户网络）来发起攻击的尝试，如下一节所述。

### 9.3. 与IPv6 数据平面上的 SR 相关的注意事项
本节提供了与 SRv6 相关的安全注意事项的简短提醒和概述，以及现有规范的指针。 从 SRv6 数据平面的角度来看，本文档本身没有引入新的安全注意事项。

SRv6 在受信任的 SR 域内运行。 与 PE 路由器之间的服务流相对应的数据包被封装（使用通过 BGP 通告的 SRv6 SID）并在该可信 SR 域内（例如，在单个 AS 内或在单个提供商网络内的多个 AS 之间）传输。

[RFC8402] 涵盖了 SR 架构的安全注意事项。 [RFC8754] 涵盖了更详细的安全考虑因素，特别是 SRv6 和 SRH，因为它们与 SR 攻击（第 7.1 节）、服务盗窃（第 7.2 节）和拓扑泄露（第 7.3 节）相关。 因此，部署 SRv6 的运营商必须遵循 [RFC8754] 第 7 节中描述的注意事项来实施基础设施访问控制列表 (ACL) 以及 BCP 38 [RFC2827] 和 BCP 84 [RFC3704] 中描述的建议。

如 [RFC8986] 中所述，SRv6 部署和 SID 分配指南简化了 ACL 过滤器的部署（例如，与应用于边界节点上的外部接口的 SRv6 块相对应的单个 ACL 足以阻止来自外部/未经授权网络的域中的 SID 发往任何 SRv6 的数据包）。 虽然 SR 域内存在假定的信任模型，因此假定允许任何向 SRv6 SID 发送数据包的节点这样做，但也可以选择使用 SRH 哈希消息认证码 (Hashed Message Authentication Code， HMAC) TLV [ RFC8754] 进行验证，如[RFC8986]中所述。

[RFC8986] 中定义了实现本文档中指示的服务的 SRv6 端点行为； 因此，该文件的安全考虑适用。 这些注意事项独立于用于服务部署的协议，即独立于 SRv6 服务的 BGP 信令。

这些注意事项有助于保护传输流量以及 VPN 等服务，以避免服务盗窃或将流量注入客户 VPN。

## 10. 参考文献
### 10.1. 规范性参考文献

[RFC2119]
Bradner, S., "Key words for use in RFCs to Indicate Requirement Levels", BCP 14, RFC 2119, DOI 10.17487/RFC2119, March 1997, <https://www.rfc-editor.org/info/rfc2119>.

[RFC2545]
Marques, P. and F. Dupont, "Use of BGP-4 Multiprotocol Extensions for IPv6 Inter-Domain Routing", RFC 2545, DOI 10.17487/RFC2545, March 1999, <https://www.rfc-editor.org/info/rfc2545>.

[RFC4271]
Rekhter, Y., Ed., Li, T., Ed., and S. Hares, Ed., "A Border Gateway Protocol 4 (BGP-4)", RFC 4271, DOI 10.17487/RFC4271, January 2006, <https://www.rfc-editor.org/info/rfc4271>.

[RFC4364]
Rosen, E. and Y. Rekhter, "BGP/MPLS IP Virtual Private Networks (VPNs)", RFC 4364, DOI 10.17487/RFC4364, February 2006, <https://www.rfc-editor.org/info/rfc4364>.

[RFC4456]
Bates, T., Chen, E., and R. Chandra, "BGP Route Reflection: An Alternative to Full Mesh Internal BGP (IBGP)", RFC 4456, DOI 10.17487/RFC4456, April 2006, <https://www.rfc-editor.org/info/rfc4456>.

[RFC4659]
De Clercq, J., Ooms, D., Carugi, M., and F. Le Faucheur, "BGP-MPLS IP Virtual Private Network (VPN) Extension for IPv6 VPN", RFC 4659, DOI 10.17487/RFC4659, September 2006, <https://www.rfc-editor.org/info/rfc4659>.

[RFC4760]
Bates, T., Chandra, R., Katz, D., and Y. Rekhter, "Multiprotocol Extensions for BGP-4", RFC 4760, DOI 10.17487/RFC4760, January 2007, <https://www.rfc-editor.org/info/rfc4760>.


[RFC6514]
Aggarwal, R., Rosen, E., Morin, T., and Y. Rekhter, "BGP Encodings and Procedures for Multicast in MPLS/BGP IP VPNs", RFC 6514, DOI 10.17487/RFC6514, February 2012, <https://www.rfc-editor.org/info/rfc6514>.

[RFC7432]
Sajassi, A., Ed., Aggarwal, R., Bitar, N., Isaac, A., Uttaro, J., Drake, J., and W. Henderickx, "BGP MPLS-Based Ethernet VPN", RFC 7432, DOI 10.17487/RFC7432, February 2015, <https://www.rfc-editor.org/info/rfc7432>.

[RFC7606]
Chen, E., Ed., Scudder, J., Ed., Mohapatra, P., and K. Patel, "Revised Error Handling for BGP UPDATE Messages", RFC 7606, DOI 10.17487/RFC7606, August 2015, <https://www.rfc-editor.org/info/rfc7606>.

[RFC8174]
Leiba, B., "Ambiguity of Uppercase vs Lowercase in RFC 2119 Key Words", BCP 14, RFC 8174, DOI 10.17487/RFC8174, May 2017, <https://www.rfc-editor.org/info/rfc8174>.

[RFC8200]
Deering, S. and R. Hinden, "Internet Protocol, Version 6 (IPv6) Specification", STD 86, RFC 8200, DOI 10.17487/RFC8200, July 2017, <https://www.rfc-editor.org/info/rfc8200>.

[RFC8214]
Boutros, S., Sajassi, A., Salam, S., Drake, J., and J. Rabadan, "Virtual Private Wire Service Support in Ethernet VPN", RFC 8214, DOI 10.17487/RFC8214, August 2017, <https://www.rfc-editor.org/info/rfc8214>.

[RFC8277]
Rosen, E., "Using BGP to Bind MPLS Labels to Address Prefixes", RFC 8277, DOI 10.17487/RFC8277, October 2017, <https://www.rfc-editor.org/info/rfc8277>.

[RFC8317]
Sajassi, A., Ed., Salam, S., Drake, J., Uttaro, J., Boutros, S., and J. Rabadan, "Ethernet-Tree (E-Tree) Support in Ethernet VPN (EVPN) and Provider Backbone Bridging EVPN (PBB-EVPN)", RFC 8317, DOI 10.17487/RFC8317, January 2018, <https://www.rfc-editor.org/info/rfc8317>.

[RFC8365]
Sajassi, A., Ed., Drake, J., Ed., Bitar, N., Shekhar, R., Uttaro, J., and W. Henderickx, "A Network Virtualization Overlay Solution Using Ethernet VPN (EVPN)", RFC 8365, DOI 10.17487/RFC8365, March 2018, <https://www.rfc-editor.org/info/rfc8365>.

[RFC8402]
Filsfils, C., Ed., Previdi, S., Ed., Ginsberg, L., Decraene, B., Litkowski, S., and R. Shakir, "Segment Routing Architecture", RFC 8402, DOI 10.17487/RFC8402, July 2018, <https://www.rfc-editor.org/info/rfc8402>.

[RFC8669]
Previdi, S., Filsfils, C., Lindem, A., Ed., Sreekantiah, A., and H. Gredler, "Segment Routing Prefix Segment Identifier Extensions for BGP", RFC 8669, DOI 10.17487/RFC8669, December 2019, <https://www.rfc-editor.org/info/rfc8669>.

[RFC8754]
Filsfils, C., Ed., Dukes, D., Ed., Previdi, S., Leddy, J., Matsushima, S., and D. Voyer, "IPv6 Segment Routing Header (SRH)", RFC 8754, DOI 10.17487/RFC8754, March 2020, <https://www.rfc-editor.org/info/rfc8754>.

[RFC8950]
Litkowski, S., Agrawal, S., Ananthamurthy, K., and K. Patel, "Advertising IPv4 Network Layer Reachability Information (NLRI) with an IPv6 Next Hop", RFC 8950, DOI 10.17487/RFC8950, November 2020, <https://www.rfc-editor.org/info/rfc8950>.

[RFC8986]
Filsfils, C., Ed., Camarillo, P., Ed., Leddy, J., Voyer, D., Matsushima, S., and Z. Li, "Segment Routing over IPv6 (SRv6) Network Programming", RFC 8986, DOI 10.17487/RFC8986, February 2021, <https://www.rfc-editor.org/info/rfc8986>.

[RFC9136]
Rabadan, J., Ed., Henderickx, W., Drake, J., Lin, W., and A. Sajassi, "IP Prefix Advertisement in Ethernet VPN (EVPN)", RFC 9136, DOI 10.17487/RFC9136, October 2021, <https://www.rfc-editor.org/info/rfc9136>.

[RFC9251]
Sajassi, A., Thoria, S., Mishra, M., Patel, K., Drake, J., and W. Lin, "Internet Group Management Protocol (IGMP) and Multicast Listener Discovery (MLD) Proxies for Ethernet VPN (EVPN)", RFC 9251, DOI 10.17487/RFC9251, June 2022, <https://www.rfc-editor.org/info/rfc9251>.

### 10.2. 信息性参考文献

[IGP-FLEX-ALGO]
Psenak, P., Ed., Hegde, S., Filsfils, C., Talaulikar, K., and A. Gulko, "IGP Flexible Algorithm", Work in Progress, Internet-Draft, draft-ietf-lsr-flex-algo-20, 18 May 2022, <https://datatracker.ietf.org/doc/html/draft-ietf-lsr-flex-algo-20>.

[RFC2827]
Ferguson, P. and D. Senie, "Network Ingress Filtering: Defeating Denial of Service Attacks which employ IP Source Address Spoofing", BCP 38, RFC 2827, DOI 10.17487/RFC2827, May 2000, <https://www.rfc-editor.org/info/rfc2827>.

[RFC3704]
Baker, F. and P. Savola, "Ingress Filtering for Multihomed Networks", BCP 84, RFC 3704, DOI 10.17487/RFC3704, March 2004, <https://www.rfc-editor.org/info/rfc3704>.

[RFC4272]
Murphy, S., "BGP Security Vulnerabilities Analysis", RFC 4272, DOI 10.17487/RFC4272, January 2006, <https://www.rfc-editor.org/info/rfc4272>.

[RFC5925]
Touch, J., Mankin, A., and R. Bonica, "The TCP Authentication Option", RFC 5925, DOI 10.17487/RFC5925, June 2010, <https://www.rfc-editor.org/info/rfc5925>.

[RFC6513]
Rosen, E., Ed. and R. Aggarwal, Ed., "Multicast in MPLS/BGP IP VPNs", RFC 6513, DOI 10.17487/RFC6513, February 2012, <https://www.rfc-editor.org/info/rfc6513>.

[RFC6952]
Jethanandani, M., Patel, K., and L. Zheng, "Analysis of BGP, LDP, PCEP, and MSDP Issues According to the Keying and Authentication for Routing Protocols (KARP) Design Guide", RFC 6952, DOI 10.17487/RFC6952, May 2013, <https://www.rfc-editor.org/info/rfc6952>.

[RFC8126]
Cotton, M., Leiba, B., and T. Narten, "Guidelines for Writing an IANA Considerations Section in RFCs", BCP 26, RFC 8126, DOI 10.17487/RFC8126, June 2017, <https://www.rfc-editor.org/info/rfc8126>.

[RFC9012]
Patel, K., Van de Velde, G., Sangli, S., and J. Scudder, "The BGP Tunnel Encapsulation Attribute", RFC 9012, DOI 10.17487/RFC9012, April 2021, <https://www.rfc-editor.org/info/rfc9012>.

[SEGMENT-ROUTING-POLICY]
Filsfils, C., Talaulikar, K., Ed., Voyer, D., Bogdanov, A., and P. Mattes, "Segment Routing Policy Architecture", Work in Progress, Internet-Draft, draft-ietf-spring-segment-routing-policy-22, 22 March 2022, <https://datatracker.ietf.org/doc/html/draft-ietf-spring-segment-routing-policy-22>.

****

本文参考

> 1. [BGP Overlay Services Based on Segment Routing over IPv6 (SRv6)](https://datatracker.ietf.org/doc/html/rfc9252)