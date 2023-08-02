---
title: Linux命令教程
author: fnoobt
date: 2022-11-27 20:36:00 +0800
categories: [Linux,Linux教程]
tags: [linux,command]
---

## Linux目录结构

linux的文件系统是采用级层式的树状目录结构，在此结构中的最上层是根目录"/"，然后在此目录下再创建其他的目录。

> 在linux中一切皆文件。
{: .prompt-tip }

### 具体的目录结构

/bin (/usr/bin、/usr/local/bin)
: 是Binary的缩写，这个目录存放着最经常使用的命令，由系统、系统管理员和用户共享

/sbin(/usr/sbin、/usr/local/sbin)
: s就是Super User的意思，这里存放系统管理员使用的系统管理程序

/home
: 存放普通用户的主目录，在Linux中每个用户都有一个自己的目录，一般该目录名是以用户的账号命名

/root
: 该目录为系统管理员，也称作超级权限者的用户主目录。注意根目录 / 和根用户的主目录 /root 之间的区别

/lib
: 系统开机所需要最基本的动态链接库，其作用类似于windows下的dll文件。几乎所有应用程序都要用到这些共享库。

/lost+found
: 每个分区在其上目录中都有一个lost+found。故障期间保存的文件在这里，这个目录一般情况下是空的，当系统非法关机后，这里会存放一些文件。

/etc
: 所有的系统管理所需要的配置文件和子目录  
比如安装了mysql，那么my.conf会放在这个目录下

/usr
: 这是一个非常重要的目录，用户的很多应用程序和文件都会放在这个目录下，类似于windows下的program files目录

/boot
: 存放的是启动Linux时的核心文件。包括一些连接文件和镜像文件

/proc
: 包含有关系统资源信息的虚拟文件系统。这个目录是虚拟目录。它是系统内存的映射，访问这个目录来获取系统信息。这个目录的内容不在硬盘上而是在内存里，我们也可以直接修改里面的某些文件

/srv
: service缩写，存放一些服务启动之后需要提取的数据

/sys
: 安装了2.6内核中新出现的一个文件系统sysfs

/tmp
: 存放临时文件，在重新启动时清理，所以不要使用它来保存任何工作!该目录对于所有用户都可以访问，不要把重要文件放置于该目录

/dev
: dev是Device(设备)的缩写, 该目录下存放的是Linux的外部设备，在Linux中访问设备的方式和访问文件的方式是相同的，类似windows下的设备管理器，把所有的硬件用文件的形式存储

/media
: linux系统会自动识别一些设备，例如u盘、光驱等，识别后，linux会把识别的设备挂载到这个目录下

/mnt
: 系统提供该目录是为了让用户临时挂载别的文件系统，例如CD-ROM(光驱)或数码相机  
我们可以将外部的存储挂载到/mnt上,然后进入该目录就可以查看里面的内容  
如:将d:/myshare挂载至/mnt/hgfs/myshare

/opt
: 这是给主机额外安装软件所存放的目录(安装包目录)  
如安装oracle数据库就可放在该目录下,默认为空

/usr/local
: 这是另一个给主机额外安装软件所安装的目录(软件目录)

/var
: 用户创建的所有可变文件和临时文件的存储空间，这个目录中存放着在不断扩充的东西  
将经常被修改的文件/目录放在这个目录之下.如日志文件、邮件队列、打印假脱机程序区、从Internet下载的文件的临时存储空间，或在刻录CD之前保存它的映像

selinux
: security-enhanced linux  
安全子系统,它能控制程序只能访问特定文件,有三种工作模式,可以自行设置

## 开机重启和用户登录注销

### 关机&重启
- 基本介绍
  + `shutdown -h now` 立刻关机
  + `shutdown -h 1` 一分钟后关机
  + `shutdown -r now`重启
  + `halt` 关机
  + `reboot` 重启
  + `sync`把内存的数据同步到磁盘
- 注意细节
  + 重启或关闭系统，首先要运行`sync`命令，把内存中的数据写入磁盘
  + 目前的`shutdown/reboot/halt` 等命令均默认执行`sync`

### 用户登录和注销
- 基本介绍
  + 尽量少用root账号，避免误操作
  + 普通用户登录后再用`su - 用户名`命令来切换成系统管理员
  + 命令行下输入`logout`注销
- 使用细节
  + `logout`注销指令在图形运行级别无效。在运行级别3下有效

## 用户管理
> Linux是一个多用户多任务的操作系统，任何一个要使用系统资源的用户，都必须首先向系统管理员申请一个账号，并以这个账号的身份进入系统

