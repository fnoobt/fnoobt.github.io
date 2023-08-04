---
title: 在CentOS上面利用Wgcf使用CF WARP作为出口
author: fnoobt
date: 2021-09-19 11:44:00 +0800
categories: [VPS,服务搭建]
tags: [cloudflare,warp,wgcf,vps,ipv6,linux]
---

## 下载WGCF

首先需要通过[WGCF][wgcf]来在服务器上面连接CF WARP作为出口。也可为没有IPv6的VPS添加IPv6。

下载wgcf文件：

```bash
mkdir wgcf
cd wgcf
wget -O wgcf https://github.com/ViRb3/wgcf/releases/download/v2.2.15/wgcf_2.2.15_linux_amd64
wget -O wgcf https://down.loukky.com/wgcf/wgcf_2.2.15_linux_amd64
chmod +x wgcf
```

>**TIP:**如需要其他平台请自行通过github替换下载链接
{: .prompt-tip }

初次使用需要注册用户并生成配置文件：

```bash
./wgcf register
./wgcf generate
```

>**TIP:**如果提示`429`的话就多试几次。
{: .prompt-tip }

随后你就可以在程序目录中找到`wgcf-account.toml`和`wgcf-profile.conf`两个新生成的文件。  
前者是你的`WARP`账户信息，如果你有WARP+账户可以替换成你自己的账户；后者就是`WireGuard`的配置文件了，下载到本地保存。

其中可以自己把`engage.cloudflareclient.com`解析成IP，对Endpoint修改成ipv4或者ipv6保存即可。

修改`wgcf-profile.conf`配置文件：

![Wgcf Conf](/assets/img/commons/vps/wgcf_config.png)

>**注意：**WG连接后是内核层级的软件，会建立自己的虚拟网卡，且WARP客户端均为内网NAT地址，当双栈流量均被WG接管后我们就无法再从原有的IP连接到服务器了。因此在IPv4与IPv6之间我们必须做一个取舍，以防这样的情况发生。
{: .prompt-danger }

修改配置文件就两种情况：
  - 1.`wgcf-profile.conf`中11行Endpoint修改为162.159.192.1:2408，删除掉第9行`AllowedIPs = 0.0.0.0/0`接管本地IPv4路由的配置
  - 2.`wgcf-profile.conf`中11行Endpoint修改为[2606:4700:d0::a29f:c001]:2408，删除掉第10行`AllowedIPs = ::/0`接管本地IPv6路由的配置

修改DNS建议同时添加上IPV4与IPV6的DNS，这里使用的Cloudflare的DNS。

```
For IPv4: 1.1.1.1 and 1.0.0.1
For IPv6: 2606:4700:4700::1111 and 2606:4700:4700::1001
```

同时记得如果需要`cf warp`的`IPV6`就删掉 `AllowedIPs = 0.0.0.0/0`，如果需要`cf warp`的`ipv4`就删掉 `AllowedIPs = ::/0`。
不要两个同时启用，不然会导致你vps原本的ip地址无法连接上去。

最后保存配置文件，修改文件名为`wgcf.conf`。

## 安装wireguard客户端。

centos7安装wireguard客户端：

```bash
sudo yum install epel-release elrepo-release
sudo yum install yum-plugin-elrepo
sudo yum install kmod-wireguard wireguard-tools
```

安装好以后需要重启VPS。更多命令可以参考[官方网站][wireguard]

把刚刚修改好的配置文件`wgcf.conf`上传至`/etc/wireguard`，然后继续：

```bash
#加载内核模块
modprobe wireguard
#检查WG模块加载是否正常
lsmod | grep wireguard
```

最后开关隧道的命令为：

```bash
#开启隧道
sudo wg-quick up wgcf
#关闭隧道
sudo wg-quick down wgcf
```

启用守护进程：

```bash
sudo systemctl start wg-quick@wgcf
```

开机自动运行：

```bash
sudo systemctl enable wg-quick@wgcf
```

执行以下命令检查是否连通。同时也能看到正在使用的是 Cloud­flare 的网络。

```bash
curl -6 ip.p3terx.com
```

如果VPS出口没有走ipv6的话，编辑/etc/gai.conf文件（没有的话就新建），修改为以下内容：

```
label ::1/128 0
label ::/0 1
label fd01::/16 1
label 2002::/16 2
label ::/96 3
label ::ffff:0:0/96 4
label fec0::/10 5
label fc00::/7 6
label 2001:0::/32 7
precedence ::1/128 50
precedence ::/0 40
precedence fd01::/16 40
precedence 2002::/16 30
precedence ::/96 20
precedence ::ffff:0:0/96 10
```

## 解锁Netflix

Xray使用IPv6解锁Netflix。  
在Xray配置文件中增加Outbount和routing配置。  
v2-ui需要在前台面板设置里添加。

Outbount指定IPV6出口：

```json
      {
      "protocol": "freedom",
      "settings": {
         "domainStrategy": "UseIPv6"         
      },
      "tag": "IP-V6"
    },
```    

routing路由NETFLIX数据到IPV6出口:

```json
      {
        "type":"field",
        "domain": [
          "geosite:netflix"
        ],
        "inboundTag":  [
          "all-in"
         ],
        "outboundTag": "IP-V6"
      },
```

****

本文参考

> 1. [ZJ's Blog](https://www.zhangjun.sh.cn/linux/centos_wgcf_cf_ipv6.html)  
> 2. [loukky](https://loukky.com/archives/1440#gsc.tab=0)


[wgcf]:https://github.com/ViRb3/wgcf
[wireguard]:https://www.wireguard.com/install/