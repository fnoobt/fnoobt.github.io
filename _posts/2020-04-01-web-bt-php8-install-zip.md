---
title: PHP8.1安装zip扩展(宝塔面板)
author: fnoobt
date: 2020-04-01 09:49:00 +0800
categories: [Web,PHP]
tags: [web,php,bt]
---

## 如何安装zip扩展
如果你没有启用zip扩展，则需要先安装libzip，再安装zip，在php.ini中启用，重启即可。

## 安装libzip-1.2.0
你可以在任意的路径下载libzip，然后解压缩，进入libzip，编译安装即可

```bash
wget https://libzip.org/download/libzip-1.2.0.tar.gz
tar -zxvf libzip-1.2.0.tar.gz
cd libzip-1.2.0
./configure
make && make install
```

如果官方的libzip-1.2.0下载很慢，可以考虑使用其他的cdn下载：  
<https://v.iculture.cc/libzip-1.2.0.tar.gz>  
<https://nih.at/libzip/libzip-1.2.0.tar.gz>

## 设置临时的环境变量（非必须）
如果安装libzip-1.2.0成功之后，则可以设置环境变量。  
如果你不确定是否成功安装，可以查看`/usr/local/lib/pkgconfig`{: .filepath}路径是否存在，存在则代表上面的库已经安装成功了

```bash
cd /usr/local/lib/pkgconfig
```

接下来我们设置环境变量

```bash
export PKG_CONFIG_PATH="/usr/local/lib/pkgconfig/"
```

## 编译zip模块

### 下载zip
如果没有下载zip，可以通过下面的命令下载解压，
```bash
wget https://pecl.php.net/get/zip
tar -zxvf zip
```

### 安装zip
在宝塔中其实zip相关的扩展，已经在极速安装时下载好了，我们可以进入`/www/server/php/81/src/ext/zip`{: .filepath}路径，您可以运行下面的命令，进行编译安装

```bash
cd /www/server/php/81/src/ext/zip/
/www/server/php/81/bin/phpize
./configure --with-php-config=/www/server/php/81/bin/php-config
make && make install
```

> 注意：直接下载的和宝塔下载的zip文件路径不同。
{: .prompt-tip }

如果在末尾出现`Build complete.`字样，代表已经安装成功了。

## 配置php.ini扩展
在php.ini最后一行增加
```ini
extension = zip.so
```
{: file='/www/server/php/81/etc/php.ini'}

宝塔中，则是进入php-8.1管理，点击配置文件，增加zip.so扩展

记得保存之后重载配置或者重启，之后就可以生效了！

当然，如果你也可以用命令行操作

```bash
echo "extension = zip.so" >> /www/server/php/81/etc/php.ini
restart php
```

****

本文参考

> 1. [鑫空之眼](https://blog.csdn.net/TaiYang_5339/article/details/129572714)
> 2. [编程网](https://www.lsjlt.com/news/276089.html)