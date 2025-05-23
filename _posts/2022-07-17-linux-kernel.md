---
title: Linux内核
author: fnoobt
date: 2022-07-17 21:36:00 +0800
categories: [Linux,kernel]
tags: [linux,kernel]
media_subpath: '/assets/img/commons/linux/kernel/'
---

## 一、前言

### 1.1内核在操作系统中的位置

为了更具象地理解内核，不妨将Linux计算机想象成有三层结构：

1. **硬件**：物理机（这是系统的底层结构或基础）是由内存（RAM）、处理器（或 CPU）以及输入/输出（I/O）设备（例如存储、网络和图形）组成的。其中，CPU 负责执行计算和内存的读写操作。
2. **Linux 内核**：操作系统的核心。（没错，内核正处于核心的位置）它是驻留在内存中的软件，用于告诉 CPU 要执行哪些操作。
3. **用户进程**：这些是内核所管理的运行程序。用户进程共同构成了用户空间。用户进程有时也简称为进程。内核还允许这些进程和服务器彼此进行通信（称为进程间通信或 IPC）。

系统执行的代码在CPU上以以下两种模式之一运行:内核模式或用户模式。运行在内核态的代码可以不受限制地访问硬件，而用户态会限制SCI对CPU和内存的访问。内存也有类似的分离(内核空间和用户空间)。这两个小细节构成了一些复杂操作的基础，比如安全保护，构建容器和虚拟机的权限分离。  
这也意味着，如果进程在用户模式下失败，损失是有限且无害的，并且可以由内核修复。另一方面，由于内核进程要访问内存和处理器，内核进程的崩溃可能会导致整个系统的崩溃。因为用户进程之间会有适当的保护措施和权限要求，所以一个进程的崩溃通常不会造成太多问题。  
此外，由于Linux内核在实时补丁期间可以连续工作，因此在应用补丁进行安全修复时不会出现宕机。

### 1.2Linux 内核的作用是什么？

内容有以下四项作用：
1. **内存管理**：追踪记录有多少内存存储了什么以及存储在哪里
2. **进程管理**：确定哪些进程可以使用中央处理器（CPU）、何时使用以及持续多长时间
3. **设备驱动程序**：充当硬件与进程之间的调解程序/解释程序
4. **系统调用和安全防护**：从流程接受服务请求

正确实现时，内核对用户是不可见的，它在自己的小世界(称为内核空间)中工作，从中分配内存，跟踪所有内容的存储位置。用户看到的东西(比如Web浏览器和文件)叫做用户空间。这些应用程序通过系统调用接口(SCI)与内核交互。  
可以这样理解:内核就像一个忙碌的私人助理，为高管(硬件)服务。助理的工作是将员工和公众(用户)的信息和请求(流程)传递给高管，记住存储的内容和位置(内存)，并确定谁可以在任何给定的时间访问高管，以及会议时间有多长。

### 1.3学习Linux内核准备工作：

1. 熟悉C语言，这个是最基本的
2. 了解编译连接过程，如果写过ld、lcf类的链接文件最好，这样就能理解类似percpu变量的实现方法
3. 学过或者自学过计算机组成原理或者微机原理，知道smp、cpu、cache、ram、hdd、bus的概念，明白中断、dma、寄存器，这样才能理解所谓的上下文context、barrier是什么

#### Linux内核的特点：结合了unix操作系统的一些基础概念

![Relationship](relationship.jpg)

#### Linux内核的任务：

1. 从技术层面讲，内核是硬件与软件之间的一个中间层。作用是将应用层序的请求传递给硬件，并充当底层驱动程序，对系统中的各种设备和组件进行寻址。
2. 从应用程序的层面讲，应用程序与硬件没有联系，只与内核有联系，内核是应用程序知道的层次中的最底层。在实际工作中内核抽象了相关细节。
3. 内核是一个资源管理程序。负责将可用的共享资源(CPU时间、磁盘空间、网络连接等)分配得到各个系统进程。
4. 内核就像一个库，提供了一组面向系统的命令。系统调用对于应用程序来说，就像调用普通函数一样。

#### 内核实现策略：

1. 微内核。最基本的功能由中央内核（微内核）实现。所有其他的功能都委托给一些独立进程，这些进程通过明确定义的通信接口与中心内核通信。
2. 宏内核。内核的所有代码，包括子系统（如内存管理、文件管理、设备驱动程序）都打包到一个文件中。内核中的每一个函数都可以访问到内核中所有其他部分。目前支持模块的动态装卸(裁剪)。Linux内核就是基于这个策略实现的。

#### 哪些地方用到了内核机制？

