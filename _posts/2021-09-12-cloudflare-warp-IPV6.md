---
title: Cloudflare WARP 教程：给 VPS 额外添加“原生” IPv4/IPv6 双栈网络出口
author: fnoobt
date: 2021-09-12 00:34:00 +0800
categories: [Cloudflare,WARP]
tags: [cloudflare,warp,vps,ipv6,linux]
---

## 前言

[Cloudflare WARP][CloudflareWARP] (简称 WARP) 是 Cloud­flare 提供的一项基于 Wire­Guard 的网络流量安全及加速服务，能够让你通过连接到 Cloud­flare 的边缘节点实现隐私保护及链路优化。早年有很多小伙伴拿来当梯子工具来直接使用，应该很熟悉了。不过由于 Wire­Guard 数据传输使用的 UDP 协议，中国大陆的网络运营商会对其进行 QoS ，加上很多节点的 IP 被封锁，现在可以说几乎处于不可用的状态了。而对于自由网络的地区来说则没有这些限制，加上有开发者制作的开源工具可以生成通用的 Wire­Guard 配置文件，这使得我们可以在安装了某科学的上网工具的国外 VPS 上部署它，并实现一些骚操作。

本篇是相关知识科普和纯手动部署教程，小伙伴们可边学习边折腾。如果想直接部署可以使用[p3terx][p3terx]大神的 [Cloudflare WARP 一键安装脚本][warp]。

## WARP 的使用场景和局限性

使用场景：
- WARP 网络出入口均为双栈 (IPv4+IPv6)，因此单栈 VPS 云服务器可以连接到 WARP 网络来获取额外的网络连通性支持：
  + IPv6 Only VPS 可获得 IPv4 网络的访问能力，不再局限于 NAT64/DNS64 的束缚，能自定义任意 DNS 解析服务器，这对使用某科学的上网工具有奇效。
  + IPv4 Only VPS 可获得 IPv6 网络的访问能力，比如可作为 IPv6 Only VPS 的 SSH 跳板。
  + WARP 的 IPv6 网络的质量比 [HE IPv6 Tunnel Broker][tunnel-broker] 及 VPS 自带的都要好，很少绕路，可作为本机 IPv6 网络的替代。

- WARP 对外访问网络的出口 IP 被很多网站视为真实用户，获得原生 IP 或私人家庭住宅 IP 的效果，可以解除某些网站基于 IP 的封锁限制：
  + 解锁 Netflix 非自制剧 (几乎失效，原因详见《[为什么 WARP 解锁 Netflix 失效了？][unlock-netflix]》)
  + 解决 Google 搜索流量异常频繁跳出人机身份验证的问题
  + 解决无法打开 Google Scholar (谷歌学术) 403 访问限制的问题
  + 解决 Google 的 IP 定位漂移到中国(送中)，无法使用 YouTube Premium 的问题
  + 解锁 ChatGPT 访问限制(Access denied/1020)的问题 (仅限已开放服务的地区)。注意：实际使用不稳定，且有封号的风险。建议使用可安全访问并解锁 ChatGPT 的 VPS。

局限性：
- WARP 是以 NAT 的方式去访问外部网络，网络出口 IP 由 Cloudflare 根据地区随机分配且为多人共用，只能用于对外网络访问，不能从外部访问 VPS 。可以理解为连上了由 Cloudflare 提供的大内网。**TIPS:** 如果是需要可从外部访问 VPS 的公网 IPv6 地址可使用 HE IPv6 Tunnel Broker 解决，而 IPv4 地址只能加钱找商家买。
- 滥用严重。WARP 一直有着各种滥用和不规范使用的情况，比如注册僵尸账号、暴力发包网络攻击，随着时间的推移已经被很多 IP 评级机构列为不可信状态，大量 IP 段已经被拉黑。未来解除 IP 封锁限制的作用会越来越少，反而越来越多的网站对其设访问限制。
- IP 万人骑。WARP 多人共用一个网络出口 IP 是相当大的量级，在热门地区几万甚至十几万人共用一个 IP 都是有可能的。很多网站会设单 IP 访问频率限制来阻止网络攻击、防止资源滥用。所以使用 WARP 访问有严格限制的网站被屏蔽、无法注册、甚至封号未来可能会是常态。
- 随着使用人数的增加减速效果越来越明显，尤其在网络高峰时段，热门地区甚至到了几乎不可用的状态。

