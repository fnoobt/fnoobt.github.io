---
title: 开始
author: fnoobt
date: 2019-08-09 20:55:00 +0800
categories: [Blog,博客教程]
tags: [开始]
media_subpath: '/assets/img/commons/demo'
description: 了解如何使用 Chirpy 的基础知识，通过这个全面的概述开始。你将学习如何安装、配置并使用你的第一个基于 Chirpy 的网站，以及如何将其部署到 web 服务器上。
---

## 创建站点仓库

有两种方法可以为此主题创建新的存储库：

- [**使用Chirpy启动器**](#选项-1-使用chirpy启动器) - 易于升级，隔离不相关的项目文件，以便您可以专注于编写。
- [**GitHub Fork**](#选项-2-github-fork) - 方便自定义开发，但难以升级。除非您熟悉 Jekyll 并决心调整或为该项目做出贡献，否则不建议使用此方法。

### 选项 1. 使用Chirpy启动器

1. 登录到 GitHub 并导航到 [**Chirpy Starter**][starter]。
2. 单击按钮 <kbd>Use this template</kbd> > <kbd>Create a new repository</kbd>。
3. 将新存储库命名为 `USERNAME.github.io`, 其中 `USERNAME` 表示您的github用户名（小写）。

### 选项 2. GitHub Fork

1. 登录到GitHub
2. [fork **Chirpy**](https://github.com/cotes2020/jekyll-theme-chirpy/fork),
3. 将新的仓库重命名为 `USERNAME.github.io`，用你的小写 GitHub 用户名替换  `USERNAME`。

## 设置环境
创建好仓库后，是时候设置你的开发环境了。主要有两种方法：

### 使用 Dev Containers（推荐用于 Windows）
Dev Containers 提供一个使用 Docker 的隔离环境，这可以防止与系统发生冲突，并确保所有依赖项都在容器内进行管理。

1. 安装 Docker：
 - 在 Windows/macOS 上安装 [Docker Desktop](https://www.docker.com/products/docker-desktop/)。
 - 在 Linux 上安装 [Docker Engine](https://docs.docker.com/engine/install/)。
2. 安装 [VS Code](https://code.visualstudio.com/) 并添加 [Dev Containers 扩展](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)。
3. 克隆你的仓库：
 - 对于 Docker Desktop：启动 VS Code 并[在容器卷中克隆你的仓库](https://code.visualstudio.com/docs/devcontainers/containers#_quick-start-open-a-git-repository-or-github-pr-in-an-isolated-container-volume)。
 - 对于 Docker Engine：在本地克隆你的仓库，然后通过 VS Code  [在容器中打开它](https://code.visualstudio.com/docs/devcontainers/containers#_quick-start-open-an-existing-folder-in-a-container)。
4. 等待 Dev Containers 设置完成。

### 本地设置（推荐用于类 Unix 操作系统）
对于类 Unix 系统，你可以本地设置环境以获得最佳性能，尽管你也可以使用 Dev Containers 作为替代方案。
1. 按照 [Jekyll 安装指南](https://jekyllrb.com/docs/installation/)安装 Jekyll 并确保已安装 [Git](https://git-scm.com/)。
2. 将您的仓库克隆到本地机器。
3. 如果您克隆了主题，请安装 Node.js 并在根目录中运行 `bash tools/init.sh` 以初始化仓库。
4. 在你的仓库根目录运行命令 `bundle` 以安装依赖。

`bash tools/init.sh`命令的作用如下:

1. 查看 [最新标签][latest-tag]的代码（以确保网站的稳定性：因为默认分支的代码正在开发中）。
2. 删除非必要的示例文件，并处理与GitHub相关的文件。
3. 构建JavaScript文件并导出到`assets/js/dist/`{: .filepath }，然后Git跟踪它们。
4. 自动创建一个新的提交以保存上面的更改。

> 如果您不想在 GitHub 页面上部署站点，请在`bash tools/init.sh`命令末尾附加选项`--no-gh`。
{: .prompt-info }

## 使用方法
### 启动 Jekyll 服务器
要在本地运行站点，请使用以下命令:

```console
$ bundle exec jekyll s
```

如果在 Docker 中上述的命令不可用，可以尝试以下命令：

```console
$ docker run -it --rm \
    --volume="$PWD:/srv/jekyll" \
    -p 4000:4000 jekyll/jekyll \
    jekyll serve
```

> 如果你使用了 Dev Containers，必须在 **VS Code** 终端中运行该命令。
{: .prompt-info }

几秒钟后，本地服务将在 ***127.0.0.1:4000*** 发布.

### 配置
根据需要更新`_config.yml`{: .filepath}的变量。一些典型的选项包括：

- `url`
- `avatar`
- `timezone`
- `lang`

### 社交联系方式
社交联系选项显示在侧边栏的底部。您可以在`_data/contact.yml`{: .filepath}文件中启用或禁用特定的联系人。

### 自定义样式表

如果您需要自定义样式表，请将主题的 `assets/css/style.scss`{: .filepath} 复制到Jekyll网站上的同一路径，然后在其末尾添加自定义样式。

从版本 `4.1.0` 开始，如果您想覆盖在 `_sass/addon/variables.scss`{: .filepath} 中定义的SASS变量，请将SASS主文件 `_sass/jekyll-theme-chirpy.scss`{: .filepath} 复制到站点源中的`_sass`{: .filepath} 目录中，然后创建一个新文件`_sass/variables-hook.scss`{: .filepath} ，并分配新值。

### 自定义静态资源

在`5.1.0`版本中引入了静态资源配置。静态资源的CDN由文件 `_data/origin/cors.yml`{: .filepath } 定义，您可以根据网站发布地区的网络条件替换其中的一些。

此外，如果您想自托管静态资源，请参阅[_chirpy-static-assets_](https://github.com/cotes2020/chirpy-static-assets#readme)。

## 部署

在部署开始之前，请检查文件 `_config.yml`{: .filepath} ，并确保`url`配置正确。此外，如果您更喜欢[**project site**](https://help.github.com/en/github/working-with-github-pages/about-github-pages#types-of-github-pages-sites)并且不使用自定义域，或者您想使用**GitHub Pages**以外的web服务器上的基本URL访问您的网站，请记住将 `baseurl` 更改为以斜杠开头的项目名称，例如 `/project-name` 。

现在，您可以选择 _以下方法之一_ 来部署您的 Jekyll 站点。

### 使用 GitHub 操作进行部署

准备以下内容：

- 如果您使用的是 GitHub 免费计划，请确保您的站点仓库是公开的。
- 如果您已将 `Gemfile.lock`{: .filepath} 提交到存储库，并且本地机器不是运行 Linux 的系统，请转到站点的根目录并更新锁定文件的平台列表：

```console
$ bundle lock --add-platform x86_64-linux
```

接下来，配置页面服务。

1. 在GitHub上浏览到您的存储库。选择选项卡 _Settings_ ，然后单击左侧导航栏中的 _Pages_ 。然后，在 **Source** 部分（在 _Build and deployment_ 下），从下拉菜单中选择[**GitHub Actions**][pages-workflow-src]。
![Build source](pages-source-light.png){: .light .border .normal w='375' h='140' }
![Build source](pages-source-dark.png){: .dark .normal w='375' h='140' }

2. 将任何提交推送到GitHub以触发 _Actions_ 工作流。在存储库的 _Actions_ 选项卡中，您应该看到工作流 _Build 和 Deploy_ 正在运行。一旦构建完成并成功，站点将自动部署。

此时，您可以转到 GitHub 指示的 URL 以访问您的网站。

### 手动构建和部署

在自托管服务器上，您无法享受 **GitHub Actions** 的便利。因此，需要在本地计算机上生成站点，然后将站点文件上载到服务器。

转到源项目的根目录，并按以下命令构建你的站点：

```console
$ JEKYLL_ENV=production bundle exec jekyll b
```

如果在 Docker 中上述的命令不可用，可以尝试以下命令构建：

```console
$ docker run -it --rm \
    --env JEKYLL_ENV=production \
    --volume="$PWD:/srv/jekyll" \
    jekyll/jekyll \
    jekyll build
```

除非指定了输出路径，否则生成的站点文件将放置在项目根目录的文件夹 `_site`{: .filepath} 中。现在，您应该将这些文件上传到目标服务器。

****

本文参考

> 1. [Getting Started](https://chirpy.cotes.page/posts/getting-started/)

[nodejs]: https://nodejs.org/
[starter]: https://github.com/cotes2020/chirpy-starter
[pages-workflow-src]: https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site#publishing-with-a-custom-github-actions-workflow
[latest-tag]: https://github.com/cotes2020/jekyll-theme-chirpy/tags