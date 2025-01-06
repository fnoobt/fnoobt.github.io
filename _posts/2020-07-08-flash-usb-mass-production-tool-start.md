---
title: U盘量产工具教程
author: fnoobt
date: 2020-07-08 10:24:00 +0800
categories: [Flash,U盘量产]
tags: [flash,刷机,量产]
---


U盘插到电脑上，U盘图标显示灰色，双击打开提示：`请将磁盘插入驱动器`，无法格式化，在U盘点右键-->属性，显示为容量等为0。

## 解决办法

### 1.获取U盘芯片信息
首先要下载一个U盘芯片检测工具`ChipGenius`，打开后选中U盘，就可以在下方看到U盘的主控制造商、主控型号和主控识别码。
```
　设备描述: [E:]USB 大容量存储设备(NAND USB2DISK)
　设备类型:　　大容量存储设备

　协议版本: USB 2.00
　当前速度: 高速(HighSpeed)
　电力消耗: 100mA

 USB设备ID: VID = FFFF PID = 1201

设备修订版: 0000

产品制造商: NAND
　产品型号: USB2DISK
产品修订版: 0.00

　主控厂商: FirstChip(一芯)
　主控型号: chipYC2019
闪存识别码:　　D87CD8F3766B - 1CE/单通道 [TLC]
```

### 2.下载对应的量产工具

ChipGenius检测出U盘主控是一芯的chipYC2019，`FFC1179 = FC2279 = ChipYC2019` 他们的工具都是共用的，对应的量产工具为FirstChip Mass-Product Tools，在[FirstChip官网](https://www.szfirstchip.com/col.jsp?id=142)下载对应的量产工具版本：[FirstChip_MpTools_20220601](https://14013833.s21d.faiusrd.com/0/ABUIABBQGAAgkuGRlQYojJfJkwM?f=FirstChip_MpTools_20220601.rar&v=1656489317)  

### 3.开始量产

把U盘插到电脑上，然后打开量产工具，这时候就量产工具里就会出现你的U盘信息，没有的话可以点击<kbd>刷新</kbd>，然后我点<kbd>设定</kbd>进行设置
- 量产设定：
  + `1 - 原厂扫描`        
  + `2 - 低级格式化`      物理级的格式化，将磁盘内容重新清空，恢复出厂时的状态（一般选择该项）
  + `3 - 高级格式化`      对磁盘的各个分区进行格式化，在逻辑上划分磁道
  + `4 - 清空+原厂扫描`   
  + `5 - 清空`           清空磁盘内容
- 产品设定： VID = FFFF PID = 1201 (和ChipGenius中一致)
- Bin设定：固定容量 --> 按照bin级固定

> 针对`FFC1179`，如果出现 `unkown flash` ，说明量产工具没有识别到U盘，可以在量产设定 --> Flash设置 --> Flash类型中选择 `89D3AC32C204` 或 `89D3AC32C600`
{: .prompt-tip }

> 如果报错 `LockPort Fail` 不能锁定，就打开量产设置，找到Bin设定，把固定容量的选项反选，然后保存再量产，如果失败了，以后量产之前还是得让其处于没选中状态，就算默认没有选中,也要打开这个选项先勾选然后再取消勾选再保存就可以量产了。
{: .prompt-tip }

> 如果最后提示`量产失败，错误代码=5`，必须设置bin设定 --> 固定容量 --> 按bin级固定
{: .prompt-tip }

量产所需要的时间比较久，可能需要一个小时，大家耐心等待，不要让电脑进入休眠模式，否则量产失败U盘会变砖。

****

本文参考

> 1. [一芯FC1178BC量产教程](https://www.upantool.com/jiaocheng/liangchan/Netac/14565.html)
> 2. [一芯FC1179量产教程](https://blog.csdn.net/qq_38767359/article/details/125414380)
> 3. [一芯FlashID](https://tieba.baidu.com/p/8245215083)
> 4. [一芯主控U盘量产修复](https://www.upantool.com/jiaocheng/liangchan/Netac/13496.html)