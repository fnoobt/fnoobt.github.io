---
title: FRP简单入门安装配置教程
author: fnoobt
date: 2021-06-24 09:56:00 +0800
categories: [VPS,服务搭建]
tags: [linux,vps,frp]
img_path: '/assets/img/commons/vps/'
---


虽然现在宽带速度都很快，但对于电脑玩家来说，最大的问题是**没有公网IP**！这使得想要在外访问家里的电脑、NAS、树莓派、摄像头等网络设备或远程控制等，都无法轻松实现。

这时你就需要一款内网穿透工具来让外网与你家内网建立起连接，实现无公网 IP 的远程访问了。[Frp][frp]是一款流行的跨平台开源免费内网穿透工具，支持 Windows、macOS 与 Linux。你只需一台快速稳定的 VPS 服务器即可愉快地进行内网穿透，实现家中设备公网直接访问了

## 安装要求

很多地方宽带都已不再提供公网 IP 了，如果你想家里的设备如 NAS、电脑可在外网访问，那么只能通过内网穿透工具实现。考虑到安全和稳定性，最优方案是买一台 VPS 服务器用于内网穿透。

在内网也需要一台机器用于运行 Frp 的客户端，可以是 Windows 电脑、Mac，或者是树莓派、NAS 等 Linux 设备。

市面上也有其他方案，比如花生壳相关软硬件产品，免费限制很多，付费价格贵，浪不起来。其他小公司的产品安全性又无法保证，那还不如自己买 VPS 建一个，有自己的服务器，日后各种建站的玩法还更多更实用，还能顺便学学 Linux 呢。

## FRP是什么

内网穿透工具有很多，其中 Frp (Fast Reverse Proxy) 是比较流行的一款。FRP 是一个免费开源的用于内网穿透的反向代理应用，它支持 TCP、UDP 协议， 也为 http 和 https 协议提供了额外的支持。你可以粗略理解它是一个中转站，帮你实现 公网 ←→ FRP(服务器) ←→ 家庭内网 的连接，让内网里的设备也可以被公网访问到。

![Architecture](frp_architecture.png)
_Frp架构原理示意图_

而目前 FRP 还推出了“点对点穿透”的试验性功能，连接成功后可以让公网设备直接跟内网设备**点对点**传输，数据流不再经过 VPS 中转，这样可以不受服务器带宽的限制，传输大文件会更快更稳定。当然，此功能并不能保证在你的网络环境 100% 可用，而且还要求访问端也得运行 FRP 客户端 (因此目前手机是无法实现的，只有电脑可以)。由于实现条件较多，所以有文件传输需求的朋友还是建议买**带宽稍大一点的 VPS** 会比较省心。

## Frp安装配置教程

现在假设你已经有一台 VPS 服务器了，那么只需按照下面的步骤，一步一步来来即可搞定 FRP 的安装和配置。当然，这里涉及到一些 Linux 基础操作命令，如果完全未接触过的朋友，可以找一些「Linux 入门教程」先了解一下。

### 1、Frp服务器端安装配置

FRP 使用 Go 语言开发，可以支持 Windows、Linux、macOS、ARM 等多平台部署。FRP 安装非常容易，只需下载对应系统平台的软件包并解压就可用了。这里以 Linux 系统为例：

```bash
export FRP_VERSION=0.51.0
sudo mkdir -p /etc/frp
cd /etc/frp
sudo wget "https://github.com/fatedier/frp/releases/download/v${FRP_VERSION}/frp_${FRP_VERSION}_linux_amd64.tar.gz"
sudo tar xzvf frp_${FRP_VERSION}_linux_amd64.tar.gz
sudo mv frp_${FRP_VERSION}_linux_amd64/* /etc/frp
```

- 其中，第一行等号后面的 `0.51.0` 是 frp 的版本号 (截稿为止最新版本，不一定要最新版本)。你安装的时候可以到官网查看下有没更新的版本，只需将新版本的号码替换掉 `0.51.0` 即可。
- FRP 默认提供了 2 个服务端配置文件，一个是简化版的 `frps.ini`，另一个是完整版的 `frps_full.ini`。初学者只需用简版配置即可，在简版 `frps.ini` 配置文件里，默认设置了监听端口为 `7000`，你可以按需修改它。

#### 服务器端配置文件的编写

准备编写服务器端配置文件 `frps.ini`
```bash
cd /etc/frp
vim frps.ini
```

> 普通用户选择`/usr/local/etc/frp`{: .filepath}路径
{: .prompt-tip }

将以下内容写入配置文件：

```markdown
[common]
bind_addr = 0.0.0.0
bind_port = 7000
vhost_http_port = 8000		#http服务映射端口
vhost_https_port = 8001		#https服务映射端口
dashboard_port = 7500		# frps控制台端口号
privilege_token = 123456	#可自定义，需与客户端（树莓派端）保持一致才可连接
dashboard_user = root		# frps控制台登陆用户名
dashboard_pwd = root		#frps控制台登陆密码
log_file = ./frps.log
log_level = info
log_max_days = 3
max_pool_count = 5
authentication_timeout = 900
tcp_mux = true
```

#### 防火墙和安全组开放指定的端口：

>**注意：**请一定要记住，你需要将服务器的系统防火墙，以及阿里云、腾讯云后台里找到“安全组策略”的相关配置，设置 `7000` 或你修改过的对应端口的**允许入站和出站**，否则会一直连接不上的哦！！！这个切记！！
{: .prompt-tip }

#### 启动 FRP 服务端

```bash
./frps -c ./frps.ini
```

如服务器使用 Win 系统，假设解压到 c:\frp 文件夹，那么只需这样启动：

```bash
c:\frp\frps.exe -c c:\frp\frps.exe
```

