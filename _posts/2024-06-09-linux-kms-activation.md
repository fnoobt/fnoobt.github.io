---
title: KMS 激活 Windows/Office
author: fnoobt
date: 2024-06-09 12:24:00 +0800
categories: [Linux,Linux教程]
tags: [linux,kms]
---

## 什么是KMS？跟其他激活方式有什么不同？
KMS是密钥管理服务（Key Management Server），是自从Windows Vista后微软开始使用的一种大型组织中的批量激活技术。在没有这个技术的XP时代，大批量的激活都使用VLK（Volume License Key）激活，因此只要把一些泄露的VLK密钥（比如某某PC制造商、某某银行、某某单位的密钥）抄下来，你就可以随便安装其他电脑上。因此在XP的时代，盗版就没有激活的问题，一个镜像一个序列号走天下，Ghost XP也是满大街。同时期的Office 2007也是静态的序列号，都不需要联网验证随便装。 为了杜绝激活毫无存在感这种尴尬的情况，从Windows Vista开始，常见的激活方式分成了以下几种：

| 方式            | 适用系统/软件                                | 使用条件                             |
|-----------------|------------------------------------|------------------------------------|
| 零售Key         | 对应的零售产品（Retail）         | 你能买到的DVD/数字版的正版产品，或者其他代理商渠道的零售Key，<br>需要联网验证，或者找客服电话激活                  |
| SLIC OEM        | 制造商在BIOS中添加SLIC信息，<br>适用于win7等较旧的系统，<br>当然也有Server在用的。<br>比如Server 2022依然有SLIC激活 | BIOS中有完整SLIC表（有些电脑出厂带了个空表，<br>没啥用还妨碍用KMS）、有对应OEM厂商的证书和OEM产品密钥（这个是不变的）     |
| 数字权利        | 仅适用于部分有OEM SLP密钥的Windows10+系统，<br>不能用于Server        | 使用正版Windows7等产品升级或者其他永久激活的渠道<br>（比如商店买的、随机附带的、用了有效的密钥等）联网同步硬件信息后，<br>使用对应的数字权利key即可在同一台电脑上重装联网激活（比如经典的3V66T）       |
| KMS            | 对应的批量授权产品（Volume），或者可以被转换证书的产品        | 对于Win10 1703+之前的系统，需要安装专业版或者企业版（Server对应批量版），并且BIOS中不含SLIC信息（或者企业版）。 <br> Win10 1703+的系统使用对应的GVLK即可，只要系统中证书不缺或者补上证书，会自动转换产品。<br> Office可以通过转换安装证书来使用，但建议安装批量版本。<br> 最少180天内要成功连接一次KMS服务器，否则激活失效。     |
| 批量多次密钥MAK | 对应的批量授权产品（Volume），<br>与KMS适用范围一样      | 与KMS一样用于批量激活，但是永久激活      |

虽然激活方式很多花样，但只要是使用正规渠道激活的，都是正版产品。因此“KMS”并不是很多小白理解的“盗版”、“病毒”，而是微软提供一种批量激活方式。那为什么会有这个印象呢？因为针对上面的几种激活方式，都有各自的盗版利用方式，而KMS成了较为通用的一种。

- 首先，对于**零售key**，这个没什么好的利用方式，一般是开发者测试账户订阅倒卖的key或者去电脑城偷拍了未激活笔记本的key。
- 对于批量多次密钥MAK，这个也没什么好说的，网上经常有泄露出来的MAK密钥，可用激活次数比较多但也不是无限，运气好的话可能蹲到一个能用，次数用完了可用电话渠道激活尝试，属于是可遇不可求的方式。可以用工具`PidKeyTool`来检测密钥有效性。
- 对于**SLIC**，修改BIOS加入SLIC表，或者修改引导程序加载SLIC到内存中（所以可以看到有些Win7电脑开机有个Grub4dos一闪而过），用对应的证书和key即可激活（这个不难找，可以直接搜索SLIC证书合集），但对于office没啥用，并且现在win10也用不了，另外修改BIOS有风险，在UEFI引导中加载SLIC也稍微复杂一点（可以用Clover/OpenCore等程序）。SLIC也有不同版本，比如最初的vista是2.0，win7是用的比较多的2.1，win8开始就用不了，但Server还有在用，比如SLIC2.6激活Server2022，SLIC版本向下兼容。一句话简单来说SLIC适用范围有限，盗版需要修改BIOS或者引导，操作复杂。
- **数字权利**是在Win10刚出来的时候，给Win7升级免费升级到win10衍生出来的产品，正常来说你需要有一个正版的win7或者win8等系统，然后升级，程序会收集你电脑的硬件信息和密钥，生成一个“门票”提交给微软服务器，以后联网微软就可以判断硬件信息是否对的上就能激活。当然你使用零售的key或者MAK等永久激活的方式都可以获取数字权利，要是你的key来源不怎么干净，这就相当于偷渡洗白了，下次重装还能用。office没有数字权利，不过有效的key可以绑定到自己的微软账户，类似于steam兑换码。