### 添加用户
```bash
useradd 用户名
```
细节说明
- 当创建用户成功后，会自动创建和用户同名的家(home)目录
- 也可以通过`useradd -d 指定目录` 新的用户名给新创建的用户指定家目录  
d就是directory

### 指定/修改密码
```bash
passwd 用户名
```

> 如果不加用户名。那么默认改变当前用户的密码
{: .prompt-tip }

### 删除用户
```bash
userdel 用户名

# 删除用户，但是保留其home目录下的内容
userdel king
# 删除用户和他的主目录
userdel -r king
```
参数
- `-r` 递归(recurisive)

### 查询用户信息指令
```bash
id 用户名
```

### 切换用户
> 如果当前用户权限不够，可以通过 `su - 用户名`指令切换到高权限用户
```
su - 切换用户名
```

细节说明
- 从权限高的用户切换到权限低的用户，不需要输入密码，反之需要
- 当需要返回到原来用户时，使用`exit/logout`指令

### 查看当前用户/登录用户
```bash
# 查看当前操作的用户
whoami
# 查看第一次登录的用户名
who am i
```

### 用户组
>类似于角色，系统可以对有共性(权限)的多个用户进行统一的管理  
必须由root用户创建

```bash
# 新增组
groupadd 组名
# 删除组
groupdel 组名
# 改组名
groupmod -n 新组名 旧组名

# 增加用户时直接加上组
useradd -g 用户组 用户名
# 为用户修改所在组
usermod -g 用户组 用户名
# 改变该用户登录的初始目录
usermod -d 目录名 用户名
```

相关的文件

- /etc/passwd
  + 用户的配置文件，记录用户各种信息
  + 每行的含义->用户名:口令:用户标识号:组标识号:注释性描述:主目录:登录shell
- /etc/shadow
  + 口令的配置文件
  + 每行的含义->登录名:加密口令:最后一次修改时间:最小时间间隔:最大时间间隔:警告时间:不活动时间:失效时间:标志
- /etc/group
  + 组的配置文件
  + 每行的含义->组名:口令:组标识号:组内用户列表

## vi和vim

### vi三种模式
1. 一般模式（normal mode），默认即为一般模式。
2. 编辑模式（insert mode）。
3. 命令模式（last line mode）。

### 三种模式的切换：
- 1.一般模式 --> 编辑模式
  + `i`：当前光标处输入内容。
  + `I`：在当前光标所在行的行首。
  + `a`：在当前光标所在处的后面。
  + `A`: 在当前光标所在行的行尾输入。
  + `o`：在光标所在行的下方新增一行空白行。
  + `O`：在光标所在行的上方新增一行空白行。
- 2.编辑模式 --> 一般模式
  + 使用：ESC键
- 3.一般模式 --> 命令模式
  + 使用：冒号`:`（英文状态下）
- 4.命令模式 --> 一般模式
  + 使用：ESC键

### 退出编辑器
命令模式下，输入下述内容可退出：
1. `q！`：强制退出，不保存并退出。
2. `wq`：保存修改并退出。
3. `x`：保存并退出。

### vi操作流程
1. 新建或编辑文件：`vi filename`
2. `i`或`insert`键，即可进入插入模式。
3. 编辑内容。
4. ESC键，退出到一般模式。
5. 键入英文冒号`:`进入命令模式，按`wq`（保存并修改）后回车。
6. 查看编辑内容是否正确：`cat filename`

### 拓展
在一般模式下

| 操作 |                 功能                 | 操作 |                 功能                 |
|:----:|:------------------------------------:|:----:|:------------------------------------:|
|   d  |                 删除                 | dd   | 删除一行                             |
|   y  |                 复制                 | yy   | 复制一行                             |
|   p  |                 粘贴                 |      |                                      |
|   x  |                 清除                 |      |                                      |
|   G  |              页尾行跳转              |  gg  | 页首行跳转：如10gg，表示跳转到第10行 |
|   /  |             正向往下搜索             |   ?  |             反向往上搜索             |
|   n  |             正向往下查找             |   N  |             往上反向查找             |
|   u  |            撤销之前的操作            |      |                                      |
|   v  | 可视化模式。该模式可移动光标选择文本 |      |                                      |

## Linux基本命令操作

### 指定运行级别
> 常用运行级别是3和5. 可以指定默认运行级别

- 0:关机
- 1:单用户【可找回丢失密码】
- 2:多用户状态，无网络服务
- 3:多用户状态，有网络服务
- 4:系统未使用，保留给用户
- 5:图形界面
- 6:系统重启