1. 进程（在cpu的虚拟内存中分配地址空间，各个进程的地址空间完全独立;同时执行的进程数最多不超过cpu数目）之间进行通信，需要使用特定的内核机制。
2. 进程间切换(同时执行的进程数最多不超过cpu数目)，也需要用到内核机制。
进程切换也需要像FreeRTOS任务切换一样保存状态，并将进程置于闲置状态/恢复状态。
3. 进程的调度。确认哪个进程运行多长的时间。

#### Linux进程

1. 采用层次结构，每个进程都依赖于一个父进程。内核启动init程序作为第一个进程。该进程负责进一步的系统初始化操作。init进程是进程树的根，所有的进程都直接或者间接起源于该进程。
2. 通过`pstree`命令查询。实际上得系统第一个进程是`systemd`，而不是init（这也是疑问点）
3. 系统中每一个进程都有一个唯一标识符(ID),用户（或其他进程）可以使用ID来访问进程。

#### Linux内核源代码的目录结构

Linux内核源代码包括三个主要部分：

1. 内核核心代码，包括第3章所描述的各个子系统和子模块，以及其它的支撑子系统，例如电源管理、Linux初始化等
2. 其它非核心代码，例如库文件（因为Linux内核是一个自包含的内核，即内核不依赖其它的任何软件，自己就可以编译通过）、固件集合、KVM（虚拟机技术）等
3. 编译脚本、配置文件、帮助文档、版权说明等辅助性文件使用ls命令看到的内核源代码的顶层目录结构，具体描述如下。include/ ---- 内核头文件，需要提供给外部模块（例如用户空间代码）使用。

```markdown
kernel/ ---- Linux内核的核心代码，包含了3.2小节所描述的进程调度子系统，以及和进程调度相关的模块。
mm/ ---- 内存管理子系统（3.3小节）。
fs/ ---- VFS子系统（3.4小节）。
net/ ---- 不包括网络设备驱动的网络子系统（3.5小节）。
ipc/ ---- IPC（进程间通信）子系统。
arch// ---- 体系结构相关的代码，例如arm, x86等等。 
arch//mach- ---- 具体的machine/board相关的代码。 
arch//include/asm ---- 体系结构相关的头文件。 
arch//boot/dts ---- 设备树（Device Tree）文件。
init/ ---- Linux系统启动初始化相关的代码。 
block/ ---- 提供块设备的层次。 
sound/ ---- 音频相关的驱动及子系统，可以看作“音频子系统”。 
drivers/ ---- 设备驱动（在Linux kernel 3.10中，设备驱动占了49.4的代码量）。
lib/ ---- 实现需要在内核中使用的库函数，例如CRC、FIFO、list、MD5等。 
crypto/ ----- 加密、解密相关的库函数。 
security/ ---- 提供安全特性（SELinux）。 
virt/ ---- 提供虚拟机技术（KVM等）的支持。 
usr/ ---- 用于生成initramfs的代码。 
firmware/ ---- 保存用于驱动第三方设备的固件。
samples/ ---- 一些示例代码。 
tools/ ---- 一些常用工具，如性能剖析、自测试等。
Kconfig, Kbuild, Makefile, scripts/ ---- 用于内核编译的配置文件、脚本等。
COPYING ---- 版权声明。 
MAINTAINERS ----维护者名单。 
CREDITS ---- Linux主要的贡献者名单。 
REPORTING-BUGS ---- Bug上报的指南。
Documentation, README ---- 帮助、说明文档。
```

## 二、为什么要学习 Linux 内核

大部分程序员可能永远没有机会开发Linux内核或者驱动Linux，那么我们为什么还需要学习Linux内核呢？Linux的源代码和架构都是开放的，我们可以学到很多操作系统的概念和实现原理。Linux的设计哲学体系继承了UNIX，现在整个设计体系相当稳定和简化，这是大部分服务器使用Linux的重要原因。

**那学习Linux内核的原因就在于此。**  

进一步了解内核的原理，有助于你更好地使用命令和程序设计，让你的面试和开发更上一层楼。但是不建议直接看源代码，因为Linux代码太大，容易丢失。
而最好的办法是，先了解一下Linux内核机制，知道基本的原理与流程。
不过，Linux内核机制也非常复杂，而且其中互相关联。  
比如说，进程运行要分配内存，内存映射涉及文件的关联，文件的读写需要经过块设备，从文件中加载代码才能运行起来进程。这些知识点要反复对照，才能理清。
 
**但是一旦攻克！你会发现Linux这个复杂的系统开始透明起来。**

****

本文参考

> 1. [Linux内核](https://zhuanlan.zhihu.com/p/635315467)