---
title: RFC 4451：BGP 多出口标识符 MULTI_EXIT_DISC (MED) 注意事项
author: fnoobt
date: 2023-10-21 15:29:00 +0800
categories: [Network,路由协议]
tags: [network,bgp]
---

## 本备忘录的状态

本备忘录为互联网社区提供信息。 不规定任何互联网标准。 本备忘录的分发不受限制。

## 版权声明

版权所有（c）2006 互联网协会 (The Internet Society )。

## 摘要

BGP MULTI_EXIT_DISC (MED) 属性为 BGP 发言者提供了一种机制，用于向相邻 AS 传达本地 AS 的最佳入口点。 虽然 BGP MED 在许多情况下都能正常工作，但在动态或复杂拓扑中使用 MED 时可能会出现许多问题。

本文档讨论有关 BGP MED 的实施和部署注意事项，并提供实施者和网络运营商应熟悉的信息。

## 1. 简介

BGP MED 属性为 BGP 发言者提供了一种机制，可将本地 AS 的最佳入口点传达给相邻 AS。 虽然 BGP MED 在许多情况下都能正常工作，但在动态或复杂拓扑中使用 MED 时可能会出现许多问题。

阅读本文档时，请注意，目标是讨论有关 BGP MED 的实施和部署注意事项。 此外，其目的是提供实施者和网络运营商都应该熟悉的指导。 在某些情况下，实施建议与部署建议有所不同。

## 2. 要求说明

本文档中的关键词“必须”、“不得”、“需要”、“应”、“不应”、“应该”、“不应该”、“推荐”、“可以”和“可选”应按照[RFC2119] 中的描述进行解释。("MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL")

### 2.1. 关于 MULTI_EXIT_DISC (MED) 属性

BGP MULTI_EXIT_DISC (MED) 属性，以前称为 INTER_AS_METRIC，目前在 [BGP4] 的 5.1.4 节中定义如下：

> MED 是一个可选的非传递属性，旨在用于外部（AS 间）链路，以区分同一相邻 AS 的多个出口或入口点。 MED 属性的值是一个四个八位字节的无符号数，称为度量。 在所有其他因素相同的情况下，应优先选择具有较低度量的退出点。 如果通过外部 BGP (EBGP) 接收，则 MED 属性可以通过内部 BGP (IBGP) 传播到同一 AS 内的其他 BGP 发言者（另请参见 9.1.2.2）。 从相邻 AS 接收到的 MED 属性不得传播到其他相邻 AS。
>
> BGP发言者必须实现一种机制（基于本地配置），允许从路由中删除 MED 属性。 如果 BGP 发言者配置为从路由中删除 MED 属性，则必须在确定路由的优先级和执行路由选择（决策过程阶段 1 和 2）之前完成此删除。
>
> 实现还可以（基于本地配置）更改通过 EBGP 接收的 MED 属性的值。 如果 BGP 发言者配置为更改通过 EBGP 接收的 MED 属性的值，则必须在确定路由的优先级和执行路由选择（决策过程阶段 1 和 2）之前更改该值。 有关对此的必要限制，请参见第 9.1.2.2 节。

[BGP4] 的第 9.1.2.2 (c) 节定义了以下有关 MED 的路由选择标准：

> c) 从考虑中删除具有不太优选的 MED 属性的路由。 MED 仅在从同一相邻 AS 获知的路由之间具有可比性（相邻 AS 由 AS_PATH 属性确定）。 不具有 MED 属性的路由被视为具有尽可能低的 MED 值。
> 
> 以下过程中也对此进行了描述: 
>
> &emsp; for m = 仍在考虑中的所有路由   
    &emsp; &emsp; for n = 仍在考虑中的所有路由  
        &emsp; &emsp; &emsp; if (neighborAS(m) == neighborAS(n)) and (MED(n) < MED(m))  
            &emsp; &emsp; &emsp; &emsp; 从考虑中删除路由 m
