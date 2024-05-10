---
title: Wireshark Lua 开发
author: fnoobt
date: 2024-05-10 19:29:00 +0800
categories: [Network,网络工具]
tags: [network,wireshark,lua]
---

Wireshark是非常强大的报文解析工具，是网络定位中不可缺的使用工具，在物联网中很多为自定义协议，wireshark无法解析，此时lua脚本就有了用武之地。Lua是一个脚本语言，不需要编译可以直接调用，完美解决了自定义报文解析。

## 1. Lua教程
Lua 是一种轻量小巧的脚本语言，用标准C语言编写并以源代码形式开放，其设计目的是为了嵌入应用程序中，从而为应用程序提供灵活的扩展和定制功能。

### 关键词
以下列出了 Lua 的保留关键词。保留关键字不能作为常量或变量或其他用户自定义标示符：

| 流程控制 |  循环  | 逻辑运算 |   其他   |
|:--------:|:------:|:--------:|:--------:|
|    if    |  while |    and   |   local  |
|  elseif  |   do   |    or    | function |
|   then   | repeat |    not   |    in    |
|   else   |  until |          |   goto   |
|  return  |   for  |          |    nil   |
|    end   |  break |          |   TRUE   |
|          |        |          |   FALSE  |

一般约定，以下划线开头连接一串大写字母的名字（比如 _VERSION）被保留用于 Lua 内部全局变量。

在默认情况下，变量总是认为是全局的。

如果你想删除一个全局变量，只需要将变量赋值为`nil`。

### 注释
```lua
-- 两个减号是单行注释

--[[
    多行注释
    多行注释
--]] 
```

### 数据类型

| 数据类型 |     描述    |
|:--------:|:-------------------:|
|    nil   |  这个最简单，只有值nil属于该类，表示一个无效值（在条件表达式中相当于false）。   |
|  boolean |  包含两个值：false和true。     |
|  number  |  表示双精度类型的实浮点数                  |
|  string  |  字符串由一对双引号或单引号来表示      |
| function |  由 C 或 Lua 编写的函数               |
| userdata |  表示任意存储在变量中的C数据结构   |
|  thread  |  表示执行的独立线路，用于执行协同程序           |
|   table  | Lua 中的表（table）其实是一个"关联数组"（associative arrays），数组的索引可以是数字、字符串或表类型。在 Lua 里，table 的创建是通过"构造表达式"来完成，最简单构造表达式是{}，用来创建一个空表。 |

> Lua 把 false 和 nil 看作是 false，其他的都为 true，数字 0 也是 true。
{: .prompt-info }

### 循环

|    循环类型    |  描述                       |
|:-------------:|:-------------------:|
|   while 循环   | 在条件为 true 时，让程序重复地执行某些语句。执行语句前会先检查条件是否为 true。 |
|    for 循环    | 重复执行指定语句，重复次数可在 for 语句中控制。                 |
| repeat...until | 重复执行循环，直到 指定的条件为真时为止                     |

```lua
-- while 循环
while(condition)
do
   statements
end

--[[ 
    for 循环
    var 从 exp1 变化到 exp2，每次变化以 exp3 为步长递增 var，并执行一次 "执行体"。
    exp3 是可选的，如果不指定，默认为1。
--]]
for var=exp1,exp2,exp3 do  
    <执行体>  
end

-- repeat 循环
repeat
   statements
until( condition )
```

### 流程控制

|      语句      |    描述                                       |
|:--------------:|:-------------------------------:|
|     if 语句    |    if 语句 由一个布尔表达式作为条件判断，其后紧跟其他语句组成。     |
| if...else 语句 |    if 语句 可以与 else 语句搭配使用, 在 if 条件表达式为 false 时执行 else 语句代码。 |
|   if 嵌套语句  |     你可以在if 或 else if中使用一个或多个 if 或 else if 语句 。     |

