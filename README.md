<div align="center">

  # Chirpy 启动器

  一个Jekyll主题，记录学习博客。

  [![Gem Version](https://img.shields.io/gem/v/jekyll-theme-chirpy)](https://rubygems.org/gems/jekyll-theme-chirpy) [![GitHub license](https://img.shields.io/github/license/cotes2020/chirpy-starter.svg?color=blue)][mit]

  [**Live Demo →**][demo]

</div>

## 特征

<details>
  <summary>
    <i>单击以查看功能</i>
  </summary>
  <p>

  - 深色/浅色主题模式
  - 本地化的UI语言
  - 置顶帖子
  - 分层类别
  - 趋势标签
  - 目录
  - 帖子的最后修改日期
  - 语法突出显示
  - 数学表达式
  - 美人鱼图和流程图
  - 暗/亮模式图像
  - 嵌入视频
  - 话语/话语/话语评论
  - 搜索
  - 原子源
  - 谷歌分析
  - SEO和性能优化
  
  </p>
</details>

## 主题

当通过[RubyGems.org][gem]安装[**Chirpy**][chirpy]主题时，Jekyll只能读取文件夹`/_data`、`/_layouts`、`/includes`、`/_sass`和`/assets`中的文件，以及主题的gem中`/_config.yml`文件的一小部分选项。如果您曾经安装过这个主题gem，则可以使用该命令`bundle info --path jekyll-theme-chirpy`来定位这些文件。

Jekyll团队声称，这是为了将球留在用户的球场上，但这也导致用户在使用功能丰富的主题时无法享受开箱即用的体验。

要充分使用**Chirpy**的所有功能，您需要将主题的gem中的其他关键文件复制到您的Jekyll网站。以下是目标列表：

```shell
.
├── _config.yml
├── _plugins
├── _tabs
└── index.html
```

为了节省您的时间，也为了防止您在复制时丢失一些文件，我们将最新版本的**Chirpy**主题和[CD][CD]工作流的文件/配置提取到此处，以便您可以在几分钟内开始写作。

## 先决条件

按照[Jekyll Docs](https://jekyllrb.com/docs/installation/)中的说明完成基本环境的安装。还需要安装[Git](https://git-scm.com/)。

## 安装

登录GitHub并[**使用此模板**][use-template]生成一个全新的存储库，并将其命名为`USERNAME.github.io`，其中`USERNAME`表示您的GitHub用户名。

然后将其克隆到本地计算机并运行：

```
$ bundle
```

本地运行预览命令 `bundle exec jekyll s` 或 `jekyll server`.

## 用法

请参阅 [主题文档](https://github.com/cotes2020/jekyll-theme-chirpy#documentation).

## 许可证

本作品在 [MIT][mit] 许可下发布。

[demo]: https://fnoobt.github.io
[gem]: https://rubygems.org/gems/jekyll-theme-chirpy
[chirpy]: https://github.com/cotes2020/jekyll-theme-chirpy/
[use-template]: https://github.com/cotes2020/chirpy-starter/generate
[CD]: https://en.wikipedia.org/wiki/Continuous_deployment
[mit]: https://github.com/cotes2020/chirpy-starter/blob/master/LICENSE
