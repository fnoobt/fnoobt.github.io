---
title: EVE-NG 使用教程
author: fnoobt
date: 2023-10-23 15:29:00 +0800
categories: [Network,虚拟环境]
tags: [network,eve-ng]
---

EVE-NG（Emulated Virtual Environment - NextGeneration）下一代仿真虚拟环境，是一种基于软件定义网络（SDN）和网络功能处理（NFV）技术的网络模拟器和实验平台，可以模拟各大厂商（如华为、华三、思科等）的网络设备，路由器，交换机，防火墙等等，强大的模拟功能是因为三大组件：Dynamips，IOL，QEMU：
- **Dynamips**是用于模拟思科路由器的，基于它的模拟器有小凡，以及大名鼎鼎的GNS3，但它只能模拟路由，交换基本上都不行
- **IOL**是将思科的ios运行在linux上，可以很好的支持交换功能，基于它的模拟器有web-iou
- **QEMU**是一套开源产品，是用纯软件实现的模拟器，几乎可以模拟任何硬件设备。[eve支持的镜像](https://www.eve-ng.net/index.php/documentation/supported-images/)

## 1. 安装VMware Workstation
EVE-NG是运行在VMware Workstation中，所以需要先下载安装VMware Workstation，版本不限。

[Workstation 17 Pro for Windows官方下载链接](https://www.vmware.com/go/getworkstation-win)

## 2. 安装EVE-NG

### 2.1. 下载EVE-NG
[EVE-NG官方下载链接](https://www.eve-ng.net/index.php/download/)

### 2.2. 导入VMware

把`.ovf`文件直接拖入VMware，设置虚拟机名称和存储路径。

### 2.3. 虚拟机设置配置

一般就用默认的就好，为方便连接防止IP变化，建议将网络模式改成NAT。

### 2.4. 启动eve
默认用户名root，密码eve（SSH和FTP都是这个账号密码）

首次登录，会提示密码、名称、 域名、联网方式等信息，一般这些都不需要改动，直接回车重启之后就可以正常使用了。

初始界面会提示IP地址，这是SSH、FTP和浏览器登录的连接地址

### 2.5. 浏览器登录
官方说明火狐兼容性最好，默认用户名admin，密码eve

## 3. 镜像

### 3.1. 镜像的导入使用

- iol镜像位置：`/opt/unetlab/addons/iol/bin`{: .filepath}

- qemu镜像位置：`/opt/unetlab/addons/qemu`{: .filepath}
: 镜像文件一般以`.qcow`或`.qcow2`结尾，放在自己创建的文件夹中，该文件夹名称一般为 '设备名称' + ' - ' + '版本号'三部分构成，并且该文件名的第一部分必须与模版文件名相同，才能被识别成镜像

- 设备图标位置：`/opt/unetlab/html/images/icons`{: .filepath}
: 一般以png或者是jpg图片，直接导入即可，没有严格的命名要求，最好是与设备名相同，好区分

- 设备模版位置：`/opt/unetlab/html/templates`{: .filepath}
: 路径下有`amd`和`intel`两个文件夹，根据自己的cpu设备品牌对应，模版一般以`.yml`结尾，注：模版文件名应与镜像文件夹的第一部分相同，这样镜像文件才能被识别成镜像

- 设备脚本位置：`/opt/unetlab/scripts`{: .filepath}
: 常见的一般以`.py`结尾，用来给部分设备镜像导入配置用。

以AMD CPU电脑导入H3CvSR为例，通过FTP上传镜像文件，镜像文件存储路径：`/opt/unetlab/addons/qemu/h3cvsr-710-CMV710-R1340P1201/hda.qcow2`{: .filepath}

模版文件存储路径: `/opt/unetlab/html/templates/amd/h3cvsr.yml`{: .filepath}

如果不确定电脑cpu架构，可以通过以下方式来查看是intel还是amd：
```bash
root@eve-ng:~# lsmod | grep ^kvm_
kvm_amd               131072  0
```

可以参照模板文件夹中已有的文件进行创建，下面是H3CvSR的模版文件

```yml
---
type: qemu
description: H3C VSR1000   # 添加设备节点在列表显示的名称
name: VSR1000              # 添加设备后默认显示的名称
cpulimit: 1
icon: Router.png           # 设备图标，存放在/opt/unetlab/html/images/icons路径下
cpu: 2                     # CPU数
ram: 4096                  # 内存（MB）
ethernet: 2                # 端口数量
eth_format: em{0}          # 设备接口命名
console: telnet            # 连接设备的方式，交换机、路由器一般选择telnet连接，其他需要可视化界面的可以改为vnc连接
qemu_arch: x86_64
qemu_version: 4.1.0
qemu_options: -machine type=pc,accel=kvm -nographic -rtc base=utc
...
```
{: file='/opt/unetlab/html/templates/amd/h3cvsr.yml'}

### 3.2. 修正文件权限
导入镜像一系列文件后，需要使用下面这条命令修正新上传的镜像文件的权限

```bash
/opt/unetlab/wrappers/unl_wrapper -a fixpermissions
```

### 3.3. 验证镜像是否可用
在lab平台上右击点击新建节点就可以看到自己新导入的节点了

## 4. 关联软件及相关问题解决

### 4.1. eve-ng客户端
eve-ng客户端（EVE-NG-Win-Client-Pack-2.0）为必需安装的前置软件，大多数人软件关联失败都是因为没下这个，[官方下载链接](https://www.eve-ng.net/index.php/download/#DL-WIN)

### 4.2. sercureCRT（控制台连接软件）
这个软件是为了响应eve-ng telnet连接，安装教程暂略。

如果不想每次点击设备都弹出一个新窗口，可通过如下方式处理：
1. 通过sercureCRT软件的 <kbd>Options</kbd> --> <kbd>Global Options</kbd> --> <kbd>Configuration Paths</kbd> --> <kbd>Configuration folder</kbd>查找到配置文件路径
2. 在该路径下找到`Global.ini`文件
3. 修改该文件中`D:"Single Instance"=00000000`的值为`D:"Single Instance"=00000001`
4. 保存退出就行了

### 4.3. wireshark（抓包软件）
为了进行抓包分析，安装教程暂略。

抓包问题：

1. 打不开抓包软件

打开eve-ng客户端安装路径`C:\Program Files\EVE-NG`{: .filepath}下的`wireshark_wrapper.bat`，编辑路径改成wireshark实际安装路径
```
"C:\Program Files\EVE-NG\plink.exe" -ssh -batch -pw %PASSWORD% %USERNAME%@%HOST% "tcpdump -U -i %INT% -s 0 -w -%FILTER%" | "C:\Program Files\Wireshark\Wireshark.exe" -k -i -
```
{: file='C:\Program Files\EVE-NG\wireshark_wrapper.bat'}

2. 抓包为空

在eve-ng客户端中打开`putty.exe`，连接EVE虚拟机保存密钥，再尝试打开wireshark发现就能抓到包了

****

本文参考

> 1. [EVE-NG详细安装使用指南](https://blog.csdn.net/balabalado/article/details/131867478)
> 2. [EVE-NG模拟器升级并添加H3C设备](https://blog.csdn.net/feidrang/article/details/104991344)
> 3. [EVE-NG社区版使用指南](https://zhuanlan.zhihu.com/p/593432015)