```lua
-- if 语句
if(布尔表达式)
then
   --[ 在布尔表达式为 true 时执行的语句 --]
end

-- if...else 语句
if(布尔表达式)
then
   --[ 布尔表达式为 true 时执行该语句块 --]
else
   --[ 布尔表达式为 false 时执行该语句块 --]
end

-- if...elseif...else 语句
if( 布尔表达式 1)
then
   --[ 在布尔表达式 1 为 true 时执行该语句块 --]
elseif( 布尔表达式 2)
then
   --[ 在布尔表达式 2 为 true 时执行该语句块 --]
elseif( 布尔表达式 3)
then
   --[ 在布尔表达式 3 为 true 时执行该语句块 --]
else 
   --[ 如果以上布尔表达式都不为 true 则执行该语句块 --]
end
```

### 运算符

算术运算符,设定 A 的值为10，B 的值为 20

| 操作符 |         描述         |        实例        |
|:------:|:--------------------:|:------------------:|
|    +   |         加法         |  A + B 输出结果 30 |
|    -   |         减法         | A - B 输出结果 -10 |
|    *   |         乘法         | A * B 输出结果 200 |
|    /   |         除法         |  B / A 输出结果 2  |
|    %   |         取余         |  B % A 输出结果 0  |
|    ^   |         乘幂         |  A^2 输出结果 100  |
|    -   |         负号         |   -A 输出结果 -10  |
|   //   | 整除运算符(>=lua5.3) |   5//2 输出结果 2  |

关系运算符,设定 A 的值为10，B 的值为 20

| 操作符 |                                描述                     |          实例         |
|:------:|:------------------------------------------------------:|:---------------------:|
|   ==   |  等于，检测两个值是否相等，相等返回 true，否则返回 false   |  (A == B) 为 false。  |
|   ~=   |  不等于，检测两个值是否相等，不相等返回 true，否则返回 false |   (A ~= B) 为 true。  |
|    >   |  大于，如果左边的值大于右边的值，返回 true，否则返回 false     |   (A > B) 为 false。  |
|    <   |  小于，如果左边的值大于右边的值，返回 false，否则返回 true     |   (A < B) 为 true。   |
|   >=   |  大于等于，如果左边的值大于等于右边的值，返回 true，否则返回 false| (A >= B) 返回false。 |
|   <=   |  小于等于， 如果左边的值小于等于右边的值，返回 true，否则返回 false |  (A <= B) 返回 true。 |

逻辑运算符,设定 A 的值为 true，B 的值为 false

| 操作符 |                                 描述                 |          实例          |
|:------:|:---------------------------------------------------:|:----------------------:|
|   and  |   逻辑与操作符。 若 A 为 false，则返回 A，否则返回 B。 |  (A and B) 为 false。  |
|   or   |   逻辑或操作符。 若 A 为 true，则返回 A，否则返回 B。  |   (A or B) 为 true。   |
|   not  | 逻辑非操作符。与逻辑运算结果相反，如果条件为 true，逻辑非为 false。 | not(A and B) 为 true。 |

其他运算符
| 操作符 |         描述           |         实例                        |
|:------:|:-----------------------:|:----------------------------------:|
|   ..   |   连接两个字符串  | a..b，其中a为 "Hello "，b 为"World", 输出结果为"Hello World"。|
|    #   | 一元运算符，返回字符串或表的长度。 |      #"Hello" 返回 5           |

运算符优先级（从高到低的顺序）
```
^
not    - (unary)
*      /       %
+      -
..
<      >      <=     >=     ~=     ==
and
or
```

除了 ^ 和 .. 外所有的二元运算符都是左连接的。

## 2. Wireshark 的常用 Lua API

### 2.1 新协议和解析器的功能
用户可以通过 Lua 脚本为 Wireshark 创建新协议。`Proto` 协议对象可以具有 `Pref` 首选项、可以在详细视图树中显示的可过滤值的 `ProtoField` 字段、用于解析新协议的函数等。

解析函数可以通过 `DissectorTable` 连接到现有的协议表中，以便新的协议解析器函数被该协议调用，并且新的解析器本身可以通过检索和调用 `Dissector` 对象。 `Proto` 解析器也可以用作后解析器，在每帧解析的末尾，或用作启发式解析器。

#### 2.1.1 解析器 Dissector
对解析器的引用，用于针对数据包或其一部分调用解析器。

- **Dissector.get(name)**
 + 按名称获取解析器引用。
 + 参数：
  * name:解析器的名字
 + 返回：如果找到，则为 Dissector 引用，否则 nil 。
