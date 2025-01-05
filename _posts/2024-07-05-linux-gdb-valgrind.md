---
title: C/C++ 调试工具 GDB 和 Valgrind
author: fnoobt
date: 2024-07-05 23:24:00 +0800
categories: [Linux,Linux教程]
tags: [linux,c,c++,gdb,valgrind]
---

## GDB
GDB 是由 GUN 软件系统社区提供的调试工具，同 GCC 配套组成了一套完整的开发环境，GDB 是 Linux 和许多类Unix系统的标准开发环境。

> **调试**：就是让代码一步一步慢慢执行，跟踪程序的运行过程。比如，可以让程序停在某个地方，查看当前所有变量的值，或者内存中的数据；也可以让程序一次只执行一条或者几条语句，看看程序到底执行了哪些代码。帮助我们发现代码中的错误，改进代码。
{: .prompt-info }

一般来说，GDB 主要能够提供以下四个方面的帮助：
- 程序启动时，可以按照自定义的要求运行程序，例如设置参数和环境变量；
- 可以让被调试的程序在所指定的代码处暂停运行，并查看当前运行状态 （例如当前变量的值，函数的执行结果），即支持断点调试
- 当程序被停住时，可以检查当前程序的中的变量的状态；
- 在程序执行过程中，可以改变某个变量的值，还可以改变代码的执行顺序，从而尝试修改程序中出现的逻辑错误

### 安装
ubuntu安装
```bash
sudo apt-get install gdb
```

CentOS安装
```bash
sudo yum install -y gdb
```

检查是否安装，若没有反应，则没有安装
```bash
rpm -qa | grep gdb
```

### 调用方式
如果要进入gdb开始调试，你只需要调用GDB运行你的程序即可
```bash
gdb your-prog [your-prog-options]
```

> 当遇到`no debugging symbols found`问题时,是因为可执行程序没有开启 DeBug ，只需要修改一下我们的Makefile，给gcc后面带上一个`-g`的命令选项
{: .prompt-info }

### 指令集
当以 gdb 方式启动gdb后，gdb会在PATH路径和当前目录中搜索源文件。如要确认gdb是否读到源文件，可使用l或list命令，看看gdb是否能列出源代码。

在gdb中，运行程序使用r或是run命令。
- **l(list)** 行号/函数名 —— 显示对应的code，每次10行
- **r(run)** —— F5【无断点直接运行、有断点从第一个断点处开始运行】

#### 暂停/恢复程序运行
调试程序中，暂停程序运行是必须的，GDB可以方便地暂停程序的运行。你可以设置程序的在哪行停住，在什么条件下停住，在收到什么信号时停往等等。以便于你查看运行时的变量，以及运行时的流程。

当进程被gdb停住时，你可以使用`info program` 来查看程序的是否在运行，进程号，被暂停的原因。

在gdb中，我们可以有以下几种暂停方式：**断点（BreakPoint）**、**观察点（WatchPoint）**、**捕捉点（CatchPoint）**、**信号（Signals）**、**线程停止（Thread Stops）**。如果要恢复程序运行，可以使用`c`或是`continue`命令。
##### 设置断点(breakpoint)
我们用`break`命令来设置断点。
- **b function** —— 在指定函数的入口处打断点
- **b filename:function** —— 在指定源文件中的指定函数的入口处打断点
- **b filename:linenum** —— 在指定源文件中的指定行号打断点
- **b linenum** —— 在指定行号打断点
- **b +/- offset** —— 在当前行号的前面/后面的offset行打断点
- **break *address** —— 在程序运行的内存地址处打断点
- **b +/- offset** —— 在当前行号的前面/后面的offset行打断点
- **b** —— break命令没有参数时，表示在下一条指令处打断点
- **break ... if cond** —— 满足条件时打断点，condition表示条件，在条件成立时停住

##### 设置观察点（WatchPoint）
观察点一般来观察某个表达式（变量也是一种表达式）的值是否有变化了，如果有变化，马上停住程序。
- **watch expr** —— 为表达式（变量）expr设置一个观察点。量表达式值有变化时，马上停住程序。
- **rwatch expr** —— 当表达式（变量）expr被读时，停住程序。
- **awatch expr** —— 当表达式（变量）的值被读或被写时，停住程序

##### 设置捕捉点（CatchPoint）
捕捉点用来补捉程序运行时的一些事件。如：载入共享库（动态链接库）或是C++的异常。

- **catch event** —— 当event发生时，停住程序。event可以是下面的内容：
1. throw 一个C++抛出的异常。（throw为关键字）
2. catch 一个C++捕捉到的异常。（catch为关键字）
3. exec 调用系统调用exec时。（exec为关键字，目前此功能只在HP-UX下有用）
4. fork 调用系统调用fork时。（fork为关键字，目前此功能只在HP-UX下有用）
5. vfork 调用系统调用vfork时。（vfork为关键字，目前此功能只在HP-UX下有用）
6. load 或 load 载入共享库（动态链接库）时。（load为关键字，目前此功能只在HP-UX下有用）
7. unload 或 unload 卸载共享库（动态链接库）时。（unload为关键字，目前此功能只在HP-UX下有用）

- **tcatch event** —— 只设置一次捕捉点，当程序停住以后，应点被自动删除。

##### 维护停止点
上面说了如何设置程序的停止点，GDB中的停止点也就是上述的三类。在GDB中，如果你觉得已定义好的停止点没有用了，你可以使用`delete`、`clear`、`disable`、`enable`这几个命令来进行维护。

1. 查看(info)
- **info breakpoints [n]** —— 查看断点的信息(n表示断点号)
- **info break [n]** —— 查看断点的信息(n表示断点号)
- **info watchpoints [n]** —— 查看观察点的信息(n表示断点号)

2. 删除(delete)
- **d [breakpoints] [range...]** —— 删除指定的断点，breakpoints为断点号。如果不指定断点号，则表示删除所有的断点。range 表示断点号的范围（如：3-7）。

3. 清除(clear)
- **clear** —— 清除所有的已定义的停止点。
- **clear linenum** —— 清除所有设置在指定行上的停止点。
- **clear filename:linenum** —— 清除所有设置在指定文件：指定行上的停止点。

4. 禁用(disable)
比删除更好的一种方法是disable停止点，disable了的停止点，GDB不会删除，当你还需要时，enable即可。
- **dis [breakpoints] [range...]** —— disable所指定的停止点，breakpoints为停止点号。如果什么都不指定，表示disable所有的停止点。

