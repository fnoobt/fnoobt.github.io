---
title:  QoS 简介
author: fnoobt
date: 2025-04-26 20:17:00 +0800
categories: [Network,路由协议]
tags: [network,qos]
---

## QoS优先级

|  QoS标记 |   DSCP值  | 流量类型  |        承载业务                            |
|----------|----------|-----------|--------------------------------------------|
| CS7      | 56       | 网内控制   | 网络内部协议控制流量（如OSPF、BGP等路由协议） |
| CS6      | 48       | 网间控制   | 跨网络的关键控制流量（如LLDP、STP协议）     |
| EF       | 46       | 加速转发   | 实时业务（如VoIP、视频通话）    |
| AF4类    | 34 36 38 | 保证转发   | 高优先级业务（如语音信令、关键专线）     |
| AF3类    | 26 28 30 | 保证转发   | 中高优先级业务（如IPTV直播、视频监控） |
| AF2类    | 18 20 22 | 保证转发   | 中等优先级业务（如VOD点播、普通视频流）   |
| AF1类    | 10 12 14 | 保证转发   | 普通数据业务（如邮件、文件传输） |
| BE       | 0        | 尽力而为   | 普通互联网流量（如网页浏览）    |

完整优先级链（高优先级 --> 低优先级）：
​CS7 > CS6 > EF > AF41 > AF42 > AF43 > AF31 > AF32 > AF33 > AF21 > AF22 > AF23 > AF11 > AF12 > AF13 > BE​

- 每个AF类（AF1~AF4）包含三个子类（如AF11、AF12、AF13），优先级顺序为 ​​AFx1 > AFx2 > AFx3​​（x为1~4）
- 优先级可能因设备队列策略而异。例如，若EF队列配置为严格优先级（PQ），则实时流量绝对优先；AF类若采用加权公平队列（WFQ），则按权重分配带宽

> 虽然 CS7/CS6 在**重要性**上可能更高（为了网络稳定），但 EF 在**调度**上通常享有绝对优先权，一旦有 EF 流量就优先发送。
{: .prompt-warning } 

DSCP 标记本身只是一个标签，实际的 QoS 行为（带宽分配、调度优先级、丢包策略）取决于网络路径上各个路由器和交换机的具体配置。

### 网络调度算法

1. PQ（优先级队列，Priority Queuing）
严格优先级调度，高优先级队列（如路由协议报文）优先处理，低优先级队列需等待高优先级队列空闲。牺牲低优先级流量的公平性，保证高优先级的网络，适用于时延敏感型业务。
2. WFQ（加权公平队列，Weighted Fair Queuing）​
基于权重分配带宽，按流（Flow）分类，自动平衡不同数据流的带宽。每个队列的带宽比例由权重决定，确保高优先级流量获得更多资源，同时避免低优先级流量“饿死”，适用于网络拥塞管理、流量整形。
3. SFQ（随机公平队列，Stochastic Fairness Queueing）​
通过哈希算法将流量分散到多个子队列，轮询调度以避免单一流垄断带宽。适用于短报文优先场景，减少抖动。

## ubuntu 配置 QoS
以如下的组网需求为例：
1. 出方向调度模式为 EF 绝对保证，最大带宽为接口带宽 40%，采用PQ调度算法；
2. CS7、CS6、AF4x、AF3x、AF2x、AF1x之间按照优先级进行比例分配但不可以抢占 BE 带宽，比例为 4:3:3:3:2:1，最大保证带宽为接口带宽 90%，采用WFQ调度算法；
3. BE流量最小保证带宽为接口带宽的 5%，最大保证带宽为接口带宽的 100%。

基于 `tc` 配置QoS，HTB结构如下所示：