- **dissector:call(tvb, pinfo, tree)**
 + 针对给定数据包（或其一部分）调用解析器。
 + 参数：
  * tvb:要剖析的缓冲区
  * pinfo:数据包信息
  * tree:要在其上添加协议项的树。
 + 返回：剖析的字节数。请注意，某些解析器总是返回传入缓冲区中的字节数，因此请注意。

#### 2.1.2 解析器表 DissectorTable
特定协议的子解析器表（例如，http、smtp、sip 等 TCP 子解析器被添加到表“tcp.port”中）。

有助于向表中添加更多解析器，以便它们出现在“解码为...”对话框中。

- **DissectorTable.new(tablename, [uiname], [type], [base], [proto])**
 + 创建一个新的解析器表供您的解析器使用。
 + 参数：
  * tablename:表名，表的简称，使用小写字母数字、点和/或下划线
  * uiname:用户界面中表的名称。默认为tablename中给定的名称，但可以是任何字符串
  * type:类型，ftypes.UINT8、ftypes.UINT16、ftypes.UINT24、ftypes.UINT32、ftypes.STRING、ftypes.NONE、ftypes.GUID、ftypes.UINT32之一。
  * base:进制类型，base.NONE、base.DEC、base.HEX、base.OCT、base.DEC_HEX、base.HEX_DEC、base.DEC之一。
  * proto:使用此解析器表的 Proto 对象。
 + 返回：新创建的 DissectorTable。
- **DissectorTable.get(tablename)**
 + 获取对现有解析器表的引用。
 + 参数：
  * tablename:表名
 + 返回：如果找到，则为 DissectorTable 引用，否则 nil 。
- **dissectortable:add(pattern, dissector)**
 + 将带有解析器函数的 Proto 或 Dissector 对象添加到解析器表中。
 + 参数：
  * pattern:要匹配的模式（整数、整数范围或字符串，具体取决于表的类型）。
  * dissector:要添加的解析器（ Proto 或 Dissector ）。
- **dissectortable:try(pattern, tvb, pinfo, tree)**
 + 尝试从表中调用解析器。
 + 参数：
  * pattern:要匹配的模式（整数、整数范围或字符串，具体取决于表的类型）。
  * tvb:要剖析的缓冲区
  * pinfo:数据包信息
  * tree:要在其上添加协议项的树。
 + 返回：剖析的字节数。请注意，某些解析器总是返回传入缓冲区中的字节数，因此请注意。
- **dissectortable:get_dissector(pattern)**
 + 尝试从表中获取解析器。
 + 参数：
  * pattern:要匹配的模式（整数、整数范围或字符串，具体取决于表的类型）。
 + 返回：如果找到则 Dissector 句柄，否则 nil 。
- **dissectortable:add_for_decode_as(proto)**
 + 将给定的 Proto 添加到此 DissectorTable 的“解码为...​”列表中。传入的 Proto 对象的函数用于解析。
 + 参数：
  * proto:要添加的 Proto 。

#### 2.1.3 偏好 Pref
Proto 的偏好。

- **Pref.bool(label, default, description)**
 + 创建要添加到 Proto.prefs Lua 表的布尔首选项。
 + 参数：
  * label:此首选项的标签（首选项输入右侧的文本）。
  * default:此首选项的默认值。
  * description:此偏好的描述。

#### 2.1.4 首选项 Prefs
协议的首选项表。

- **prefs:__newindex(name, pref)**
 + 创建新的偏好。
 + 参数：
  * name:此首选项的缩写。
  * pref:有效但仍未分配的 Pref 对象。

#### 2.1.5 协议 Proto
Wireshark 中的新协议。协议有多种用途。主要是解析协议，但它们也可以是用于注册用于其他目的的偏好的虚拟对象。

- **Proto.new(name, description)**
 + 创建一个新的 Proto 对象。
 + 参数：
  * name:协议的名称。
  * description:协议的长文本描述（通常是小写）。
 + 返回：新创建的 Proto 对象。

- **proto.dissector**
 + 协议的解析器，您定义的函数。稍后调用时，将给出该函数。
  * 一个 Tvb 对象
  * 一个 Pinfo 对象
  * 一个 TreeItem 对象