那这个数字权利怎么被盗版方便地利用呢？在这个免费升级的过程中，有个`gatherosstate.exe`，他检查当前系统已经有效永久激活的话，他就收集硬件信息和密钥等生成一个`GenuineTicket.xml`门票，发送给微软服务器，微软认可这个“门票”之后我们就可以使用数字权利了。而关键在于这个`gatherosstate.exe`，通过魔改的`gatherosstate.exe`，可以忽略一切条件不管你有没有激活、在什么系统下，直接生成门票，然后使用`clipup`安装门票即可。理论上微软可以轻松筛选出这些使用不正常方法生成的“门票”，但微软没有这么做，毕竟收入最大来源并不是普通用户，况且Windows现在也不怎么赚钱了。这个方法的缺点也很明显，需要联网提交信息给微软，需要运行魔改版的`gatherosstate`，因此还需要关闭自带的Windows Defender杀毒，需要一个个手动操作，当然能一劳永逸也是个选择。常见工具有`HWIDGEN`等。
- 最后就是**KMS**激活了。KMS需要一台Windows服务器上安装批量激活服务，添加对应的KMS KEY，达到一定条件后客户端就可以通过自动发现或者手动输入KMS服务器地址进行激活。关键就在于这个服务器，经过国外大神们的不断研究，开发了不少版本的KMS服务模拟器，相当于游戏的私服，很多激活工具集成了KMS模拟器，在本机通过虚拟网卡的形式弄了个虚拟局域网给本机激活，又或者通过注册表劫持服务的形式让所有激活请求都劫持到本地。很多小白就是用的这种激活工具，因为他通用，简单，但又容易被坏坏的人往里面加木马病毒广告。基于这些工具要么运行在后台要么要劫持服务，这里很不推荐使用任何KMS工具。当然你自己搭建一个KMS模拟器或者使用别人的公网模拟器的话，就比较干净一点，只要你是手动输入命令行应用KMS的话，想使用其他激活方式直接更换密钥即可，无任何影响，后文再细说。使用KMS激活会定期向服务器请求激活信息，如果超过最大180天的宽限期联系不上服务器，激活就会失效。平时看到那么多的“未激活的Windows”估计是工具激活之后把后台都杀掉了。Enterprise G（G是Gov的G，神州网信的特供版）版本使用KMS可以激活宽限400年（但会自动应用一些组策略）。近年还有一种“KMS38”的激活方式，他并不是属于微软的正常渠道的激活方式，原理是利用了上面的数字权利程序，正常来说KMS最大宽限期只有180天，但gatherosstate.exe生成门票的时候可以继承剩余的KMS日期，bug就来了，我们可以欺骗他把日期修改到2038年，所以他叫做KMS38。这个过程甚至不需要发送门票给微软，完全离线。至于为什么是2038年？因为1970年是计算机元年，在有符号32位整数时间戳里面，到2038年会出现回归现象，所以长时间为2038年。

## 常见Windows/Office 版本密钥

> 注意
> - 无法激活任何评估版本的产品
> - GLVK仅适用于批量通道产品以及可以转换的版本的产品
> - OEM用于数字权利，适用于Windows 1607+（14393）的产品
{: .prompt-info }

### GVLK KEY

#### Windows 11