此时便可以通过任意一台能上网的电脑在浏览器中访问位于服务器端的frps控制台了。
>**注意：**访问地址为：frps端公网IP:7500，如：1.2.3.4:7500。
{: .prompt-tip }

#### 设置开机自启动

编写 `frps.service` 文件(如相关文件夹不存在，则需要自行创建):

```bash
vim /etc/systemd/system/frps.service
```

`frps.service`文件内容如下：

```markdown
[Unit]
Description=Frp Server Service
After=network.target nss-lookup.target

[Service]
Type=simple
#启动服务的命令（此处写你的frps的实际安装目录）
ExecStart=/etc/frp/frps -c /etc/frp/frps.ini

[Install]
WantedBy=multi-user.target
```

最后，启动 frps 并设置开机自启动

```bash
#打开自启动
systemctl enable frps.service
#启动frps
systemctl start frps.service
```

如果要重启应用，可以输入，`sudo systemctl restart frps`  
如果要停止应用，可以输入，`sudo systemctl stop frps`  
如果要查看应用的日志，可以输入，`sudo systemctl status frps`

### 2、配置 Frp 客户端

设置好服务器上 Frp 服务端后，我们就需要在内网的机器上安装 Frp 的客户端了。 Frp 的客户端程序 frpc (frpc.exe) 与服务器端都在同一个压缩包里， 我们同样下载对应系统版本的软件包。

你可以将 Frp 客户端安装在内网的 Windows 电脑、Linux 设备 (比如树莓派) 或者 NAS，甚至部分路由器等设备上。Linux 客户端的安装和启动与服务器端没有太多区别，只是对应运行程序是 `frpc` 而不是 `frps`。

为了简单起见，我们这里以 Windows 电脑来安装 Frp 客户端，因为 Frp 是绿色程序，下载软件包回来解压后，启动 `frpc.exe` 即可。

但在启动前，我们需要先修改配置文件，我们以配置“Windows 远程桌面控制”以及“群晖 NAS 管理界面”为例。假设你的 FRP 服务端所在的 VPS 公网 IP 为 `1.2.3.4`， 而客户端是 Win 电脑，我们来修改 `frpc.ini` 配置文件：

```markdown
[common]
# server_addr 为 FRP 服务端 (VPS 服务器) 的公网 IP
server_addr = 1.2.3.4
server_port = 7000
privilege_token = 123456		#可自定义，需与服务器端（云主机端）保持一致才可连接

[DSM]
type = tcp
local_ip = 192.168.1.40 #群晖 NAS 在局域网中的内网 IP
local_port = 5000
remote_port = 7001

[RDP]
type = tcp
local_ip = 192.168.1.30 #电脑在局域网中的内网 IP (如是本机，也可使用 127.0.0.1)
local_port = 3389
remote_port = 7002
```

这样就在本地上新增了`DSM`和`RDP`两个可供公网访问的服务了 (它们名称可以自己取)，这里分别对应内网的群晖 NAS 的后台管理界面和 PC 远程桌面。如果你需要添加更多的设备和服务供外网访问，那么只需照样画葫芦，指定正确的 IP 地址和端口号即可。

#### 注意放行端口

每个服务的 remote_port 是远程访问时要用到的端口号，注意这些端口号也要在服务器的防火土啬和安全组里放行才能顺利访问的，如上面的 7001、7002。

#### 启动 FRP 客户端

假设你已将 Frp 的客户端解压缩到 `c:\frp` 目录中，那么启动 Frp 客户端的命令就是：

```bash
c:\frp\frpc.exe -c c:\frp\frpc.ini
```

Linux 启动 Frp 客户端命令：

```bash
./frpc -c ./frpc.ini
```

启动之后看到 `start proxy success`字样就表示启动成功了。

### 3、进行远程访问

前面搞了这么多，我们终于可以正式使用 Frp 内网穿透来进行远程访问内网里的设备了！按照上面的配置，我们想要访问群晖 NAS 的界面，只需打开**浏览器**，在地址栏输入 `服务器公网IP:7001` 即可访问到群晖后台管理界面。

而如果需要远程桌面连接到家里的 Windows 电脑，那么打开“微软远程桌面客户端”后，在地址栏里填入 `服务器公网IP:7002` 即可连接。

由此，借助 Frp，你就能轻松地为本地局域网内网的设备提供公网直接访问的能力了，你可以用 Frp 来转发包括但不限于 `ssh`、`http`、`https`、`转发 Unix 域套接字`等服务。

上面只是最基础的教程，Frp 还有很多很多高级功能，比如给 Web 增加密码保护、点对点内网穿透、设置端口白名单等等，Frp 官网上也提供了很详细的[文档][frp_doc]，感兴趣的朋友可以去研究一下。

## 写在后面

最后，有了 Frp，我们就能轻松解决没有公网 IP 的老难题了！无论家里的 NAS 、电脑还是其他网络设备，都能轻松在外访问，这可以说是无公网 IP 用户必备的工具了。

希望这篇简单 Frp 入门教程能对有远程访问需求的同学有所帮助吧。毕竟 Frp 带来的便利性是非常大的，特别是需要出差移动办公的用户，值得大家去研究和折腾一番。与 Frp 同类的软件还有 `Ngrok`、`n2n`、`lanproxy` 等，它们工作方式都大同小异，都需要一台服务器作为中转。

****

本文参考

> 1. [异次元](https://www.iplaysoft.com/frp.html)  
> 2. [yangjianyu](https://blog.csdn.net/yangjianyu_csdn/article/details/90475122)

[frp_doc]:https://github.com/fatedier/frp/blob/master/README_zh.md
[frp]:https://github.com/fatedier/frp