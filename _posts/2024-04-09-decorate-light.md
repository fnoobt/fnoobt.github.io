---
title: 灯具
author: fnoobt
date: 2024-04-09 18:54:00 +0800
categories: [Decorate,装修]
tags: [decorate,灯具]
math: true
---

按照[《建筑照明设计标准》(GB/T50034-2024)](https://www.mohurd.gov.cn/gongkai/zc/wjk/art/2024/art_17339_777466.html),照明要求如下：

1. 功率因数：输入功率与额定功率之差

<table>
    <tr>
        <td colspan=2>额定功率（W）</td>
        <td>功率因素限值</td>
    </tr>
    <tr> 
        <td colspan=2>$ \leq 5 $</td>
        <td>0.5</td>
    </tr>
    <tr> 
        <td rowspan=2>>5</td>
        <td>家居用</td>
        <td>0.7</td>
    </tr>
    <tr> 
        <td>非家居用</td>
        <td>0.9</td>
    </tr>
</table>

2. LED灯的初始光效值（lm/W）

|   规格                 | 2700K/3500K | 3500K/4000K/5000K |
|:----------------------:|:------------:|:-----------------:|
|  非定向LED              | 85          | 95                |
|  定向LED（PAR16/PAR20） | 80          | 85                |
|  定向LED（PAR30/PAR38） | 85          | 90                |
|  LED平板灯              | 95          | 105               |

3. 住宅建筑照明标准值

<table>
    <tr>
        <td colspan=2>房间或场所</td>
        <td>参考平面及其高度</td>
        <td>照度标准值(lx)</td>
        <td> $R_{a}$ </td>
    </tr>
    <tr> 
        <td rowspan=2>起居室</td>
        <td>一般活动</td>
        <td rowspan=2>0.75m水平面</td>
        <td>100</td>
        <td rowspan=2>80</td>
    </tr>
    <tr> 
        <td>书写、阅读</td>
        <td>300 (混合照明)</td>
    </tr>
    <tr> 
        <td rowspan=2>卧室</td>
        <td>一般活动</td>
        <td rowspan=2>0.75m水平面</td>
        <td>75</td>
        <td rowspan=2>80</td>
    </tr>
    <tr> 
        <td>床头、阅读</td>
        <td>200 (混合照明)</td>
    </tr>
    <tr> 
        <td colspan=2>餐厅</td>
        <td>0.75m餐桌面</td>
        <td>150</td>
        <td>80</td>
    </tr>
     <tr> 
        <td rowspan=2>厨房</td>
        <td>一般活动</td>
        <td>0.75m水平面</td>
        <td>100</td>
        <td rowspan=2>80</td>
    </tr>
    <tr> 
        <td>操作台</td>
        <td>台面</td>
        <td>300 (混合照明)</td>
    </tr>
     <tr> 
        <td rowspan=2>卫生间</td>
        <td>一般活动</td>
        <td>0.75m水平面</td>
        <td>100</td>
        <td>80</td>
    </tr>
    <tr> 
        <td>化妆台</td>
        <td>台面</td>
        <td>300 (混合照明)</td>
        <td>90</td>
    </tr>
     <tr> 
        <td colspan=2>走廊、楼梯间</td>
        <td>地面</td>
        <td>100</td>
        <td>60</td>
    </tr>
    <tr> 
        <td colspan=2>电梯前厅</td>
        <td>地面</td>
        <td>75</td>
        <td>60</td>
    </tr>
</table>

4. LED灯的功率选择

房间平均照度公式:

$$ E = Φ⋅UF⋅MF/A $$
​
- E: 照度（单位：lx）
- Φ: 灯具的光通量（单位：lm）
- UF: 灯具的利用系数（Unitary Factor，表示光通量利用效率，即灯具发出的光通量中，有效用于指定区域的比例。一般在0.4-0.8之间），北美常称为CU (Coefficient of Utilization)，该值和灯具类型、发光形式（直发光、侧发光）、房间尺寸、安装高度、墙地反射等有关。
- MF: 灯具的维护系数（Maintenance Factor，表示光通量随时间的衰减，用于修正由于灯具老化、灰尘等原因造成的光通量衰减。一般在0.6-0.9之间）。对于家庭室内环境，新装修的取0.9，维护到位（一年两次）比较清洁的取0.8，不怎么维护的一般取0.7，长时间使用的厨房等污染严重的取0.6。
- A: 照射面积（单位：$m^2$）

以卧室为例子，按照度75 lx，利用系数0.5，维护系数0.8，LED平板灯 105 lm/W，功率因数0.7，进行计算，需要的**最低功率**为每平方2.55 W。
以餐厅为例子，按照度150 lx，利用系数0.5，维护系数0.8，LED平板灯 105 lm/W，功率因数0.7，进行计算，需要的**最低功率**为每平方5.1 W。
以客厅为例子，按照度300 lx，利用系数0.5，维护系数0.8，LED平板灯 105 lm/W，功率因数0.7，进行计算，需要的**最低功率**为每平方10.2 W。

综上所述，对于卧室建议每平方3~4 W，同时应该配备台灯、壁灯等补充光源，对于客厅建议每平方不低于10 W (包括主灯、筒灯、射灯、线性灯功率)。

****

本文参考

> 1. [建筑照明设计标准](https://www.mohurd.gov.cn/gongkai/zc/wjk/art/2024/art_17339_777466.html)