>
> 在上面的伪代码中，MED(n) 是一个返回路由 n 的 MED 属性值的函数。 如果路由 n 没有 MED 属性，则该函数返回可能的最低 MED 值（即 0）。
>  
> 类似地，neighborAS(n) 是一个返回接收路由的邻居 AS 的函数。 如果该路由是通过 IBGP 学习到的，并且另一个 IBGP 发言者没有发起该路由，则该路由是另一个 IBGP 发言者从邻居 AS 中学习到该路由的。 如果该路由是通过 IBGP 学到的，并且另一个 IBGP 发言者 (a) 发起该路由，或者 (b) 通过聚合创建该路由，并且聚合路由的 AS_PATH 属性为空或以 AS_SET 开头，则它是本地AS。
>
> 如果在将路由重新通告到 IBGP 之前删除了 MED 属性，则仍可以基于接收到的 EBGP MED 属性执行比较。 如果实现选择删除 MED，则对 MED 的可选比较（如果执行）必须仅在 EBGP 学习的路由中执行。 然后，可以将最佳 EBGP 获悉的路由与删除 MED 属性后的 IBGP 获悉的路由进行比较。 如果从 EBGP 学习路由的子集中删除 MED，并且所选“最佳”EBGP 学习路由不会删除 MED，则必须使用 MED 与 IBGP 学习路由进行比较。 对于 IBGP 学到的路由，MED 必须用于到达决策过程中此步骤的路由比较。 将EBGP学习到的路由的MED包含在与IBGP学习到的路由的比较中，然后删除MED属性，并发布该路由已被证明会导致路由环路。

### 2.2. MED 和 Potatoes

让我们考虑这样一种情况，流量在一对主机之间流动，每个主机连接到不同的传输网络，该网络本身在两个或多个位置互连。 每个传输网络都可以选择将流量发送到距离相邻传输网络最近的对等点，或者将流量传递到通告到目标主机的最低成本路径的互连位置。

前一种方法被称为“hot potato 路由”（或最近出口），因为就像赤手空拳地握着热土豆一样，谁拿到了它就想尽快摆脱它。 热土豆路由是通过不将 EBGP 学习的 MED 传递到 IBGP 来完成的。 这最大限度地减少了提供商路由流量的中转流量。 不太常见的是“cold potato 路由”（或最佳出口），其中传输提供商使用自己的传输能力将流量传输到相邻传输提供商所宣传的最靠近目的地的点。 冷土豆路由是通过将 EBGP 学习到的 MED 传递到 IBGP 来完成的。

如果一个传输提供商使用热土豆路由，而另一个传输提供商使用冷土豆，则两者之间的流量往往会更加对称。 然而，如果两家提供商在其网络之间都采用冷土豆路由或热土豆路由，则可能会存在大量的不对称性。

根据业务关系，如果一个提供商具有更大的容量或较不拥挤的骨干网络，则该提供商可能会使用冷土豆路由。 冷土豆路由广泛使用的一个例子是 20 世纪 90 年代中期 NSF 资助的 NSFNET 骨干网和 NSF 资助的区域网络。

在某些情况下，提供商可能对给定对等 AS 的某些目的地使用热土豆路由，而对其他目的地使用冷土豆路由。 一个例子是 20 世纪 90 年代中期 NSFNET 中对商业和研究流量的不同处理。 如今，许多商业网络与客户交换 MED，但不与双边对等方交换 MED。 然而，MED 的商业用途差异很大，从普遍使用到根本不使用都是存在的。

此外，当今许多 MED 部署的行为可能与网络运营商的预期不同（例如，导致次优路由），这导致的不是热土豆或冷土豆，而是土豆泥！ 本文档中提供了有关 MED 导致的意外行为的更多信息。

## 3. 实施和协议注意事项
已发现许多与 MED 相关的实现和协议特性可能会影响网络行为。 以下部分提供有关这些问题的信息。

### 3.1. MED 是可选的非传递属性

MED 是一个非传递性可选属性，其向 IBGP 和 EBGP 对等方的通告是可自行决定的。 因此，某些实现默认启用向 IBGP 对等方发送 MED，而其他实现则不启用。 此行为可能会导致 AS 内的次优路由选择。 此外，某些实现默认将 MED 发送到 EBGP 对等体，而其他实现则不发送。 此行为可能会导致域间路由的选择次优。

### 3.2. MED 值和首选项

一些实现认为 MED 值为零不如没有 MED 值。 此行为导致 AS 内的路径选择不一致。 BGP 规范的当前版本 [BGP4]指出如果路由 n 没有 MED 属性，则应将尽可能低的 MED 值（即 0）分配给该属性，消除了 [RFC1771] 中存在的歧义。

很明显，BGP 规范的不同实现和不同版本对于缺少 MED 的解释各不相同。 例如，该规范的早期版本要求为缺失的 MED 分配尽可能高的 MED 值（即 2^32-1）。

此外，一些实现已被证明在内部使用最大可能的 MED 值 (2^32-1) 作为“无穷大”度量（即，MED 值用于将路由标记为不可行）； 当收到 MED 值为 2^32-1 的更新时，他们会将该值重写为 2^32-2。 随后，新的 MED 值将被传播，并可能导致路由不一致或意外的路径选择。

由于实现不一致和协议修订差异，如今许多网络运营商明确重置（即设置为零或某个其他“固定”值）入口上的所有 MED 值，以符合其内部路由策略（即包括要求在配置中不使用 MED 值 0 和 2^32-1的策略，无论 MED 是直接计算的还是配置的），这样就不必依赖于所有路由器都具有相同的丢失 MED 行为。

