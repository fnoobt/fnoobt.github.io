---
title: 自定义图标
author: cotes
date: 2019-08-11 09:49:00 +0800
categories: [Blog,博客教程]
tags: [图标]
---

[**Chirpy**](https://github.com/cotes2020/jekyll-theme-chirpy/) 的[图标](https://www.favicon-generator.org/about/)放在目录 `assets/img/favicons/`{: .filepath} 中。您可能希望将它们替换为自己的。以下部分将指导您创建和替换默认网站图标。

## 生成图标

准备一个大小为512x512或更大的方形图像（PNG、JPG或SVG），然后转到在线工具[**真实图标生成器**](https://realfavicongenerator.net/)并单击按钮<kbd>Select your Favicon image</kbd>上传您的图像文件。

在下一步中，网页将显示所有使用场景。您可以保留默认选项，滚动到页面底部，然后单击按钮<kbd>Generate your Favicons and HTML code</kbd>以生成网站图标。

## 下载和替换

下载生成的包，解压缩并从提取的文件中删除以下两个：

- `browserconfig.xml`{: .filepath}
- `site.webmanifest`{: .filepath}

然后复制剩余的图像文件(`.PNG`{: .filepath}和`.ICO`{: .filepath})，以覆盖Jekyll网站的目录`assets/img/favicons/`{: .filepath}中的原始文件。如果你的Jekyll网站还没有这个目录，只需创建一个。

下表将帮助您了解对图标文件的更改：

| 文件                | 从在线工具                          | 从 Chirpy |
|---------------------|:---------------------------------:|:-----------:|
| `*.PNG`             | ✓                                 | ✗           |
| `*.ICO`             | ✓                                 | ✗           |

>  ✓ 表示保留，✗ 表示删除。
{: .prompt-info }

下次构建网站时，网站图标将替换为自定义版本。

****

> [原文出处](https://chirpy.cotes.page/posts/customize-the-favicon/)