---
title: RFC 7209：以太网VPN（EVPN）的要求
author: fnoobt
date: 2023-10-19 15:21:00 +0800
categories: [Network,路由协议]
tags: [network,bgp,evpn]
---

## 摘要

以太网 L2VPN （Layer 2 Virtual Private Networks，二层虚拟专用网络）服务的广泛采用以及该技术的新应用（例如数据中心互连）最终产生了一系列新的要求，而当前的虚拟专用局域网服务（Virtual Private LAN Service，VPLS）解决方案无法轻松解决这些要求。特别是，不支持具有全活转发的多宿主，并且还没有现有的解决方案可以利用多点到多点（Multipoint-to-Multipoint，MP2MP）标签交换路径（Label Switched Paths，LSP）来优化多目标帧的传递。此外，即使在基于BGP的自动发现的情况下，VPLS的配置也要求网络运营商在访问配置之上指定各种网络参数。本文档指定了解决上述问题的以太网VPN（Ethernet VPN，EVPN）解决方案的要求。

## 本备忘录的状态

本文档不是Internet Standards Track规范，其发布仅供参考。

本文档是互联网工程任务组 (Internet Engineering Task Force，IETF) 的产品。 它代表了IETF社区的共识。 它已接受公众审查，并已被互联网工程指导小组 (Internet Engineering Steering Group，IESG) 批准发布。并非 IESG 批准的所有文件都是任何级别互联网标准的候选文件； 请参阅[RFC 5741 第2节](https://datatracker.ietf.org/doc/html/rfc5741#section-2)。

有关本文档当前状态、任何勘误表以及如何提供反馈的信息，请访问<https://www.rfc-editor.org/info/rfc7209>。

## 版权声明

版权所有 (c) 2014 IETF Trust 和文档作者。 版权所有。

本文件受本文件发布之日生效的 [BCP 78](https://datatracker.ietf.org/doc/html/bcp78) 和 [IETF 信托基金有关 IETF 文件的法律规定](https://trustee.ietf.org/license-info) 的约束。请仔细阅读这些文件，因为它们描述了您与本文件相关的权利和限制。从本文档中提取的代码组件必须包括信托法律条款第 4.e 节中所述的简化 BSD 许可证文本，并且不提供简化 BSD 许可证中所述的保证。


## 1. 简介

虚拟专用 LAN 服务 (VPLS)，如 [RFC4664: Framework for L2VPNs](https://datatracker.ietf.org/doc/html/rfc4664)、[RFC4761: VPLS Using BGP for Auto-Discovery and Signaling](https://datatracker.ietf.org/doc/html/rfc4761) 和 [RFC4762: VPLS Using LDP Signaling](https://datatracker.ietf.org/doc/html/rfc4762) 中所定义，是一项经过验证且广泛部署的技术。然而，现有解决方案在冗余性、多播优化和配置简便性方面存在许多限制。 此外，新的应用程序正在推动对其他 L2VPN 服务一些新要求，例如以太网树 (Ethernet Tree，E-Tree) 和虚拟专线服务 (Virtual Private Wire Service，VPWS)。

在多宿主方面，例如，如[VPLS-BGP-MH](https://datatracker.ietf.org/doc/html/rfc7209/#ref-VPLS-BGP-MH)中所述，当前的VPLS只能支持单活冗余模式（第3节中定义）的多宿主。 当前的 VPLS 解决方案不支持具有全活冗余模式（第 3 节中定义）的灵活多宿主。

在多播优化领域，[RFC7117: Multicast in VPLS](https://datatracker.ietf.org/doc/html/rfc7117)描述了如何将多播LSP与VPLS结合使用。 然而，该解决方案仅限于点对多点 (Point-to-Multipoint，P2MP) LSP，因为还没有使用 VPLS 来利用多点对多点 (MP2MP) LSP的已定义解决方案。

在配置简化方面，当前的 VPLS 确实通过依赖基于 BGP 的服务自动发现 [RFC4761](https://datatracker.ietf.org/doc/html/rfc4761) 、[RFC6074: Provisioning, Auto-Discovery, and Signaling in L2VPNs](https://datatracker.ietf.org/doc/html/rfc6074) 提供了一种单面配置机制。 这仍然需要操作员在访问侧以太网配置的基础上配置许多网络侧参数。

在数据中心互连领域，应用程序正在推动对新服务接口类型的需求，这些新服务接口类型是 VLAN 捆绑和基于 VLAN 的服务接口的混合组合。 这些被称为“VLAN 感知捆绑”服务接口。

虚拟化应用程序还推动了网络处理的 MAC（Media Access Control，介质访问控制）地址数量的增加。 这就提出了使网络在故障时重新收敛与服务商边缘 (Provider Edge，PE) 获知的 MAC 地址数量无关的要求。

需要最小化多目标框架的泛洪量并将泛洪局限在给定站点的范围内。

除了目前 VPLS 和分层 VPLS (Hierarchical VPLS，H-VPLS) 所涵盖的范围之外，还需要支持灵活的 VPN 拓扑和策略。

本文档的重点是定义新解决方案的要求，即解决上述问题的以太网 VPN (EVPN)。

第 4 节讨论冗余要求。 第 5 节描述了多播优化要求。 第 6 节阐明了配置要求的难易程度。 第 7 节重点介绍新的服务接口要求。 第 8 节强调了快速收敛的要求。 第 9 节描述了泛洪抑制要求，最后第 10 节讨论了支持灵活 VPN 拓扑和策略的要求。

## 2. 要求说明

本文档中的关键词“必须”、“不得”、“需要”、“应”、“不应”、“应该”、“不应该”、“推荐”、“可以”和“可选”是 按照 [RFC2119: Key words for use in RFCs to Indicate Requirement Levels](https://datatracker.ietf.org/doc/html/rfc2119) 中的描述进行解释。("MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL")

本文档不是协议规范，本文档中的关键词是为了清晰和强调需求语言而使用的。

## 3. 术语

- AS：Autonomous System，自治系统
- CE：Customer Edge，客户侧边缘
- E-Tree：Ethernet Tree，以太网树
- MAC地址：Media Access Control address，介质访问控制地址，也称为MAC
- LSP：Label Switched Path，标签交换路径
- PE：Provider Edge，服务提供商边缘
- MP2MP：Multipoint to Multipoint，多点到多点
- VPLS：Virtual Private LAN Service，虚拟专用局域网服务
- 单活冗余模式：当一台设备或一个网络被多宿主到两个或多个PE的组中，并且当该冗余组中只有一个PE可以向/从给定VLAN的多宿主设备或网络转发流量时，这种多宿主称为“单活”。
- 全活冗余模式：将一台设备多宿主到两个或多个PE的组中，并且当该冗余组中的所有PE可以向/从给定VLAN的多宿主设备或网络转发通信时，这种多宿主称为“全活”。

## 4. 冗余要求
### 4.1 基于流的负载均衡

用于将CE节点多宿主到一组PE节点的常见机制涉及利用基于 [802.1AX](https://datatracker.ietf.org/doc/html/rfc7209/#ref-802.1AX) 的多机箱以太网链路聚合组 (Link Aggregation Groups，LAG)。 [PWE3-ICCP](https://datatracker.ietf.org/doc/html/rfc7209/#ref-PWE3-ICCP)描述了一种这样的方案。 CE通过负载均衡算法在连接到PE的连接电路上分配流量的方法非常灵活。 唯一的要求是该算法确保给定流量的有序逐帧传递 在典型的实现中，这些算法涉及基于哈希函数选择捆绑包内的出站链接，该哈希函数根据以下一个或多个字段识别流：

- i. 第 2 层：源 MAC 地址、目标 MAC 地址、VLAN 
- ii. 第 3 层：源 IP 地址、目标 IP 地址 
- iii. 第 4 层：UDP 或 TCP 源端口、目标端口

这里需要注意的一个关键点是，[802.1AX](https://datatracker.ietf.org/doc/html/rfc7209/#ref-802.1AX) 没有为以太网捆绑定义标准负载平衡算法，因此，不同的实现方式会有不同的动作。 事实上，即使在链路上存在不对称负载平衡的情况下，捆绑包也可以正确运行。 在这种情况下，全主动多宿主的第一个要求是能够适应来自 CE 节点的基于 L2、 L3、 L4 报头字段的基于流的灵活负载平衡的能力。

(R1a) 解决方案必须能够如上所述支持来自CE的灵活的基于流的负载平衡。

(R1b) 即使CE连接到多个PE上，解决方案也必须能够支持针对CE的流量的基于流的负载平衡。因此，该解决方案必须能够使用连接到CE的多个链路，而不用考虑CE连接到的PE的数量。

应当注意的是，当一个CE多宿主到多个PE时，从每个远程PE到每个多宿主PE可能会有多个等价多径（Equal-Cost Multipath，ECMP）路径。 此外，对于全活动多宿主CE，远程PE可以选择任意多宿主PE来发送目的地为多宿主CE的流量。 因此，当解决方案支持全活多宿主时，它必须为目的地为多宿主CE的流量使用尽可能多的这些路径。

(R1c) 解决方案应该支持 PE 之间基于流的负载平衡，而这些 PE 是跨越多个自治系统的冗余组的成员。

### 4.2 基于流的多路径

任何满足 4.1 节中描述的全活冗余模式（例如，基于流的负载平衡）的解决方案，也需要在给定的一对 PE 之间使用多条路径。 例如，如果在全主动冗余组中的远程 PE 和一对 PE 之间存在两个或多个 LSP，则解决方案需要能够在这些 LSP 之间基于每个流对目的地流量进行负载均衡到冗余组中的PE。 此外，如果远程 PE 与冗余组中的其中一个 PE 之间存在两条或更多条 ECMP 路径，则解决方案需要利用所有等价 LSP。 对于后者，该解决方案还可以利用基于熵标签[RFC6790:   The Use of Entropy Labels in MPLS Forwarding](https://datatracker.ietf.org/doc/html/rfc6790)的负载平衡功能。

(R2a) 解决方案必须能够使用远程 PE 和具有全活多宿主的冗余组中的所有 PE 之间的所有 LSP。

(R2b) 解决方案必须能够使用远程 PE 与具有全活多宿主的冗余组中的任何 PE 之间的所有 ECMP 路径。

例如，考虑一种场景，其中CE1多宿主到PE1和PE2，CE2多宿主到以全活冗余模式运行的PE3和PE4的。 此外，请考虑在 CE1 和 CE2 的任意多宿主 PE 之间存在 3 条 ECMP 路径。 从 CE1 到 CE2 的流量可以通过 MPLS/IP 核心在 12 条不同的路径上转发，如下所示： CE1 将流量负载均衡到 PE1 和 PE2。 PE1 和 PE2 中的每一个都有 3 条到 PE3 和 PE4 的 ECMP 路径，总共 12 条路径。 最后，当流量到达 PE3 和 PE4 时，它通过以太网通道（也称为链路捆绑）转发到 CE2。

值得指出的是，基于流的多路径是对上一节中基于流的负载均衡的补充。

### 4.3 异地冗余 PE 节点

提供与 CE 或接入网络的多宿主连接的 PE 节点可以位于同一物理位置（共置），也可以分布在不同的地理位置（例如，在不同的中心局 (Central Offices，CO) 或存在点 (Points of Presence，POP) 中）。 当提供异地冗余解决方案时需要后者，以确保在断电、自然灾害等情况下关键应用程序的业务连续性。全活多宿主机制需要支持共置和异地冗余 PE 放置。 从成本角度来看，后一种情况通常意味着，为运行多宿主机制需要 PE 之间的专用链路而不是主要考虑成本原因。 此外，当后面的 PE 具有异地冗余时，不能假定从远程 PE 到双宿主设置中的一对 PE 的 IGP 成本相同。

(R3a) 解决方案必须支持全活多宿主，而无需多宿主组中的 PE 之间有专用的控制/数据链路。

(R3b) 解决方案必须支持从远程 PE 到多宿主组中每个 PE 的不同 IGP 成本。

(R3c) 解决方案必须支持同一自治系统内不同 IGP 域的多宿主。

(R3d) 解决方案应该支持跨多个自治系统的多宿主。

### 4.4 最优流量转发

在典型网络中，当考虑一对指定的 PE 时，通常会发现连接到这些 PE 的单宿主 CE 和多宿主 CE 。

(R4) 全主动多宿主解决方案应该支持以下所有场景的单播流量的最佳转发。 所谓“**最佳转发**”，是指除非目标 CE 连接到多宿主 PE 之一，否则不会在作为多宿主组成员的 PE 设备之间转发流量。

- i. 单宿主 CE 到多宿主 CE
- ii. 多宿主 CE 到单宿主 CE
- iii. 多宿主 CE 到多宿主 CE

对于地理冗余 PE 来说，将流量从同一多宿主组中的一个 PE 转发到另一个 PE 会引入额外的延迟，而且 PE 节点和核心节点的交换容量的使用效率低下，需要着重考虑。 多宿主组（也称为跨设备链路聚合）是一组支持多宿主 CE 的 PE 组。

### 4.5 支持灵活的冗余分组

(R5)为了支持灵活的冗余分组，多宿主机制应该允许将PE节点任意分组为冗余组，其中每个冗余组代表共享同一组PE的所有多宿主设备/网络。

最好用一个例子来解释这一点：考虑三个 PE 节点 – PE1、PE2 和 PE3。 多宿主机制必须允许给定的 PE（例如 PE1）同时成为多个冗余组的一部分。 例如，可以存在一个组（PE1、PE2）、一个组（PE1、PE3）和另一个组（PE2、PE3），其中CE可以多宿主到这三个冗余组中的任意一个。

### 4.6 多宿主网络

有些应用程序需要以太网而不是单个设备来多宿主到一组 PE。 以太网通常会运行弹性机制，例如多生成树协议[802.1Q]或以太网环保护交换[G.8032]。 PE可以参与也可以不参与以太网的控制协议。 对于运行 [802.1Q] 或 [G.8032] 的多宿主网络，这些协议要求每个 VLAN 仅在其中一个多宿主链路上处于活动状态。

(R6a) 解决方案必须支持具有单主动冗余模式的多宿主网络连接，其中所有 VLAN 在一个 PE 上均处于活动状态。

(R6b) 解决方案还必须支持具有单主动冗余模式的多宿主网络，其中不相交的 VLAN 集在不同的 PE 上处于活动状态。

(R6c) 解决方案应该支持作为跨越多个 AS 的冗余组成员的 PE 之间的单主动冗余模式。

(R6d) 解决方案可以支持具有基于 MAC 的负载平衡的多宿主网络的全主动冗余模式（即，VLAN 上的不同 MAC 地址可通过不同的 PE 访问）。

## 5. 组播优化要求

在某些环境中，可能需要使用 MP2MP LSP 来优化多播、广播和未知单播流量，以便减少核心路由器中的多播状态量。 [RFC7117](https://datatracker.ietf.org/doc/html/rfc7117)禁止使用 MP2MP LSP ，因为当前 VPLS 解决方案要求出口 PE 在通过 LSP 接收到未知单播数据包时执行学习。 当使用 MP2MP LSP 时，这是一个挑战，因为它们没有识别发送者的固有机制。 如果不再需要识别用于执行学习的发送方，则可以轻松地将 MP2MP LSP 进行多播优化。

(R7a) 解决方案必须能够提供一种机制，当通过 MP2MP LSP 接收数据包时，不需要针对 MPLS LSP 进行 MAC 学习。

(R7b) 解决方案应该能够提供使用 MP2MP LSP 来优化多播、广播和未知单播流量传送的过程。

## 6. 易于配置要求
随着 L2VPN 技术扩展到企业部署，易于配置变得至关重要。 尽管当前的 VPLS 具有自动发现机制，可以通过 MPLS/IP 核心网络自动发现属于给定 VPN 实例的成员 PE，但仍需要进一步简化，如下所述：

(R8a) 该解决方案必须支持 MPLS/IP 核心网络上 VPN 成员 PE 的自动发现，类似于 [RFC4761](https://datatracker.ietf.org/doc/html/rfc4761) 和 [RFC6074](https://datatracker.ietf.org/doc/html/rfc6074) 中描述的 VPLS 自动发现机制。

(R8b) 该解决方案应该支持自动发现属于给定冗余或多宿主组的 PE。

(R8c) 该解决方案应该支持多宿主设备或网络的站点 ID 的自动感知，并支持根据站点 ID 自动生成冗余组 ID。

(R8d) 该解决方案应该支持参与冗余（多宿主）组的 PE 之间的自动指定转发器（Designated Forwarder，DF）选举，并且能够在冗余组的成员 PE 之间划分服务实例（例如 VLAN）。

(R8e) 对于 VLAN 标识符在 MPLS 网络中是全局的部署（即网络仅限于最多 4K 服务），PE 设备应从VLAN 标识符派生 MPLS 特定属性（例如 VPN ID、BGP 路由目标等）。 这样，网络运营商为接入电路配置 VLAN 标识符就足够了，并且通过核心网络建立服务所需的所有 MPLS 和 BGP 参数都将自动获得，无需任何显式配置 。

(R8f) 实现应该恢复对为没有配置新值的参数使用默认值。

## 7. 新的服务接口要求
[MEF] 和 [802.1Q](https://datatracker.ietf.org/doc/html/rfc7209/#ref-802.1Q) 指定了以下服务：

- **端口模式：**在此模式下，端口上的所有流量都映射到单个桥接域和单个相应的 L2VPN 服务实例。 端到端保证客户 VLAN 透明度。
- **VLAN模式：**该模式下，端口上的每个VLAN都映射到唯一的桥接域和对应的L2VPN服务实例。 此模式允许通过端口进行服务多路复用，并支持可选的 VLAN 转换。
- **VLAN捆绑：**在该模式下，端口上的一组VLAN被共同映射到一个唯一的桥接域和相应的L2VPN服务实例。 客户 MAC 地址在映射到同一服务实例的所有 VLAN 中必须是唯一的。

对于上述每个服务，在支持相关服务的 PE 上为每个服务实例分配一个桥接域。 例如，在端口模式的情况下，将为属于该服务实例的所有端口分配单个桥接域，无论通过这些端口的 VLAN 数量如何。

值得注意的是，上面使用的术语“桥接域”是指 IEEE 桥接模型中定义的 MAC 转发表，并不表示或暗示任何具体实现。

[RFC4762]定义了两种基于“离散和集中学习”（unqualified and qualified learning）的VPLS业务，分别映射到端口模式和VLAN模式。

(R9a) 解决方案必须支持上述三种服务类型（端口模式、VLAN 模式和 VLAN 捆绑）。

对于数据中心互连的托管应用程序，网络运营商需要能够使用单个 L2VPN 实例在 WAN 上扩展以太网 VLAN，同时保持与该实例关联的各个 VLAN 之间的数据平面分离。 这称为“VLAN 感知捆绑服务”。

(R9b) 解决方案可以支持 VLAN 感知捆绑服务。

这就产生了两种新的服务接口类型：**无需转换的 VLAN 感知捆绑**和**带转换的 VLAN 感知捆绑**。

无需转换的 VLAN 感知捆绑服务接口具有以下特征：

- 服务接口可将客户 VLAN 捆绑到单个 L2VPN 服务实例中。
- 服务接口可保证客户VLAN端到端的透明性。
- 服务接口可维护客户 VLAN 之间的数据平面分离（即，为每个 VLAN 创建专用桥接域）。

在多对一捆绑的特殊情况下，服务接口不得假设任何客户 VLAN 具有先验知识。 即不应在PE上配置客户VLAN； 相反，该接口的配置就像基于端口的服务一样。

带转换的 VLAN 感知捆绑服务接口具有以下特征：

- 服务接口可将客户 VLAN 捆绑到单个 L2VPN 服务实例中。
- 服务接口可维护客户 VLAN 之间的数据平面分离（即，为每个 VLAN 创建专用桥接域）。
- 业务接口支持客户VLAN ID转换，以应对不同接口上使用不同VLAN标识符（VID）来指定同一客户VLAN的场景。

新服务类型和之前定义的三种类型在服务提供者资源分配方面的主要区别是，新服务要求为每个L2VPN服务实例分配多个桥接域（每个客户VLAN一个），而不是每个L2VPN服务实例分配单个桥接域。

## 8. 快速收敛
(R10a) 对于多宿主设备和多宿主网络，解决方案必须具备从 PE-CE 连接电路故障以及 PE 节点故障中恢复的你能力。

(R10b) 恢复机制必须提供与 PE 获知的 MAC 地址数量无关的收敛时间。 这在虚拟化应用的环境中尤其重要，虚拟化应用正在推动第 2 层网络处理的 MAC 地址数量的增加。

(R10c) 此外，恢复机制应该提供收敛时间，该收敛时间应于与绑定在电路或 PE 关联的服务实例数量无关。

## 9. 泛洪抑制
(R11a) 解决方案应该允许网络操作员选择是否丢弃未知单播帧还是进行泛洪传输。 该属性需要在每个服务实例的基础上进行配置。

(R11b) 此外，对于该解决方案用于数据中心互连的情况，该解决方案应该最大限度地减少给定站点范围之外的广播帧的泛洪。 特别需要注意的是周期性地址解析协议 (ARP) 流量。

(R11c) 此外，该解决方案应该在拓扑变化时消除任何不必要的单播流量泛洪，特别是在多宿主站点的情况下，其中 PE 具有给定 MAC 地址的备份路径的先验知识。

## 10. 支持灵活的VPN拓扑和策略
(R12a) 解决方案必须能够支持不受该解决方案底层机制限制的灵活 VPN 拓扑。

其中一个示例是 E-Tree 拓扑，其中 VPN 中的一个或多个站点是根，其他站点是叶。 允许根向其他根和叶子发送流量，而叶子只能与根通信。 该解决方案必须提供支持 E-Tree 拓扑的能力。

(R12b) 该解决方案可以提供在 MAC 地址粒度上应用策略的能力，以控制 VPN 中的哪些 PE 学习哪个 MAC 地址以及如何转发特定的 MAC 地址。 应该可以应用策略来仅允许 VPN 中的某些成员 PE 发送或接收特定 MAC 地址的流量。

(R12c) 解决方案必须能够支持 [RFC4364: BGP/MPLS IP VPNs](https://datatracker.ietf.org/doc/html/rfc4364) 中描述的 AS 间 option-C 和 AS 间 option-B 两种场景。

## 11. 安全考虑
为 EVPN 解决方案开发的任何协议扩展均应包括适当的安全分析。 除了在数据平面中执行 MAC 学习时 [RFC4761](https://datatracker.ietf.org/doc/html/rfc4761) 和 [RFC4762](https://datatracker.ietf.org/doc/html/rfc4762) 中涵盖的安全要求以及在控制平面中执行 MAC 学习时 [RFC4364](https://datatracker.ietf.org/doc/html/rfc4364) 中涵盖的安全要求之外，还需要涵盖以下附加要求。

(R13) 解决方案必须能够检测并正确处理同一 MAC 地址出现在两个不同以太网段后面的情况（无论是无意还是恶意）。

(R14) 解决方案必须能够将 MAC 地址与特定以太网段（也称为“粘性 MAC”）相关联，以帮助限制该 MAC 地址进入网络的恶意流量。 此功能可以限制网络上欺骗性 MAC 地址的出现。 启用此功能后，将禁止此类粘性 MAC 地址的 MAC 移动性，并且必须丢弃来自任何其他以太网段的此类 MAC 地址的流量。

## 12. 规范性引用
[802.1AX] IEEE, “IEEE Standard for Local and metropolitan area networks - Link Aggregation”, Std. 802.1AX-2008, IEEE Computer Society, November 2008.

[802.1Q] IEEE, “IEEE Standard for Local and metropolitan area networks - Virtual Bridged Local Area Networks”, Std. 802.1Q-2011, 2011.

[G.8032] ITU-T, “Ethernet ring protection switching”, ITU-T Recommendation G.8032, February 2012.

[RFC2119] Bradner, S., “Key words for use in RFCs to Indicate Requirement Levels”, BCP 14, RFC 2119, March 1997.

[RFC4364] Bersani, F. and H. Tschofenig, “The EAP-PSK Protocol: A Pre-Shared Key Extensible Authentication Protocol (EAP) Method”, RFC 4764, January 2007.

[RFC4761] Kompella, K., Ed., and Y. Rekhter, Ed., “Virtual Private LAN Service (VPLS) Using BGP for Auto-Discovery and Signaling”, RFC 4761, January 2007.

[RFC4762] Lasserre, M., Ed., and V. Kompella, Ed., “Virtual Private LAN Service (VPLS) Using Label Distribution Protocol (LDP) Signaling”, RFC 4762, January 2007.

[RFC6074] Rosen, E., Davie, B., Radoaca, V., and W. Luo, “Provisioning, Auto-Discovery, and Signaling in Layer 2 Virtual Private Networks (L2VPNs)”, RFC 6074, January 2011.

## 13. 信息性引用
[VPLS-BGP-MH] Kothari, B., Kompella, K., Henderickx, W., Balue, F., Uttaro, J., Palislamovic, S., and W. Lin, “BGP based Multi-homing in Virtual Private LAN Service”, Work in Progress, July 2013.

[PWE3-ICCP] Martini, L., Salam, S., Sajassi, A., and S. Matsushima, “Inter-Chassis Communication Protocol for L2VPN PE Redundancy”, Work in Progress, March 2014.

[MEF] Metro Ethernet Forum, “Ethernet Service Definitions”, MEF 6.1 Technical Specification, April 2008.

[RFC4664] Andersson, L., Ed., and E. Rosen, Ed., “Framework for Layer 2 Virtual Private Networks (L2VPNs)”, RFC 4664, September 2006.

[RFC6790] Kompella, K., Drake, J., Amante, S., Henderickx, W., and L. Yong, “The Use of Entropy Labels in MPLS Forwarding”, RFC 6790, November 2012.

[RFC7117] Aggarwal, R., Ed., Kamite, Y., Fang, L., Rekhter, Y., and C. Kodeboniya, “Multicast in Virtual Private LAN Service (VPLS)”, RFC 7117, February 2014.

****

本文参考

> 1. [Requirements for Ethernet VPN (EVPN)](https://datatracker.ietf.org/doc/html/rfc7209/)
> 2. [RFC7209：以太网VPN（EVPN）的要求](https://zhuanlan.zhihu.com/p/607742050?utm_id=0)