切换运行级别 `init [0|1|2|3|4|5|6]`

#### centos7下对应的运行级别

```
# /etc/inittab
# multi-user.target: analogous to runlevel 3
# graphical.target: analogous to runlevel 5
```
{: file='/etc/inittab'}

```bash
# 查看现在默认的运行级别
systemctl get-default
# 设置默认运行级别
systemctl set-default TARGET.target
```

### 帮助指令（man、help）

#### man
> 获得帮助信息

```bash
man [命令或配置文件]

# 查看ls命令的帮助
man ls
```

#### help
>获得shell内置命令的帮助信息

```
help 命令
```

### 目录的相关命令(cd、pwd、ls、mkdir、rmdir、du)

#### cd

> 切换到指定目录

```bash
# 未进行切换目录。 因为'.'代表当前路径
cd .
# 切换到当前目录的上一级目录。'..'代表上一级目录
cd ..
# 切换到用户的家目录
cd ~ 
# 切回到上一次所在的目录
cd -
```

#### pwd

> 打印当前所在目录 (print working directory)

```bash
pwd
```

#### ls

> 查看当前目录的所有内容信息

```bash
# 列出的文件以长格式输出，一个文件显示一行（可简写为ll）
ls -l
# 显示以 “.”开头的文件，“.”开头的为隐藏文件，默认不显示
ls -a
# 显示目录本身而不显示目录下的文件，默认ls 后面的参数如果是目录，则会显示目录下的文件，如：ls /root
ls -d
# 长格式输出的文件字节数转换为K,M,G的形式方便人来阅读
ls -lh
# 列出的文件按照修改时间的晚和早排序（最近修改的先显示）
ls -t
# 列出的文件按照修改时间的早和晚排序（最近修改的后显示）
ls -tr
# 列出当前目录下的所有文件，如果有目录遍历所有目录及其子目录下的文件
ls -R
```

#### mkdir

> 创建目录

```bash
mkdir  [选项] 要创建的目录
```
常用选项
- `-p`创建多级目录

#### rmdir

> 删除空目录

```bash
rmdir [选项] 要删除的空目录
```
使用细节  
- rmdir删除的是空目录，要求这个目录下没有子文件也没有文件夹
- 如果需要删除非空目录，需要使用`rm -rf` 要删除的目录  
例如`rm -rf /home/animal
`
#### du

> 打印当前所在目录 (print working directory)

```bash
pwd
```

### 文件操作命令(touch、cp、mv、rm、file)

#### touch
> 创建空文件，如果文件已经存在修改文件的修改日期

```bash
touch 空文件名称
```

#### cp
> 拷贝文件到指定目录

```bash
cp [选项] source dest
```

常用选项
- `-r`递归复制整个文件夹

使用细节
- 使用`\cp`强制覆盖不提示

#### mv
> 移动文件与目录 或重命名

```bash
mv oldFileName newFileName (重命名)
mv /temp/movefile /targetFolder (移动文件)
```

#### rm
> 删除文件或目录

```bash
rm [选项] 要删除的文件或目录
```

常用选项
- `-r`递归删除所有文件
- `-f`强制删除，不提示（不用用户键入'y'或'n'进行确认）

#### file
> 查看文件的类型

```bash
file -f 文件名称
```

### 文件内容查看命令(cat、tac、more、less、head/tail、echo、>/`>>`、ln、history)

#### cat
> 查看文件内容

```bash
cat [选项] 要查看的文件
```

常用选项
- -n显示行号

使用细节
- cat只能浏览文件，而不能修改文件
- 为了浏览方便，一般会使用**管道命令**如`| more`

#### tac
> 查看文本文件内容，倒序输出，按照行号倒序打印文本文件的内容

```bash
tac [选项] 要查看的文件
```

#### more
> 一个基于VI编辑器的文本过滤器

```bash
more 要查看的文件
```

|  操作  |       功能说明       |
|:------:|:--------------------:|
|  space |      向下翻一页      |
|  enter |      向下翻一行      |
|    q   |       立刻离开       |
| ctrl+f |    向下滚动一个屏    |
| ctrl+b |      返回上一屏      |
|    =   |     输出当前行号     |
|   :f   | 输出文件名和当前行号 |

#### less
> 分屏查看文件，对大型文件有较高的效率

```bash
less 要查看的文件
```

