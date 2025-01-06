---
title: 前端与后端的数据交互（jquery ajax + python flask)
author: fnoobt
date: 2018-03-06 09:49:00 +0800
categories: [Web,JavaScript]
tags: [web,ajax,javascript,python,flask]
media_subpath: '/assets/img/commons/web/'
---

## 概述

> 前端与后端的数据交互，最常用的就是GET、POST，比较常用的用法是：提交表单数据到后端，后端返回json

- 前端的数据发送与接收
  1. 提交表单数据
  2. 提交JSON数据
- 后端的数据接收与响应
  1. 接收GET请求数据
  2. 接收POST请求数据
  3. 响应请求

## 前端的数据发送与接收

### 提交表单数据

GET请求:
```js
var data = {
    "name": "test",
    "age": 1
};
$.ajax({
    type: 'GET',
    url: /your/url/,
    data: data, // 最终会被转化为查询字符串跟在url后面，例如： /your/url/?name=test&age=1
    dataType: 'json', // 注意：这里是指希望服务端返回json格式的数据
    success: function(data) { // 这里的data就是json格式的数据
    },
    error: function(xhr, type) {
    }
});
```

POST请求:
```js
var data = {}
// 如果页面并没有表单，只是input框，请求也只是发送这些值，那么可以直接获取放到data中
data['name'] = $('#name').val()

// 如果页面有表单，那么可以利用jquery的serialize()方法获取表单的全部数据
data = $('#form1').serialize();

$.ajax({
    type: 'POST',
    url: /your/url/,
    data: data,
    dataType: 'json', // 注意：这里是指希望服务端返回json格式的数据
    success: function(data) { // 这里的data就是json格式的数据
    },
    error: function(xhr, type) {
    }
});
```

> 注意：
> 1. 参数dataType：期望的服务器响应的数据类型，可以是`null`, `xml`, `script`, `json`
> 2. 请求头中的Content-Tpye默认是`Content-Type:application/x-www-form-urlencoded`，所以参数会被编码为 name=xx&age=1 这种格式，提交到后端，后端会当作表单数据处理
{: .prompt-tip }

### 提交JSON数据

如果要给后端传递json数据，就需要增加content-type参数，告诉后端，传递过来的数据格式，并且需要将data转为字符串进行传递。实际上，服务端接收到后，发现是json格式，做的操作就是将字符串转为json对象。  
另外，不转为字符串，即使加了content-type的参数，也默认会转成 name=xx&age=1，使后端无法获取正确的json

POST一个json数据:
```js
var data = {
    "name": "test",
    "age", 1
}
$.ajax({
    type: 'POST',
    url: /your/url/,
    data: JSON.stringify(data), // 转化为字符串
    contentType: 'application/json; charset=UTF-8',
    dataType: 'json', // 注意：这里是指希望服务端返回json格式的数据
    success: function(data) { // 这里的data就是json格式的数据
    },
    error: function(xhr, type) {
    }
});
```

## 后端的数据接收与返回

### 接收GET请求数据

```js
name = request.args.get('name', '')
age = int(request.args.get('age', '0'))
```

### 接收POST请求数据

接收表单数据
```js
name = request.form.get('name', '')
age = int(request.form.get('age', '0'))
```

接收Json数据
```js
data = request.get_json()
```

> 另外，如果前端提交的数据格式不能被识别，可以用`request.get_data()`接收数据。  
> 微信公众号后台接收微信推送的xml格式的消息体，就可以用`request.get_data()`来接收

### 响应请求

Flask可以非常方便的返回json数据

```js
from Flask import jsonify

return jsonify({'ok': True})
```

看一下源码就可以知道，jsonify就是帮我们做了点添加mimetype这样的杂事，所以如果不嫌麻烦和难看，你也可以这样写

```js
// 太丑了，还是别这么干了
return Response(data=json.dumps({'ok': True}), mimetype='application/json')
```

放两张截图来直观看一下请求

![PostFormat](post_format_data.png)
_post表单数据到服务端_

![PostJson](post_json_data.png)
_Post JSON数据到服务端_

****

本文参考

> 1. [前端与后端的数据交互](https://www.jianshu.com/p/4350065bdffe)