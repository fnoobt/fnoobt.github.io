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

## 文章描述
默认情况下，帖子的第一句话会在主页的帖子列表、进一步阅读部分以及 RSS feed 的 XML 中显示。如果您不想显示帖子的自动生成描述，可以在前端元数据中使用 `description` 字段进行自定义，如下所示：

```yaml
---
description: 帖子的简短摘要。
---
```

此外， `description` 文本还会显示在文章页面的标题下方。

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

## 媒体
在 Chirpy 中将图片、音频和视频称为媒体资源。

### URL前缀
有时在一篇文章中需要为多个资源定义重复的 URL 前缀，对于吧这个无聊的任务，可以通过设置以下两个参数来避免。

1. 如果你使用 CDN 托管媒体文件，可以在 `_config.yml`{: .filepath}  中指定 cdn 。站点头像和帖子的媒体资源 URL 将会前缀 CDN 域名。

```yaml
cdn: https://cdn.com
```

2. 要在当前帖子/页面范围中指定资源路径前缀，请在帖子的前端事项中设置 `media_subpath` ：

```yaml
---
media_subpath: /path/to/media/
---
```

`site.cdn` 和 `page.media_subpath` 这两个选项可以单独使用或结合使用，以灵活地组成最终的资源 URL： `[site.cdn/][page.media_subpath/]file.ext`

### 图像

#### 标题

在图像的下一行添加斜体，然后它将成为标题并出现在图像的底部：

```markdown
![img-description](/path/to/image)
_Image Caption_
```
{: .nolineno}

#### 大小

为了防止页面内容布局在加载图像时移动，我们应该设置每个图像的宽度和高度。

```markdown
![Desktop View](/assets/img/commons/demo/mockup.png){: width="700" height="400" }
```
{: .nolineno}

> 对于 SVG，您必须至少指定其 _宽度_，否则它将不会呈现。
{: .prompt-info }

从 _Chirpy v5.0.0_ 开始，`height`和`width`支持缩写（`height` → `h`，`width` → `w`）。以下示例具有与上述示例相同的效果：

```markdown
![Desktop View](mockup.png){: w="700" h="400" }
```
{: .nolineno}

#### 位置

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

#### 暗/亮模式

您可以在暗/亮模式下使图像遵循主题偏好。这需要您准备两个图像，一个用于暗模式，另一个用于亮模式，然后为它们指定一个特定的类（`dark` 或 `light`）：

```markdown
![Light mode only](/path/to/light-mode.png){: .light }
![Dark mode only](/path/to/dark-mode.png){: .dark }
```

#### 阴影

可以为程序窗口的屏幕截图添加显示阴影效果：

```markdown
![Desktop View](/assets/img/sample/mockup.png){: .shadow }
```
{: .nolineno}

#### 预览图片

如果你想在帖子顶部添加一张图片，请提供一张分辨率为 `1200 x 630` 的图片。请注意，如果图像纵横比不满足 `1.91 : 1` ，则图像将被缩放和裁剪。

了解这些先决条件后，可以开始设置图片的属性：

```yaml
---
image:
  path: /path/to/image
  alt: image alternative text
---
```