- **proto.name**
 + 该解析器的名字。
- **proto.description**
 + 该解析器的描述。
- **proto.fields**
 + 该解析器的 Lua 表。ProtoField

#### 2.1.6 协议专家 ProtoExpert
协议专家信息字段，在将项目添加到解剖树时使用。

一般用不到，需要了解参考官方文档。

#### 2.1.7 协议字段 ProtoField
协议字段（将项目添加到解析树时使用）。

- **ProtoField.new(name, abbr, type, [valuestring], [base], [mask], [description])**
 + 创建一个新的 ProtoField 对象用于协议字段。
 + 参数：
  * name:字段的实际名称（出现在树中的字符串）。
  * abbr:字段的过滤器名称（过滤器中使用的字符串）。
  * type:类型，ftypes.BOOLEAN、ftypes.CHAR、ftypes.UINT8、ftypes.UINT16、
  ftypes.UINT24、ftypes.UINT32、ftypes.UINT64、ftypes.INT8、ftypes.INT16、ftypes.INT24、ftypes.INT32、ftypes.INT64、ftypes.FLOAT、ftypes.DOUBLE、ftypes.ABSOLUTE_TIME、ftypes.RELATIVE_TIME、ftypes.STRING、ftypes.STRINGZ、ftypes.UINT_STRING、ftypes.ETHER、ftypes.BYTES、ftypes.UINT_BYTES、ftypes.IPv4、ftypes.IPv6、ftypes.IPXNET、ftypes.FRAMENUM、ftypes.PCRE、ftypes.GUID、ftypes.OID、ftypes.PROTOCOL、ftypes.REL_OID、ftypes.SYSTEM_ID、ftypes.EUI64、ftypes.NONE之一。
  * valuestring:包含与值相对应的文本的表，或者包含与值 ({min, max, "string"}) 相对应的范围字符串值表的表，或者包含值的单位名称的表如果基数是后续base之一，或者如果字段类型是 ftypes.FRAMENUM。base.RANGE_STRING、base.UNIT_STRING、frametype.NONE、frametype.REQUEST、frametype.RESPONSE、frametype.ACK、frametype.DUP_ACK
  * base:进制类型，base.NONE、base.DEC、base.HEX、base.OCT、base.DEC_HEX、base.HEX_DEC、base.UNIT_STRING、base.RANGE_STRING之一。
  * mask:要使用的位掩码。
  * description:字段的描述。
 + 返回：新创建的 新创建的 ProtoField 对象。
- **ProtoField.char(abbr, [name], [base], [valuestring], [mask], [description])**
- **ProtoField.uint8(abbr, [name], [base], [valuestring], [mask], [description])**
- **ProtoField.uint16(abbr, [name], [base], [valuestring], [mask], [description])**
- **ProtoField.uint24(abbr, [name], [base], [valuestring], [mask], [description])**
- **ProtoField.uint32(abbr, [name], [base], [valuestring], [mask], [description])**
- **ProtoField.uint64(abbr, [name], [base], [valuestring], [mask], [description])**
- **ProtoField.int8(abbr, [name], [base], [valuestring], [mask], [description])**
- **ProtoField.int16(abbr, [name], [base], [valuestring], [mask], [description])**
- **ProtoField.int24(abbr, [name], [base], [valuestring], [mask], [description])**
- **ProtoField.int32(abbr, [name], [base], [valuestring], [mask], [description])**
- **ProtoField.int64(abbr, [name], [base], [valuestring], [mask], [description])**
- **ProtoField.bool(abbr, [name], [display], [valuestring], [mask], [description])**
- **ProtoField.float(abbr, [name], [valuestring], [description])**
- **ProtoField.double(abbr, [name], [valuestring], [description])**
- **ProtoField.string(abbr, [name], [display], [description])**
- **ProtoField.bytes(abbr, [name], [display], [description])**
- **ProtoField.ipv4(abbr, [name], [description])**
- **ProtoField.ipv6(abbr, [name], [description])**

- **ProtoField.type**
 + 字段的类型。
- **ProtoField.abbr**
 + 字段的缩写名称。
- **ProtoField.name**
 + 字段的实际名称。
