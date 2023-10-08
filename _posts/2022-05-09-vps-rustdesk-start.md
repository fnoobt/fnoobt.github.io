---
title: RustDesk 安装
author: fnoobt
date: 2022-05-09 15:32:00 +0800
categories: [VPS,服务搭建]
tags: [vps,rustdesk]
---

RustDesk有三部分组成，中继服务器(hbbr)、ID注册服务器(hbbs)与客户端(win、linux、mac)，其中hbbr与hbbs需要部署在服务器上，客户端提供控制与被控服务。

## 安装要求
RustDesk对硬件要求非常低，基础云服务器的最低配置就足够了。如果TCP打孔直接连接失败，将使用服务器中继，中继连接的流量介于 30 K/s 和 3 M/s（1920x1080 屏幕）之间，具体取决于分辨率设置和屏幕刷新率。如果只是为了办公需求，流量在100K/s左右。  
基础服务器配置：
- CPU：1核  
- 内存：1 GB
- 硬盘：10 GB
- 宽带：2M

## 服务器端安装方法

### 放行防火墙端口
请在运行脚本之前在服务器上设置防火墙。

在设置防火墙之前，请确保您已通过 SSH 或其他设置获得访问权限。UFW（基于 Debian）的示例命令是：
```bash
ufw allow proto tcp from YOURIP to any port 22
```

如果安装了 UFW，请使用以下命令配置防火墙（仅当您要使用自动生成的安装文件时才需要端口 8000）：
```bash
ufw allow 21115:21119/tcp
ufw allow 8000/tcp
ufw allow 21116/udp
sudo ufw enable
```

### 1、安装脚本方式安装
建议使用此方法，运行以下命令：
```bash
wget https://raw.githubusercontent.com/techahold/rustdeskinstall/master/install.sh
chmod +x install.sh
./install.sh
```

如果需要更新:
```bash
wget https://raw.githubusercontent.com/techahold/rustdeskinstall/master/update.sh
chmod +x update.sh
./update.sh
```

> 如果服务器无法访问github会导致该方法无效。
{: .prompt-tip }

会自动创建`etc/systemd/system/gohttpserver.service`{: .filepath}实现开机自启

### 2、下载服务器端程序安装

下载rustdesk server
```bash
wget --no-check-certificate https://github.com/rustdesk/rustdesk-server/releases/download/1.1.8-2/rustdesk-server-linux-amd64.zip
unzip rustdesk-server-linux-amd64.zip
```

建议使用PM2方式(方式二)安装，这样便于管理。

#### 方式一、在没有PM2的情况下运行hbbs/hbbr

启动
```bash
./hbbs -r <relay-server-ip[:port]>
./hbbr
```

#### 方式二、用PM2运行hbbs/hbbr
安装pm2
```bash
sudo apt install npm
sudo npm install -g pm2
```

启动
```bash
pm2 start hbbs -- -r <relay-server-ip[:port]>
pm2 start hbbr
```

查看运行状态
```
pm2 list
```

> 其中<relay-server-ip[:port]>是你服务器的公网ip与端口。比如服务器IP为1.2.3.4，命令为`pm2 start hbbs -- -r 1.2.3.4 -k _`
> `hbbs`的参数`-r`不是强制性的，只是方便您不要在受控客户端指定中继服务器。如果使用默认 21117 端口，则无需指定端口。客户端指定的中继服务器的优先级高于此优先级。 

如果上述操作都没问题，那服务端已经部署完成了。在rustdesk文件夹中找到公钥文件id_ed25519.pub，保存了公钥Key。

## 客户端安装方法

下载对应的[RustDesk客户端版本](https://github.com/rustdesk/rustdesk/releases/)。  
安装完成后进入`设置`-->`网络`-->`ID/中继服务器`，然后在ID服务器里面输入`server-ip:21116`，在中继服务器里面输入`server-ip:21117`,Key里面输入`公钥`，最后点击`应用`。  
手机或者其他电脑同样配置，当左下角出现`就绪`的时候，就是成功了。

****

[官方文档](https://rustdesk.com/docs/en/self-host/install/)