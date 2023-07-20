---
title: V2Ray自定义路由设置规则的写法
author: fnoobt
date: 2022-03-16 00:34:00 +0800
categories: [V2ray,路由规则]
tags: [v2ray,路由规则]
---

目前，主流的V2Ray手机、电脑客户端都移除了对PAC模式的支持，这是因为通过功能强大的“（高级）路由设置”功能，同样可以实现PAC模式自动代理功能。

现在介绍一下在Proxy (代理)、Direct (直连)、Block (阻止)三栏内，怎么填写域名或IP地址。

在代理、直连、阻止三类代理规则输入框内，根据需要按以下规则填写域名或IP。

## 一、域名路由规则的写法：

### 预定义域名列表 geosite:

以 geosite: 开头，后面是一个预定义域名列表名称，如 geosite:google ，意思是包含了Google旗下绝大部分域名；或者 geosite:cn，意思是包含了常见的大陆站点域名。常用名称及域名列表：

```markdown
geosite:category-ads　包含了常见的广告域名。
geosite:category-ads-all　包含了常见的广告域名，以及广告提供商的域名。
geosite:cn　相当于 geolocation-cn 和 tld-cn 的合集。
geosite:apple　包含了 Apple 旗下绝大部分域名。
geosite:google　包含了 Google 旗下绝大部分域名。
geosite:microsoft　包含了 Microsoft 旗下绝大部分域名。
geosite:facebook　包含了 Facebook 旗下绝大部分域名。
geosite:twitter　包含了 Twitter 旗下绝大部分域名。
geosite:telegram　包含了 Telegram 旗下绝大部分域名。
geosite:geolocation-cn　包含了常见的大陆站点域名。
geosite:geolocation-!cn　包含了常见的非大陆站点域名，同时包含了 tld-!cn。
geosite:tld-cn　包含了 CNNIC 管理的用于中国大陆的顶级域名，如以 .cn、.中国 结尾的域名。
geosite:tld-!cn　包含了非中国大陆使用的顶级域名，如以 .hk（香港）、.tw（台湾）、.jp（日本）、.sg（新加坡）、.us（美国）.ca（加拿大）等结尾的域名。
```

更多域名类别，请查看 [data 目录](https://github.com/v2fly/domain-list-community/tree/master/data)

### 域名 domain:

由 domain: 开始，后面是一个域名。例如 domain:baiyunju.cc ，匹配 www.baiyunju.cc 、baiyunju.cc，以及其他baiyunju.cc主域名下的子域名。

不过，前缀domain:可以省略，只输入域名，其实也就成了纯字符串了。

### 完整匹配 full:

由 full: 开始，后面是一个域名。例如 full:baiyunju.cc 只匹配 baiyunju.cc，但不匹配 www.baiyunju.cc 。

### 纯字符串

比如直接输入 sina.com, 可以分行，也可以不分行以“,”隔开，可以匹配 sina.com、sina.com.cn 和 www.sina.com，但不匹配 sina.cn。

### 正则表达式 regexp:

由 regexp: 开始，后面是一个正则表达式。例如 regexp:\.goo.*\.com$ 匹配 www.google.com、fonts.googleapis.com，但不匹配 google.com。

### 从外部文件中加载域名规则 ext:

比如 ext:file:tag，必须以 ext:（全部小写）开头，后面跟文件名（不含扩展名）file 和标签 tag，文件必须存放在 V2Ray 核心的资源目录中，文件格式与 geosite.dat 相同，且指定的标签 tag 必须在文件中存在。

>**说明：**普通用户常用的也就是上面的`纯字符串`规则写法，比如，在代理（或直连）栏下填写 baiyunju.cc,　就可以让网站通过代理（或直连）上网。

## 二、IP 路由规则的写法：

### geoip:

以 geoip:（全部小写）开头，后面跟双字符国家代码，如 geoip:cn ，意思是所有中国大陆境内的 IP 地址，geoip:us 代表美国境内的 IP 地址。

#### 常用geoip列表：

```markdown
geoip:private
geoip:cn
geoip:cloudflare
geoip:cloudfront
geoip:facebook
geoip:fastly
geoip:google
geoip:netflix
geoip:telegram
geoip:twitter
```

>**重要提示：**以上geoip列表需要配合加强版资源文件geoip.dat、geosite.dat。
{: .block-tip }

### 特殊值：

geoip:private，包含所有私有地址，如127.0.0.1（本条规则仅支持 V2Ray 3.5 以上版本）。

### IP：

如 127.0.0.1，20.194.25.232

### CIDR：

如 10.0.0.0/8。

### 从外部文件中加载 IP 规则：

如 ext:file:tag，必须以 ext:（全部小写）开头，后面跟文件名（不含扩展名）file 和标签 tag，文件必须存放在 V2Ray 核心的资源目录中，文件格式与 geoip.dat 相同，且指定的 tag 必须在文件中存在。

****

[原文出处](https://baiyunju.cc/7246)