|    操作    |                 功能说明                 |
|:----------:|:----------------------------------------:|
|    space   |                向下翻一页                |
| [pagedown] |                向下翻一页                |
|  [pageup]  |                向上翻一页                |
|    /字串   | 向下搜寻[字串]；n：向下查找，N：向上查找 |
|    ?字串   | 向上搜寻[字串]；n：向上查找，N：向下查找 |
|      q     |                 立刻离开                 |

#### head/tail
> 显示文件开头/结尾的部分内容，默认前10行

```bash
# 显示文件开头前10行
head 文件名
# 显示文件开头前5行
head -n 5 文件名
# 显示文件结尾后10行
tail 文件名
# 显示文件结尾后5行
tail -n 5 文件名
# 实时监控文件追加的内容
tail -f 文件名
```

#### echo
> 输出内容到控制台

```bash
echo [选项] [输出内容]
# 输出环境变量
echo $PATH
# 输出主机名
echo $HOSTNAME
# 输出hello,world
echo hello,world
```

#### >/`>>`
> `>`输出重定向，`>>`追加

```bash
# 列表的内容写入文件(覆盖)
ls -l > 文件
# 列表的内容追加进文件
ls -l >> 文件
# 将文件1的内容覆盖到文件2
cat 文件1 > 文件2
echo "内容" >> 文件
```

#### ln
> 软链接，存放其他文件的路径 ln->link

```bash
ln -s [原文件或目录] [软连接名] (给文件创造一个软连接)
rm 软连接名 (删除软连接)
```
细节说明
- 使用pwd查看目录时，看到的是软连接所在目录

#### history
> 查看已经执行的命令，也可以执行某个历史命令

```bash
# 显示所有历史命令
history
# 显示近期10个历史命令
history 10
# 执行历史中第10条命令
!10
```
### 系统管理类命令（hutdown、reboot、lscpu）

#### shutdown
> 关机命令

```bash
# 立刻关机
shutdown -h now
# 每个登录用户收到“10分钟后关机”的消息，并于10分钟后关机
shutdown -h +10 "10分钟后关机"
# 取消关机
shutdown -c
```

#### reboot
> 重启系统

```bash
reboot
```

#### lscpu
> 查看系统cpu信息

```bash
lscpu
```

### 日期时间管理类命令（date、clock、cal）

#### date
> 打印操作系统时钟

```bash
# 按照格式设置打印时间信息
date 
date +%Y
date +%m
date +%d
date "+%Y-%m-%d %H-%M-%S"
# 按照指定日期重新设定日期和时间
date -s 2022-11-27 20:02:10
```

#### clock
> 打印硬件时钟（主板中依靠纽扣电池保存在芯片中的时钟）

```bash
# 按照硬件时钟设置操作系统时钟
clock -s
# 按照操作系统时钟设置硬件时钟
clock -w
```

#### cal
> 查看日历

```bash
# 显示当前日历
cal
# 显示2022年日历
cal 2022
```

### 搜索查找类（find、locate、which、grep与|、who、w）

#### find
> 从指定目录向下递归地遍历其各个子目录并显示

```bash
find [搜索范围] [选项]
# 在/home目录下查找java文件
find /home -name *.java
# 在/opt目录下查找属于jack的文件
find /opt -user jack
# 查找整个linux系统下大于200M的文件，+n大于n，-n小于n，n等于n
find / -size +200M
```

|  选项 |               功能              |
|:-----:|:-------------------------------:|
| -name |      按照指定文件名查找模式     |
| -user |      查找属于指定用户的文件     |
| -size | 按指定大小查找(单位可以是M,G,k) |

#### locate
> 快速定位文件路径，使用前需要执行`updatedb`

```bash
# locate基于数据库查询，必须定期更新
updatedb
locate 文件名
```

#### which
> 查找某个指令在磁盘的什么位置

```bash
which [命令名称]
```

#### grep与|
> grep过滤查找，|表示将前一个命令的结果传递给后面的命令处理

```bash
grep [选项] 查找内容 源文件
```

常用选项
- `-n`显示行号
- `-i`忽略字母大小写

#### who
> 当前用户登录的信息

```bash
who [选项]
```

#### w
> 当前用户登录的信息，以什么程序登录的

```bash
w [选项] [用户名]
```

### 压缩和解压类（gzip/gunzip、zip/unzip、bzip2/bunzip2、tar）

#### gzip/gunzip
> 默认压缩之后不会保留源文件，生成*.gz文件

```bash
gzip 文件
gunzip 文件
```

#### zip/unzip
> 解压缩指令，生成*.zip文件