5. 启用(enable)
- **enable [breakpoints] [range...]** —— enable所指定的停止点，breakpoints为停止点号。
- **enable [breakpoints] once range...** —— enable所指定的停止点一次，当程序停止后，该停止点马上被GDB自动disable。
- **enable [breakpoints] delete range...** —— enable所指定的停止点一次，当程序停止后，该停止点马上被GDB自动删除。

##### 停止条件
前面在说到设置断点时，我们提到过可以设置一个条件，当条件成立时，程序自动停止，这是一个非常强大的功能，这里，我想专门说说这个条件的相关维护命令。一般来说，为断点设置一个条件，我们使用if关键词，后面跟其断点条件。并且，条件设置好后，我们可以用condition命令来修改断点的条件。（只有 break和watch命令支持if，catch目前暂不支持if）

- **condition bnum expression** —— 修改断点号为bnum的停止条件为expression。
- **condition bnum** —— 清除断点号为bnum的停止条件。
- **ignore bnum count** —— 表示忽略断点号为bnum的停止条件count次。

##### 在停止点添加命令
我们可以使用GDB提供的command命令来设置停止点的运行命令。也就是说，当运行的程序在被停止住时，我们可以让其自动运行一些别的命令，这很有利行自动化调试。对基于GDB的自动化调试是一个强大的支持。
```bash
commands [bnum]
... command-list ...
end
```

为断点号bnum指写一个命令列表。当程序被该断点停住时，gdb会依次运行命令列表中的命令。

如果你要清除断点上的命令序列，那么只要简单的执行一下commands命令，并直接在打个end就行了。

##### 单步和多步调试
1. 单步调试(step/next)
- **s [count]** —— 单步跟踪，如果有函数调用，他会进入该函数。进入函数的前提是，此函数被编译有debug信息。加了count表示执行后面的count条指令，然后再停住。
- **n [count]** —— 同样单步跟踪，如果有函数调用，他不会进入该函数。

2. 多步调试

继续(continue)

当程序被停住了，你可以用continue命令恢复程序的运行直到程序结束，或下一个断点到来。
- **c [ignore-count]** —— 恢复程序运行，直到程序结束，或是下一个断点到来。ignore-count表示忽略其后的断点次数。
- **fg [ignore-count]** —— continue，c，fg三个作用一样

直到(until)
- **u** —— 当你厌倦了在一个循环体内单步跟踪时，这个命令可以运行程序直到退出循环体。
- **u location** —— 

- **set step-mode on** —— 打开step-mode模式，于是，在进行单步跟踪时，程序会因为没有debug信息而停住。这个参数有很利于查看机器码。
- **set step-mod off** —— 关闭step-mode模式。默认为off。
- **show step-mode** —— 打印变量值

- **finish** —— 运行程序，直到当前函数完成返回。并打印函数返回时的堆栈地址和返回值及参数值等信息。

#### 堆栈和打印等
- **bt(backtrace)** —— 打印当前的函数调用栈的所有信息，可以看到底层函数调用的过程
- **set var** —— 修改变量的值
- **p(print) var** —— 打印变量值
- **display expr** —— 跟踪查看一个变量，每次停下来都显示它的值【变量/结构体…】
- **undisplay + dnums...** —— 取消对先前设置的变量的跟踪

#### 排查问题三剑客
- **until + linenum** —— 进行指定位置跳转，执行完区间代码
- **finish** —— 在一个函数内部，执行到当前函数返回，然后停下来等待命令
- **c(continue)** —— 从一个断点处，直接运行至下一个断点处


## Valgrind
Valgrind是一个用于内存调试、内存泄漏检测以及性能分析的开源软件开发工具包。每个工具都执行某种调试、分析或类似任务，以帮助程序员改进程序。

- **Memcheck**：内存错误检测器，最为知名和常用。它可以帮助您使程序（尤其是用 C 和 C++ 编写的程序）更加正确。
- **Cachegrind**：缓存和分支预测分析器。它可以帮助您使程序运行得更快。
- **Callgrind**：调用图生成缓存分析器。它与 Cachegrind 有一些重叠，但也收集了一些 Cachegrind 没有的信息。
- **Helgrind**：线程错误检测器。它可以帮助您使多线程程序更加正确。
- **DRD**：线程错误检测器。它类似于 Helgrind，但使用不同的分析技术，因此可能会发现不同的问题。
- **Massif**：堆分析器。它可以帮助您使程序使用更少的内存。
- **DHAT**：堆分析器。它可以帮助您了解区块生命周期、区块利用率和布局效率低下的问题。
- **BBV**：实验性的 SimPoint 基本块向量生成器。它对从事计算机体系结构研究和开发的人来说很有用

还有一些小工具对大多数用户没有用：**Lackey** 是一个示例工具，展示了一些仪器基础知识；**Nulgrind** 是最小的 Valgrind 工具，不进行分析或仪器分析，仅用于测试目的。