| 操作系统版本                                        | KMS 客户端产品密钥            |
|:----------------------------------------------------|:------------------------------|
| Windows 11 专业版 Windows 10 专业版                 | W269N-WFGWX-YVC9B-4J6C9-T83GX |
| Windows 11 专业版 N Windows 10 专业版 N             | MH37W-N47XK-V7XM9-C7227-GCQG9 |
| Windows 11 专业工作站版 Windows 10 专业工作站版     | NRG8B-VKK3Q-CXVCJ-9G2XF-6Q84J |
| Windows 11 专业工作站版 N Windows 10 专业工作站版 N | 9FNHH-K3HBT-3W4TD-6383H-6XYWF |
| Windows 11 专业教育版 Windows 10 专业教育版         | 6TP4R-GNPTD-KYYHQ-7B7DP-J447Y |
| Windows 11 专业教育版 N Windows 10 专业教育版 N     | YVWGF-BXNMC-HTQYQ-CPQ99-66QFC |
| Windows 11 教育版 Windows 10 教育版                 | NW6C2-QMPVW-D7KKK-3GKT6-VCFB2 |
| Windows 11 教育版 N Windows 10 教育版 N             | 2WH4N-8QGBV-H22JP-CT43Q-MDWWJ |
| Windows 11 企业版 Windows 10 企业版                 | NPPR9-FWDCX-D2C8J-H872K-2YT43 |
| Windows 11 企业版 N Windows 10 企业版 N             | DPH2V-TTNVB-4X9Q3-TJR4H-KHJW4 |
| Windows 11 企业版 G Windows 10 企业版 G             | YYVX9-NTFWV-6MDM3-9PT4T-4M68B |
| Windows 11 企业版 G N Windows 10 企业版 G N         | 44RPN-FTY23-9VTTB-MP9BX-T84FV |

#### Windows 10

| 操作系统版本                                        | KMS 客户端产品密钥            |
|:----------------------------------------------------|:------------------------------|
| Windows 10 企业版 LTSC 2021 Windows 10 企业版 LTSC 2019     | M7XTQ-FN8P6-TTKYV-9D4CC-J462D |
| Windows 10 企业版 N LTSC 2021 Windows 10 企业版 N LTSC 2019 | 92NFX-8DJQP-P6BBQ-THF9C-7CG2H |
| Windows 10 企业版 LTSB 2016                                 | DCPHK-NFMTC-H88MJ-PFHPY-QJ4BJ |
| Windows 10 企业版 N LTSB 2016                               | QFFDN-GRT3P-VKWWX-X7T3R-8B639 |
| Windows 10 企业版 2015 LTSB                                 | WNMTR-4C88C-JK8YV-HQ7T2-76DF9 |
| Windows 10 企业版 2015 LTSB N                               | 2F77B-TNFGY-69QQF-B8YKP-D69TJ |

#### Windows 7

| 操作系统版本                                        | KMS 客户端产品密钥            |
|:----------------------------------------------------|:------------------------------|
| Windows 7 专业版                                     | FJ82H-XT6CR-J8D7P-XQJJ2-GPDD4 |
| Windows 7 专业版 N                                   | MRPKT-YTG23-K7D7T-X2JMM-QY7MG |
| Windows 7 专业版 E                                   | W82YF-2Q76Y-63HXB-FGJG9-GF7QX |
| Windows 7 企业版                                     | 33PXH-7Y6KF-2VJC9-XBBR8-HVTHH |
| Windows 7 企业版 N                                   | YDRBP-3D83W-TY26F-D46B2-XCKRJ |
| Windows 7 企业版 E                                   | C29WB-22CC8-VJ326-GHFJW-H9DH4 |

#### Office 2021

| Office产品                         | KMS 客户端产品密钥            |
|:-----------------------------------|:-----------------------------|
| Office LTSC Professional Plus 2021 | FXYTK-NJJ8C-GB6DW-3DYQT-6F7TH |
| Office LTSC Standard 2021          | KDX7X-BNVR8-TXXGX-4Q7Y8-78VT3 |
| Project Professional 2021          | FTNWT-C6WBT-8HMGF-K9PRX-QV9H8 |
| Project Standard 2021              | J2JDC-NJCYY-9RGQ4-YXWMH-T3D4T |
| Visio LTSC Professional 2021       | KNH8D-FGHT4-T8RK3-CTDYJ-K2HT4 |
| Visio LTSC Standard 2021           | MJVNY-BYWPY-CWV6J-2RKRT-4M8QG |
| Access LTSC 2021                   | WM8YG-YNGDD-4JHDC-PG3F4-FC4T4 |
| Excel LTSC 2021                    | NWG3X-87C9K-TC7YY-BC2G7-G6RVC |
| Outlook LTSC 2021                  | C9FM6-3N72F-HFJXB-TM3V9-T86R9 |
| PowerPoint LTSC 2021               | TY7XF-NFRBR-KJ44C-G83KF-GX27K |
| Publisher LTSC 2021                | 2MW9D-N4BXM-9VBPG-Q7W6M-KFBGQ |
| Skype for Business LTSC 2021       | HWCXN-K3WBT-WJBKY-R8BD9-XK29P |
| Word LTSC 2021                     | TN8H9-M34D3-Y64V9-TR72V-X79KV |