## WARP 设置教程

### 安装 WireGuard

既然 WARP 基于 Wire­Guard ，那么我们首先就需要安装 Wire­Guard 。  
博主暂时只写了 De­bian 的 Wire­Guard 详细安装方法：《[Debian Linux VPS WireGuard 安装教程][debian-wireguard]》，其它系统可以参考[官方文档][wireguard]来进行安装。  
如果你懒得手动安装，也可以使用一键脚本来安装：

```bash
bash <(curl -fsSL git.io/warp.sh) wg
```

### 使用 wgcf 生成 WireGuard 配置文件

[ViRb3/wgcf][ViRb3]是 Cloud­flare WARP 的非官方 CLI 工具，它可以模拟 WARP 客户端注册账号，并生成通用的 Wire­Guard 配置文件。

安装 wgcf

```bash
curl -fsSL git.io/wgcf.sh | sudo bash
```

注册 WARP 账户 (将生成 wgcf-account.toml 文件保存账户信息)

```bash
wgcf register
```

生成 Wire­Guard 配置文件 (wgcf-profile.conf)

```bash
wgcf generate
```

生成的两个文件记得备份好，尤其是 `wgcf-profile.conf`，万一未来工具失效、重装系统后可能还用得着。

## 编辑 WireGuard 配置文件

将配置文件中的节点域名 `engage.cloudflareclient.com` 解析成 IP。不过一般都是以下两个结果：

```
162.159.192.1
2606:4700:d0::a29f:c001
```

这样做是因为后面的操作要根据 VPS 所配备的网络协议的不同去选择要连接 WARP 的节点是 IPv4 或 IPv6 协议。

### IPv4 Only 服务器添加 WARP IPv6 网络支持
将配置文件中的 `engage.cloudflareclient.com `替换为 `162.159.192.1`，并删除 `AllowedIPs = 0.0.0.0/0`。即配置文件中 `[Peer]` 部分为：

```markdown
[Peer]
PublicKey = bmXOC+F1FxEMF9dyiK2H5/1SUtzH0JuVo51h2wPfgyo=
AllowedIPs = ::/0
Endpoint = 162.159.192.1:2408
```

>原理：`AllowedIPs = ::/0`参数使得 IPv6 的流量均被 Wire­Guard 接管，让 IPv6 的流量通过 WARP IPv4 节点以 NAT 的方式访问外部 IPv6 网络。

此外配置文件中默认的 DNS 是 `1.1.1.1`，博主实测其延迟虽然很低，但因为缺少了 ECS 功能所以解析结果并不理想。由于它将替换掉系统中的 DNS 设置 (`/etc/resolv.conf`)，建议小伙伴们请根据实际情况来进行替换，或者直接删除 DNS 这行。以下配置供参考：

```markdown
DNS = 8.8.8.8,8.8.4.4,2001:4860:4860::8888,2001:4860:4860::8844
```

### IPv6 Only 服务器添加 WARP IPv4 网络支持

将配置文件中的 `engage.cloudflareclient.com` 替换为 `[2606:4700:d0::a29f:c001]`，并删除 `AllowedIPs = ::/0`。即配置文件中 `[Peer]` 部分为：

```markdown
[Peer]
PublicKey = bmXOC+F1FxEMF9dyiK2H5/1SUtzH0JuVo51h2wPfgyo=
AllowedIPs = 0.0.0.0/0
Endpoint = [2606:4700:d0::a29f:c001]:2408
```

>原理：`AllowedIPs = 0.0.0.0/0`参数使得 IPv4 的流量均被 Wire­Guard 接管，让 IPv4 的流量通过 WARP IPv6 节点以 NAT 的方式访问外部 IPv4 网络。

此外配置文件中默认的 DNS 是 `1.1.1.1`，由于是 IPv4 地址，故查询请求会经由 WARP 节点发出。由于它将替换掉系统中的 DNS 设置 (`/etc/resolv.conf`)，为了防止当节点发生故障时 DNS 请求无法发出，建议替换为 IPv6 地址的 DNS 优先，或者直接删除 DNS 这行。以下配置供参考：