### 下载&安装
到[Valgrind官网](https://valgrind.org)下载[最新的版本](https://valgrind.org/downloads/current.html#current)

以下载3.23.0版本为例：
```bash
wget https://sourceware.org/pub/valgrind/valgrind-3.23.0.tar.bz2
tar -jxvf valgrind-3.23.0.tar.bz2
cd valgrind-3.23.0/
./configure
make && sudo make install
```

### 调用方式
Valgrind是非侵入式的，你只需要调用Valgrind运行你的程序即可，而不需要重新链接、重新编译或以其他方式修改要检查的程序。就像这样：
```bash
valgrind [valgrind-options] your-prog [your-prog-options]
```

`valgrind-options`里最重要的的一项是`--tool`(该选项默认为`memcheck`)，它决定你使用valgrind工具包中的哪一个程序做分析。
无论那种工具，Valgrind都是非侵入性的，Valgrind通过读取调试信息获取链接库、执行库，来输出程序运行时状态，并且定位异常代码位置。这体现在，待调试程序会在Valgrind的控制下运行，Valgrind内核会将待调试程序的代码交给所选工具（比如memcheck），相应工具会添加一些自己的代码然后运行，将运行结果返回给Valgrind内核。

Valgrind能模拟待调试程序执行的每条指令，所以，所选工具不仅检查或概要分析应用程序中的代码，而且还检查所有支持的动态链接库（包括C库，GUI库等）中的代码。

### 编译选项
1. 重新编译你的程序，使其附带调试信息（开启-g选项）。如果没有调试信息，Valgrind无法定位异常代码位置。
2. 如果待调试的是C++程序，考虑去掉函数内联调用（开启`-fno-inline`），这样可以更简单的查看函数调用堆栈。或者，使用Valgrind选项`–read-inline-info = yes`可以读取内联函数的调试信息，这样，即便程序使用了内联函数也能正确的显示调用堆栈信息。
3. 关闭编译优化（-O）。在`O1`以上的优化级别下，`Memcheck`工具会误报一些未初始化值的错误。
4. 使用`-Wall`编译代码，他能识别Valgrind在较高优化级别上可能会遗漏的部分。


### 异常报告
错误消息如下所示，其中描述了问题 1，即堆块溢出：
```bash
==19182== Invalid write of size 4
==19182==    at 0x804838F: f (example.c:6)
==19182==    by 0x80483AB: main (example.c:11)
==19182==  Address 0x1BA45050 is 0 bytes after a block of size 40 alloc'd
==19182==    at 0x1B8FF5CD: malloc (vg_replace_malloc.c:130)
==19182==    by 0x8048385: f (example.c:5)
==19182==    by 0x80483AB: main (example.c:11)
```

每条错误消息中都包含了很多信息:
- `19182` 是进程 ID，这通常不重要。
- 第一行（“Invalid write...”）告诉您这是哪种类型的错误。在这里，由于堆块溢出，程序写入了它不应该具有的一些内存。
- 在第一行下方是堆栈跟踪，告诉您问题发生的位置。`at`为发生位置，`by`是上一级函数。如果堆栈跟踪不够大，请使用该 `--num-callers` 选项使其更大。
- 代码地址（例如0x804838F）通常不重要，但偶尔对于追踪更奇怪的错误至关重要。
- 一些错误消息还有第二个组件，用于描述所涉及的内存地址。这个显示写入的内存刚好超过在 `example.c` 的第 5 行使用 `malloc()` 分配的块的末尾。

以上消息表明该程序对地址0x804838F进行了非法的4字节读取。该异常发生在example.c的第6行，被同一文件的第11行调用，依此类推。

按照错误报告的顺序修复错误是值得的，因为以后的错误可能是由早期的错误引起的。

内存泄漏消息如下所示
```bash
==19182== 40 bytes in 1 blocks are definitely lost in loss record 1 of 1
==19182==    at 0x1B8FF5CD: malloc (vg_replace_malloc.c:130)
==19182==    by 0x8048385: f (a.c:5)
==19182==    by 0x80483AB: main (a.c:11)
```

堆栈跟踪告诉您泄漏的内存分配到何处。不幸的是，Memcheck无法告诉您为什么内存泄漏。（忽略“vg_replace_malloc.c”，这是一个实现细节）

有几种泄漏;两个最重要的类别是：
- **肯定丢失了(definitely lost)**：您的程序正在泄漏内存 -- 修复它！
- **可能丢失(probably lost)**：你的程序正在泄漏内存，除非你正在用指针做有趣的事情（比如移动它们以指向堆块的中间）。

Memcheck 还报告未初始化值的使用，最常见的是消息“条件跳转或移动取决于未初始化的值”。确定这些错误的根本原因可能很困难。尝试使用 获取 `--track-origins=yes` 额外信息。这使得 Memcheck 运行得更慢，但您获得的额外信息通常可以节省大量时间来弄清楚未初始化值的来源。

### 核心命令行选项
如上所述，Valgrind的核心接受一组通用选项。每种工具还接受特定的选项。

Valgrind的默认设置在大多数情况下都能成功地提供合理的行为。我们按粗略类别将可用选项分组。
#### 工具选择选项
最重要的一个选项。运行名为 toolname 的 Valgrind 工具，例如 memcheck、cachegrind、callgrind、helgrind、drd、massif、dhat、lackey、none、exp-bbv 等。
```bash
--tool=<toolname> [default: memcheck]
```

#### 基本选项
这些选项适用于所有工具。比较常用的是`--log-file=<filename>`指定保存到的文件。
```bash
#帮助
-h --help
#与 --help 相同，但也列出了通常仅对 Valgrind 的开发人员有用的调试选项。
--help-debug
#显示 Valgrind 核心的版本号
--version
#静默运行，仅打印错误消息。
-q, --quiet
#提供有关程序各个方面的额外信息。重复该选项会增加详细级别。
-v, --verbose

#启用后，Valgrind 将跟踪通过系统调用启动的 exec 子进程。这对于多进程程序是必需的。
--trace-children=<yes|no> [default: no]
#跟踪子进程时，跳过一些子进程。参数为子可执行文件的名称
--trace-children-skip=patt1,patt2,...
#与 --trace-children-skip的区别是检查子进程的参数
--trace-children-skip-by-arg=patt1,patt2,...
#启用后，Valgrind 将不会显示由 fork 调用生成的子进程的任何调试或日志记录输出。
--child-silent-after-fork=<yes|no> [default: no]

# yes 或 full 提供“gdbserver”功能，允许外部 GNU GDB 调试器在 Valgrind 上运行时控制和调试
--vgdb=<no|yes|full> [default: yes]
#启用gdbserver 时使用
--vgdb-error=<number> [default: 999999999]
--vgdb-stop-at=<set> [default: none]
--track-fds=<yes|no|all> [default: no]

#启用后，每条消息前面都会指示自启动以来经过的时间，以天、小时、分钟、秒和毫秒表示。
--time-stamp=<yes|no> [default: no]

#指定 Valgrind 应将其所有消息发送到指定的文件描述符
--log-fd=<number> [default: 2, stderr]
#指定 Valgrind 应将其所有消息发送到指定文件
--log-file=<filename>
#指定 Valgrind 应将其所有消息发送到指定 IP 地址的指定端口
--log-socket=<ip-address:port-number>

#启用后，如果环境变量中存在以空格分隔的服务器 URL，则 Valgrind 将尝试从 debuginfod 服务器下载丢失的 $DEBUGINFOD_URLS debuginfo
--enable-debuginfod=<no|yes> [default: yes]
```

#### 错误相关选项
所有可以报告错误的工具都使用这些选项，例如 Memcheck，但不能报告 Cachegrind。比较常用的是`--xml=<yes|no>`及其相关的部分。
```bash
#启用后，输出的重要部分（例如工具错误消息）将采用XML格式，而不是纯文本格式。
--xml=<yes|no> [default: no]
#指定 Valgrind 应将其 XML 输出发送到指定的文件描述符
--xml-fd=<number> [default: -1, disabled]
#指定 Valgrind 应将其 XML 输出发送到指定文件
--xml-file=<filename>
#指定 Valgrind 应将其 XML 输出发送到指定 IP 地址的指定端口
--xml-socket=<ip-address:port-number>
#在 XML 输出的开头嵌入额外的用户注释字符串
--xml-user-comment=<string>

#启用/禁用 C++ 名称的自动去角（解码）
--demangle=<yes|no> [default: yes]
#指定在标识程序位置的堆栈跟踪中显示的最大条目数
--num-callers=<number> [default: 12]
#堆栈扫描支持仅在 ARM 目标上可用
--unw-stack-scan-thresh=<number> [default: 0] , --unw-stack-scan-frames=<number> [default: 5]

#启用后，Valgrind 会在总共看到 10,000,000 个错误或 1,000 个不同的错误后停止报告错误
--error-limit=<yes|no> [default: yes]
#指定如果 Valgrind 在运行中报告任何错误，则要返回的备用退出代码
--error-exitcode=<number> [default: 0]
#如果启用此选项，Valgrind 将在出现第一个错误时退出
--exit-on-first-error=<yes|no> [default: no]
#当错误以纯文本（i.e. XML）形式输出时，指示在每个错误之前（之后）输出包含 begin（end） 字符串的行。
--error-markers=<begin>,<end> [default: none]
#如果此选项为“是”，则对于报告错误的工具，valgrind 将显示检测到的错误列表和退出时使用的抑制列表
--show-error-list=no|yes|all [default: no]
#指定 -s 等同于 --show-error-list=yes
-s
#启用/禁用非法指令诊断的打印
--sigill-diagnostics=<yes|no> [default: yes]
#启用后，保留（“存档”）符号和所有其他已卸载代码的 debuginfo
--keep-debuginfo=<yes|no> [default: no]
#如果启用此选项，则将显示所有堆栈跟踪条目，并且 main 不会对类似函数进行规范化
--show-below-main=<yes|no> [default: no]
#如果启用此选项，将显示每个源文件的路径
--fullpath-after=<string> [default: do not show source paths]
#提供绝对路径作为 Valgrind 搜索调试对象的额外最终位置
--extra-debuginfo-path=<path> [default: undefined and unused]
#版本 3.9.0 中引入的一项新的实验性功能
--debuginfo-server=ipaddr:port [default: undefined and unused]
#使用此功能时要小心：它会禁用所有一致性检查，当 main 和 debuginfo 对象不匹配时可能会导致 Valgrind 崩溃
--allow-mismatched-debuginfo=no|yes [no]
#指定一个额外的文件，从中读取要禁止显示的错误说明。最多可以使用 100 个额外的抑制文件。
--suppressions=<filename> [default: $PREFIX/lib/valgrind/default.supp]
# Valgrind 将打印 suppression行
--gen-suppressions=<yes|no|all> [default: no]
# Valgrind 将在发生每个错误时停止，并读取您的键盘输入
--input-fd=<number> [default: 0, stdin]
#此选项仅在 macOS 上运行 Valgrind 时才相关
--dsymutil=no|yes [yes]
#堆栈帧的最大大小
--max-stackframe=<number> [default: 2000000]
#指定主线程堆栈的大小
--main-stacksize=<number> [default: use current 'ulimit' value]
#最大线程数
--max-threads=<number> [default: 500]
#如果启用，则 realloc 将解除分配内存并返回 NULL。如果不启用, realloc 不会释放内存，并且将当一个字节大小处理。
--realloc-zero-bytes-frees=yes|no [default: yes for glibc no otherwise]
#指定 Valgrind 应在指定文件中生成 xtree 内存报告
--xtree-memory-file=<filename> [default: xtmemory.kcg.%p]
```
#### 设置默认选项
还有一些选项可用于调试 Valgrind 本身。您不需要在正常运行中使用它们。如果要查看列表，请使用该 `--help-debug `选项。

如果您希望调试程序而不是调试 Valgrind 本身，则应使用选项 `--vgdb=yes` 或 `--vgdb=full`。

#### 设置默认选项
请注意，Valgrind 还从三个位置读取选项：
1. 文件 `~/.valgrindrc`
2. 环境变量 `$VALGRIND_OPTS`
3. 文件 `./.valgrindrc`

这些选项在命令行选项之前按给定顺序进行处理。稍后处理的选项将覆盖之前处理的选项;例如，中的 `./.valgrindrc` 选项将优先于 中的 `~/.valgrindrc` 选项。

请注意，如果该文件不是常规文件，或者被标记为全局可写文件，或者不属于当前用户，则该 `./.valgrindrc` 文件将被忽略。这是因为这些 `./.valgrindrc` 选项可能包含可能有害的选项，或者本地攻击者可以使用这些选项在您的用户帐户下执行代码。

放入 `$VALGRIND_OPTS` 的任何特定于工具的选项或 `./.valgrindrc` 文件都应以工具名称和冒号为前缀。例如，如果您希望 Memcheck 始终进行泄漏检查，则可以在 `~/.valgrindrc` ：

```bash
--memcheck:leak-check=yes
```

如果运行了 Memcheck 以外的任何工具，则将忽略此功能。如果您选择其他不支持 `--leak-check=yes` 的工具，这将可能出现异常。


### 三步使用 Valgrind gdbserver 和 GDB 调试程序
在 Valgrind 下运行的程序不是由 CPU 直接执行的。相反，它运行在 Valgrind 提供的合成 CPU 上。这就是为什么调试器在 Valgrind 上运行程序时无法本机调试程序的原因。

最简单的入门方法是使用标志 `--vgdb-error=0` 运行 Valgrind 。然后按照屏幕上的说明进行操作，这些说明为您提供了启动 GDB 并将其连接到程序所需的精确命令。

1. 如果要在使用 Memcheck 工具时使用 GDB 调试程序，请像这样启动 Valgrind：
```bash
valgrind --vgdb=yes --vgdb-error=0 prog
```

2. 在另一个 shell 中，启动 GDB：
```bash
gdb prog
```

3. 然后向 GDB 发出以下命令：
```bash
valgrind --vgdb=yes --vgdb-error=0 prog
```

您现在可以调试程序，例如通过插入断点，然后使用 GDB continue 命令。

此快速入门信息足以基本使用 Valgrind gdbserver。 Valgrind 和 GDB 组合的更高级功能请参见[官方文档](https://valgrind.org/docs/manual/manual-core-adv.html#manual-core-adv.gdbserver-concept)。

> 可以省略命令行标志 `--vgdb=yes` ，因为这是默认值。

### Memcheck使用
Memcheck 是一个内存错误检测器。它可以检测 C 和 C++ 程序中常见的以下问题。

- 内存非法访问，例如溢出和欠载堆块，溢出堆栈的顶部，以及在释放内存后访问内存。
- 使用未定义的变量，即尚未初始化的值，或从其他未定义的值派生的值。
- 错误的释放堆内存，例如双重释放堆块，或者 malloc / new / new[] 与 free / delete /delete[] 使用不匹配。
- 在memcpy及其相关函数中，源指针和目标指针重叠。
- 向内存分配函数的 size 参数传递了一个可疑的（很可能是负数）值。
- 使用size为0的realloc函数。
- 使用不是2的幂的对齐值
- 内存泄漏。

像这样的问题可能很难通过其他方式发现，通常长时间未被发现，然后导致偶尔的、难以诊断的崩溃。

#### memcheck的异常信息

##### 1. 非法读取/非法写入错误(Illegal read / Illegal write errors)
```bash
Invalid read of size 4
   at 0x40F6BBCC: (within /usr/lib/libpng.so.2.1.0.9)
   by 0x40F6B804: (within /usr/lib/libpng.so.2.1.0.9)
   by 0x40B07FF4: read_png_image(QImageIO *) (kernel/qpngio.cpp:326)
   by 0x40AC751B: QImageIO::read() (kernel/qimage.cpp:3621)
 Address 0xBFFFF0E0 is not stack'd, malloc'd or free'd
```
当您的程序在Memcheck认为不应该读取或写入内存的地方时，就会发生这种情况。在此示例中，程序在地址 0xBFFFF0E0 执行了 4 字节读取，该地址位于系统提供的库 libpng.so.2.1.0.9 中的某个位置，该地址是从同一库中的其他位置调用的，从 的 qpngio.cpp 第 326 行调用，依此类推。

请注意，Memcheck 仅告诉您您的程序即将访问非法地址的内存。它无法阻止访问的发生。因此，如果你的程序进行了通常会导致分段错误的访问，你的程序仍然会遭受同样的命运 -- 但你会在此之前收到来自Memcheck的消息。在这个特定示例中，读取堆栈上的垃圾是非致命的，并且程序保持活动状态。

##### 2. 使用未初始化的值(Use of uninitialised values)
```bash
Conditional jump or move depends on uninitialised value(s)
   at 0x402DFA94: _IO_vfprintf (_itoa.h:49)
   by 0x402E8476: _IO_printf (printf.c:36)
   by 0x8048472: main (tests/manuel1.c:8)
```

当程序使用尚未初始化的值（换句话说，未定义）时，将报告未初始化的值使用错误。在这里，未定义的值在 C 库 printf 的机器内部的某个地方使用。
```c
int main()
{
  int x;
  printf ("x = %d\n", x);
}
```

你的程序可以随意复制（未初始化的）数据。当Memcheck 观察到这一点并跟踪这些数据，但不会报错。只有当你的程序试图以可能影响程序外部可见行为的方式使用未初始化的数据时，Memcheck 才会报错。在这个例子中，变量 x 是未初始化的。Memcheck 观察到值被传递给 `_IO_printf`，然后传递给 `_IO_vfprintf`，但不会报错。然而，`_IO_vfprintf` 需要检查 x 的值，以便将其转换为相应的 ASCII 字符串，因为这个原因 Memcheck 会发出错误信息。

未初始化数据的来源往往是：
- 过程中尚未初始化的局部变量
- 堆块的内容（用 `malloc` 、 `new` 或类似函数分配）在你（或构造函数）在那里写东西之前。

##### 3. 在系统调用中使用未初始化或不可寻址的值(Use of uninitialised or unaddressable values in system calls)
```bash
Invalid read of size 4
   at 0x40F6BBCC: (within /usr/lib/libpng.so.2.1.0.9)
   by 0x40F6B804: (within /usr/lib/libpng.so.2.1.0.9)
   by 0x40B07FF4: read_png_image(QImageIO *) (kernel/qpngio.cpp:326)
   by 0x40AC751B: QImageIO::read() (kernel/qimage.cpp:3621)
 Address 0xBFFFF0E0 is not stack'd, malloc'd or free'd
```

当您的程序在Memcheck认为不应该读取或写入内存的地方时，就会发生这种情况。在此示例中，程序在地址 0xBFFFF0E0 执行了 4 字节读取，该地址位于系统提供的库 `libpng.so.2.1.0.9` 中的某个位置，该地址是从同一库中的其他位置调用的，从 的 `qpngio.cpp` 第 326 行调用，依此类推。

请注意，Memcheck 仅告诉您您的程序即将访问非法地址的内存。它无法阻止访问的发生。因此，如果你的程序进行了通常会导致分段错误的访问，你的程序仍然会遭受同样的命运 -- 但你会在此之前收到来自Memcheck的消息。在这个特定示例中，读取堆栈上的垃圾是非致命的，并且程序保持活动状态。

##### 3. 使用未初始化的值(Use of uninitialised values)
Memcheck 检查系统调用的所有参数：
- 它检查所有直接参数本身，无论它们是否被初始化。
- 此外，如果系统调用需要从程序提供的缓冲区读取，Memcheck 会检查整个缓冲区是否可寻址，以及其内容是否已初始化。
- 此外，如果系统调用需要写入用户提供的缓冲区，Memcheck 会检查缓冲区是否可寻址。

在系统调用后，Memcheck 会更新其跟踪的信息，以精确反映由系统调用引起的内存状态的任何更改。

下面是两个具有无效参数的系统调用的示例：
```c
#include <stdlib.h>
#include <unistd.h>
int main( void )
{
char* arr  = malloc(10);
int*  arr2 = malloc(sizeof(int));
write( 1 /* stdout */, arr, 10 );
exit(arr2[0]);
}
```

报错信息：
```bash
Syscall param write(buf) points to uninitialised byte(s)
    at 0x25A48723: __write_nocancel (in /lib/tls/libc-2.3.3.so)
    by 0x259AFAD3: __libc_start_main (in /lib/tls/libc-2.3.3.so)
    by 0x8048348: (within /auto/homes/njn25/grind/head4/a.out)
Address 0x25AB8028 is 0 bytes inside a block of size 10 alloc'd
    at 0x259852B0: malloc (vg_replace_malloc.c:130)
    by 0x80483F1: main (a.c:5)

Syscall param exit(error_code) contains uninitialised byte(s)
    at 0x25A21B44: __GI__exit (in /lib/tls/libc-2.3.3.so)
    by 0x8048426: main (a.c:8)
```

因为程序（a）从堆区块写入了未初始化的垃圾数据到标准输出，并且（b）向exit函数传递了一个未初始化的值。注意，第一个错误指的是由buf指向的内存（而不是buf本身），但第二个错误直接指向exit函数的参数arr2[0]。

##### 4. 非法释放(Illegal frees)
```bash
Invalid free()
   at 0x4004FFDF: free (vg_clientmalloc.c:577)
   by 0x80484C7: main (tests/doublefree.c:10)
 Address 0x3807F7B4 is 0 bytes inside a block of size 177 free'd
   at 0x4004FFDF: free (vg_clientmalloc.c:577)
   by 0x80484C7: main (tests/doublefree.c:10)
```

Memcheck 跟踪你的程序使用 `malloc`/`new` 分配的区块，因此它可以确切地知道传递给 `free`/`delete` 的参数是否合法。在这里，这个测试程序两次释放了同一个区块。与非法读写错误一样，Memcheck 试图理解释放的地址。如果地址像这里一样之前已经被释放，你会被告知这一点——这使得发现对同一个区块进行重复释放变得容易。如果你尝试释放一个不指向堆区块起始位置的指针，你也会遇到这个信息。

##### 5. 当使用不适当的释放函数释放堆块时
```bash
Mismatched free() / delete / delete []
   at 0x40043249: free (vg_clientfuncs.c:171)
   by 0x4102BB4E: QGArray::~QGArray(void) (tools/qgarray.cpp:149)
   by 0x4C261C41: PptDoc::~PptDoc(void) (include/qmemarray.h:60)
   by 0x4C261F0E: PptXml::~PptXml(void) (pptxml.cc:44)
 Address 0x4BB292A8 is 0 bytes inside a block of size 64 alloc'd
   at 0x4004318C: operator new[](unsigned int) (vg_clientfuncs.c:152)
   by 0x4C21BC15: KLaola::readSBStream(int) const (klaola.cc:314)
   by 0x4C21C155: KLaola::stream(KLaola::OLENode const *) (klaola.cc:416)
   by 0x4C21788F: OLEFilter::convert(QCString const &) (olefilter.cc:272)
```

在 C++ 以与内存分配方式兼容的方式解除分配内存非常重要。

在 C++ 中，大多数时候你会编写使用 new 表达式和 delete 表达式。一个 new 表达式会调用 operator new 来执行分配，然后调用对象的构造函数（如果存在）。类似地，delete 表达式会调用对象的析构函数（如果存在），然后调用 operator delete。数组重载会为数组中的每个对象调用构造函数/析构函数。

- 如果使用 `malloc` 、 `calloc` 、 `realloc` 或 `valloc memalign` 分配，则必须使用 `free` 解除分配。
- 如果使用 `new` 分配，则必须使用 `delete` 解除分配 。
- 如果使用 `new[] `分配，则必须使用 `delete[]` 解除分配 。

##### 6. 重叠的源块和目标块(Overlapping source and destination blocks)
以下 C 库函数将一些数据从一个内存块复制到另一个内存块（或类似操作）：`memcpy`、`strcpy`、`strncpy`、`strcat`、`strncat`。由它们的 src（源）和 dst（目标）指针指向的区块不允许重叠。POSIX 标准有类似的措辞：“如果复制发生在重叠的对象之间，行为是未定义的。”因此，Memcheck 会检查这一点。
```bash
==27492== Source and destination overlap in memcpy(0xbffff294, 0xbffff280, 21)
==27492==    at 0x40026CDC: memcpy (mc_replace_strmem.c:71)
==27492==    by 0x804865A: main (overlap.c:40)
```

您不希望这两个块重叠，因为其中一个块可能会被复制部分覆盖。

你可能会认为，在 dst 小于 src 的情况下，Memcheck 没必要报告这个问题。例如，实现 memcpy 的明显方式是从第一个字节复制到最后一个字节。然而，一些架构的优化指南建议从最后一个字节复制到第一个字节。此外，一些 memcpy 的实现在复制之前将 dst 置零，因为将目标的缓存行清零可以提高性能。

> 注意：如果你想写出真正可移植的代码，不要对语言实现做出任何假设。
{: .prompt-info }

##### 7. 可疑参数值(Fishy argument values)
所有内存分配函数都接受一个参数，指定应分配的内存块的大小。显然，请求的大小应该是一个非负值，通常不会过大。例如，在64位机器上，分配请求的大小不可能超过 \(2^{63}\) 字节。更有可能的是，这样一个值是错误的大小计算的结果，实际上是一个负值（由于位模式被解释为无符号整数，所以看起来非常大）。这样的值被称为“可疑值”。以下分配函数的大小参数会检查是否可疑：`malloc`、`calloc`、`realloc`、`memalign`、`posix_memalign`、`aligned_alloc`、`new`、`new[]`、`__builtin_new`、`__builtin_vec_new`。对于 `calloc`，两个参数都会被检查。
```bash
==32233== Argument 'size' of function malloc has a fishy (possibly negative) value: -3
==32233==    at 0x4C2CFA7: malloc (vg_replace_malloc.c:298)
==32233==    by 0x400555: foo (fishy.c:15)
==32233==    by 0x400583: main (fishy.c:23)
```

在早期的 Valgrind 版本中，这些值被称为“愚蠢的参数(silly arguments)”，并且不包括回溯跟踪。

##### 8. Realloc size 为 0
长期以来，对 realloc 的（误）用，即用它来做 free 的工作，一直被误解。在 C17 标准 ISO/IEC 9899:2017 中，当 realloc 的 size 参数为 0 时的行为被指定为实现定义。Memcheck 警告 realloc 的非移植性使用。
```bash
==77609== realloc() with size 0
==77609==    at 0x48502B8: realloc (vg_replace_malloc.c:1450)
==77609==    by 0x201989: main (realloczero.c:8)
==77609==  Address 0x5464040 is 0 bytes inside a block of size 4 alloc'd
==77609==    at 0x484CBB4: malloc (vg_replace_malloc.c:397)
==77609==    by 0x201978: main (realloczero.c:7)
```

##### 9. 对齐错误(Alignment Errors)
C 和 C++ 具有多个允许用户获取对齐内存的功能。通常，这样做是出于性能原因，以便内存的缓存行或内存页对齐。C 具有和 `posix_memalign` `aligned_alloc` 的函数 `memalign` 。C++ 有许多 operator new 和 operator delete 的重载。其中，`posix_memalign`非常明确地指定，但其他的在实现之间差异很大。Valgrind 将为在任何平台上无效的对齐值生成错误。
- `memalign` 如果对齐方式为零或不是 2 的倍数，则会产生错误。
- `posix_memalign` 如果对齐方式小于 sizeof（size_t）、不是 2 的倍数或大小为零，则会产生错误。
- `aligned_alloc` 如果对齐不是 2 的倍数，如果大小为零或大小不是对齐的整数倍，则会产生错误。
- `aligned new` 如果对齐方式为零或不是 2 的倍数，则会产生错误。 nothrow 重载将返回一个 NULL 指针。非 nothrow 重载将中止 Valgrind。
- `aligned delete` 如果对齐方式为零或不是 2 的倍数，或者对齐方式与 使用的 aligned new 对齐方式不同，则会产生错误。
- `sized delete` 如果大小与 使用的 new 大小不同，则会产生错误。
- `sized aligned delete` 组合单个大小和对齐的删除运算符的错误条件。

```bash
==65825== Invalid alignment value: 3 (should be power of 2)
==65825==    at 0x485197E: memalign (vg_replace_malloc.c:1740)
==65825==    by 0x201CD2: main (memalign.c:39)
```

##### 10. 内存泄漏检测(Memory leak detection)
Memcheck 跟踪响应对 `malloc` / `new` 等的调用而发出的所有堆块。因此，当程序退出时，它知道哪些块尚未释放。如果正确设置了`–leak-check`，则对于每个剩余的块，Memcheck会根据根集中的指针确定该块是否可访问。根集包括所有线程的通用寄存器，以及可访问的客户端内存（包括堆栈）中初始化、对齐、指针大小的数据字。

有两种方法可以到达块。第一种是“起始指针”，即指向块起点的指针。第二种是带有“内部指针”，即指向块中间的指针。我们知道有几种方式可以发生内部指针：
- 指针最初可能是一个起始指针，并且由程序有意（或无意）移动。特别是，如果您的程序使用标记指针，即如果它使用指针的底部一位、两位或三位（由于对齐而通常始终为零），则可能会发生这种情况，以便存储额外的信息。
- 它可能是内存中的随机垃圾值，完全无关，只是巧合。
- 它可能是指向 C++ 的内部 char 数组的指针 `std::string`。例如，某些编译器在 `std::string` 的开头添加 3 个单词，以在包含字符数组的内存之前存储长度、容量和引用计数。它们在这 3 个单词之后返回一个指针，指向 char 数组。
- 某些代码可能会分配一个内存块，并使用前 8 个字节（块大小 - 8）作为 64 位数字存储。 `sqlite3MemMalloc` 这样做。
- 它可能是指向分配了 `new[]` 的 C++ 对象（具有析构函数）数组的指针。在这种情况下，一些编译器在分配块的开头存储一个包含数组长度的“魔术 cookie”，并返回一个指向刚好经过该魔术 cookie 的指针，即内部指针。
- 它可能是指向使用多重继承的 C++ 对象内部的指针。

在Memcheck官方手册上，列出了以下九种内存泄露的情况：
```bash
     指针链                    AAA 泄漏案例     BBB 泄漏案例
     -------------            -------------   -------------
(1)  RRR ------------> BBB                    DR
(2)  RRR ---> AAA ---> BBB    DR              IR
(3)  RRR               BBB                    DL
(4)  RRR      AAA ---> BBB    DL              IL
(5)  RRR ------?-----> BBB                    (y)DR, (n)DL
(6)  RRR ---> AAA -?-> BBB    DR              (y)IR, (n)DL
(7)  RRR -?-> AAA ---> BBB    (y)DR, (n)DL    (y)IR, (n)IL
(8)  RRR -?-> AAA -?-> BBB    (y)DR, (n)DL    (y,y)IR, (n,y)IL, (_,n)DL
(9)  RRR      AAA -?-> BBB    DL              (y)IL, (n)DL

指针链图例:
- RRR: 根集节点或 DR 块
- AAA, BBB: 堆块
- --->: 起始指针
- -?->: 内部指针

泄漏案例图例:
- DR: 可直接访问(Directly reachable)
- IR: 间接可达(Indirectly reachable)
- DL: 直接丢失(Directly lost)
- IL: 间接丢失(Indirectly lost)
- (y)XY: 如果内部指针是真实指针，则为 XY
- (n)XY: 如果内部指针不是真实指针，则为 XY
- (_)XY: 无论哪种情况都是 XY
```

Memcheck在其输出中合并了其中一些情况，从而导致以下四种泄漏类型：
- **Still reachable**：内存仍然可访问。对于 BBB 块来说包括上述的情况1和2。意味着BBB内存块仍然是可访问的，有可能在未来某个时刻被释放。这是非常普遍的，可以说不是问题。因此，默认情况下，Memcheck 不会单独报告此类块。
- **Definitely lost**：肯定的内存泄漏。对于 BBB 块来说包括上述情况3。在程序退出时，BBB无法再被访问，确认泄漏。这可能是在程序的早期某个时间点丢失指针的症状。这种情况应该由程序员修复。
- **Indirectly lost**：间接的内存泄漏。对于 BBB 块来说包括上述情况4和9。由于AAA无法被访问，AAA和BBB都无法被正常释放。
- **Possibly lost**：可能的内存泄漏。对于 BBB 块来说包括上述情况5-8。这意味着已找到指向块的一个或多个指针的链，但至少有一个指针是内部指针。这可能只是内存中的一个随机值，恰好指向一个块，所以你不应该认为这是可以的，除非你知道你有内部指针。


在存在内存异常的情况下，memcheck会给输出一个汇总的信息，格式如下：
```bash
LEAK SUMMARY:
   definitely lost: 48 bytes in 3 blocks.
   indirectly lost: 32 bytes in 2 blocks.
     possibly lost: 96 bytes in 6 blocks.
   still reachable: 64 bytes in 4 blocks.
        suppressed: 0 bytes in 0 blocks.
```

如果指定了`–leak-check = full`，则Memcheck将提供每个绝对丢失或可能丢失的块的详细信息，包括分配位置。（实际上，它会将泄漏类型相同且堆栈跟踪充分相似的所有块的结果合并到单个“丢失记录”中。`–leak-resolution`使您可以控制“充分相似”的含义。）它无法告诉你何时，如何或为什么丢失指向泄漏块的指针；您必须自己解决。通常，你应该尝试确保你的程序在退出时没有任何绝对丢失或可能丢失的块。
```bash
8 bytes in 1 blocks are definitely lost in loss record 1 of 14
   at 0x........: malloc (vg_replace_malloc.c:...)
   by 0x........: mk (leak-tree.c:11)
   by 0x........: main (leak-tree.c:39)

88 (8 direct, 80 indirect) bytes in 1 blocks are definitely lost in loss record 13 of 14
   at 0x........: malloc (vg_replace_malloc.c:...)
   by 0x........: mk (leak-tree.c:11)
   by 0x........: main (leak-tree.c:25)
```

第一条消息描述了一个确定丢失的单个 8 字节块的简单情况。第二种情况提到了另一个肯定丢失的 8 字节块;不同之处在于，由于这个丢失的块，其他块中的另外 80 个字节会间接丢失。损失记录不按任何显着顺序显示，因此损失记录数字不是特别有意义。丢失记录编号可用于 Valgrind gdbserver 中，以列出泄漏块的地址和/或提供有关块如何仍可访问的更多详细信息。

该选项 `--show-leak-kinds=<set>` 控制要显示指定时间 `--leak-check=full` 的泄漏类型集。

泄漏类型集通过以下方式之一指定：
- 一个或多个以逗号分隔的 `definite、indirect、possible、reachable` 列表。
- `all` 指定完整的集合（所有泄漏类型）。
- `none` 对于空集。

要显示的泄漏类型的默认值为 `--show-leak-kinds=definite,possible` 。

要显示除了绝对丢失和可能丢失的块之外，还可以显示 `--show-leak-kinds=all` 可访问和间接丢失的块，您可以使用 .要仅显示可访问和间接丢失的块，请使用 `--show-leak-kinds=indirect`,reachable .然后，将呈现可访问和间接丢失的块，如以下两个示例所示。

首先，只有在 `--leak-check=full` 指定的情况下，泄漏才算作真正的“错误”。然后，该选项 `--errors-for-leak-kinds=<set>` 控制要视为错误的泄漏类型集。默认值为 `--errors-for-leak-kinds=definite,possible`。

#### Memcheck 命令行选项
其中比较常用的是`--leak-check=full`，显示详细信息。
```bash
#启用后，在客户端程序完成时搜索内存泄漏。如果设置为 summary ，则表示发生了多少次泄漏。如果设置为 full 或 yes ，则每个单独的泄漏都将详细显示和/或计为错误
--leak-check=<no|summary|yes|full> [default: summary]

#在进行泄漏检查时，确定 Memcheck 是否愿意将不同的回溯视为相同的回溯，以便将多个泄漏合并到单个泄漏报告中
--leak-resolution=<low|med|high> [default: high]
#内存泄漏的类别，definite、indirect、possible、reachable中的一个或多个
--show-leak-kinds=<set> [default: definite,possible]
#指定在 full 泄漏搜索中计为错误的泄漏类型
--errors-for-leak-kinds=<set> [default: definite,possible]
#指定在泄漏搜索期间要使用的泄漏检查启发式方法集,stdstring length64 newarray multipleinheritance
--leak-check-heuristics=<set> [default: all]
#指定要显示的泄漏类型的替代方法
--show-reachable=<yes|no> , --show-possibly-lost=<yes|no>
#如果设置为 yes，则在退出时完成的泄漏搜索结果将输出到“Callgrind 格式”执行树文件中
--xtree-leak=<no|yes> [no]
#指定 Valgrind 应在指定文件中生成 xtree 泄漏报告
--xtree-leak-file=<filename> [default: xtleak.kcg.%p]
#控制 Memcheck 是否报告未定义值错误的使用情况
--undef-value-errors=<yes|no> [default: yes]
#控制 Memcheck 是否跟踪未初始化值的来源
--track-origins=<yes|no> [default: no]
#控制 Memcheck 如何处理 32 位、64 位、128 位和 256 位自然对齐的负载
--partial-loads-ok=<yes|no> [default: yes]
#控制 Memcheck 在检查某些值的定义性时是否应采用更精确但更耗时的检测
--expensive-definedness-checks=<no|auto|yes> [default: auto]
#控制要为 malloc'd 和/或 free'd 块保留的堆栈跟踪。
--keep-stacktraces=alloc|free|alloc-and-free|alloc-then-free|none [default: alloc-and-free]
#指定队列中块的最大总大小（以字节为单位）
--freelist-vol=<number> [default: 20000000]
#增加了发现“小”块的悬空指针的可能性，即使释放了大块也是如此。
--freelist-big-blocks=<number> [default: 1000000]
#启用后，假设在堆栈指针下方的一小段距离内读取和写入是由于 GCC 2.96 中的错误造成的，并且不会报告它们
--workaround-gcc296-bugs=<yes|no> [default: no]
#指定后，它会导致 Memcheck 不报告堆栈指针下方指定偏移量的访问错误
--ignore-range-below-sp=<number>-<number>
#启用后，Memcheck 会检查是否使用与分配函数匹配的函数解除分配堆块
--show-mismatched-frees=<yes|no> [default: yes]
#启用后，Memcheck 会检查大小为零的 的使用 realloc
--show-realloc-size-zero=<yes|no> [default: yes]
#此选项中列出的任何范围（可以指定多个范围，用逗号分隔）将被 Memcheck 的可寻址性检查忽略
--ignore-ranges=0xPP-0xQQ[,0xRR-0xSS]
#用指定的字节填充由 malloc 、 new 等分配的块，但不填充 由 calloc 分配的块
--malloc-fill=<hexnumber>
#用指定的字节值填充 、 free delete 等释放的块
--free-fill=<hexnumber>
```

****

本文参考

> 1. [GDB用法详解](https://www.cnblogs.com/lvdongjie/p/8994092.html)
> 2. [什么是GDB？GDB如何使用？](https://blog.csdn.net/weixin_45031801/article/details/134399664)
> 3. [Valgrind官方文档](https://valgrind.org/docs/manual/QuickStart.html)
> 4. [使用Valgrind调试Linux C++程序](https://simonzgx.github.io/2020/08/04/Valgrind%E4%BD%BF%E7%94%A8/)