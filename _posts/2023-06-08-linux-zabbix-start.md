---
title: zabbix介绍及部署
author: fnoobt
date: 2023-06-08 19:29:00 +0800
categories: [Linux,Linux教程]
tags: [linux,zabbix]
---

zabbix是一个监控软件，其可以监控各种网络参数，保证企业服务架构安全运营，同时支持灵活的告警机制，可以使得运维人员快速定位故障、解决问题。zabbix支持分布式功能，支持复杂架构下的监控解决方案，也支持web页面，为主机监控提供了良好直观的展现。

## zabbix的构成
zabbix主要由以下5个组件构成
### 1、Server  
zabbix server是zabbix的核心组件，server内部存储了所有的配置信息、统计信息和操作信息。zabbix agent会向zabbix server报告可用性、完整性及其他统计信息。
### 2、web页面  
web页面也是zabbix的一部分，通常和zabbix server位于一台物理设备上，但是在特殊情况下也可以分开配置。web页面主要提供了直观的监控信息，以方便运维人员监控管理。
### 3、数据库  
zabbix数据库内存储了配置信息、统计信息等zabbix的相关内容。
### 4、proxy  
zabbix proxy可以根据具体生产环境进行采用或者放弃。如果使用了zabbix proxy，则其会替代zabbix server采集数据信息，可以很好的分担zabbix server的负载。zabbix proxy通常运用与架构过大、zabbix server负载过重，或者是企业设备跨机房、跨网段、zabbix server无法与zabbix agent直接通信的场景。
### 5、Agent  
zabbix agent通常部署在被监控目标上，用于主动监控本地资源和应用程序，并将监控的数据发送给zabbix server。

## zabbix的监控对象
zabbix支持监控各种系统平台，包括Linux和Windows等主流操作系统，也可以借助SNMP或者是SSH协议监控路由交换设备。

zabbix如果部署在服务器上，可以监控其CPU、内存、网络性能等硬件参数，也可以监控具体的服务或者应用程序、服务运行情况及性能。

- 硬件监控：Zabbix IPMI Interface ，通过IPMI接口进行监控，我们可以通过标准的IPMI硬件接口，监控被监控对象的物理特征，比如电压、温度、风扇状态、电源状态等。
- 系统监控：Zabbix Agent Interface ，通过专用的代理程序进行监控，与常见的master/agent模型类似，如果被监控对象支持对应的agent，推荐首选这种方式。
- Java监控：Zabbix JMX Interface ，通过JMX进行监控，JMX（java management extensions，即java管理扩展），监控JVM虚拟机时，使用这种方法是非常不错的选择。
- 网络设备监控：Zabbix SNMP Interface ，通过SNMP协议与被监控对象进行通信，SNMP协议的全称为simple network management protocol，被译为简单网络管理协议，通常来说，我们无法在路由器、交换机这种硬件上安装agent，但是这些硬件都支持SNMP协议。
- 应用服务监控：Zabbix Agent UserParameter
- MySQL数据库监控：percona-monitoring-plulgins
- URL监控：Zabbix Web 监控

## zabbix的常用术语
zabbix的学习需要掌握一些zabbix的常用术语，zabbix常用术语列举如下：

### 1、主机（host）  
要监控的设备，可以由IP或者是主机名（必须可解析）指定。
### 2、主机组（host group）  
主机的逻辑容器，包含主机和模板，主机组通常在给用户或者是用户组指派监控权限时使用。
### 3、监控项（item）  
一个特定监控指标的相关数据，比如内存的大小、CPU的使用率，甚至是服务的运行状态等等。监控项数据来源于被监控对象，并且每个监控项都由一个key来标识。
### 4、触发器（trigger）  
一个表达式，用于评估监控项的值是否在合理的范围内。当接收的值超出触发器的规定时，就被认为是故障，如果超出后再次符合，就被认为是正常。
### 5、事件（event）  
触发器触发的一个特定事件，或者是zabbix定义的一个自动上线注册主机的事件。
### 6、动作（action）  
指根据配置，zabbix对于触发器触发的特定事件进行处理的具体措施，如执行某个脚本，或者是向管理员邮箱发送邮件等等。
### 7、报警升级（escalation）  
发送警报或者是执行远程命令的自定义方案。
### 8、媒介（media）  
发送通知（告警）的手段，如微信、邮件、钉钉等等。
### 9、通知（notification）  
通过指定的媒介，向用户发送的有关事件的信息。
### 10、远程命令（remote command）  
指运维人员提前写好的命令，可以让被监控主机在触发事件后执行。
### 11、模板（template）  
用于快速定义被监控主机的预设条目集合，通常包括了监控项、触发器、应用等，模板可以直接链接至某个主机。
### 12、应用（application）  
一组监控项的集合。
### 13、web场景（web scennario）  
用于检测web站点可用性的一个或多个HTTP请求。
### 14、前端（frontend）  
zabbix的web接口。  
这些术语，我们都会在后文中直接使用而不过多赘述，在企业技术交流中也会经常使用。