```bash
zip [选项] xxx.zip 将要压缩的内容
unzip [选项] xxx.zip

# 递归压缩，压缩目录
zip -r
# 指定解压后文件的存放目录
unzip -d <目录>
# 将home目录及其子文件和子文件夹都压缩
zip -r myhome.zip ~/
# 将myhome.zip解压到/opt/tmp下
unzip -d /opt/tmp myhome.zip
```

#### bzip2/bunzip2
> 默认压缩之后不会保留源文件，生成*.bz2文件

```bash
# 压缩/root/install.log文件
bzip2 /root/install.log
# 解压缩/root下install.log.bz2
bunzip2 /root /install.log.bz2
```

#### tar
> 打包指令，生成.tar.gz文件

```bash
tar [选项] xxx.tar.gz 打包的内容

# 将后面的两个文件压缩成pc.tar.gz
tar -zcvf pc.tar.gz /home/pig.txt /home/cat.txt
# 将文件夹/home/压缩为myhome.tar.gz
tar -zcvf myhome.tar.gz /home/
# 将文件解压至当前文件夹
tar -zxvf pc.tar.gz
# 将文件解压至/opt/tmp
tar -zxvf myhome.tar.gz -C /opt/tmp
```

| 选项 |        功能        |
|:----:|:------------------:|
|  -c  |  产生.tar打包文件  |
|  -v  |    显示详细信息    |
|  -f  | 指定压缩后的文件名 |
|  -z  |    打包同时压缩    |
|  -x  |    解包.tar文件    |

## 组管理和权限管理

>在linux中的每个用户必须属于一个组，不能独立于组外。  
在linux中每个文件都有所有者，所在组，其他组的概念。

### 文件/目录所有者

```bash
# 查看文件所有者
ls -ahl
# 修改文件所有者(单独修改文件所有者，不会影响其所在组)，可选参数 -R 递归执行
chown 用户名 文件名
# 修改文件所在组，可选参数 -R 递归执行
chgrp 组名 文件名
```

### 组的创建
```bash
# 创建组
groupadd monster
# 创建一个用户并指定组
useradd -g monster tom
```

### 权限的基本介绍

```bash
-rw-r--r--   1 root root     21 Oct  4  2022 hello.txt
```

`-rw-r--r--`说明
- 第0位 文件类型
  + `l`是链接
  + `d`是目录
  + `c`是字符设备文件，如鼠标/键盘
  + `b`是块设备，如硬盘
  + `-`是文件
- 第1~3位 文件所有者的权限
- 第4~6位 同组用户的权限
- 第7~9位 其他组用户的权限

其他说明
- `1`          文件：硬连接数或目录：子目录数
- `root`       用户
- `root`       组
- `21`         文件大小(单位：字节)
- `Oct 4 2022` 最后修改时间
- `hello.txt`  文件名

rwx权限详解
- rwx作用于文件
  + r代表可读
  + w代表可写  
  (不代表可删除，可删除文件的前提是对此文件所在目录具有写权限)
  + x代表可执行
- rwx作用于目录
  + r代表可读，可以使用ls查看目录内容
  + w代表可写，可以修改。可在目录内创建+删除+重命名目录
  + x代表可执行，即可以进入该目录

> 可以用数字表示；r=4,w=2,x=1；rwx=4+2+1=7

### 修改权限
> chmod(全称:change mode);控制用户对文件的权限的命令

#### 1.`+`、`-`、`=`变更权限

`u`所有者;`g`所有组;`o`其他人;`a`所有人(即u+g+o)
```
chmod u=rwx,g=rx,o=x 文件/目录
chmod o+w 文件/目录
chmod a-x 文件/目录
```

#### 2.通过数字变更权限
`r=4 w=2 x=1`
```bash
# 文件/目录相当于chmod 751 文件/目录
chmod u=rwx,g=rx,o=x 
# 将/home/abc.txt文件的权限修改成rwxr-xr-x
chmod 755 /home/abc.txt
```

### 修改文件所有者

```bash
# 文件/目录 (改变所有者)
chown newowner
# 改变所有者和所在组
chown newowner:newgroup 文件/目录
```
参数 -R：如果是目录，则递归生效

## 定时任务调度

### crontab循环定时任务
> crontab 进行定时任务的设置

```
crontab [选项]
```
选项参数：
- `-e` 编辑crontab定时任务
- `-l` 查询crontab定时任务
- `-r` 删除当前用户所有的crontab任务

`service crond restart` 重启任务调度

#### 快速入门
设置任务调度文件:`/etc/crontab`

