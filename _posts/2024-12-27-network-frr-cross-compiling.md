---
title:  FRR 交叉编译
author: fnoobt
date: 2024-12-27 10:17:00 +0800
categories: [Network,路由协议]
tags: [network,frr,cross-compiling]
---

FRR 能够交叉编译到许多不同的架构。只要有了足够的工具链，这个过程就相当简单，但在尝试编译 FRR 或其依赖项之前，必须小心谨慎地验证此工具链的正确性。要注意的是构建工具的构建过程中的小疏忽可能会导致问题，会导致这些问题变得难以诊断。

> FRR这种大型软件的依赖项很多，交叉编译的环境搭建比较复杂，选择使用[QEMU安装Ubuntu ARM系统](https://fnoobt.github.io/posts/vps-quem-ubuntu-arm/)可能是一个更好的选择。
{: .prompt-info }

## 官方版本
本教程使用的构建机器是`x86_64-pc-linux-gnu`，目标机器是`aarch64-linux-gnu`。如果您计划运行本教程中的命令，请将其替换为下面的目标元组：
```bash
export HOST_ARCH="aarch64-linux-gnu"
```

### 工具链准备
交叉编译任何程序的第一步是确定程序（FRR） 将要在什么系统上运行。按照自动工具的惯例，将需要运行 FRR 的机器称为“主机（host）”机器，而构建 FRR 的机器将被称为“构建（build）”机器。工具链将安装到构建机器上，并用于构建 FRR 以供主机运行。

对于给定的目标，构建系统的操作系统可能对本地构建交叉编译器有一定的支持，甚至可能提供为目标体系结构上游构建的二进制工具链。在承诺从头开始构建工具链之前，请检查您的包管理器或操作系统文档。

首先需要在构建机器安装构建交叉编译工具链和 FRR 依赖项。
```bash
sudo apt-get update
# 安装必要的工具和依赖
sudo apt-get install -y build-essential git cmake autoconf automake libtool \
                        pkg-config bison flex python3-dev python3-apt gawk \
                        libc-ares-dev install-info libsnmp-dev perl \
                        protobuf-c-compiler libprotobuf-c-dev libcap-dev \
                        libelf-dev libunwind-dev libgrpc++-dev protobuf-compiler-grpc \
                        libsqlite3-dev libzmq5 libzmq3-dev libpcre2-dev \
                        libjson-c-dev libpam0g-dev libsystemd-dev libcap-ng-dev \
                        libxml2-dev libevent-dev libssl-dev texinfo 

# 安装ARM64交叉编译工具链
sudo apt-get install -y gcc-aarch64-linux-gnu g++-aarch64-linux-gnu
```

在[交叉编译依赖项](#交叉编译依赖项)介绍了交叉编译工具链使用**CMake**、**Autotools** 和 **Makefile** 编译包的一般说明；这三种情况适用于几乎所有的 FRR 依赖项。

> 确保 C 标准库（glibc 或其他库）的版本和实现与主机和构建工具链匹配。`ldd --version` 会在这里为您提供帮助。如果它们不匹配，请升级其中一个。
{: .prompt-warn }

### 测试工具链
在开始任何交叉编译之前，应该通过编写、编译和链接一个简单程序来测试新的工具链。
```bash
# 一个小程序
cat > nothing.c <<EOF
int main() { return 0; }
EOF

# 使用交叉编译器构建和链接
${HOST_ARCH}-gcc -o nothing nothing.c

# 检查生成的二进制文件，结果可能有所不同
file ./nothing

# ./nothing: ELF 64-bit LSB pie executable, ARM aarch64, version 1 (SYSV), 
# dynamically linked, interpreter /lib/ld-linux-aarch64.so.1, 
# BuildID[sha1]=b300dacd1d046bd148f43e92e4d8edfb23442271, for GNU/Linux 3.7.0, not stripped
```

如果这没有产生任何错误，则已安装的工具链可能已准备好开始编译构建依赖项并最终编译 FRR 本身。虽然可能仍然存在潜在问题，但从根本上来说，工具链可以生成二进制文件，这足以开始使用它。

> 如果在上一次功能测试期间发生任何错误，请回顾并解决它们，然后再继续；因为这表明您的交叉编译工具链无法构建 FRR 或其依赖项。即使一切正常，请记住，从现在开始的许多错误可能仍与您的工具链有关（例如 libstdc++.so 或其他组件），并且这个小测试不能保证完整的工具链一致性。
{: .prompt-warn }

### 交叉编译依赖项
编译 FRR 时，需要在构建机器上同时编译其依赖项。这样，共享库中的符号（将在运行时加载到主机上）就可以在编译时链接到 FRR 二进制文件；此外，在编译阶段需要这些库的标头才能成功构建。

ubunut国内官方源配置网站：

1. [清华源](https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/)
2. [中科大源](https://mirrors.ustc.edu.cn/help/ubuntu)
3. [阿里源](https://developer.aliyun.com/mirror/ubuntu)

#### Sysroot概述
所有构建依赖项都应安装到构建计算机上的“根”目录中，以下称为“sysroot”。在构建过程中搜索必需的库和标头时，此目录将作为路径的前缀。通常，这可以在构建依赖包时通过`--prefix`标志进行设置，这意味着`make install`会将编译的库复制到（例如） `/usr/${HOST_ARCH}/usr` 。

如果工具链是在构建机器上构建的，那么可能已经有一个安装了这些工具和标准库的 sysroot；将该目录用作此构建的 sysroot 也可能会有所帮助。

#### 基本工作流程
在编译或构建任何依赖项之前，请记下目标守护程序以及需要哪些库。如果仅使用守护程序的子集进行构建，则并非所有依赖项都是必需的。

1. **CMake**：对于使用CMake的库（例如 `json-c`和`libyang`），编译和安装工作流程如下：
```bash
mkdir build
cd build
CC=${HOST_ARCH}-gcc \
CXX=${HOST_ARCH}-g++ \
cmake \
    --install-prefix /usr/${HOST_ARCH} \
    ..
make
sudo make install
```

1. **Autotools**：对于使用Autotools的库，编译和安装工作流程如下。生成的库将安装到 sysroot 的 `/usr/${HOST_ARCH}` 中。
```bash
./configure \
   CC=${HOST_ARCH}-gcc \
   CXX=${HOST_ARCH}-g++ \
   --build=${HOST_ARCH} \
   --prefix=/usr/${HOST_ARCH}
make
sudo make install
```

1. **Makefile**：对于仅具有 Makefile 的程序（例如 `libcap`），过程可能略有不同：
```bash
# CC=${HOST_ARCH}-gcc make 相当于运行以下两个命令
# export CC=${HOST_ARCH}-gcc
# make
CC=${HOST_ARCH}-gcc make
sudo make install DESTDIR=/usr/${HOST_ARCH}
```

这三个工作流程应该可以处理 FRR 的大部分构建和安装构建时依赖项。验证已安装的文件是否正确放入 sysroot 中，并且实际上是使用交叉编译工具链构建的，而不是由本机工具链意外构建的。

#### 依赖项说明
交叉编译期间可能会出现很多问题。下面收集了一些较常见的错误和一些特殊注意事项以供参考。

##### libyang
应在 CMake 步骤期间提供`-DENABLE_LYD_PRIV=ON` 。

还要确保安装的 `libyang` 版本与目标 FRR 版本所需的版本相对应。
```bash
git clone https://github.com/CESNET/libyang.git
cd libyang
git checkout v2.1.128
mkdir build; cd build
CC=${HOST_ARCH}-gcc \
CXX=${HOST_ARCH}-g++ \
cmake --install-prefix /usr/${HOST_ARCH} \
      -D CMAKE_BUILD_TYPE:String="Release" ..
make && sudo make install
```

##### gRPC
仅当 `--enable-grpc` 标志稍后将传递给 FRR 时，此部分才是必需的。如果构建机器上的协议版本与 gRPC 构建期间链接到的协议版本不同，则在编译 gRPC 时可能会出现问题。此缺陷的错误消息如下所示：
```yaml
gens/src/proto/grpc/channelz/channelz.pb.h: In member function ‘void grpc::channelz::v1::ServerRef::set_name(const char*, size_t)’:
gens/src/proto/grpc/channelz/channelz.pb.h:9127:64: error: ‘EmptyDefault’ is not a member of ‘google::protobuf::internal::ArenaStringPtr’
 9127 |   name_.Set(::PROTOBUF_NAMESPACE_ID::internal::ArenaStringPtr::EmptyDefault{}, ::std::string(
```

发生这种情况的原因是，协议缓冲区代码生成使用 `protoc` 创建具有不同 getter 和 setter 的类，这些类与源树的 `.proto `文件定义的 protobuf 数据相对应。显然，交叉编译的 `protoc` 不能用于此代码生成，因为该二进制文件是为不同的 CPU 构建的。

解决方案是安装匹配本机和交叉编译的协议缓冲区的版本；这样，本机二进制文件将生成代码，交叉编译库将通过 gRPC 链接到，并且这些版本不会不一致。

如果使用 `libstdc++`，那么 `-latomic` 链接器标志可能也是必要的，因为 GCC 的 C++11 实现在某些情况下会对 `<atomic>` 进行库调用，所以不能假定 `-latomic`。

### 交叉编译 FRR 本身
下载frr，并切换到对应的版本
```bash
# 克隆并启动构建
git clone 'https://github.com/FRRouting/frr.git'
cd frr
# 切换版本
git checkout stable/10.2
```

所有必要的库都交叉编译并安装到 sysroot 中后，最后要实际构建的是 FRR 本身
```bash
# 运行 autoreconf 以生成 configure 文件
./bootstrap.sh

# 使用本机工具链构建 clippy
mkdir build-clippy
cd build-clippy
../configure --enable-clippy-only
make clippy-only
cd ..

# 接下来，配置 FRR 并使用我们刚刚构建的 clippy
./configure \
   CC=${HOST_ARCH}-gcc \
   CXX=${HOST_ARCH}-g++ \
   --host=${HOST_ARCH} \
   --with-sysroot=/usr/${HOST_ARCH} \
   --with-clippy=./build-clippy/lib/clippy \
   --sysconfdir=/etc \
   --localstatedir=/var \
   --sbindir="\${prefix}/lib/frr" \
   --prefix=/usr/ \
   --enable-user=frr \
   --enable-group=frr \
   --enable-vty-group=frrvty \
   --disable-doc \
   --enable-grpc

# 构建
make
```

> 如果构建 clippy 时提示 source directory already configured，需要在frr源码目录使用`make distclean`命令清理原有的配置。
{: .prompt-info }

### 安装到主机
如果在前面的步骤中没有发现任何错误，则可以安全地`make install`放入其自己的目录中。
```bash
# 安装 FRR 自己的“sysroot”
make install DESTDIR=/some/path/to/sysroot
```

运行上述命令后，FRR 二进制文件、模块和示例配置文件将安装到构建机器上的某个路径中。该目录将包含`/usr`和`/etc`等文件夹；现在应该将该“root”复制到主机并安装在根目录之上。
```bash
# 将此 sysroot 打包成 Tar（保留权限）
tar -C /some/path/to/sysroot -cpvf frr-${HOST_ARCH}.tar .

# 将 tar 文件传输到主机
scp frr-${HOST_ARCH}.tar me@host-machine:

# 将打包好的 sysroot 覆盖在主机根目录之上
ssh me@host-machine <<-EOF
   # 可能需要在此处提升权限
   tar -C / -xpvf frr-${HOST_ARCH}.tar.gz .
EOF
```

现在应该安装 FRR，就像在主机上运行`make install`一样。创建配置文件并根据需要分配权限。最后，确保主机上存在正确的 FRR 用户和组。

#### 故障排除
即使采取了一切预防措施，仍然可能会出错！本节详细介绍了一些常见的运行时问题。

##### 库不匹配
如果在主机上安装后看到类似以下内容：
```bash
/usr/lib/frr/zebra: error while loading shared libraries: libyang.so.1: cannot open shared object file: No such file or directory
```

… 至少一个先前链接到二进制文件的 FRR 依赖项在主机操作系统上不可用。即使已安装，主机存储库的版本也可能落后于较新的 FRR 构建所需的版本（目前 YANG 很可能会发生这种情况）。

如果主机操作系统包管理器中没有匹配的库，则可以使用用于编译 FRR 的相同工具链来编译它们。在构建机器上编译 FRR 时，该库可能已经构建完毕，在这种情况下，它可能就像遵循[安装到主机](#安装到主机)期间布置的相同工作流程一样简单。

##### Glibc 版本不匹配
C 标准库的版本和实现必须在主机和构建工具链上匹配。与此错误配置相对应的错误将如下所示：
```bash
/usr/lib/frr/zebra: /lib/${HOST_ARCH}/libc.so.6: version `GLIBC_2.32' not found (required by /usr/lib/libfrr.so.0)
```

请参阅之前有关防止[glibc 不匹配](https://docs.frrouting.org/projects/dev-guide/en/latest/cross-compiling.html#glibc-mismatch)的警告。

## 关键流程
### 1. “构建（build）”机器
```bash
# 1.安装aarch64交叉编译工具链
sudo apt update
sudo apt-get upgrade
sudo apt install -y gcc-aarch64-linux-gnu g++-aarch64-linux-gnu

# 2.安装 Ubuntu 的构建依赖项
sudo apt-get install \
            git autoconf automake libtool make libreadline-dev texinfo \
            pkg-config libpam0g-dev libjson-c-dev bison flex \
            libc-ares-dev python3-dev python3-sphinx \
            install-info build-essential libsnmp-dev perl \
            libcap-dev python2 libelf-dev libunwind-dev \
            libyang2 libyang2-dev

# 3.设置交叉编译环境变量
export CROSS_COMPILE=aarch64-linux-gnu-
export HOST_ARCH="aarch64-linux-gnu"

# 4.准备交叉编译库的目录
mkdir -p /usr/local/${HOST_ARCH}
export PREFIX=/usr/local/${HOST_ARCH}

# 5.交叉编译并安装依赖项
# 需要交叉编译所并安装所有frr的依赖项，下面以libyang为例
# libyang
git clone https://github.com/CESNET/libyang.git
cd libyang
git checkout v2.1.128
mkdir build; cd build
CC=${HOST_ARCH}-gcc \
CXX=${HOST_ARCH}-g++ \
cmake --install-prefix $PREFIX CMAKE_INSTALL_PREFIX=$PREFIX \
      -D CMAKE_TOOLCHAIN_FILE=../cmake/toolchain/aarch64-linux-gnu.cmake \
      CMAKE_BUILD_TYPE:String="Release" ..
make && sudo make install

# 下载 FRR 源代码
git clone https://github.com/FRRouting/frr.git
cd frr

# 运行 autoreconf 以生成 configure 文件
./bootstrap.sh

mkdir -p build
cd build
../configure \
    --host=${HOST_ARCH} \
    --prefix=/usr/kd/${HOST_ARCH} \
    --includedir="\${prefix}/include" \
    --bindir="\${prefix}/bin" \
    --sbindir="\${prefix}/lib/frr" \
    --libdir="\${prefix}/lib/frr" \
    --libexecdir="\${prefix}/lib/frr" \
    --with-pkg-git-version \
    --enable-user=frr \
    --enable-group=frr \
    --enable-vty-group=frrvty \
    --enable-vtysh \
    --enable-logging \
    --disable-doc

# 根据系统的 CPU 核心数来并行编译
make -j$(nproc)

# FRR 安装到之前指定的路径
sudo make install
```

frr configure配置：
- `--host=aarch64-linux-gnu`: 设置交叉编译目标架构为 ARM64。
- `--prefix`: 设置安装路径（安装到 ARM64 架构的路径）。
- `--with-pkg-git-version`: 启用 Git 版本信息。
- `--enable-vtysh`: 启用 vtysh。
- `--enable-logging`: 启用日志功能。
- `--disable-doc`: 禁用文档生成（如果不需要文档，可以禁用以加快编译速度）。

### 2. “主机（host）”机器
```bash
# 使用 SCP 或其他文件传输工具，将编译后的文件传输到 ARM64 机器上
scp -r /usr/kd/${HOST_ARCH} username@arm64-ip:/path/to/destination

# 启动 FRR
/path/to/destination/sbin/frr
```

> 注意：请确保所有必需的库也已经在 ARM64 机器上安装，并且环境变量设置正确。如果目标 ARM64 机器上缺少某些库，需要在 ARM64 环境中安装对应的依赖包。
{: .prompt-info }

****

本文参考

> 1. [FRR 交叉编译](https://docs.frrouting.org/projects/dev-guide/en/latest/cross-compiling.html)