- **ProtoField.base**
 + 字段的进制类型。
- **ProtoField.valuestring**
 + 字段的值字符串。
- **ProtoField.mask**
 + 字段的位掩码。
- **ProtoField.description**
 + 字段的描述。

#### 2.1.8 全局函数 Global Functions

- **register_postdissector(proto, [allfields])**
 + 将 Proto 协议（带有解析器函数）设为后解析器。解剖后的每一帧都会调用它。
- **dissect_tcp_pdus(tvb, tree, min_header_size, get_len_func, dissect_func, [desegment])**
 + 让 TCP 层为 TCP 段中的每个 PDU 调用给定的 Lua 解析函数，其长度由给定的 get_len_func 函数返回。

### 2.2 获取解析数据
#### 2.2.1 字段 Field 
用于获取字段值的字段提取器。 Field 对象只能在解析器、后解析器、启发式解析器和 Tap 的回调函数之外创建。

创建后，它会在回调函数中使用，以生成 FieldInfo 对象。

- **Field.new(fieldname)**
 + 创建一个字段提取器。
 + 参数：
  * fieldname:字段的过滤器名称（例如 ip.addr）
 + 返回：字段提取器

> 必须在调用 Taps 或 Dissector 之前定义字段提取器
{: .prompt-warning }

- **Field.name**
 + 该字段的过滤器名称，或者nil。
- **Field.display**
 + 该字段的完整显示名称，或 nil。
- **Field.type**
 + 该字段的 ftype ，或者nil。

#### 2.2.2 字段信息 FieldInfo
从解析的数据包数据中提取的字段。 FieldInfo 对象只能在解析器、后解析器、启发式解析器和 Tap 的回调函数中使用。

FieldInfo 可以通过预先使用 Field.new() 或 Field() 在现有的 Wireshark 字段上调用，也可以在 Lua 创建的新字段上调用一个 ProtoField。

- **fieldinfo.len**
 + 该字段的长度。
- **fieldinfo.offset**
 + 该字段的偏移量。
- **fieldinfo.value**
 + 该字段的值。
- **fieldinfo.label**
 + 表示该字段的字符串。
- **fieldinfo.display**
 + 该字段的字符串显示如 GUI 中所示。
- **fieldinfo.type**
 + 内部字段类型，一个与 ftype 值之一匹配的数字。
- **fieldinfo.little_endian**
 + 该字段是否采用小端编码（布尔值）。
- **fieldinfo.big_endian**
 + 该字段是否采用大端编码（布尔值）。
- **fieldinfo.name**
 + 该字段的过滤器名称。

#### 2.2.3 全局函数

- **all_field_infos()**
 + 获取当前树中的所有字段。请注意，这仅获取底层解析器此时为此数据包填充的任何字段 - 可能有适用于该数据包的字段根本没有被填充，因为此时任何东西都不需要它们。此函数仅获取 C 端代码当前填充的内容，而不是完整列表。

### 2.3 获取数据包信息

#### 2.3.1 地址 Address
代表一个地址。

- **Address.ip(hostname)**
 + 创建表示 IPv4 地址的地址对象。
  + 参数：
  * hostname:IP 主机的地址或名称。
 + 返回：地址对象。
- **Address.ipv6(hostname)**
 + 创建表示 IPv6 地址的地址对象。
  + 参数：
  * hostname:IP 主机的地址或名称。
 + 返回：地址对象。
- **Address.ether(eth)**
 + 创建表示以太网地址的地址对象。
  + 参数：
  * eth:以太网地址。
 + 返回：地址对象。

#### 2.3.2 列 Column
数据包列表中的一列。

#### 2.3.3 列 Columns
数据包列表的 Column 。

#### 2.3.4 NS时间 NSTime
NSTime代表一个nstime_t。这是一个具有秒和纳秒的对象。

#### 2.3.5 数据包信息 Pinfo
Packet information数据包信息。

- **pinfo.visited**
 + 该数据包是否已被访问过。
- **pinfo.number**
 + 当前文件中该数据包的编号。
- **pinfo.len**
 + 帧的长度。
- **pinfo.caplen**
 + 捕获的帧长度。
- **pinfo.curr_proto**
 + 我们正在解析哪个协议。