#### Office 2019

| Office产品                    | KMS 客户端产品密钥            |
|:------------------------------|:-----------------------------|
| Office Professional Plus 2019 | NMMKJ-6RK4F-KMJVX-8D9MJ-6MWKP |
| Office Standard 2019          | 6NWWJ-YQWMR-QKGCB-6TMB3-9D9HK |
| Project Professional 2019     | B4NPR-3FKK7-T2MBV-FRQ4W-PKD2B |
| Project Standard 2019         | C4F7P-NCP8C-6CQPT-MQHV9-JXD2M |
| Visio Professional 2019       | 9BGNQ-K37YR-RQHF2-38RQ3-7VCBB |
| Visio Standard 2019           | 7TQNQ-K3YQQ-3PFH7-CCPPM-X4VQ2 |
| Access 2019                   | 9N9PT-27V4Y-VJ2PD-YXFMF-YTFQT |
| Excel 2019                    | TMJWT-YYNMB-3BKTF-644FC-RVXBD |
| Outlook 2019                  | 7HD7K-N4PVK-BHBCQ-YWQRW-XW4VK |
| PowerPoint 2019               | RRNCX-C64HY-W2MM7-MCH9G-TJHMQ |
| Publisher 2019                | G2KWX-3NW6P-PY93R-JXK2T-C9Y9V |
| Skype for Business 2019       | NCJ33-JHBBY-HTK98-MYCV8-HMKHJ |
| Word 2019                     | PBX3G-NWMT6-Q7XBW-PYJGG-WXD33 |

#### Office 2016

| Office产品                    | KMS 客户端产品密钥            |
|:------------------------------|:------------------------------|
| Office Professional Plus 2016 | XQNVK-8JYDB-WJ9W3-YJ8YR-WFG99 |
| Office Standard 2016          | JNRGM-WHDWX-FJJG3-K47QV-DRTFM |
| Project Professional 2016     | YG9NW-3K39V-2T3HJ-93F3Q-G83KT |
| Project Standard 2016         | GNFHQ-F6YQM-KQDGJ-327XX-KQBVC |
| Visio Professional 2016       | PD3PC-RHNGV-FXJ29-8JK7D-RJRJK |
| Visio Standard 2016           | 7WHWN-4T7MP-G96JF-G33KR-W8GF4 |
| Access 2016                   | GNH9Y-D2J4T-FJHGG-QRVH7-QPFDW |
| Excel 2016                    | 9C2PK-NWTVB-JMPW8-BFT28-7FTBF |
| OneNote 2016                  | DR92N-9HTF2-97XKM-XW2WJ-XW3J6 |
| Outlook 2016                  | R69KK-NTPKF-7M3Q4-QYBHW-6MT9B |
| PowerPoint 2016               | J7MQP-HNJ4Y-WJ7YM-PFYGF-BY6C6 |
| Publisher 2016                | F47MM-N3XJP-TQXJ9-BP99D-8837K |
| Skype for Business 2016       | 869NQ-FJ69K-466HW-QYCP2-DDBV6 |
| Word 2016                     | WXY84-JN2Q9-RBCCQ-3Q3J3-3PFJ6 |

#### Office 2013

| Office产品                    | KMS 客户端产品密钥            |
|:------------------------------|:------------------------------|
| Office 2013 Professional Plus | YC7DK-G2NP3-2QQC3-J6H88-GVGXT |
| Office 2013 Standard          | KBKQT-2NMXY-JJWGP-M62JB-92CD4 |
| Project 2013 Professional     | FN8TT-7WMH6-2D4X9-M337T-2342K |
| Project 2013 Standard         | 6NTH3-CW976-3G3Y2-JK3TX-8QHTT |
| Visio 2013 Professional       | C2FG9-N6J68-H8BTJ-BW3QX-RM3B3 |
| Visio 2013 Standard           | J484Y-4NKBF-W2HMG-DBMJC-PGWR7 |
| Access 2013                   | NG2JY-H4JBT-HQXYP-78QH9-4JM2D |
| Excel 2013                    | VGPNG-Y7HQW-9RHP7-TKPV3-BG7GB |
| InfoPath 2013                 | DKT8B-N7VXH-D963P-Q4PHY-F8894 |
| Lync 2013                     | 2MG3G-3BNTT-3MFW9-KDQW3-TCK7R |
| OneNote 2013                  | TGN6P-8MMBC-37P2F-XHXXK-P34VW |
| Outlook 2013                  | QPN8Q-BJBTJ-334K3-93TGY-2PMBT |
| PowerPoint 2013               | 4NT99-8RJFH-Q2VDH-KYG2C-4RD4F |
| Publisher 2013                | PN2WF-29XG2-T9HJ7-JQPJR-FCXK4 |
| Word 2013                     | 6Q7VD-NX8JD-WJ2VH-88V73-4GBJ7 |