```md
# HTB结构
#    root (1:)
#    ├── 1:10 EF类（PQ调度算法）           上限带宽40%
#    ├── 1:20 Assured组父类（WFQ调度算法） 上限带宽90%
#    │   ├── 1:21 CS7           权重 4
#    │   ├── 1:22 CS6           权重 3
#    │   ├── 1:23 AF41          权重 3
#    │   ├── 1:24 AF31          权重 3
#    │   ├── 1:25 AF21          权重 2
#    │   └── 1:26 AF11          权重 1
#    └── 1:30 BE类            保证带宽5%，上限带宽100%
```

### 环境准备
```bash
# 安装依赖
sudo apt update
sudo apt install -y iproute2 tc

# 启用IPv6转发
sudo sysctl -w net.ipv6.conf.all.forwarding=1

# 加载内核模块
sudo modprobe sch_htb
sudo modprobe sch_sfq
```

### 配置脚本
```bash
#!/bin/bash

# --- 基础配置 ---
IFACE="eth0"           # 替换实际的接口名称
TOTAL_BW=1000          # 替换为实际的接口总带宽
BW_UNIT="mbit"         # 单位，kbit, mbit, gbit

# 带宽分配
BW_EF_CEIL=$((TOTAL_BW*40/100))       # EF 上限带宽40%
BW_ASSURED_CEIL=$((TOTAL_BW*90/100))  # Assured 组上限带宽90%
BW_BE_GUARANTEE=$((TOTAL_BW*5/100))   # BE 保证带宽5%
BW_BE_CEIL=$TOTAL_BW                  # BE 上限带宽100%

# Assured 组权重比例(CS7:CS6:AF4x:AF3x:AF2x:AF1x = 4:3:3:3:2:1)
WEIGHTS=(4 3 3 3 2 1)
for w in "${WEIGHTS[@]}"; do TOTAL_WEIGHTS=$((TOTAL_WEIGHTS+w)); done

# 计算 Assured 组各等级带宽
calc_bw() {
    #使用 bc 进行浮点运算，tc 通常处理这个问题
    local rate=$(echo "scale=2; $1 * $2 / $3" | bc)
    # 确保最低速率为 1kbit
    local is_zero=$(echo "$rate == 0" | bc)
    if [ "$is_zero" -eq 1 ]; then
        echo "1kbit" # 返回最小值
    else
        # 检查 rate 是整数还是浮点数
        if [[ "$rate" == *"."* ]]; then
            printf "%.2f%s\n" $rate $BW_UNIT # 保持浮动单位
        else
            echo "${rate}${BW_UNIT}" # 使用带单位的整数
        fi
    fi
}

BW_CS7=$(calc_bw $BW_ASSURED_CEIL ${WEIGHTS[0]} $TOTAL_WEIGHTS)
BW_CS6=$(calc_bw $BW_ASSURED_CEIL ${WEIGHTS[1]} $TOTAL_WEIGHTS)
BW_AF41=$(calc_bw $BW_ASSURED_CEIL ${WEIGHTS[2]} $TOTAL_WEIGHTS)
BW_AF31=$(calc_bw $BW_ASSURED_CEIL ${WEIGHTS[3]} $TOTAL_WEIGHTS)
BW_AF21=$(calc_bw $BW_ASSURED_CEIL ${WEIGHTS[4]} $TOTAL_WEIGHTS)
BW_AF11=$(calc_bw $BW_ASSURED_CEIL ${WEIGHTS[5]} $TOTAL_WEIGHTS)

# --- DSCP 值（TC 过滤器的十六进制） ---
declare -A DSCP_MAP=(
    ["EF"]="0x2e"   # 加速转发 (46)
    ["CS7"]="0x38"  # 网内控制 (56)
    ["CS6"]="0x30"  # 网间控制 (48)
    ["AF41"]="0x22" # 高优先级 (34)
    ["AF31"]="0x1a" # 中高优先级 (26)
    ["AF21"]="0x12" # 中优先级 (18)
    ["AF11"]="0x0a" # 普通业务 (10)
    ["BE"]="0x00"   # 尽力而为 (0)
)

# IPv4需左移2位
declare -A IPv4_TOS_MAP=(
    ["EF"]="0xb8"   # 0x2e << 2 = 0xB8
    ["CS7"]="0xe0"  # 0x38 << 2 = 0xE0
    ["CS6"]="0xc0"  # 0x30 << 2 = 0xC0
    ["AF41"]="0x88" # 0x22 << 2 = 0x88
    ["AF31"]="0x68" # 0x1a << 2 = 0x68
    ["AF21"]="0x48" # 0x12 << 2 = 0x48
    ["AF11"]="0x28" # 0x0a << 2 = 0x28
    ["BE"]="0x00"   
)

# --- TC 命令 ---

# 清除之前的配置
sudo tc qdisc del dev $IFACE root 2> /dev/null

# --- 创建HTB层次结构 ---

# 创建 HTB 根队列，默认流量进入 BE 类 (1:30)
sudo tc qdisc add dev $IFACE root handle 1: htb default 30 

# 1. EF 类（最高优先级 - PQ 模拟）
#    rate 定义最低保证带宽。ceil 定义允许使用的最大可用带宽。
#    prio 定义优先级，优先级 0 最高。quantum 手动设置为 MTU 大小。
sudo tc class add dev $IFACE parent 1: classid 1:10 htb \
    rate 1kbit ceil ${BW_EF_CEIL}${BW_UNIT} prio 0 quantum 1500
echo "Added EF class (1:10) ceil ${BW_EF_CEIL}${BW_UNIT} prio 0"

# 2. Assured 组父类（管理 Assured 组的集体带宽）
#    rate 定义组内总的最低保证带宽。ceil 定义组内总允许使用的最大可用带宽。
#    优先级 1 低于 EF。
sudo tc class add dev $IFACE parent 1: classid 1:20 htb \
    rate ${BW_ASSURED_CEIL}${BW_UNIT} ceil ${BW_ASSURED_CEIL}${BW_UNIT} prio 1 quantum 15000
echo "Added Assured Group parent class (1:20) rate ${BW_ASSURED_CEIL}${BW_UNIT} ceil ${BW_ASSURED_CEIL}${BW_UNIT} prio 1"

# 3. BE 类（最低优先级组）
#    优先级 2 低于 Assured 组。
sudo tc class add dev $IFACE parent 1: classid 1:30 htb \
    rate ${BW_BE_GUARANTEE}${BW_UNIT} ceil ${BW_BE_CEIL}${BW_UNIT} prio 2 quantum 1500
echo "Added BE class (1:30) rate ${BW_BE_GUARANTEE}${BW_UNIT} ceil ${BW_BE_CEIL}${BW_UNIT} prio 2"

# 创建 Assured 组的子类
#    Assured 组成员类（共享优先级 1，类似 WFQ，基于 rate）
#    rate 是其在组内的保证带宽，ceil 允许借用至父类的 ceil。
CLASS_IDS=(21 22 23 24 25 26)
BANDWIDTHS=($BW_CS7 $BW_CS6 $BW_AF41 $BW_AF31 $BW_AF21 $BW_AF11)
for i in {0..5}; do
    # 添加具有计算速率、组上限、优先级 1 和手动量程的 HTB 类
    sudo tc class add dev $IFACE parent 1:20 classid 1:${CLASS_IDS[$i]} htb \
        rate ${BANDWIDTHS[$i]} ceil ${BW_ASSURED_CEIL}${BW_UNIT} prio 1 \
        quantum $((1500*${WEIGHTS[$i]}))

    # 添加WFQ队列（通过quantum实现权重），perturb 15 增加了散列的随机性，避免多个流同步
    sudo tc qdisc add dev $IFACE parent 1:${CLASS_IDS[$i]} handle 1${CLASS_IDS[$i]}: \
        sfq quantum $((1514*${WEIGHTS[$i]})) perturb 15
done

# --- 添加叶子队列Qdisc ---

# EF使用PFIFO，FIFO 队列适合于 HTB 父队列的严格优先级处理
sudo tc qdisc add dev $IFACE parent 1:10 handle 10: pfifo limit 1000
# BE使用FQ_CODEL（结合了公平队列FQ、受控延迟CoDel、主动队列管理AQM）
sudo tc qdisc add dev $IFACE parent 1:30 handle 30: fq_codel       

# --- 流量分类规则 ---

# 基于 DSCP 对数据包进行分类，使用u32 过滤器匹配 IP 头中的 DSCP 字段（掩码 0xfc 忽略 ECN 位）
FILTER_PRIO=10
for key in "${!DSCP_MAP[@]}"; do
    case $key in
        "EF") flowid=1:10 ;;
        "CS7") flowid=1:21 ;;
        "CS6") flowid=1:22 ;;
        "AF41") flowid=1:23 ;;
        "AF31") flowid=1:24 ;;
        "AF21") flowid=1:25 ;;
        "AF11") flowid=1:26 ;;
        "BE") flowid=1:30 ;;
        *)    echo "Skipping unknown key: $key"; continue ;;   # 如果未找到 key 则跳过
    esac

    # IPv6匹配DSCP
    # prio 过滤规则的匹配顺序，优先检查高优先级流量的规则
    # flowid 将匹配到的数据包导向相应的 HTB 类
    sudo tc filter add dev $IFACE parent 1: protocol ipv6 \
        prio $FILTER_PRIO u32 \
        match ip6 dscp ${DSCP_MAP[$key]} 0xfc \
        flowid $flowid
    # 在 tc 命令后立即检查退出状态 ($?)（可选，但建议）
    if [ $? -ne 0 ]; then
        echo "Error adding IPv6 filter for $key (DSCP ${DSCP_MAP[$key]}) at prio $FILTER_PRIO" >&2
    else
        echo "Added IPv6 filter prio $FILTER_PRIO for $key (DSCP ${DSCP_MAP[$key]}) -> $flowid"
    fi
    ((FILTER_PRIO++))
    
    # IPv4匹配TOS 字段（DSCP左移2位）
    sudo tc filter add dev $IFACE parent 1: protocol ipv4 \
        prio $FILTER_PRIO u32 \
        match ip tos ${IPv4_TOS_MAP[$key]} 0xfc \
        flowid $flowid
    if [ $? -ne 0 ]; then
        echo "Error adding IPv4 filter for $key (TOS ${IPv4_TOS_MAP[$key]}) at prio $FILTER_PRIO" >&2
    else
        echo "Added IPv4 filter prio $FILTER_PRIO for $key (TOS ${IPv4_TOS_MAP[$key]}) -> $flowid"
    fi

    ((FILTER_PRIO++))
done

echo "--- TC Configuration Complete for $IFACE ---"

# --- 验证命令 ---
echo "TC configuration completed. Verification commands:"
echo "1. Show classes and stats: tc -s -d class show dev $IFACE"
echo "2. Show qdiscs and stats: tc -s -d qdisc show dev $IFACE"
echo "3. Show filters: tc filter show dev $IFACE parent 1:"
echo "4. Real-time bandwidth: nload -u M $IFACE"
echo "5. Send ICMPv6 with EF flag: sudo ping6 -Q 0xb8 <IPv6_Destination> -c 10" # Note: ping -Q uses TOS value (DSCP<<2)
echo "6. Send ICMPv4 with EF flag: sudo ping -Q 0xb8 <IPv4_Destination> -c 10"
# Tshark command depends on whether you expect IPv4 or IPv6 mostly
echo "7. IPv6 DSCP distribution: sudo tshark -i $IFACE -Y ipv6 -T fields -e ipv6.dsfield.dscp"
echo "8. IPv4 DSCP distribution: sudo tshark -i $IFACE -Y ip -T fields -e ip.dsfield.dscp"
```