---
title: Newifi3刷三方固件（华硕老毛子Padavan）
author: fnoobt
date: 2020-03-02 18:44:00 +0800
categories: [刷机,Newifi]
tags: [刷机,newifi,padavan]
---

## 准备阶段

下载Padavan固件：

到[Padavan固件下载](https://opt.cn2qq.com/padavan/)这里搜`newifi3`就能找到[RT-N56UB1-newifi3D2-512M_3.4.3.9-099.trx](https://opt.cn2qq.com/padavan/RT-N56UB1-newifi3D2-512M_3.4.3.9-099.trx)。

其他固件可查看[新路由三 NEWifi D2 固件集合贴](https://www.right.com.cn/forum/forum.php?mod=viewthread&tid=658359&page=1&extra=#pid3990638)

## 刷入不死Breed

1. 初始化路由器，不联网登录路由器管理后台<http://192.168.99.1>设置登录密码；
2. 连接路由器LAN口，访问<http://192.168.99.1/newifi/ifiwen_hss.html>，激活`SSH`(回车后网页会出现*succss*,既代表激活成功)；
3. 下载Breed文件[newifi-d2-jail-break.ko](https://s.razeen.cn/firmwares/newifi-d2-jail-break.ko)，需要最新的可以到[Index of Breed](https://breed.hackpascal.net/)下载;
4. WinScp工具上传Breed模块到路由器`/tmp`目录（SCP协议），或使用命令行上传 `scp newifi-d2-jail-break.ko root@192.168.99.1:/tmp`
5. ssh 登陆路由器后台，输入命令 `cd /tmp && insmod newifi-d2-jail-break.ko` 开始刷入;
6. 系统重启，成功刷入。

## 进入Breed控制台
拔掉电源，按住路由器reset按钮，然后再插入电源，等几秒，电源灯闪烁，就可以松开reset按钮了。  
电脑上浏览器输入192.168.1.1进入**Breed Web 恢复控制台**了。

进入 **Breed Web 恢复控制台**后，先不要急着进行升级，我们先到<kbd>固件备份</kbd>页面把`EEPROM`和`编程器固件`备份一下。

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

> 1. [Razeen](https://razeen.me/posts/start-use-newifi3/)
> 2. [jzhang](https://www.jianshu.com/p/6629e5e23274)