```markdown
DNS = 2001:4860:4860::8888,2001:4860:4860::8844,8.8.8.8,8.8.4.4
```

### WARP 双栈全局网络置换

WARP 双栈全局网络是指 IPv4 和 IPv6 都通过 WARP 网络的出口对外进行网络访问，实际上默认生成的 Wire­Guard 配置文件就是这个效果。不过默认的配置文件没有外部对本机 IP 访问的相关路由规则，一旦直接使用 VPS 就会直接失联，所以我们还需要对配置文件进行修改。路由规则需要添加在配置文件的 [Interface] 和 [Peer] 之间的位置，以下是路由规则示例：

```markdown
[Interface]
...
PostUp = ip -4 rule add from <替换IPv4地址> lookup main
PostDown = ip -4 rule delete from <替换IPv4地址> lookup main
PostUp = ip -6 rule add from <替换IPv6地址> lookup main
PostDown = ip -6 rule delete from <替换IPv6地址> lookup main
[Peer]
...
```

>**TIPS:** 包含`<>`(尖括号)的部分一起替换掉，这只是为了看起来明显。

替换配置中的 IP 地址部分为 VPS 的公网 IP 地址，如果 IDC 提供的是 VPC 内网方案则需要替换为内网 IP 。像 AWS 、Azure 、Google Cloud 、Or­a­cle Cloud 等大厂都是 VPC 内网方案，内网地址一般会在网页面板有提供。如果不确定是哪种网络方案，输入 `ip a | grep <公网IP地址>` 看是否有显示，没有那么就说明是 VPC 内网方案。

## 启用 WireGuard 网络接口
- 将 Wire­Guard 配置文件复制到 `/etc/wireguard/` 并命名为 `wgcf.conf`。

```bash
sudo cp wgcf-profile.conf /etc/wireguard/wgcf.conf
```

- 开启网络接口（命令中的 `wgcf` 对应的是配置文件 `wgcf.conf` 的文件名前缀）。

```bash
sudo wg-quick up wgcf
```

- 执行执行`ip a`命令，此时能看到名为`wgcf`的网络接口

- 执行以下命令检查是否连通。同时也能看到正在使用的是 Cloud­flare 的网络。

```bash
# IPv4 Only VPS
curl -6 ip.p3terx.com
# IPv6 Only VPS
curl -4 ip.p3terx.com
```

- 测试完成后关闭相关接口，因为这样配置只是临时性的。

```bash
sudo wg-quick down wgcf
```

- 正式启用 Wire­Guard 网络接口

```bash
# 启用守护进程
sudo systemctl start wg-quick@wgcf
# 设置开机启动
sudo systemctl enable wg-quick@wgcf
```

## 其它操作和说明

### DNS 优化和 IPv4/IPv6 优先级设置

>**TIPS:** 以下设置仅针对操作系统 DNS ，代理软件应单独设置内置的 DNS 和 IP 分流策略，具体参考相关软件文档中的 DNS 和路由部分。原因参见《优先级设置在特殊场景中的局限性》章节。

当访问的网站是双栈且服务器也是双栈，默认情况下 IPv6 优先级高于 IPv4，应用程序优先使用 IPv6 地址。  
理论上应该是如下情况：
- IPv4 Only 服务器优先通过新增的 WARP IPv6 网络去访问外部网络。
- IPv6 Only 服务器优先通过原来的 IPv6 网络去访问外部网络。

然而 WARP 的情况有点特殊，可能与 Wire­Guard 的路由规则有关，所以现实的情况可能是：
- IPv4 Only 服务器优先通过原来的 IPv4 网络去访问外部网络。
- IPv6 Only 服务器优先通过原来的 IPv6 网络去访问外部网络。

如果你对于这个设定不满意，可以根据实际的需求手动去设置 IPv4 与 IPv6 的优先级。

### 验证 IPv4/IPv6 优先级
执行 `curl ip.p3terx.com` 命令，显示 IPv4 地址则代表 IPv4 优先，否则为 IPv6 优先。

### 设置 IPv4 优先
编辑 `/etc/gai.conf` 文件，在末尾添加下面这行配置：

