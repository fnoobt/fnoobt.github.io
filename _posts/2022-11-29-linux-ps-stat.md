---
title: Linux进程查看中STAT的含义(进程的状态)
author: fnoobt
date: 2022-11-29 17:31:00 +0800
categories: [Linux,Linux教程]
tags: [linux,stat]
---

- D ：不可中断 `Uninterruptible sleep (ususally IO)` 收到信号不唤醒和不可运行, 进程必须等待直到有中断发生
- R ：运行 `Runnable` 正在运行的或在运行队列中等待的
- S ：睡眠 `Sleeping` 休眠中, 受阻, 在等待某个条件的形成或接受到信号
- T ：终止 `Terminate` 进程收到SIGSTOP, SIGSTP, SIGTIN, SIGTOU信号后停止运行运行
- W ：无驻留页 `has no resident pages` 没有足够的记忆体分页可分配(自2.6.xx 内核起无效)
- X ：死掉的进程(一般不会看到，杀死后就没有了)
- Z ：僵死 `Zombie` 进程已终止, 但进程描述符存在, 直到父进程调用wait4()系统调用后释放

在BSD模式下显示STAT项的话，还可能出现以下的几种情况：
- < ：高优先级(对其它用户不利)
- N ：低优先级 (对其它用户有利)
- L ：页面锁定到内存(实时的且与客户进行IO的)
- s ：进程的领导者，在它之下有子进程
- l ：多线程的进程（使用 CLONE_THREAD，就像 NPTL pthreads 那样）
- \+ ：前端进程组内的进程

****

本文参考

> 1. [lclc](https://www.cnblogs.com/lcword/p/13541931.html)