- **pinfo.desegment_offset**
 + tvbuff 中的偏移量，下次调用时解析器将继续处理该偏移量。
- **pinfo.port_type**
 + .src_port 和 .dst_port 的端口类型。
- **pinfo.src_port**
 + 该数据包的源端口。
- **pinfo.dst_port**
 + 该数据包的目标端口。
- **pinfo.net_src**
 + 该数据包的网络层源地址。
- **pinfo.net_dst**
 + 该数据包的网络层目标地址。

#### 2.3.6 私有表 PrivateTable
PrivateTable代表pinfo→private_table。主要调试使用。

### 2.4 处理数据包数据的函数

#### 2.4.1 字节数组 ByteArray
- **ByteArray.new([hexbytes], [separator])**
 + 创建一个新的 ByteArray 对象。
 + 参数：
  * hexbytes:由十六进制字节组成的字符串，如“00 B1 A2”或“1a2b3c4d”。
  * separator:十六进制字节/单词之间的字符串分隔符（默认=“”），或者如果使用布尔值 true ，则第一个参数被视为原始二进制数据
 + 返回：新的 ByteArray 对象。

> 从版本 1.11.3 开始，如果第二个参数是布尔值 true ，则第一个参数将被视为要使用的原始 Lua 字节字符串，而不是十六进制字符串。
{: .prompt-info }

#### 2.4.2 缓冲区 Tvb
Tvb 代表数据包的缓冲区。它作为参数传递给侦听器和解析器，并可用于从数据包的数据中提取信息（通过 TvbRange ）。

要创建 TvbRange ，必须使用偏移量和长度作为可选参数来调用 Tvb ；偏移量默认为 0，长度默认为 tvb:captured_len() 。

> Tvb 只能由当前侦听器或解析器调用使用，并在侦听器或解析器返回后立即销毁，因此一旦函数返回，对它们的引用将不可用。
{: .prompt-warning }

- **tvb:reported_len()**
 + 获取 Tvb 的报告长度（网络上的长度）。
- **tvb:captured_len()**
 + 获取 Tvb 的捕获长度（捕获过程中保存的数量）。
- **tvb:len()**
 + 获取 Tvb 的报告长度（网络上的长度）。
- **tvb:reported_length_remaining([offset])**
 + 获取到 Tvb 末尾的数据包数据的报告（未捕获）长度；如果偏移量超出 Tvb 末尾，则获取 0。
- **tvb:bytes([offset], [length])**
 + 从 Tvb 获取 ByteArray 。
 + 参数：
  * offset:距 Tvb 开头的偏移量（以八位字节为单位）。默认为 0。
  * length:范围的长度（以八位字节为单位）。默认为 Tvb 结束。
 + 返回：ByteArray 对象或 nil。
- **tvb:offset()**
 + 返回子 Tvb 的原始偏移量（从源 Tvb 的开头）。
- **tvb:range([offset], [length])**
 + 从此 Tvb 创建一个 TvbRange 。
 + 参数：
  * offset:距 Tvb 开头的偏移量（以八位字节为单位）。默认为 0。
  * length:范围的长度（以八位字节为单位）。默认为 -1，指定 Tvb 中的剩余字节。
 + 返回：TVB范围
- **tvb:raw([offset], [length])**
 + 从此 Tvb 创建一个 TvbRange 。
 + 参数：
  * offset:第一个字节的位置。默认值为 0，即第一个字节。
  * length:要获取的段的长度。默认值为 -1，或 Tvb 中的剩余字节。
 + 返回：Tvb 中二进制字节的 Lua 字符串。

#### 2.4.3 Tvb的可用范围 TvbRange
TvbRange 表示 Tvb 的可用范围，用于从生成它的 Tvb 中提取数据。

TvbRange 是通过调用 Tvb 创建的（例如“tvb(offset,length)”）。长度默认为 -1，表示使用到 Tvb 末尾的字节。如果 TvbRange 范围超出 Tvb 的范围，则创建将导致运行时错误。

- **tvbrange:uint()**
 + 从 TvbRange 获取 Big Endian（网络顺序）无符号整数。该范围必须为 1-4 个八位字节长。
