---
title: Wireshark 源码编译（Windows 10）
author: fnoobt
date: 2024-05-19 17:40:00 +0800
categories: [Network,网络工具]
tags: [network,wireshark,compile]
---

## 编译环境搭建

### 1、推荐：安装 Chocolatey
[Chocolatey](https://chocolatey.org/)是一个Windows的原生包管理器，在[Chocolatey社区包存储库](https://community.chocolatey.org/packages)可以查看支持的包。它支持 Windows 软件包和 Python 包。

[官方安装方法](https://chocolatey.org/install)，在Windows 10上以管理员身份运行打开`Windows PowerShell`，执行如下命令：

```bash
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
```

Chocolatey 倾向于将软件包安装到自己的路径（%ChocolateyInstall%）中，尽管软件包可以自由使用自己的首选项。您可以使用命令 `choco install` （或其简写 `cinst` ）安装 Chocolatey 软件包。
```bash
rem Flex is required.
choco install -y winflexbison3
rem Git, CMake, Python, etc are also required, but can be installed
rem via their respective installation packages.
choco install -y git cmake python3
```

> 在window中`rem`代表注释。
{: .prompt-info }

### 2、安装 Microsoft Visual Studio
下载并安装[Microsoft Visual Studio 2022 Community Edition](https://visualstudio.microsoft.com/zh-hans/thank-you-downloading-visual-studio/?sku=Community&channel=Release&version=VS2022&source=VSLandingPage&cid=2030&passive=false#installvs)。如果您愿意，可以下载并安装[Microsoft Visual Studio 2019 Community Edition](https://visualstudio.microsoft.com/thank-you-downloading-visual-studio/?sku=Community&rel=16)。

需要安装的插件以 Visual Studio 2022为例（Visual Studio 2019也可以参考）。选中`Desktop development with C++`复选框，以下是必选项

- `MSVC …​ VS 2022 C++`项，其中`…​ build tools (Latest)`
- `Windows 10 SDK`
- `C++ CMake tools for Windows`

### 3、安装Qt

#### 3.1 在线安装
Wireshark 主应用程序使用 Qt 窗口化工具包。要安装Qt，请转到[下载Qt](https://www.qt.io/download)页面，选择`Download open source`，然后选择下载`Qt Online Installer`，然后下载`Qt Online Installer for Windows`。执行时，请注册或登录，然后使用“下一步”按钮继续。当系统提示时，选择自定义安装`Custom installation`。

在选择组件`Select Components`页面中，选择所需的Qt版本。我们推荐最新的 LTS 版本，稳定的 Windows 安装程序目前附带 Qt 6.7.1。选择以下组件：

- `MSVC 2019 64-bit`
- `Qt 5 Compatibility Module`
- `Qt Debug Information Files` (包含可用于调试的PDB文件)
- 在 `Additional Libraries`下，选择`Qt Multimedia` 以支持在RTP Player对话框中播放流的高级控件
- 您可以取消选择所有其他组件，例如`Qt Charts`或`Android xxxx`，因为它们不是必需的。

CMake 变量 [CMAKE_PREFIX_PATH](https://doc.qt.io/qt-6/cmake-get-started.html) 应根据您的环境进行设置，并应指向 Qt 安装目录，例如 `C:\Qt\6.7.1\msvc2019_64` 或者您也可以使用环境变量 `WIRESHARK_QT6_PREFIX_PATH`。

Qt 6 是构建 Wireshark 的默认选项，但 Wireshark 支持 Qt 5.12 及更高版本。要使 Wireshark 能够使用 Qt 5 进行构建，请传递 `-DUSE_qt6=OFF` 给 cmake。

#### 3.2 离线安装
在[Qt官方发布](https://download.qt.io/official_releases/qt/)中，选择相应版本，离线安装。

> QT 5.15 之后不再支持直接下载安装包，需下载源码编译安装。
{: .prompt-warning }

### 4、安装 Python
从 [Python官方](https://www.python.org/downloads/) 获取 Python 3 安装程序并安装 Python。其安装位置因安装程序中选择的选项和要安装的 Python 版本而异。以Python3.10为例，常见的安装目录是:
`C:\Users\username\AppData\Local\Programs\Python\Python310`, `C:\Program Files\Python310`, `C:\Python310`。

或者，您可以使用 Chocolatey 安装 Python：
```bash
choco install -y python3
```

> Chocolatey 会自动添加环境变量。
{: .prompt-info }

Chocolatey 可能会在上述位置之一安装 Python，或者可能在 `C:\Tools\Python3` 中安装 Python。

### 5、安装 Git
从 [Git官方](https://git-scm.com/download/win) 获取 Git 安装程序并安装 Git。确保将包含`git.exe`的目录添加到您的环境变量Path中。

或者，您可以使用 Chocolatey 安装 Git：
```bash
choco install -y git
```

如果安装 Windows 版本的 git，请从 Windows 命令提示符中选择`使用 Git`（在 chocolatey 中为 `/GitOnlyOnPath` 选项）。不要选择`从Windows命令提示符中使用 Git 和可选的 Unix 工具`选项（在 chocolatey 中为 `/GitAndUnixToolsOnPath `选项）。

### 6、安装 CMake
虽然构建 Wireshark 需要 CMake，但它可能已作为 Visual Studio 或 Qt 的组件安装。如果是这种情况，您可以跳过此步骤。如果您确实想要或需要安装 CMake，您可以从 [CMake官方](https://cmake.org/download/)获取它。建议将 CMake 安装到默认位置。确保将包含`cmake.exe`的目录添加到您的环境变量Path中。

或者，您可以使用 Chocolatey 安装 CMake：
```bash
choco install -y cmake
```

### 7、安装 Asciidoctor、Xsltproc 和 DocBook
Asciidoctor 可以直接作为 Ruby 脚本运行，也可以通过 Java 包装器 （AsciidoctorJ） 运行。尚不支持 JavaScript 风格 （Asciidoctor.js）。它与 Xsltproc 和 DocBook 结合使用，以生成您正在阅读的文档和用户指南。

您可以使用 Chocolatey 安装 AsciidoctorJ、Xsltproc 和 DocBook。AsciidoctorJ 需要 Java 运行时，并且有很多可供选择。Chocolatey 目前不支持替代包依赖项，包括对 Java 的依赖项。因此，安装 asciidoctorj 软件包不会自动安装 Java 运行时，您必须单独安装一个。

在Windows 10上以管理员身份运行打开`Windows PowerShell`，执行如下命令：
```bash
choco install -y <your favorite Java runtime>
choco install -y asciidoctorj xsltproc docbook-bundle
```

### 8、安装winflexbison
可以从 [winflexbison官方](https://sourceforge.net/projects/winflexbison/)获取 winFlexBison 安装程序并安装到默认位置。确保将包含`win_flex.exe`的目录添加到您的环境变量Path中。

或者，您可以使用 Chocolatey 安装 Winflexbison:
```bash
choco install -y winflexbison3
```

### 9、可选：安装 Perl
如果需要，您可以从 [strawberryperl](http://strawberryperl.com/)或[activestate](https://www.activestate.com/)获取 Perl 安装程序并安装到默认位置。

或者，您可以使用 Chocolatey 安装 Perl:
```bash
choco install -y strawberryperl
# ...或者...
choco install -y activeperl
```

## 获取源码与编译

### 1、安装和准备源代码

> 在你开始为自己的项目编译 Wireshark 源代码之前，最好至少编译和运行一次 Wireshark。
{: .prompt-info }

**下载源代码**：使用命令行或 Git 将 Wireshark 源代码下载到 `C:\Development\wireshark` 中：
```bash
cd C:\Development
git clone https://gitlab.com/wireshark/wireshark.git
```

### 2、打开 Visual Studio 命令提示符
从`开始`菜单中，导航到`Visual Studio 2022`文件夹，然后选择适合要进行的生成的命令提示符，例如，64 位版本的`x64 Native Tools Command Prompt for VS 2022`。

> 所有后续操作都在此命令提示符窗口中进行。
{: .prompt-info }

#### 设置环境变量
使用适合您安装的路径和值设置以下环境变量：
```bash
rem Let CMake determine the library download directory name under
rem WIRESHARK_BASE_DIR or set it explicitly by using WIRESHARK_LIB_DIR.
rem Set *one* of these.
set WIRESHARK_BASE_DIR=C:\Development
rem set WIRESHARK_LIB_DIR=C:\wireshark-x64-libs
rem Set the Qt installation directory
set WIRESHARK_QT6_PREFIX_PATH=C:\Qt\6.7.1\msvc2019_64
rem Append a custom string to the package version. Optional.
set WIRESHARK_VERSION_EXTRA=-YourExtraVersionInfo
# YourExtraVersionInfo 可设置版本信息，比如202405191844
```

可以将设置这些变量添加到要在打开 Visual Studio Tools 命令提示符后运行的批处理文件中，比如`env.bat`。64 位和 32 位构建需要单独的构建目录，创建并跳转到64位构建目录如下所示：
```bash
mkdir C:\Development\wsbuild64
cd C:\Development\wsbuild64
```

### 3、生成构建文件
CMake 用于处理源代码树中的CMakeLists.txt文件，并生成适合您系统的构建文件。

只有在首次创建生成目录时才需要初始生成步骤。后续生成将根据需要重新生成生成文件。

```bash 
cmake -G "Visual Studio 17 2022" -A x64 ..\wireshark
```

根据需要调整 Visual Studio 版本和Wireshark 源代码树的路径。要使用其他生成器，请修改 -G 参数。 cmake -G 列出了所有 CMake 支持的生成器，Wireshark仅支持 Visual Studio，不支持 32 位版本。比如Visual Studio 2019的命令为：
```bash 
cmake -G "Visual Studio 16 2019" -A x64 ..\wireshark
```

在config的过程中，会自动下载依赖包，位于`C:\Development\wireshark-win64-libs`目录。

在 CMake 生成过程结束时，应显示以下内容：
```bash
-- Configuring done
-- Generating done
-- Build files have been written to: C:/Development/wsbuild64
```

如果获得任何其他输出，则环境中存在问题，必须在生成之前进行纠正。检查传递给 CMake 的参数，尤其是 Wireshark 源和环境变量 `WIRESHARK_BASE_DIR` 的 `CMAKE_PREFIX_PATH -G` 选项和路径。

### 4、构建 Wireshark
在Visual Studio cmd窗口，运行以下命令：
```bash 
msbuild /m /p:Configuration=RelWithDebInfo Wireshark.sln
```

生成的可执行程序路径`C:\Development\wsbuild64\run\RelWithDebInfo\Wireshark.exe`，双击打开Wireshark.exe进行测试。

打开`帮助`=>`关于`。如果它显示您的`私有`程序版本，例如：版本 `4.3.0-myprotocol123` 恭喜！您已经编译了自己的 Wireshark 版本！

> 如果在更改某些源文件后由于问题导致编译失败，请尝试通过运行 `msbuild /m /p:Configuration=RelWithDebInfo Wireshark.sln /t:Clean` 然后再次生成构建方案来清理生成文件。
{: .prompt-info }

### 5、调试环境设置
可以使用 Visual Studio 调试器或 WinDbg 进行调试。请参阅有关使用调试器工具的部分。

#### 可选：创建用户和开发者指南
在Visual Studio cmd窗口，运行以下命令：
```bash 
msbuild /m /p:Configuration=RelWithDebInfo docbook\all_guides.vcxproj
```

生成的两种文档developer-guide和user-guide位于`C:\Development\wsbuild64\docbook`目录。

#### 可选：创建 Wireshark 安装程序
注意：在执行以下操作之前，您应该已成功构建 Wireshark。

##### 安装NSIS
需要先安装 [NSIS](https://nsis.sourceforge.io/Main_Page)，安装完毕后设置环境变量Path。

> 请注意，NSIS 的 32 位版本适用于 64 位和 32 位版本的 Wireshark。需要 NSIS 版本 3。

##### 创建安装程序
在Visual Studio cmd窗口，运行以下命令：
```bash 
msbuild /m /p:Configuration=RelWithDebInfo wireshark_nsis_prep.vcxproj
msbuild /m /p:Configuration=RelWithDebInfo wireshark_nsis.vcxproj
```

构建 Wireshark 安装程序。如果对可执行文件进行签名，则应在`wireshark_nsis_prep`和`wireshark_nsis`步骤之间进行签名。若要对安装程序进行签名，应将签名批处理脚本放在路径上。它必须命名为`sign-wireshark.bat`。它应该由 CMake 自动检测，要始终需要签名，请设置 `-DENABLE_SIGNED_NSIS=On` CMake选项。

生成的安装程序位于`C:\Development\wsbuild64\packaging\nsis`目录，在其它机器上安装测试。

## 问题与解决方案

### error C2220: 以下警告被视为错误
在Visual Studio中，将警告视为错误是一个提高代码质量的设置，它强制开发人员解决编译过程中出现的所有警告，以确保代码更加健壮和标准合规。如果遇到`error C2220: 以下警告被视为错误`，修改`CMakeLists.txt`中的`set(WERROR_COMMON_FLAGS "/WX")`为`set(WERROR_COMMON_FLAGS "/WX-")`

- `/WX` 或 `是 (/WX)`：这表示所有警告都被当作错误处理。
- `/WX-` 或 `否 (/WX-)`：这表示警告不会导致编译失败。

### 打包找不到文件
如果在打包的时候遇到以下类型的报错，找不到`androiddump.html`这个文件，但是可以找到`androiddump.exe`文件，这是因为在window中使用的是exe文件，没有html文件。
```yaml
File: "D:\project\wireshark\wsbuild64\run\RelWithDebInfo\androiddump.html" -> no files found.
Usage: File [/nonfatal] [/a] ([/r] [/x filespec [...]] filespec [...] |
/oname=outfile one_file_only)
Error in macro InstallExtcap on macroline 3
Error in script "wireshark.nsi" on line 1150 -- aborting creation process
```

需要将`InstallExtcap`宏定义进行修改，确保它能够正确处理传入的参数，并在指定的路径下找到相应的文件。
```yaml
!macro InstallExtcap EXTCAP_NAME

  SetOutPath $INSTDIR
  #修改 File "${STAGING_DIR}\${EXTCAP_NAME}.html"
  File "${STAGING_DIR}\${EXTCAP_NAME}.exe"
  SetOutPath $INSTDIR\extcap\wireshark
  File "${STAGING_DIR}\extcap\wireshark\${EXTCAP_NAME}.exe"
```

重新打包

****

本文参考

> 1. [在Windows 10中源码编译Wireshark](https://blog.csdn.net/u013098162/article/details/104330514)
> 2. [Windows：使用 Microsoft Visual Studio编译Wireshark](https://www.wireshark.org/docs/wsdg_html_chunked/ChSetupWindows.html)