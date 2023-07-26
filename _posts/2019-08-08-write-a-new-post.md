---
title: 编辑帖子
author: cotes
date: 2019-08-08 14:10:00 +0800
categories: [Blog,博客教程]
tags: [编辑帖子]
render_with_liquid: false
---

本教程将指导您如何在 Chirpy 模板中撰写帖子，即使您以前使用过 Jekyll 也值得一读，因为许多功能都需要设置特定的变量。

## 命名和路径

创建一个名为 `YYYY-MM-DD-标题.后缀名`{: .filepath} 的新文件并将其放在根目录的 `_posts`{: .filepath} 中. 请注意，这个 `后缀名`{: .filepath} 必须是 `md`{: .filepath}和 `markdown`{: .filepath}之一. 如果您想节省创建文件的时间，请考虑使用插件[`Jekyll-Compose`](https://github.com/jekyll/jekyll-compose) 来完成此操作。

## 前言

基本上，您需要在帖子顶部填写以下[前言](https://jekyllrb.com/docs/front-matter/):

```yaml
---
title: 标题
date: YYYY-MM-DD HH:MM:SS +/-TTTT
categories: [顶级类别, 子类别]
tags: [标签]     # TAG名称应始终小写
---
```

> 帖子的布局已默认设置为 `post` ,因此无需在`前言`块中添加post变量。
{: .prompt-tip }

### 日期时区

为了准确记录帖子的发布日期，您不仅应该在 `_config.yml`{: .filepath}中设置帖子的 `时区` ，而且还应该在其`前言`块的 `date` 变量中提供帖子的时区. 格式：`+/-TTTT`, 例如：`+0800`.

### 类别和标签

每个帖子的类别`categories`设计为最多包含两个元素, 标签`tags`中的元素数量可以是零到无穷大. 例如:

```yaml
---
categories: [动物, 昆虫]
tags: [蜜蜂]
---
```

### 作者信息

帖子的作者信息通常不需要填写在`前言`中，默认情况下，它们将从配置文件的变量`social.name`和`social.links`的第一个条目中获得。但您也可以按如下方式覆盖它：

在`_data/authers.yml`中添加作者信息（如果您的网站没有此文件，请毫不犹豫地创建一个）。

```yaml
<author_id>:
  name: <full name>
  twitter: <twitter_of_author>
  url: <homepage_of_author>
```
{: file="_data/authors.yml" }


然后使用`author`指定单个条目，或使用`authors`指定多个条目：

```yaml
---
author: <author_id>                     # 用于单个条目
# or
authors: [<author1_id>, <author2_id>]   # 用于多个条目
---
```


话虽如此，关键字`author` 也可以识别多个条目。

> 从文件`_data/authors.yml`{: .filepath }读取作者信息的好处是页面将具有元标记`twitter:creator`, 这丰富了[Twitter Cards](https://developer.twitter.com/en/docs/twitter-for-websites/cards/guides/getting-started#card-and-content-attribution)，并且有利于SEO.
{: .prompt-info }

## 目录

默认情况下，**T**able **o**f **C**ontents（TOC）显示在帖子的右侧面板上。如果要全局关闭它，请转到`_config.yml`{: .filepath}并将变量`toc`的值设置为`false`。如果你想关闭特定帖子的目录，请将以下内容添加到帖子的[前言](https://jekyllrb.com/docs/front-matter/)中：

```yaml
---
toc: false
---
```

## 评论

评论的全局切换由文件`_config.yml`{: .filepath}中的变量`comments.active`定义。为该变量选择评论系统后，将为所有帖子打开评论。

如果您想关闭特定帖子的评论，请将以下内容添加到帖子的 **前言** 中：

```yaml
---
comments: false
---
```

## 数学

出于网站性能原因，默认情况下不会加载数学功能（由 **[MathJax](https://www.mathjax.org/)** 提供支持）。但它可以通过以下方式启用：

```yaml
---
math: true
---
```

## Mermaid

[**Mermaid**](https://github.com/mermaid-js/mermaid)是一个很棒的图表生成工具。要在帖子中启用它，请将以下内容添加到 YAML 块：

```yaml
---
mermaid: true
---
```

然后，您可以像使用其他标记语言一样使用它：用 ```` ```mermaid ```` 和 ```` ``` ```` 围绕图形代码。可以使用[Mermaid在线编辑器](https://mermaid.live/)轻松创建详细的图表。

## 图像

### 标题

在图像的下一行添加斜体，然后它将成为标题并出现在图像的底部：

```markdown
![img-description](/path/to/image)
_Image Caption_
```
{: .nolineno}

### 大小

为了防止页面内容布局在加载图像时移动，我们应该设置每个图像的宽度和高度。

```markdown
![Desktop View](/assets/img/commons/demo/mockup.png){: width="700" height="400" }
```
{: .nolineno}

> 对于 SVG，您必须至少指定其 _宽度_，否则它将不会呈现。
{: .prompt-info }

从 _Chirpy v5.0.0_ 开始，`height`和`width`支持缩写（`height` → `h`，`width` → `w`）。以下示例具有与上述示例相同的效果：

```markdown
![Desktop View](/assets/img/commons/demo/mockup.png){: w="700" h="400" }
```
{: .nolineno}

### 位置

默认情况下，图像居中，但您可以使用类`normal`、`left`和`right`之一指定位置。

> 一旦指定了位置，就不应添加图像标题。
{: .prompt-warning }

- **正常位置**

  图像将在下面的示例中左对齐：

  ```markdown
  ![Desktop View](/assets/img/commons/demo/mockup.png){: .normal }
  ```
  {: .nolineno}

- **向左浮动**

  ```markdown
  ![Desktop View](/assets/img/commons/demo/mockup.png){: .left }
  ```
  {: .nolineno}

- **向右浮动**

  ```markdown
  ![Desktop View](/assets/img/commons/demo/mockup.png){: .right }
  ```
  {: .nolineno}

### 暗/亮模式

您可以在暗/亮模式下使图像遵循主题偏好。这需要您准备两个图像，一个用于暗模式，另一个用于亮模式，然后为它们指定一个特定的类（`dark` 或 `light`）：

```markdown
![Light mode only](/path/to/light-mode.png){: .light }
![Dark mode only](/path/to/dark-mode.png){: .dark }
```

### 阴影

可以为程序窗口的屏幕截图添加显示阴影效果：

```markdown
![Desktop View](/assets/img/sample/mockup.png){: .shadow }
```
{: .nolineno}

### CDN URL

如果您将图像托管在CDN上，则可以通过分配`_config.yml`{: .filepath}文件的变量`img_cdn`来节省重复写入CDN URL的时间

```yaml
img_cdn: https://cdn.com
```
{: file='_config.yml' .nolineno}

一旦分配了`img_cdn`，CDN URL将添加到以`/`开头的所有图像（网站头像和帖子的图像）的路径中。

例如，在使用图像时：

```markdown
![The flower](/path/to/flower.png)
```
{: .nolineno}

解析结果会在图像路径之前自动添加CDN前缀`https://cdn.com`：

```html
<img src="https://cdn.com/path/to/flower.png" alt="The flower">
```
{: .nolineno }

### 图像路径

当帖子包含许多图像时，重复定义图像的路径将是一项耗时的任务。为了解决这个问题，我们可以在帖子的 YAML 块中定义此路径：

```yml
---
img_path: /img/path/
---
```

然后，Markdown 的图片源可以直接写入文件名：

```md
![The flower](flower.png)
```
{: .nolineno }

输出将是：

```html
<img src="/img/path/flower.png" alt="The flower">
```
{: .nolineno }

### 预览图像

如果你想在帖子顶部添加一张图片，请提供一张分辨率为 `1200 x 630` 的图片。请注意，如果图像纵横比不满足 `1.91 : 1` ，则图像将被缩放和裁剪。

了解这些先决条件后，可以开始设置图像的属性：

```yaml
---
image:
  path: /path/to/image
  alt: image alternative text
---
```

注意，[`img_path`](#图像路径)也可以传递给预览图像，也就是说，当设置了它时，属性`path`只需要图像文件名。

为了简单使用，您也可以只使用`image`来定义路径。

```yml
---
image: /path/to/image
---
```

### LQIP

对于预览图像：

```yaml
---
image:
  lqip: /path/to/lqip-file # or base64 URI
---
```

> 您可以在帖子 _[文本和排版](/posts/text-and-typography/)_ 的预览图像中观察 LQIP。


对于普通图像：

```markdown
![Image description](/path/to/image){: lqip="/path/to/lqip-file" }
```
{: .nolineno }

## 置顶帖子

您可以将一个或多个帖子固定到首页顶部，固定帖子会根据其发布日期以相反的顺序排序。启用方式：

```yaml
---
pin: true
---
```

## 提示

提示有几种类型：`tip`, `info`, `warning`, 和 `danger`。它们可以通过将类`prompt-{type}`添加到块引号中来生成。例如，定义`info`类型的提示如下：

```md
> Example line for prompt.
{: .prompt-info }
```
{: .nolineno }

## 语法

### 内联代码

```md
`inline code part`
```
{: .nolineno }

### 文件路径突出显示

```md
`/path/to/a/file.extend`{: .filepath}
```
{: .nolineno }

### 代码块

标记符号```` ``` ````可以很容易地创建如下代码块：

````md
```
This is a plaintext code snippet.
```
````

#### 指定语言

使用```` ```{language} ````，您将获得一个带有语法高亮显示的代码块：

````markdown
```yaml
key: value
```
````

> Jekyll标记`{% highlight %}`与此主题不兼容。
{: .prompt-danger }

#### 行号

默认情况下，除`明文`、`控制台`和`终端`外的所有语言都将显示行号。如果要隐藏代码块的行号，请将类`nolineno`添加到其中：

````markdown
```shell
echo 'No more line numbers!'
```
{: .nolineno }
````

#### 指定文件名

您可能已经注意到，代码语言将显示在代码块的顶部。如果您想用文件名替换它，可以添加属性`file`来实现这一点：

````markdown
```shell
# content
```
{: file="path/to/file" }
````

#### Liquid代码

如果要显示**Liquid**代码段，请在 liquid 代码两边加上`{% raw %}`和`{% endraw %}`：

````markdown
{% raw %}
```liquid
{% if product.title contains 'Pack' %}
  This product's title contains the word Pack.
{% endif %}
```
{% endraw %}
````

或者在帖子的YAML块中添加`render_with_liquid: false`（需要Jekyll 4.0或更高版本）。

## 视频

您可以使用以下语法嵌入视频：

```liquid
{% include embed/{Platform}.html id='{ID}' %}
```

其中`Platform`是平台名称的小写字母，`ID`是视频ID。

下表显示了如何在给定的视频URL中获得我们需要的两个参数，您还可以了解当前支持的视频平台。

| Video URL                                                                                          | Platform  | ID            |
|----------------------------------------------------------------------------------------------------|-----------|:--------------|
| [https://www.**youtube**.com/watch?v=**H-B46URT4mg**](https://www.youtube.com/watch?v=H-B46URT4mg) | `youtube` | `H-B46URT4mg` |
| [https://www.**twitch**.tv/videos/**1634779211**](https://www.twitch.tv/videos/1634779211)         | `twitch`  | `1634779211`  |



## 了解更多信息

有关Jekyll帖子的更多信息，请访问[Jekyll 文档：posts](https://jekyllrb.com/docs/posts/)。

****

> [原文出处](https://chirpy.cotes.page/posts/write-a-new-post/)