注意，[`media_subpath`](#url前缀)也可以传递给预览图片，也就是说，当设置了它时，属性`path`只需要图像文件名。

为了简单使用，您也可以只使用`image`来定义路径。

```yml
---
image: /path/to/image
---
```

#### LQIP

对于预览图片：

```yaml
---
image:
  lqip: /path/to/lqip-file # or base64 URI
---
```

> 您可以在帖子 _[文本和排版](/posts/blog-text-and-typography/)_ 的预览图片中观察到 LQIP。


对于普通图像：

```markdown
![Image description](/path/to/image){: lqip="/path/to/lqip-file" }
```
{: .nolineno }

### 视频

#### 社交媒体平台

您可以使用以下语法嵌入视频：

```liquid
{% include embed/{Platform}.html id='{ID}' %}
```

其中`Platform`是平台名称的小写字母，`ID`是视频ID。

下表显示了如何在给定的视频URL中获得我们需要的两个参数，您还可以了解当前支持的视频平台。

| Video URL                                                                                          | Platform  | ID            |
| -------------------------------------------------------------------------------------------------- | --------- | :------------ |
| [https://www.**youtube**.com/watch?v=**H-B46URT4mg**](https://www.youtube.com/watch?v=H-B46URT4mg) | `youtube` | `H-B46URT4mg` |
| [https://www.**twitch**.tv/videos/**1634779211**](https://www.twitch.tv/videos/1634779211)         | `twitch`  | `1634779211`  |
| [https://www.**bilibili**.tv/videos/**BV1Q44y1B7Wf**](https://www.bilibili.com/video/BV1Q44y1B7Wf)         | `bilibili`  | `BV1Q44y1B7Wf`  |

#### 视频文件
如果你想直接嵌入视频文件，请使用以下语法：

```liquid
{% include embed/video.html src='{URL}' %}
```

`URL` 是一个视频文件的 URL，例如 `/path/to/sample/video.mp4` 。

您还可以为嵌入的视频文件指定其他属性。以下是一份允许的属性完整列表。

- `poster='/path/to/poster.png'` — 视频下载时显示的海报图片
- `title='Text'` — 视频的标题，显示在视频下方，与图片的标题相同
- `autoplay=true` — 视频一可以播放时就开始播放
- `loop=true` — 到达视频结尾时自动返回到开头
- `muted=true` — 初始时音频将被静音
- `types` — 请指定其他视频格式的扩展名，用 `|` 分隔。请确保这些文件与您的主要视频文件位于同一目录中。

考虑一个包含以上所有内容的示例：

```liquid
{%
  include embed/video.html
  src='/path/to/video.mp4'
  types='ogg|mov'
  poster='poster.png'
  title='Demo video'
  autoplay=true
  loop=true
  muted=true
%}
```

### 音频
如果你想直接嵌入音频文件，请使用以下语法：

```liquid
{% include embed/audio.html src='{URL}' %}
```

`URL` 是一个音频文件的 URL，例如 `/path/to/audio.mp3` 。

你还可以为嵌入的音频文件指定额外的属性。以下是一份允许的属性列表。

- `title='Text'` — 音频下方显示的标题，与图片的标题样式相同
- `types` — 指定额外音频格式的扩展名，用 `|` 分隔。确保这些文件与主要音频文件位于同一目录中。

考虑一个包含以上所有内容的示例：

```liquid
{%
  include embed/audio.html
  src='/path/to/audio.mp3'
  types='ogg|wav|aac'
  title='Demo audio'
%}
```

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

### 文件路径高亮

```md
`/path/to/a/file.extend`{: .filepath}
```
{: .nolineno }

### 代码块

Markdown符号```` ``` ````可以很容易地创建如下代码块：

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

默认情况下，除`明文(plaintext)`、`控制台(console)`和`终端(terminal)`外的所有语言都将显示行号。如果要隐藏代码块的行号，请将类`nolineno`添加到其中：

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

## 数学

出于网站性能原因，默认情况下不会加载数学功能（由 **[MathJax](https://www.mathjax.org/)** 提供支持）。但它可以通过以下方式启用：

```yaml
---
math: true
---
```

启用数学功能后，您可以使用以下语法添加数学方程：

- 块数学应使用 `$$ math $$` 添加，并且在 `$$` 前后必须有空行
  + 添加方程编号应使用 `$$\begin{equation} math \end{equation}$$`
  + 引用方程编号应在方程块中使用 `\label{eq:label_name} `，在文本中 inline 使用时应使用 `\eqref{eq:label_name}` （请参见下面的示例）
- 内联数学（在行中）应使用 `$$ math $$` 添加，`$$`前后不得有任何空行
- 内联数学（在列表中）应添加 `\$$ math $$` 

```yaml
<!-- 块数学运算，保留所有空行 -->

$$
LaTeX_math_expression
$$

<!-- 公式编号，保留所有空行  -->

$$
\begin{equation}
  LaTeX_math_expression
  \label{eq:label_name}
\end{equation}
$$

Can be referenced as \eqref{eq:label_name}.

<!-- 在行中内联数学，无空行 -->

"Lorem ipsum dolor sit amet, $$ LaTeX_math_expression $$ consectetur adipiscing elit."

<!-- 在列表中内联数学，转义第一个“$” -->

1. \$$ LaTeX_math_expression $$
2. \$$ LaTeX_math_expression $$
3. \$$ LaTeX_math_expression $$
```

> 从 v7.0.0 开始，MathJax 的配置选项已移到文件 `assets/js/data/mathjax.js` ，你可以根据需要更改这些选项，例如添加扩展。  
如果是通过 `chirpy-starter` 构建站点，请从 gem 安装目录复制该文件（使用命令 `bundle info --path jekyll-theme-chirpy` 检查）到您仓库中相同目录。
{: .prompt-tip }

## Mermaid

[**Mermaid**](https://github.com/mermaid-js/mermaid)是一个很棒的图表生成工具。要在帖子中启用它，请将以下内容添加到 YAML 块：

```yaml
---
mermaid: true
---
```

然后，您可以像使用其他 markdown 语言一样使用它：用 ```` ```mermaid ```` 和 ```` ``` ```` 围绕图形代码。可以使用[Mermaid在线编辑器](https://mermaid.live/)轻松创建详细的图表。


## 了解更多信息

有关Jekyll帖子的更多信息，请访问[Jekyll 文档：posts](https://jekyllrb.com/docs/posts/)。

****

本文参考

> 1. [Writing a New Post](https://chirpy.cotes.page/posts/write-a-new-post/)
