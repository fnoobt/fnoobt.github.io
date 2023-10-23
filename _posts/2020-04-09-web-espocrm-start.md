---
title: EspoCRM安装
author: fnoobt
date: 2020-04-09 09:49:00 +0800
categories: [Web,WebDemo]
tags: [web,espocrm,php,mysql]
---

## 安装

下载[EspoCRM](https://www.espocrm.com/)的最新[版本](https://github.com/espocrm/espocrm/releases)  
EspoCRM的安装参见[官方文档](https://docs.espocrm.com/administration/server-configuration/)

### 网页报错
```
Warning: require_once(vendor/autoload.php): Failed to open stream: No such file or directory in /www/wwwroot/espocrm/bootstrap.php on line 33

Fatal error: Uncaught Error: Failed opening required 'vendor/autoload.php' (include_path='/www/wwwroot/espocrm') in /www/wwwroot/espocrm/bootstrap.php:33 Stack trace: #0 /www/wwwroot/espocrm/public/index.php(30): include() #1 {main} thrown in /www/wwwroot/espocrm/bootstrap.php on line 33
```

这是因为下载的是源代码，网站根目录下缺少`/vendor`这个存放php项目依赖包的目录

#### 解决方法：
1. 下载完整版本代码
2. 使用`composer install`进行安装（如果以前安装过的话使用：`composer update`）

> `composer install --ignore-platform-reqs`忽略平台要求强制安装
{: .prompt-tip }

****

> [原文出处](https://docs.espocrm.com/administration/server-configuration/)