- **tvbrange:le_uint()**
 + 从 TvbRange 获取 Little Endian 无符号整数。该范围必须为 1-4 个八位字节长。
- **tvbrange:uint64()**
 + 从 TvbRange 获取 Big Endian（网络顺序）无符号 64 位整数，作为 UInt64 对象。该范围必须为 1-8 个八位字节长。
- **tvbrange:int()**
 + 从 TvbRange 获取 Little Endian 有符号整数。该范围必须为 1-4 个八位字节长。
- **tvbrange:int64()**
 + 从 TvbRange 获取 Big Endian（网络顺序）带符号的 64 位整数，作为 Int64 对象。该范围必须为 1-8 个八位字节长。
- **tvbrange:float()**
 + 从 TvbRange 获取 Little Endian 浮点数。该范围必须为 4 或 8 个八位字节长。
- **tvbrange:ipv4()**
 + 从 TvbRange 获取 IPv4 地址作为 Address 对象。
- **tvbrange:ipv6()**
 + 从 TvbRange 获取 IPv6 Address 对象。

### 2.5 将信息添加到解析树
#### 2.5.1 树项 TreeItem 
TreeItem 代表 Wireshark 的数据包详细信息窗格和 TShark 的数据包详细信息视图中的信息。 TreeItem 表示树中的一个节点，它也可能是一个子树并具有子节点列表。子树的子树有零个或多个同级，它们是同一 TreeItem 子树的其他子树。

在解剖、启发式解剖和后解剖期间，根 TreeItem 作为函数回调的第三个参数传递给解剖器（例如 myproto.dissector(tvbuf,pktinfo,root) ）。

在某些情况下，为了提高性能，树并未真正添加到其中。例如，当前未在 Wireshark 的可见窗口窗格中显示/选择的数据包，或者未使用 -V 开关调用 TShark。然而，“add”类型 TreeItem 函数仍然可以被调用，并且仍然返回 TreeItem 对象 - 但信息并没有真正添加到树中。因此，您通常不需要担心是否有一棵真正的树。如果由于某种原因您需要知道它，您可以使用 TreeItem.visible 属性 getter 来检索状态。

- **treeitem:add_packet_field(protofield, [tvbrange], encoding, [label])**
 + 将给定 ProtoField 对象的新子树添加到此树项，并返回新子 TreeItem 。与 TreeItem:add() 和 TreeItem:add_le() 不同， ProtoField 参数不是可选的，并且不能是 Proto 对象。相反，此函数始终使用 ProtoField 来确定要从传入的 TvbRange 中提取的字段类型，并在 GUI 的“数据包字节”窗格中突出显示相关字节（如果有）是GUI）等。如果没有给出 TvbRange ，则不会突出显示任何字节，并且无法确定该字段的值；在这种情况下， ProtoField 必须已定义/创建为不具有长度，否则会发生错误。然而，出于向后兼容性的原因，仍然必须给出 encoding 参数。与 TreeItem:add() 和 TreeItem:add_le() 不同，此函数通过将 encoding 参数设置为 ENC_BIG_ENDIAN 来执行大端和小端解码或 ENC_LITTLE_ENDIAN 。
 + 参数：
  * protofield:要添加到树中的 ProtoField 字段对象。
  * tvbrange:此树项覆盖/表示的数据包中的字节数 TvbRange 。
  * encoding:TvbRange 中的字段编码。
  * label:要附加到创建的 TreeItem 的一个或多个字符串。
 + 返回：新的子元素 TreeItem ，字段提取的值或 nil，以及偏移量或 nil。
- **treeitem:add([protofield], [tvbrange], [value], [label])**
 + 向此树项目添加一个子项目，返回新的子项目 TreeItem 。如果 ProtoField 表示数值（int、uint 或 float），则将其视为 Big Endian（网络顺序）值。该函数具有复杂的形式：'treeitem:add([protofield,] [tvbrange,] value], label)'，这样如果第一个参数是 ProtoField 或 Proto ，第二个参数是 TvbRange ，第三个参数给出，它是一个值；但如果第二个参数是非 TvbRange ，那么它就是值（而不是用“nil”填充该参数，这对此函数无效）。如果第一个参数是非 ProtoField 和非 Proto ，则该参数可以是 TvbRange 或标签，并且该值不在使用。
 + 参数：
  * protofield:要添加到树中的 ProtoField 字段对象。
  * tvbrange:此树项覆盖/表示的数据包中的字节数 TvbRange 。
  * value:该字段的值，而不是 ProtoField/Proto 值。
  * label:用于树项目标签的一个或多个字符串，而不是 ProtoField/Proto 字符串。
 + 返回：新的子 TreeItem。
