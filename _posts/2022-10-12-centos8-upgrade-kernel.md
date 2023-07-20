---
title: Centos 8升级内核
author: fnoobt
date: 2022-05-09 00:34:00 +0800
categories: [Linux,kernel]
tags: [Linux,kernel]
---

1. 查看当前内核版本

使用的系统版本，当前日期CentOS最新版：

```bash
$ cat /etc/redhat-release 
CentOS Linux release 8.2.2004 (Core)
```

查看当前系统内核版本：

```bash
$ uname -r
4.18.0-193.6.3.el8_2.x86_64
```

当前日期 Linux 的内核很多都 5.x，各方面考虑还是有必要升级一下的，内核可以从这里直接下载：<https://www.kernel.org/>

2. 使用ELRepo仓库

这里使用ELRepo仓库，ELRepo 仓库是基于社区的用于企业级 Linux 仓库，提供对 RedHat Enterprise（RHEL）和其他基于 RHEL的 Linux 发行版（CentOS、Scientific、Fedora 等）的支持。ELRepo 聚焦于和硬件相关的软件包，包括文件系统驱动、显卡驱动、网络驱动、声卡驱动和摄像头驱动等。<http://elrepo.org/tiki/tiki-index.php>:

导入ELRepo仓库的公共密钥：

```bash
$ rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
```

安装ELRepo仓库的yum源：

```bash
$ yum install https://www.elrepo.org/elrepo-release-8.el8.elrepo.noarch.rpm
```

可用的系统内核安装包：

```bash
$ yum --disablerepo="*" --enablerepo="elrepo-kernel" list available
bpftool.x86_64 5.7.7-1.el8.elrepo elrepo-kernel
kernel-ml-devel.x86_64 5.7.7-1.el8.elrepo elrepo-kernel
kernel-ml-doc.noarch 5.7.7-1.el8.elrepo elrepo-kernel
kernel-ml-headers.x86_64 5.7.7-1.el8.elrepo elrepo-kernel
kernel-ml-modules-extra.x86_64 5.7.7-1.el8.elrepo elrepo-kernel
kernel-ml-tools.x86_64 5.7.7-1.el8.elrepo elrepo-kernel
kernel-ml-tools-libs.x86_64 5.7.7-1.el8.elrepo elrepo-kernel
kernel-ml-tools-libs-devel.x86_64 5.7.7-1.el8.elrepo elrepo-kernel
perf.x86_64 5.7.7-1.el8.elrepo elrepo-kernel
python3-perf.x86_64 5.7.7-1.el8.elrepo elrepo-kernel
```

3. 安装最新版内核

```bash
$ yum --enablerepo=elrepo-kernel install kernel-ml
```

4. 设置以新的内核启动

0 表示最新安装的内核，设置为 0 表示以新版本内核启动：

```bash
$ grub2-set-default 0
```

以后不需要第5步，直接使用这条指定不同数字设置不同内核版本启动。

5. 生成grub配置文件并重启系统

```bash
$ grub2-mkconfig -o /boot/grub2/grub.cfg
$ reboot
```

这一步可以不用执行生成grub配置的命令，直接重启！

6. 验证新内核

```bash
$ uname -r
5.7.7-1.el8.elrepo.x86_64
```

7. 查看系统中已安装的内核
可以看到这里一共安装了3个版本的内核，分别是 v4.18.0-193.6.3 和 v4.18.0-147.5.1以及 v5.7.7-1。

```bash
$ rpm -qa | grep kernel
kernel-core-4.18.0-193.6.3.el8_2.x86_64
kernel-modules-4.18.0-147.5.1.el8_1.x86_64
kernel-ml-modules-5.7.7-1.el8.elrepo.x86_64
kernel-devel-4.18.0-147.5.1.el8_1.x86_64
kernel-4.18.0-80.el8.x86_64
kernel-tools-libs-4.18.0-193.6.3.el8_2.x86_64
kernel-core-4.18.0-80.el8.x86_64
kernel-4.18.0-147.5.1.el8_1.x86_64
kernel-modules-4.18.0-80.el8.x86_64
kernel-4.18.0-193.6.3.el8_2.x86_64
kernel-tools-4.18.0-193.6.3.el8_2.x86_64
kernel-ml-5.7.7-1.el8.elrepo.x86_64
kernel-headers-4.18.0-193.6.3.el8_2.x86_64
kernel-core-4.18.0-147.5.1.el8_1.x86_64
kernel-devel-4.18.0-193.6.3.el8_2.x86_64
kernel-modules-4.18.0-193.6.3.el8_2.x86_64
kernel-ml-core-5.7.7-1.el8.elrepo.x86_64
```

8. 删除旧内核
删除旧内核，这一步是可选的。

```bash
$ yum remove kernel-core-4.18.0 kernel-devel-4.18.0 kernel-tools-libs-4.18.0 kernel-headers-4.18.0
```

再查看系统已安装的内核，确认旧内核版本已经全部删除：

```bash
$ rpm -qa | grep kernel
kernel-ml-modules-5.7.7-1.el8.elrepo.x86_64
kernel-ml-5.7.7-1.el8.elrepo.x86_64
kernel-ml-core-5.7.7-1.el8.elrepo.x86_64
```

也可以安装 `yum-utils` 工具，当系统安装的内核大于3个时，会自动删除旧的内核版本：

```bash
$ yum install yum-utils
```

删除旧的版本使用 `package-cleanup` 命令。

9. 参考文献


- [ELRepo官网](http://elrepo.org/tiki/index.php)

- [Centos7升级内核版本](https://www.cnblogs.com/xzkzzz/p/9627658.html)

****

版权声明：本文为CSDN博主「Erics-1996」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
[原文出处](https://blog.csdn.net/Thanlon/java/article/details/107193301)