由于实现通常不提供在决策算法中禁用 MED 比较的机制，因此“不使用 MED”通常需要在进入路由域时将所有 MED 显式设置为某个固定值。 通过为网络中的所有路由一致地分配固定的 MED 值，MED 在决策算法中实际上不再是问题。

### 3.3. 比较不同自治系统之间的 MED

MED 旨在用于外部（AS 间）链路，以区分同一相邻 AS 的多个出口或入口点。 然而，现在大量 MED 应用程序使用 MED 来确定从不同自治系统接收到的类似路由之间的路由偏好。

许多实现提供了对从不同相邻自治系统接收到的路由之间的 MED 进行比较的功能。 虽然此功能已经证明了一些好处（例如，[RFC3345] 中描述的），但运营商应警惕启用此类功能的潜在副作用。 下面的部署部分提供了一些示例来说明为什么这可能会导致不良行为。

### 3.4. BGP 的 MED、路由反射和 AS 联盟

在特定配置中，“BGP 路由反射 - 全网状 IBGP 的替代方案”[RFC2796] 和“BGP 自治系统联盟”[RFC3065] 中定义的 BGP 扩展机制将引入持续的 BGP 路由振荡 [RFC3345]。 该问题根源于BGP的工作方式：信息隐藏/层次结构与由于 MED 规则导致缺乏总排序而强加的非层次结构选择过程之间存在冲突。 鉴于当前的实践，我们发现该问题最常出现在 MED + 路由反射器或联盟的环境中。

避免这种情况的一种潜在方法是将成员 AS 间或集群间 IGP 度量配置为高于成员 AS 内 IGP 度量，和/或使用其他平局策略，来避免基于不可比较的 MED 进行 BGP 路由选择。 当然，IGP 度量约束对于某些应用程序可能过于繁重。

不比较从不同相邻自治系统获知的前缀的多个路径之间的 MED（如第 2.3 节中所述），或者根本不使用 MED，会显着降低将潜在路由振荡条件引入网络的概率。

尽管就当前规范而言可能是“合法的”，但不建议修改在任何类型的 IBGP 会话（例如，标准 IBGP、BGP 联盟的成员 AS 之间的 EBGP 会话、路由反射等）上接收到的 MED 属性。

### 3.5. 路线振荡抑制和 MED 波动

MED 通常是从 IGP 度量或与给定 BGP NEXT_HOP 的 IGP 度量相关的附加成本动态派生的。 这通常提供了一个有效的模型，用于确保向对等体通告的 BGP MED（用于表示到网络内给定目的地的最佳路径）与给定 AS 内的 IGP 的路径保持一致。

动态派生的基于 IGP 的 MED 的结果是，AS 内的不稳定，甚至 AS 内单个给定链路的不稳定，都可能导致广泛的 BGP 不稳定或跨多个域传播的 BGP 路由通告波动。 简而言之，如果每次 IGP 度量波动时您的 MED 都会“波动”，则您的路由可能会因 BGP 路由振荡抑制 [RFC2439] 而受到抑制。

MED 的使用可能会加剧 BGP 振荡抑制行为的不利影响，因为它可能导致仅为反映内部拓扑变化而重新进行路由通告。

许多实现并不存在 IGP 振荡的实际问题； 它们要么在第一次通告时锁定其 IGP 度量，要么采用某种内部抑制机制。 有些实现将 BGP 属性更改视为不如路由撤销和公告重要，以尝试减轻此类事件的影响。

### 3.6. MED 对更新打包效率的影响

一条 BGP 更新消息中可以通告多条不可行的路由。 BGP4 协议还允许在单个更新消息中通告具有一组公共路径属性的多个前缀。 这通常称为“更新打包”。 如果可能，建议使用更新打包，因为它提供了一种在许多领域实现更高效行为的机制，包括：

- 由于生成或接收较少的更新消息而减少了系统开销。
- 由于较少的数据包和较低的带宽消耗而减少了网络开销。
- 减少路径属性的处理频率并在 AS_PATH 数据库（如果有的话）中搜索匹配集。 路径属性的一致排序可以轻松地在数据库中进行匹配，因为您没有相同数据的不同表示。

更新打包要求单个更新消息中的所有可行路由共享一个公共属性集，以包括一个公共 MED 值。 因此，MED 值中潜在的大范围变化会引入另一个变量，并可能导致更新打包效率显着降低。

### 3.7. 时间路由选择