案例：每小时每分钟执行一次 `ls -l /etc/ > /tmp/to.txt`
```bash
crontab -e
*/1 * * * * ls -l /etc/ -> /tmp/to.txt
```

#### 5个占位符的说明

|   项目  |              含义              |
|:-------:|:------------------------------:|
| 第一个* |       一小时中的第几分钟       |
| 第二个* |        一天中的第几小时        |
| 第三个* |         一个月的第几天         |
| 第四个* |         一年中的第几月         |
| 第五个* | 一周中的星期几 (0,7都代表周日) |

#### 特殊符号的说明

| 特殊符号 |                                  含义                                 |
|:--------:|:---------------------------------------------------------------------:|
|     *    |             代表任何时间，如第一个*代表一小时中的每一分钟             |
|     ,    | 代表不连续的时间，如0 8,12,16 * * *代表每天的8:00,12:00,16:00执行一次 |
|     -    |     代表连续的时间范围，如0 5 * * 1-6代表周一到周六的5:00执行一次     |
|    */n   |        代表每隔多久执行一次，如*/10 * * * *代表每10分钟执行一次       |

#### 应用案例
```bash
# 每隔1分钟，将当前的日期信息追加到/tmp/mydate中
*/1 * * * * date >> /tmp/mydate/

# 每天2:00将mysql数据库testdb备份到文件中
0 2 * * * mysqldump -u root -p root testdb > /home/db.bak
```

每隔1分钟，将当前日期和日历都追加到/tmp/mycal中
1. `vim /home/my.sh`写入`date >> /tmp/mycal`和`cal >> /tmp/mycal`
2. `chmod u+x /tmp/my.sh`给`my.sh`增加执行权限
3. `crontab -e`并写入`*/1 * * * * /home/my.sh`

### at一次性定时任务
> 一次性定时计划任务。  
at的守护进程atd会以后台模式运行，检查作业队列，时间到了就运行此作业。  
使用at命令时，一定要确保atd进程的启动。(`ps -ef | grep atd`)

```bash
at [选项] [时间]

# 结束at命令的输入
ctrl+d
```

参数：
- `-m`：当at的工作完成后，即使没有输出信息，以email通知用户该工作已完成
- `-l`：at -l相当于atq，列出目前系统上面的所有该用户的at调度
- `-d`：at -d相当于atrm,可以取消一个在at调度中的工作
- `-v`：可以使用较明显的时间格式列出at调度中的任务列表
- `-c`：可以列出后面接的该项工作的实际命令内容
- `-f`：从文件中读取作业

## Linux磁盘分区、挂载

### 分区与文件系统

1. Linux中每个分区都是用来组成整个文件系统的一部分
2. Linux采用了一种叫做"载入"的处理方法，它的整个文件系统中包含了一整套的文件和目录，且将一个分区和一个目录联系起来。这时，要载入的一个分区将使它的存储空间在一个目录下获得。

查看所有设备挂载情况

```bash
lsblk 
lsblk -f
```

### 挂载的经典案例
增加一块硬盘
1. 虚拟机添加硬盘
2. 分区
3. 格式化
4. 挂载
5. 设置可以自动挂载

### 磁盘工作实用指令

```bash
# 统计/opt文件夹下文件的个数
ls -l /opt | grep "^-" | wc -l
# 统计/opt文件夹下目录的个数
ls -l /opt | grep "^d" | wc -l
# 统计/opt文件夹下文件的个数，包括子文件夹里的
ls -lR /opt | grep "^-" | wc -l
# 统计/opt文件夹下文件的个数，包括子文件夹里的
ls -lR /opt | grep "^d" | wc -l
# 以树状显示目录结构,如果没有tree指令，使用yum install tree
tree /home/
```

### wc命令
> word count 用于统计一个文件的行数，字数，字节数或字符数

参数：
- `-c` 统计字节数
- `-k` 统计字符数
- `-l` 统计行数
- `-w` 统计字数，一个字被定义为由空白、跳格或换行字符分隔的字符串

## 网络配置
### 设置主机名
1. 为了方便记忆，可以给linux主机设置主机名
2. 指令`hostname` 查看主机名
3. 修改文件`/etc/hostname`，可以指定主机名
4. 修改后，重启生效

### 设置hosts映射

windows下
: 在`C:\Windows\System32\drivers\etc\hosts`文件指定

linux下
: 在`/etc/hosts/`文件指定

### 主机名解析过程分析
#### Hosts
> 一个文本文件。记录IP和Hostname(主机名)的映射关系

#### DNS
> Domain Name System 域名系统  
互联网上作为域名和IP地址相互映射的一个分布式数据库