- **treeitem:add_le([protofield], [tvbrange], [value], [label])**
 + 向此树项目添加一个子项目，返回新的子项目 TreeItem 。如果 ProtoField 表示数值（int、uint 或 float），则将其视为 Little  Endian值。该函数具有复杂的形式：'treeitem:add_le([protofield,] [tvbrange,] value], label)'，这样如果第一个参数是 ProtoField 或 Proto ，第二个参数是 TvbRange ，第三个参数给出，它是一个值；但如果第二个参数是非 TvbRange ，那么它就是值（而不是用“nil”填充该参数，这对此函数无效）。如果第一个参数是非 ProtoField 和非 Proto ，则该参数可以是 TvbRange 或标签，并且该值不在使用。
 + 参数：
  * protofield:要添加到树中的 ProtoField 字段对象。
  * tvbrange:此树项覆盖/表示的数据包中的字节数 TvbRange 。
  * value:该字段的值，而不是 ProtoField/Proto 值。
  * label:用于树项目标签的一个或多个字符串，而不是 ProtoField/Proto 字符串。
 + 返回：新的子 TreeItem。

## 3. Lua示例
```lua
local proto_foo = Proto("foo", "Foo Protocol")
    proto_foo.fields.bytes = ProtoField.bytes("foo.bytes", "Byte array")
    proto_foo.fields.u16 = ProtoField.uint16("foo.u16", "Unsigned short", base.HEX)

    function proto_foo.dissector(buf, pinfo, tree)
            -- ignore packets less than 4 bytes long
            if buf:len() < 4 then return end

            -- ##############################################
            -- # Assume buf(0,4) == {0x00, 0x01, 0x00, 0x02}
            -- ##############################################

            local t = tree:add( proto_foo, buf() )

            -- Adds a byte array that shows as: "Byte array: 00010002"
            t:add( proto_foo.fields.bytes, buf(0,4) )

            -- Adds a byte array that shows as "Byte array: 313233"
            -- (the ASCII char code of each character in "123")
            t:add( proto_foo.fields.bytes, buf(0,4), "123" )

            -- Adds a tree item that shows as: "Unsigned short: 0x0001"
            t:add( proto_foo.fields.u16, buf(0,2) )

            -- Adds a tree item that shows as: "Unsigned short: 0x0064"
            t:add( proto_foo.fields.u16, buf(0,2), 100 )

            -- Adds a tree item that shows as: "Unsigned short: 0x0064 ( big endian )"
            t:add( proto_foo.fields.u16, buf(1,2), 100, nil, "(", nil, "big", 999, nil, "endian", nil, ")" )

            -- LITTLE ENDIAN: Adds a tree item that shows as: "Unsigned short: 0x0100"
            t:add_le( proto_foo.fields.u16, buf(0,2) )

            -- LITTLE ENDIAN: Adds a tree item that shows as: "Unsigned short: 0x6400"
            t:add_le( proto_foo.fields.u16, buf(0,2), 100 )

            -- LITTLE ENDIAN: Adds a tree item that shows as: "Unsigned short: 0x6400 ( little endian )"
            t:add_le( proto_foo.fields.u16, buf(1,2), 100, nil, "(", nil, "little", 999, nil, "endian", nil, ")" )
    end

    udp_table = DissectorTable.get("udp.port")
    udp_table:add(7777, proto_foo)
```

一个简单的lua代码可以分为三部分：
1、创建Proto对象
2、创建dissector方法
3、注册解析器

****

本文参考

> 1. [Lua 教程](https://www.runoob.com/lua/lua-tutorial.html)
> 2. [Wireshark 的 Lua API 参考手册](https://www.wireshark.org/docs/wsdg_html_chunked/wsluarm_modules.html)