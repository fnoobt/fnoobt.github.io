---
title: Clash订阅转换服务搭建
author: fnoobt
date: 2022-10-13 18:32:00 +0800
categories: [VPS,服务搭建]
tags: [v2ray,sub-Web,subConverter,clash]
---

想把V2ray的订阅节点放在Clash中使用，需要使用Github上的开源转换面板，但是在使用别的现成面板担心数据隐私的泄露，于是自己着手搭建一个前端面板。  
[Sub-Web前端](https://github.com/CareyWang/sub-web)  
[SubConverter后端](https://github.com/tindy2013/subconverter)

## Sub-Web前端搭建

### 更新系统并安装 Node 与 Yarn

Centos
```bash
yum update -y
yum install -y curl wget sudo nodejs git npm
npm install -g cnpm --registry=https://registry.npm.taobao.org
#若提示npm can not found请使用下列命令安装npm后执行上面命令
yum install npm –enablerepo=epel
cnpm install -g yarn
```

Debian/Ubuntu
```bash
sudo apt-get update -y
sudo apt-get install -y curl wget sudo nodejs git npm
sudo npm install -g yarn
```
命令执行完毕以后，请运行下面的代码查询 Node 与 Yarn 是否安装成功，若是成功会返回版本号。
```bash
node -v
#yum install npm –enablerepo=epel
yarn --version
```

### 下载并安装 Sub-Web
拉取 sub-web 程序，并进入 sub-web 文件夹
```bash
git clone https://github.com/CareyWang/sub-web.git
cd sub-web
```

在项目目录中安装构建依赖项，构建的过程稍微有点长
```bash
yarn install
```

如果报错node版本不符，可以更新node版本
```bash
# 首先，清除npm缓存:
npm cache clean -f
# 安装n, Node的版本管理器:
npm install -g n
# 安装了n模块后，用他来安装指定版本[version.number]，此处用node 14
sudo n 14
```

使用 webpack 运行 Web 客户端以进行本地开发。
```bash
yarn serve
```
这时，我们浏览器访问 http://服务器ip:8080/ 应该可以进行前端 sub-web 的预览了。

> 记住8080端口的防火墙和安全组要开放，不可以使用域名
{: .prompt-tip }

### 修改默认后端地址 && 增加远程规则
找到 VPS `/root/sub-web/src/views/Subconverter.vue`{: .filepath} 文件

找到 `backendOptions:` 将你解析好的后端地址输入进去。域名为你刚才准备的**后端域名**，要将http改成**https**，并且增加/sub?的后缀。  
这是为了后续不用每次都手动填写后端地址

因为这个版本更新以后，规则方面很少，经常用到的一些经典的 ACL4SSR 的规则并没有集成，大家可以看看，若是有，就不需要这样操作。
找到 `remoteConfig: [` 后回车将下面的规则复制进去
```json
{
    label: "ACL4SSR",
    options: [
        {
            label: "ACL4SSR_Online 默认版 分组比较全 (与Github同步)",
            value: "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online.ini"
        },

        {
            label: "ACL4SSR_Online_AdblockPlus 更多去广告 (与Github同步)",
            value: "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_AdblockPlus.ini"
        },

        {
            label: "ACL4SSR_Online_NoAuto 无自动测速 (与Github同步)",
            value:"https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_NoAuto.ini"
        },

        {
            label: "ACL4SSR_Online_NoReject 无广告拦截规则 (与Github同步)",
            value:"https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_NoReject.ini"
        },

        {
            label: "ACL4SSR_Online_Mini 精简版 (与Github同步)",
            value:"https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Mini.ini"
      },

      {
            label: "ACL4SSR_Online_Mini_AdblockPlus.ini 精简版 更多去广告 (与Github同步)",
            value:"https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Mini_AdblockPlus.ini"
      },

      {
            label: "ACL4SSR_Online_Mini_NoAuto.ini 精简版 不带自动测速 (与Github同步)",
            value:"https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Mini_NoAuto.ini"
      },

      {
            label: "ACL4SSR_Online_Mini_Fallback.ini 精简版 带故障转移 (与Github同步)",
            value:"https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Mini_Fallback.ini"
      },

      {
            label: "ACL4SSR_Online_Mini_MultiMode.ini 精简版 自动测速、故障转移、负载均衡 (与Github同步)",
            value:"https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Mini_MultiMode.ini"
      },

      {
          label: "ACL4SSR_Online_Full 全分组 重度用户使用 (与Github同步)",
          value:"https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Full.ini"
      },

      {
          label: "ACL4SSR_Online_Full_NoAuto.ini 全分组 无自动测速 重度用户使用 (与Github同步)",
          value:"https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Full_NoAuto.ini"
      },

      {
          label: "ACL4SSR_Online_Full_AdblockPlus 全分组 重度用户使用 更多去广告 (与Github同步)",
          value:"https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Full_AdblockPlus.ini"
      },

      {
          label: "ACL4SSR_Online_Full_Netflix 全分组 重度用户使用 奈飞全量 (与Github同步)",
          value:"https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Full_Netflix.ini"
      },

      {
          label: "ACL4SSR 本地 默认版 分组比较全",
          value: "config/ACL4SSR.ini"
      },

      {
          label: "ACL4SSR_Mini 本地 精简版",
          value: "config/ACL4SSR_Mini.ini"
      },

      {
          label: "ACL4SSR_Mini_NoAuto.ini 本地 精简版+无自动测速",
          value: "config/ACL4SSR_Mini_NoAuto.ini"
      },

      {
          label: "ACL4SSR_Mini_Fallback.ini 本地 精简版+fallback",
          value: "config/ACL4SSR_Mini_Fallback.ini"
      },

      {
          label: "ACL4SSR_BackCN 本地 回国",
          value: "config/ACL4SSR_BackCN.ini"
      },

      {
          label: "ACL4SSR_NoApple 本地 无苹果分流",
          value: "config/ACL4SSR_NoApple.ini"
      },

      {
            label: "ACL4SSR_NoAuto 本地 无自动测速 ",
            value: "config/ACL4SSR_NoAuto.ini"
      },

      {
            label: "ACL4SSR_NoAuto_NoApple 本地 无自动测速&无苹果分流",
            value: "config/ACL4SSR_NoAuto_NoApple.ini"
      },

      {
            label: "ACL4SSR_NoMicrosoft 本地 无微软分流",
            value: "config/ACL4SSR_NoMicrosoft.ini"
      },

      {
            label: "ACL4SSR_WithGFW 本地 GFW列表",
            value: "config/ACL4SSR_WithGFW.ini"
      }
    ]
  },
```

### 配置完毕后打包网站
配置完毕以后，程序会自动更新，再次刷新前端网页，会出现刚才添加的相关规则

至此，我们的前端调试完毕，我们现在需要打包，生成一个发布目录并将他发布了。

首先停止调试程序，CTRL+C ，退出当前调试，然后执行下面的命令进行打包：

```bash
yarn build
```

> 若是不会停止，请断开VPS，重新连接VPS以后，输入 `cd sub-web` 再执行
{: .prompt-tip }

执行以下打包命令，在 `/root/sub-web` 下面会生成一个 `dist` 目录，这个目录即为网页的发布目录。

将这个目录的里面的文件复制到你站点的根目录即可。

### 将前端发布
在宝塔面板中点击增加站点分别将前端站点增加上去，并配置好ssl证书。

将 `/root/sub-web/dist` 文件夹内的所有文件复制到前端站点的根目录下即可。

添加重定向规则，这是为了用`/sub`代替25500端口
```
location /sub{
    proxy_pass http://127.0.0.1:25500;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header REMOTE-HOST $remote_addr;
    add_header X-Cache $upstream_cache_status;
    #Set Nginx Cache
    add_header Cache-Control no-cache;
    expires 12h;
}
```

## SubConverter后端搭建

下载对应的 [subconverter版本](https://github.com/tindy2013/subconverter/releases)
```bash
wget https://github.com/tindy2013/subconverter/releases/download/v0.7.2/subconverter_linux64.tar.gz
tar zxvf subconverter_linux64.tar.gz
```

### 修改配置文件参数(这一步可不操作)

现在我们需要修改后端配置文件中的一些参数

找到VPS文件 `/root/subconverter/pref.ini`，找到如下参数进行修改
```ini
api_access_token=123123dfsdsdfsdfsdf            #随意设置自己知道就行
managed_config_prefix=https://sub.yourdomin.com  #设置成我们刚刚解析的后端域名
listen=127.0.0.1                                #这里改成 127.0.0.1 进行反代
```

> 注意不同的用户文件的路径会不同
{: .prompt-tip }

### 创建服务进程

编辑 subconverter.service
```bash
sudo vi /etc/systemd/system/subconverter.service
```

```
[Unit]
Description=A API For Subscription Convert
After=network.target

[Service]
Type=simple
ExecStart=/root/subconverter/subconverter
WorkingDirectory=/root/subconverter
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```
{: file='/etc/systemd/system/subconverter.service'}

> 注意修改 `ExecStart` 和 `WorkingDirectory` 路径
{: .prompt-tip }

设置跟随系统启动

```bash
sudo systemctl enable subconverter
sudo systemctl start subconverter
```

### 查看日志
```bash
journalctl -fu subconverter
```

## 在线生成转换地址
前端我们也可以搭建，但是没必要，下面分享一些三方的前端，当然你如果不在意隐私前后端都可以用别人的。
- [789.st](https://sub.789.st/)
- [acl4ssr](https://acl4ssr-sub.github.io/)
- [v1.mk](https://sub.v1.mk/)
- [subcloud.xyz](https://my.subcloud.xyz/)

页面后端地址填写我们的地址就可以 https://你的主机ip:25500/sub?

## 节点转换器（支持VLESS）

因为SubConverter不支持VLESS，如果有VLESS需求可以使用[节点转换器](https://v2rayse.com/node-convert/)，使用前端生成，可断网使用。

****

本文参考

> 1. [订阅转换服务 subconverter 后端搭建](https://www.ga0x.com/blog/2023/03/07/install-subconverter)
> 2. [sub-web订阅转换面板的搭建教程](https://www.mxlong.com/13.html)
> 3. [Sub-Web搭建教程](https://v2rayssr.com/sub-web.html)