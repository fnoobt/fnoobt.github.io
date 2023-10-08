---
title: 小米/红米AC2100路由器刷三方固件（华硕老毛子Padavan）
author: fnoobt
date: 2022-12-22 17:44:00 +0800
categories: [Flash,路由器]
tags: [Flash,刷机,miwifi,ac2100,padavan]
---

## 准备阶段

### 1.下载降级固件：

[红米AC2100降级固件](http://cdn.cnbj1.fds.api.mi-img.com/xiaoqiang/rom/rm2100/miwifi_rm2100_firmware_d6234_2.0.7.bin)

[小米AC2100降级固件](http://cdn.cnbj1.fds.api.mi-img.com/xiaoqiang/rom/r2100/miwifi_r2100_firmware_4b519_2.0.722.bin)

下载完成后进入后台 <http://192.168.31.1> --> <kbd>常用设置</kbd> --> <kbd>系统状态</kbd> --> <kbd>手动升级</kbd>  
加载固件，清除数据 --> <kbd>开始升级</kbd>

### 2.下载Padavan固件：

到[Padavan固件下载](https://opt.cn2qq.com/padavan/)这里搜`R2100`就能找到[R2100_3.4.3.9-099.trx](https://opt.cn2qq.com/padavan/R2100_3.4.3.9-099.trx)。

## 刷入不死Breed

### 获取stok
首先需要确保路由器有网络，有网络才能自动下载BREED。

登录192.168.31.1进入路由器管理界面，复制当前地址 `http://192.168.31.1/cgi-bin/luci/;stok=XXXXXXXXXXXXX/web/home#router` 中`XXXXXXXXXXXXX`的部分。  
每次登录 `stok=` 之后的一串字符都会改变，每次登录需要重新复制。

将第一步复制下来的字符串替换掉以下链接的`XXXXXXXXXXXXX`部分之后复制到浏览器打开，跳转页面会显示返回 {"code":0}，如果显示其他则可能是没有进行路由器降级或者stok过期，请重新尝试或恢复出厂设置。

### 检查NAND坏块（可不执行）
路由器开机超过一小时建议先重启。运行代码后，你路由器的2.4g WiFi名称会改名成：比如  "ESMT"，"Toshiba"，"Toshiba 90 768"。 90和768是坏块。 如果ESMT或者Toshiba后面没数字，那恭喜你，没有坏块！！！
```
http://192.168.31.1/cgi-bin/luci/;stok=XXXXXXXXXXXXX/api/misystem/set_config_iotdev?bssid=Xiaomi&user_id=longdike&ssid=%0A%5B%20-z%20%22%24(dmesg%20%7C%20grep%20ESMT)%22%20%5D%20%26%26%20B%3D%22Toshiba%22%20%7C%7C%20B%3D%22ESMT%22%0Auci%20set%20wireless.%24(uci%20show%20wireless%20%7C%20awk%20-F%20'.'%20'%2Fwl1%2F%20%7Bprint%20%242%7D').ssid%3D%22%24B%20%24(dmesg%20%7C%20awk%20'%2FBad%2F%20%7Bprint%20%245%7D')%22%0A%2Fetc%2Finit.d%2Fnetwork%20restart%0A
```

### 在线刷Breed
你可以先检查NAND坏块，不检查也没关系。Bootloader那里肯定不会有坏块，不然官方Uboot也会出问题的。
```
http://192.168.31.1/cgi-bin/luci/;stok=XXXXXXXXXXXXX/api/misystem/set_config_iotdev?bssid=Xiaomi&user_id=longdike&ssid=%0Acd%20%2Ftmp%0Acurl%20-o%20B%20-O%20https%3A%2F%2Fbreed.hackpascal.net%2Fr1286%2520%255b2020-10-09%255d%2Fbreed-mt7621-xiaomi-r3g.bin%20-k%20-g%0A%5B%20-z%20%22%24(sha256sum%20B%20%7C%20grep%20242d42eb5f5aaa67ddc9c1baf1acdf58d289e3f792adfdd77b589b9dc71eff85)%22%20%5D%20%7C%7C%20mtd%20-r%20write%20B%20Bootloader%0A
```

路由器开始下载 Breed 进行刷写，大约需要1~3分钟进行重启，等待路由器指示灯由蓝色变为橘黄色，然后再次变为蓝色进入系统，此时Breed已经刷入完毕。

### 可能出现的问题
如果没有出现蓝灯切换为橘黄灯重启，则可能有以下原因：
1. Breed进行了更新，需要额外进行以下操作。  
打开[Breed最新版本](https://breed.hackpascal.net/)，搜索`xiaomi`，找到 [breed-mt7621-xiaomi-r3g.bin](https://breed.hackpascal.net/breed-mt7621-xiaomi-r3g.bin)并下载。  
打开[SHA256文件HASH值](https://crypot.51strive.com/sha256_checksum.html) 将刚才下载的文件添加进去，生成生成HASH值。  
将以下字符串中##################的部分替换为新生成的HASH值。再次替换XXXX部分以后复制到浏览器打开即可。
```
http://192.168.31.1/cgi-bin/luci/;stok=XXXXXXXXXXXXX/api/misystem/set_config_iotdev?bssid=xiaomi&user_id=longdike&ssid=%0Acd%20%2Ftmp%0Acurl%20-o%20B%20-O%20https%3A%2F%2Fbreed.hackpascal.net%2Fbreed-mt7621-xiaomi-r3g.bin%20-k%0A%5B%20-z%20%22%24(sha256sum%20B%20%7C%20grep%20##################)%22%20%5D%20%7C%7C%20mtd%20-r%20write%20B%20Bootloader%0A
```
2. 网络问题或者下载的Breed固件损坏，请多尝试几次。
3. Stok后面的字段过期，请重新获取。

如果Breed更新导致刷不进固件或其他问题，请参阅[Breed官方更新日志](https://blog.hackpascal.net/)

## 进入Breed控制台
拔下电源线，用针按住路由器后面的Reset键不放手重新插上，等待路由器指示灯为蓝色闪烁状态松手，此时已成功启用Breed后台。用网线连接电脑与路由器，等待获取到ip地址以后登录 192.168.1.1 即可进入Breed界面。

在<kbd>小米 R3G 设置</kbd>中删除字段 `normal_firmware_md5` 并保存。

> 如果要直接升级 OpenWrt 的 kernel1 和 rootfs0，需要将闪存布局选择为 "小米 R3G OpenWrt"；如果要升级 Bdata，需要将闪存布局选择为 "小米 R3G 原厂"。
{: .prompt-tip }

> 为防止路由器变砖，进入 **Breed Web 恢复控制台**后，先不要急着进行升级，我们先到<kbd>固件备份</kbd>页面把`EEPROM`和`编程器固件`备份一下。EEPROM保存着出厂信息，且每台设备均为唯一，包括路由器SN，MAC地址和无线相关参数。EEPROM数据丢失可能导致无线网无法使用。
{: .prompt-danger }

## 刷机

路由如果从一个系统刷入另一个系统，最好先恢复出厂设置，这样也能保持在刷入系统之前，是最初始状态。  
到<kbd>恢复出厂设置</kbd>页面，选择固件类型`Config区（公版）`，点击<kbd>执行</kbd>。

1. 到<kbd>固件更新</kbd>页面，勾选`固件`，再点击<kbd>浏览</kbd>，选择对应的固件，最后进行<kbd>上传</kbd>。
2. 固件上传完成后，到 **Breed Web 恢复控制台**更新确认界面，点击<kbd>更新</kbd>。
3. 等待更新完成，设备重启
4. 在浏览器中输入 <http://192.168.123.1>，输入用户名及密码，Padavan系统默认用户名 `admin`，密码 `admin`。

>更新breed版本在**Breed Web 恢复控制台**<kbd>固件更新</kbd>页面勾选`Bootloader`选择bin文件升级即可。
{: .prompt-tip }

****

本文参考

> 1. [蒼天淨土](https://www.bilibili.com/read/cv14946356)
> 2. [刷BREED的URL](https://www.right.com.cn/forum/thread-4066963-1-1.html)