### 官网GVLK连接
1. [Windows/Server](https://learn.microsoft.com/zh-cn/windows-server/get-started/kms-client-activation-keys?tabs=server2025%2Cwindows1110ltsc%2Cversion1803%2Cwindows81#generic-volume-license-keys-gvlk)
2. [office2010](https://learn.microsoft.com/zh-cn/previous-versions/office/office-2010/ee624355(v=office.14)?redirectedfrom=MSDN)
3. [office2016/2019/2021/2024](https://learn.microsoft.com/zh-cn/office/volume-license-activation/gvlks?redirectedfrom=MSDN)

## Windows激活

> 以下操作需要使用管理员权限的命令行CMD
{: .prompt-info }

1. 首先需要查询当前激活信息/系统版本（确保是支持KMS的版本）：
```bash
slmgr /dlv
```

2. 安装对应GVLK密钥（如果是从VLSC下载VL版本已经内置，不需要安装）
```bash
slmgr /ipk xxxxx-xxxxx-xxxxx-xxxxx-xxxxx
```

> 注意：VL版本的镜像文件名是`SW_DVD`开头。MSDN下的是测试版本不是VL版本。
{: .prompt-info }

3. 设置KMS服务器地址
```bash
slmgr /skms kms.example.com
```

4. 手动执行激活请求(KMS服务正常的话，手动点立即激活一样效果，不点过一段时间也会自己请求)
```bash
slmgr /ato
```

5. 查询过期时间
```bash
slmgr /xpr
```

更多命令可以输入`slmgr`后回车查看。

## Office
> 以下操作需要使用管理员权限的命令行CMD，并且使用VL版本的Office
{: .prompt-info }

[安装最新VL版office快捷指南](https://blog.03k.org/post/dowload-vloffice.html)

首先找到你`的OSPP.VBS`所在的目录，一般是office安装所在目录`Program Files\Microsoft Office\OfficeXX`{: .filepath}，找不到的建议安装一个Everything或者使用windows自带的文件搜索框搜索一下，把`OSPP.VBS`拖到运行对话框即可快速获取文件路径以便复制。

设置KMS服务器地址
```bash
cscript ospp.vbs /sethst:kms.example.com
```

手动执行激活请求(KMS服务正常的话，打开任意office组件也会自动请求激活)
```bash
cscript ospp.vbs /act
```

如果提示看到successful的字样，那么就是激活成功了。更多命令可以输入`cscript ospp.vbs`后回车查看。

## 版本转换
批量版本的商业镜像文件是SW_开头的ISO。对于Win10 1703+或者对应Server，你可以直接更换对应的GVLK KEY来更改系统的版本。但很多版本在发行的时候其实内置的证书不全，更改KEY会提示非核心版本，比如一些出厂安装了家庭版的笔记本，缺了个远程桌面功能，想改成专业版或者企业版，是不是只能重装呢？其实可以添加补全证书即可转换。这个方法也适用于转换其他奇奇怪怪的版本，比如EnterpriseG。

1. 找到一台目标版本的电脑或者镜像，把`\Windows\System32\spp\tokens\skus`{: .filepath}文件夹复制出来。
2. 把上面的文件夹覆盖到要转换的系统的相同位置，补全许可证。
3. 执行命令安装许可证：`slmgr /rilc`
4. 执行命令安装对应版本的GVLK：`slmgr /ipk xxxxx-xxxxx-xxxxx-xxxxx-xxxxx`，转换完成。

Office也可以通过添加许可证来“转换”版本，但不推荐这么干，因为重装Office比重装系统简单，一来你可能在Office里面看到多个许可证一个激活一个没激活非常强迫症，二来这样干有可能会出现“你的许可证不是正版，你可能是盗版软件的受害者？”的弹窗（虽然不影响使用，果然office比windows赚钱啊）。因此，建议直接安装使用VL版本的office。

****

本文参考

> 1. [KMS激活Windows/Office口袋指南](https://blog.03k.org/post/kms.html)
> 2. [GVLK/OEM KEY速查](https://blog.03k.org/post/gvlk.html)
> 3. [安装最新VL版office快捷指南](https://blog.03k.org/post/dowload-vloffice.html)