某些实现存在错误，导致基于 MED 的最佳路径选择出现临时性行为。 这些通常涉及存储最旧路由和对 MED 排序路由的方法，这会导致关于是否真正选择最旧路由的不确定行为。

其原因是较旧的路径可能更稳定，因此更可取。 然而，路线选择中的时间行为会导致不确定性行为，因此通常是不受欢迎的。

## 4. 部署注意事项
已经讨论过，接受来自其他自治系统的 MED 可能会导致网络中的流量波动。 某些实现只会降低 MED，而不会将其调高以防止过多的流量变动。

但是，如果会话重置，则通告的 MED 可能会发生变化。 如果网络依靠收到的 MED 来正确路由流量，则流量模式可能会发生巨大变化，进而可能导致网络拥塞。 本质上，基于 MED 接受和路由流量允许其他人对您的网络进行流量工程。 您可能会接受也可能不会接受。

如前所述，许多网络运营商选择在入口处重置 MED 值。 此外，许多运营商明确不使用 0 或 2^32-1 的 MED 值，以避免与 BGP 规范的实现和各种修订版不一致。

### 4.1. 不同自治系统之间 MED 的比较

尽管 MED 原本仅用于比较来自同一 AS 中的不同外部对等方接收的路径，但许多实现也提供了比较不同自治系统之间的 MED 的功能。 AS运营商经常使用LOCAL_PREF来选择外部偏好（主要、次要上游、对等体、客户等），使用MED而不是LOCAL_PREF可能会导致最佳路由分布不一致，因为MED仅在AS_PATH长度之后进行比较。

尽管对于某些配置来说这似乎是个好主意，但在比较不同自治系统之间的 MED 时必须小心。 BGP 发言者通常通过获取与到达本地 AS 内给定 BGP NEXT_HOP 相关的 IGP 度量来导出 MED 值。 这允许 MED 在向对等体通告路由时合理地反映 IGP 拓扑。 虽然在比较从单个 AS 学习的多个路径之间的 MED 时这没有问题，但在比较不同自治系统之间的 MED 时，它可能会导致潜在的“加权”决策。 最常见的情况是，当自治系统使用不同的机制来派生 IGP 度量或 BGP MED时 ，或者当它们甚至可能使用具有截然不同的度量空间（例如，OSPF 与 IS-IS 中的传统度量空间）的不同 IGP 协议时。

### 4.2. 聚集对 MED 的影响

另一个 MED 部署考虑因素涉及 BGP 路由信息聚合对 MED 的影响。 聚合通常是从 AS 中的多个位置生成的，以适应稳定性、冗余和其他网络设计目标。 当 MED 源自与所述聚合关联的 IGP 度量时，向对等方通告的 MED 值可能会导致非常次优的路由。

## 5. 安全考虑
MED 被特意设计为“弱”度量，仅在最佳路径决策过程的后期使用。 BGP 工作组担心，如果没有指定其他首选项，远程操作员指定的任何度量只会影响本地 AS 中的路由。 MED 设计的一个首要目标是确保对等方无法“流失”或“吸收”他们所通告的网络的流量。 因此，接受来自对等方的 MED 在某种意义上可能会增加网络对对等方利用的敏感性。

## 6. 致谢
感谢John Scudder为文档提供他一贯的敏锐眼光和建设性见解。 另外，感谢 Curtis Villamizar、JR Mitchell 和 Pekka Savola 的宝贵反馈。

## 7. 参考文献

### 7.1. 规范性参考文献

[RFC1771] Rekhter, Y. and T. Li, “A Border Gateway Protocol 4 (BGP- 4)”, RFC 1771, March 1995.

[RFC2119] Bradner, S., “Key words for use in RFCs to Indicate Requirement Levels”, BCP 14, RFC 2119, March 1997.

[RFC2796] Bates, T., Chandra, R., and E. Chen, “BGP Route Reflection - An Alternative to Full Mesh IBGP”, RFC 2796, April 2000.

[RFC3065] Traina, P., McPherson, D., and J. Scudder, “Autonomous System Confederations for BGP”, RFC 3065, February 2001.

[BGP4] Rekhter, Y., Li, T., and S. Hares, “A Border Gateway Protocol 4 (BGP-4)”, RFC 4271, January 2006.

### 7.2. 信息性参考文献

[RFC2439] Villamizar, C., Chandra, R., and R. Govindan, “BGP Route Flap Damping”, RFC 2439, November 1998.

[RFC3345] McPherson, D., Gill, V., Walton, D., and A. Retana, “Border Gateway Protocol (BGP) Persistent Route Oscillation Condition”, RFC 3345, August 2002.

****

本文参考

> 1. [BGP MULTI_EXIT_DISC (MED) Considerations](https://datatracker.ietf.org/doc/html/rfc4451)