## zabbix的工作流程
Zabbix在进行监控时，zabbix客户端要安装在被监控设备上，负责定期收集数据，并将其发送给zabbix服务端；zabbix服务端要安装在监控设备上，其将zabbix客户端发送的数据存储的数据库中，zabbix web根据数据在前端进行展示和绘图。

zabbix的数据收集分为两种模式：

### 1、主动模式  
zabbix客户端主动向zabbix server请求监控项列表，并主动将监控项内需要的数据提交给zabbix server。
### 2、被动模式  
zabbix server向 agent 请求获取监控项的数据，zabbix agent返回数据。

由此可以看出zabbix的主动和被动模式是以zabbxi客户端为基准的。

## zabbix进程详解
在默认的情况下，zabbix有6个工作进程；分别是 zabbix_agentd，zabbix_get，zabbix_proxy，zabbix_sender，zabbix_server 和 zabbix_gateway。

其中，zabbix_java_gateway是可选进程。这6个进程的作用如下：

### 1、zabbix_agentd
zabbix-agentd为zabbix客户端守护进程 ，主要负责收集客户端监控项数据。
### 2、zabbix_server
zabbix_server为zabbix服务端守护进程，主要负责收集zabbix客户端数据。（端口为10051）
### 3、zabbix_proxy
zabbix_proxy是zabbix的代理程序，其功能类似于server，作用上类似于一个中转站，最终会把收集的数据再次提交给zabbix_server。
### 4、zabbix_get
zabbix_get作为zabbix工具，通常运行在zabbix_server或者zabbix_proxy上，用于远程获取客户端信息，通常用于排错。
### 5、zabbix_sender
zabbix_sender也是zabbix的一个工具，通常运行在zabbix的客户端，用于耗时比较长的检查，其作用是主动发送数据。
### 6、zabbix_java_gateway
zabbix_java_gateway是zabbix2.0以后引入的新功能，可以用于JAVA方面的设备；但是只能主动获取数据，而不能被动获取数据。

## zabbix的监控框架
在实际的工作环境中，根据网络环境和监控的规模不同，zabbix一共有三种框架，分别是server_client架构、master_node_client架构和server_proxy_client架构。

### 1、server_client架构
zabbix最简单的架构，监控设备和被监控设备之间直接相连，zabbix_server 和 zabbix_client 之间直接进行数据交互。
### 2、zabbix_proxy_client架构
proxy是连接 server 和 client 之间的桥梁，其本身不存放数据，只是将zabbix_agent端发来的数据暂存，然后再提交给server。这种架构一般用于跨机房、跨网络的中型网络架构。

在server_proxy_client架构中，server设备的宕机会导致整个系统瘫痪而无法正常工作。
### 3、master_node_client架构
master_node_client架构是zabbix最复杂的架构。一般用于跨机房、跨网络、监控设备较多的大型网络架构。与server_proxy_client架构相比，master_node_client架构的主要区别在于node与proxy上.

在master_node_client架构中，每个node可以理解为一个小的server端，在自己的配置文件和数据库，node下游可以直接连接client，也可以再次经过proxy代理后连接client。

在master_node_client架构中，master设备宕机不会影响node节点的正常工作。

### 每个模块的工作职责
#### 1、Zabbix_Server
zabbix_server作为核心组件，用来获取agent存活情况和监控数据。所有的配置、统计、操作数据均通过server进行存取到database；
#### 2、Zabbix_Database
用户存储所有的zabbix的配置信息、监控数据的数据库；
#### 3、Zabbix_Web
zabbix的web界面，管理员通过web界面管理zabbix配置以及查看zabbix相关监控信息，通常与zabbix_server运行在同一台主机上，也可以单独部署在独立的服务器上；
#### 4、Zabbix_Proxy
通常用于分布式监控，代理zabbix_server收集部分被监控的数据并统一发送给server端；（通常大于500台主机需要使用）
#### 5、Zabbix_Agent
部署在被监控主机上，负责收集被监控主机的数据，并发送给servre端或者proxy端；