#### 机制分析
用户在浏览器输入了`www.baidu.com`
1. 浏览器先检查浏览器缓存中有没有该域名解析IP地址，再检查DNS解析器缓存。二者都可以理解为本地解析器缓存。
2.计算机第一次成功访问某一网站后，一定时间内。浏览器或操作系统会缓存DNS解析记录。  
`ipconfig /displaydns` 查看dns域名解析缓存  
`ipconfig /flushdns` 手动清理dns缓存
3. 如果1中没有找到，那么检查系统的hosts文件。
4. 如果1，3都未找到，那么到域名服务dns进行解析。

## 进程管理
1. 在Linux中，每个执行的程序都称作一个进程。  
每个进程都分配一个ID号(pid)
2. 每个进程都可能以两种方式存在(前台/后台)  
前台进程就是用户目前的屏幕上可以操作的。  
后台进程就是实际在操作，但是屏幕上无法看到。
3. 一般系统的服务都以后台进程的方式存在，并且常驻在系统中，直至关机。

### 显示系统执行的进程
> ps命令用来查看目前系统中，有哪些正在执行，以及他们执行的状况

ps显示的信息

| 字段 |          说明          |
|:----:|:----------------------:|
|  PID |         进程ID         |
|  TTY |        终端机号        |
| TIME |   此进程消耗的CPU时间  |
|  CMD | 正在执行的命令或进程名 |

ps的参数
- `ps -a`显示当前终端所有进程信息
- `ps -u`以用户的形式显示进程信息
- `ps -x`显示后台进程运行的参数

运行`ps -aux`

- `USER`:执行进程的用户
- `PID`:进程ID
- `%CPU`:占用CPU的百分比
- `%MEM`:占用物理内存的百分比
- `VSZ`:占用虚拟内存；  
包括进程可以访问的所有内存（堆栈），包括进入交换分区的内容，以及共享库占用的内存。
- `RSS`:常驻内存集（Resident Set Size），表示该进程分配的内存大小。  
不包括进入交换分区的内存,包括共享库占用的内存（只要共享库在内存中）,包括所有分配的栈内存和堆内存。
- `TTY`:由虚拟控制台、串口以及伪终端设备组成的终端设备；终端名称
- `STAT`:进程状态；S->sleep，R->running/runable等等  
[linux进程查看中STAT的含义][STAT]
- `START`:启动时间
- `TIME`：占用的CPU时间
- `COMMAND`:进程名/执行该进程时的指令



### 终止进程
```bash
# 通过进程号杀死进程
kill [选项] 进程号
# 通过进程名称杀死进程，支持通配符
killall 进程名称
```

常用选项：`-9`强迫进程立刻停止

### 查看进程树pstree
```bash
pstree [选项]
```
pstree [选项]

常用选项
- `-p`显示pid
- `-u`显示进程所属用户

## 服务管理
> 服务本质就是后台进程，通常都会监听某个端口，等待其他程序的请求。  
比如mysqld,sshd,防火墙等，又称为守护进程

### service管理指令
1. `service 服务名 [start|stop|restart|reload|status]`
2. 在centos7后，很多服务不使用`service`，而是`systemctl`
3. `service`指令管理的服务在`etc/init.d`查看

### 查看服务名
1. 直接输入`setup`，可以看到全部服务  
"*"号表示自启动
2. `ls -l /etc/init.d/`可以看到`service`指令管理的服务

### 服务的运行级别
- Linux系统的7种运行级别
  + 0:关机
  + 1:单用户【可找回丢失密码】
  + 2:多用户状态，无网络服务
  + 3:多用户状态，有网络服务，控制台界面
  + 4:系统未使用，保留给用户
  + 5:图形界面
  + 6:系统重启
- 开机流程说明  
  + 1.开机
  + 2.IOS
  + 3./boot
  + 4.systemd进程1
  + 5.运行级别
  + 6.启动此运行级别对应的服务
  
### chkconfig
> 1. 通过chkconfig可以给服务的各个运行级别设置自启动/关闭
2. chkconfig能够管理的服务在etc/init.d/查看
3. centos7后，很多服务使用systemctl管理

```bash
chkconfig --list [| grep xxx]
chkconfig 服务名 --list
chkconfig --level 5 服务名 on/off
```

使用细节：
- 使用chkconfig后，重启生效

### systemctl

- 基本语法：`systemctl [start|stop|restart|status]` 服务名
- systemctl指令管理的服务在`/usr/lib/systemd/system`查看

#### 设置服务自启动状态