```bash
precedence ::ffff:0:0/96  100
```

一键添加命令如下：

```bash
# IPv4 优先
grep -qE '^[ ]*precedence[ ]*::ffff:0:0/96[ ]*100' /etc/gai.conf || echo 'precedence ::ffff:0:0/96  100' | sudo tee -a /etc/gai.conf
```

### 设置 IPv6 优先

编辑 `/etc/gai.conf` 文件，在末尾添加下面这行配置：

```bash
label 2002::/16   2
```

一键添加命令如下：

```bash
# IPv6 优先
grep -qE '^[ ]*label[ ]*2002::/16[ ]*2' /etc/gai.conf || echo 'label 2002::/16   2' | sudo tee -a /etc/gai.conf
```

### 优先级设置在特殊场景中的局限性

在 VPS 上使用某科学的上网工具时出站的 IPv4/​IPv6 优先级还取决于科学工具的 DNS 策略和分流路由策略。  
比如某些路由器上的某科学的上网工具客户端不会发送域名给服务端做 DNS 解析，而是在本地直接将域名解析为 IP 并通过服务端直接向已解析的 IP 发起连接，那么可能因为路由器 DNS 屏蔽了 AAAA 记录就只会去访问 IPv4 网络。解决方法是开启某科学的上网工具服务端的流量探测 (sniff­ing) 功能，并添加相关路由规则进行分流处理，服务端会从请求数据中嗅探出域名并进行二次 DNS 解析后对网络流量进行重定向，就比如可以将本身发往网站 IPv4 服务器的流量重定向到 IPv6 服务器（前提是这个网站使用了 IPv6 地址）。

### 科学上网代理工具 IPv4/IPv6 分流方法

有关某科学的上网工具的 DNS 和路由分流设置有很多资料，小伙伴们可自行咕鸽搜索。每个人的需求不一样，不可能以偏概全，最好是看相关工具的文档，所以博主在这里就不多赘述了。  
推荐一篇最早提及相关设置的教程作为参考：《[某科学的上网工具白话文手册 - 指定出站 IP][outbounds]》，结合相关文档食用效果更佳。

### Cloudflare WARP 网速测试

使用 speedtest.net 提供的 CLI 工具测试通过 WARP 访问外部网络的极限网速。

- 安装 [Ookla Speedtest CLI][speedtest]

```bash
curl -fsSL git.io/speedtest-cli.sh | sudo bash
```

- 执行`speedtest`命令测速。

博主随便拿了一个吃灰的 LXC 小鸡进行测试，发现即使用的是 wireguard-go 其网速依然很猛，几轮测试下来速度都在 500M 上下。可以预见的是这个速度应该远未达到 WARP 的极限，不过随着这篇教程的发布，之后是否还是这么理想就不得而知了。

## 写在最后

Cloud­flare 一直以来为广大人民群众免费提供优秀的网络服务，希望大家善待它， _不要肆意滥用_。

****

[原文出处](https://p3terx.com/archives/use-cloudflare-warp-to-add-extra-ipv4-or-ipv6-network-support-to-vps-servers-for-free.html)


[CloudflareWARP]:https://developers.cloudflare.com/cloudflare-one/connections/connect-devices/warp/
[p3terx]:https://p3terx.com/archives/use-cloudflare-warp-to-add-extra-ipv4-or-ipv6-network-support-to-vps-servers-for-free.html
[warp]:https://github.com/P3TERX/warp.sh
[unlock-netflix]:https://p3terx.com/archives/why-is-it-getting-harder-to-unlock-netflix-with-warp.html
[tunnel-broker]:https://p3terx.com/archives/use-he-tunnel-broker-to-add-public-network-ipv6-support-to-ipv4-vps-for-free.html
[debian-wireguard]:https://p3terx.com/archives/debian-linux-vps-server-wireguard-installation-tutorial.html
[wireguard]:https://www.wireguard.com/install/
[ViRb3]:https://github.com/ViRb3/wgcf
[outbounds]:https://toutyrater.github.io/app/netflix.html#%E6%8C%87%E5%AE%9A%E5%87%BA%E7%AB%99-ip
[speedtest]:https://www.speedtest.net/apps/cli