Zabbix Server、Proxy、Agent都有自己的配置文件以及log文件，重要的参数需要在这里配置，后面会详细说明。

## zabbix源码安装及部署
### ubuntu下载Zabbix6.0 deb文件并安装。
```bash
wget https://mirrors.tuna.tsinghua.edu.cn/zabbix/zabbix/6.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.0-4%2Bubuntu22.04_all.deb
sudo dpkg -i zabbix-release_6.0-4+ubuntu22.04_all.deb
sudo apt update
```

### 安装组件
```bash
sudo apt install zabbix-server-mysql
sudo apt install zabbix-frontend-php
sudo apt install zabbix-apache-conf
sudo apt install zabbix-sql-scripts
sudo apt install zabbix-agent
```

### 创建MySQL数据库
```sql
#sudo apt install mysql-server -y  #安装MySQL
#sudo mysql -uroot -p
password
mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH MYSQL_NATIVE_PASSWORD BY 'password';   #修改MySQL root密码
mysql> create database zabbix character set utf8 collate utf8_bin;
mysql> create user zabbix@localhost identified by 'password';
mysql> grant all privileges on zabbix.* to zabbix@localhost;   #授权zabbix对应数据库所有的权限
mysql> flush privileges;   #刷新
mysql> quit;
#sudo mysql_secure_installtion
#mysql -uzabbix -p			#连接测试
#systemctl start mysql.service		#启动数据库
#systemctl enable mysql
```

> `mysql> set global log_bin_trust_function_creators=1;`该命令让所有用户具有执行类似functions的权限，危险,实际应用建议使用root导入数据

### 导入初始架构
导入初始架构和数据。
```bash
sudo vim /etc/zabbix/apache.conf		#更改zabbix时区
php_value date.timezone Asia/Shanghai
zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz | mysql -uzabbix -p zabbix
```

### 配置数据库
为Zabbix server配置数据库，编辑配置文件`/etc/zabbix/zabbix_server.conf`。
```bash
sudo vi /etc/zabbix/zabbix_server.conf
# 一定不能有引号，否则会报错
>DBHost=localhost
>DBName=zabbix
>DBPassword=password
>DBUser=zabbix
```

### 启动进程
启动Zabbix server和agent进程。
```bash
sudo apt update
sudo systemctl start zabbix-server zabbix-agent apache2
sudo systemctl enable zabbix-server zabbix-agent apache2
sudo netstat -lnp |grep zabbix
```

### 网页配置
通过网址 _server_ip/zabbix_ 按照页面提示进行配置

### Centos下载Zabbix6.0储存库（二进制安装方式）
```bash
rpm -Uvh https://mirrors.tuna.tsinghua.edu.cn/zabbix/zabbix/6.0/rhel/7/x86_64/zabbix-release-6.0-4.el7.noarch.rpm
```

安装完成后你查看一下你的库里面会有一个zabbix库(/etc/yum.repos.d/zabbix.repo)
```bash
cd /etc/yum.repos.d/
ls
```

可通过`sed`修改源为清华同方源
```bash
sudo sed -i 's#https://repo.zabbix.com#https://mirrors.tuna.tsinghua.edu.cn#' /etc/yum.repos.d/zabbix.repo 
```

清空缓存，安装组件
```bash
yum clean all
yum makecache
yum install zabbix-server-mysql zabbix-agent -y
```

后续和ubuntu类似

## zabbix服务参数介绍
- zabbix server服务名：zabbix-server 端口：10051
- zabbix agent服务名：zabbix-agent端口：10050
- zabbix server主配置文件：/etc/zabbix/zabbix_server.conf
- zabbix agent主配置文件：/etc/zabbix/zabbix_agentd.conf
- zabbix企业微信报警脚本路径：/usr/lib/zabbix/alertscripts
- zabbix自定义监控项路径：/etc/zabbix/zabbix_agentd.d zabbix
- 日志文件路径：/var/log/zabbix/


****

本文参考

> 1. [zabbix介绍及部署（超详细讲解）](https://zhuanlan.zhihu.com/p/614847827)
> 2. [Ubuntu-22.04安装Zabbix](https://zhuanlan.zhihu.com/p/587415883?utm_id=0)