- `systemctl list-unit-files [|grep 服务名]`查看服务自启动状态
- `systemctl enable` 服务名设置开机启动
- `systemctl disable` 服务名关闭开机启动
- `systemctl is-enabled` 服务名查询某个服务是否自启动

应用案例
```bash
# 查看当前防火墙的状况
systemctl status firewalld
# 关闭/重启防火墙
systemctl stop/restart firewalld
```

细节讨论
1. 关闭或启用防火墙，会立刻生效。可以使用`telnet`测试  
指定IP和端口的连接 `telnet ip:port`
2. 这种方式仅临时生效，重启后回归以前的设置。
3. 如果希望永久生效，使用`systemctl [enable|disable]` 服务名

### firewall
```bash
# 打开端口
firewall-cmd --permanent --add-port = 端口号/协议
# 关闭端口
firewall-cmd --permanent --remove-port = 端口号/协议
# 重新载入，设置生效
firewall-cmd --reload
# 查询端口情况
firewall-cmd --query-port = 端口号/协议
```

动态监控进程
```bash
top [选项]
```
选项说明
- `-d`  指定top命令每隔几秒更新，默认3秒
- `-i`	使top不显示任何闲置/僵死进程
- `-p`	通过ID指定监控进程

#### 交互操作说明

| 操作 |            功能           |
|:----:|:-------------------------:|
|   P  | 以CPU使用率排序，默认选项 |
|   M  |      以内存使用率排序     |
|   N  |         以PID排序         |
|   q  |          退出top          |

案例
- 1.监视特定用户
  + 1.输入top查看执行的进程
  + 2.然后输入u回车，再输入用户名
- 2.终止指定进程
  + 1.输入top查看执行的进程
  + 2.然后输入k回车，再输入进程ID号
- 3.指定系统状态更新的时间
  + `top -d 10`

### 监控网络状态
```bash
netstat [选项]
```

选项说明
- `-an`按一定顺序排列输出
- `-p`显示哪个进程在调用

案例
```
#查看sshd服务的信息
netstat -anp | grep sshd
```

## rpm和yum
### RPM
> rpm :redhat package manager -> redhat 软件包管理工具  
用于互联网下载包的打包及安装

#### rpm包的简单查询指令
`rpm -qa | grep xx`查询已安装的rpm列表

#### rpm包名基本格式
>name-version-release.os.arch.rpm  
软件名称-版本号-发布次数.适合linux系统.硬件平台.rpm

一个rpm包名`sqlite-3.7.17-8.el7_7.1.x86_64`
- 名称：sqlite
- 版本号：3.7.17-8
- 适用操作系统：el7_7.1.x86_64
  + el 是 Red Hat Enterprise Linux 的简写  
  包括Red Hat x.x，CentOS x.x，CloudLinux x.x
  + 表示centos7.x的64位系统
  + 如果是i686、i386表示32位系统，noarch表示通用

#### rpm包其他查询
```bash
# 查询软件包是否安装
rpm -q 软件包名
# 查询软件包信息
rpm -qi 软件包名
# 查询软件包中的文件
rpm -ql 软件包名
# 查询文件所属软件包
rpm -qf 文件全路径名
```

#### 卸载rpm包

```bash
rpm -e 软件包名
```

细节讨论
1. 如果其他软件包依赖于将要卸载的软件包，卸载时会报错
`Failed dependencies:setup is needed by (installed) shadow-utils-2:4.6-5.el7.x86_64`
2. 强制删除，需要增加参数`--nodeps`  
`rpm -e --nodeps setup`

### 安装rpm包
```bash
rpm -ivh 包全路径
```
参数：
- i=install 安装
- v=verbose 提示
- h=hash 进度条

### YUM
> Yum是一个Shell前端软件包管理器，基于RPM包管理。  
可以从指定服务器自动下载rpm包并且安装，自动处理依赖

基本指令
```bash
# 查询yum服务器是否有需要安装的软件
yum list | grep xx
# 安装指定yum包
yum install xxx
```

****

本文参考

> 1. [白色的风车](https://blog.csdn.net/weixin_52333476/article/details/127269431?spm=1001.2101.3001.6650.6&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-6-127269431-blog-131484976.235%5Ev38%5Epc_relevant_sort&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-6-127269431-blog-131484976.235%5Ev38%5Epc_relevant_sort&utm_relevant_index=7)
> 2. [程序前行者](https://blog.csdn.net/m0_62808124/article/details/127540625)

[STAT]:http://fnoobt.github.io/posts/linux-ps-stat/