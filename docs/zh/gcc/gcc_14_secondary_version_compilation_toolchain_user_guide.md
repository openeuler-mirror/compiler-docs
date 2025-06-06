# 背景介绍

## 简介

为确保操作系统的稳健性，基础软件的选型策略通常倾向于采用经过时间验证、相对稳定的版本，而非最新发布版本。这一策略旨在避免版本更迭带来的潜在不稳定因素，确保在整个长期支持（LTS）周期内，系统版本保持相对稳定。因此，当前 openEuler 在 24.03 LTS 版本整个生命周期都是选择使用 GCC 12.3.1 作为基线进行开发。

这样的选择会带来如下问题。首先，许多的硬件特性需要基础 GCC 工具链的支持，选择非最新版本的 GCC 会导致新特性无法及时在新发布的操作系统上使能。另外，某些用户倾向使用最新版本的编译器使能最新特性，这些特性相较于低版本编译器会带来部分性能提升。

因此，为了使能多样算例新特性，满足不同用户对不同硬件特性支持的需求，在 openEuler 24.09 版本推出 openEuler GCC Toolset 工具链，这是一个专为 openEuler 系统设计的 GCC 多版本编译工具链，该工具链提供一个高于系统主 GCC 版本的副版本 GCC 编译工具链，为用户提供了更加灵活且高效的编译环境选择。通过使用 openEuler GCC Toolset 14 多版本编译工具链，用户可以轻松地在不同版本的 GCC 之间进行切换，以便充分利用新硬件特性，同时享受到 GCC 最新优化所带来的性能提升。

## 方案设计

### 编译工具链功能介绍

GCC 编译工具链是一套由 GNU 开发和维护的开源编译器集合，它是用于将高级语言代码转换为机器语言的工具集，GCC 编译工具链不仅包括GCC编译器本身，还包含一系列辅助工具和库，这些组件共同构成了一个完整的编译环境。

1. GCC 编译器（gcc/g++/gfrotran 等）:

    * 作用：GCC 编译器是工具链的核心，负责完成预处理和编译过程，将源代码转换成汇编代码或中间表示。对于 C++ 代码，g++ 是 GCC 的 C++ 编译器前端，除了完成编译工作外，还会自动链接 C++ 标准库。

2. Binutils 工具集：

    * 包含工具：链接器（ld）、汇编器（as）、目标文件格式查看器（readelf）、符号查看器（nm）、目标文件格式转换工具（objcopy）、反汇编工具（objdump）、尺寸查看工具（size）等。
    * 作用：这些工具在编译过程中起辅助作用，如将汇编代码转换成机器码（汇编器）、将多个目标文件链接成可执行文件（链接器）、查看目标文件或可执行文件的信息（readelf、nm、objdump）等。

3. glibc 库：

    * 作用： glibc 是 GNU C Library 的缩写，是 GNU 组织为 GNU 系统以及 Linux 系统编写的 C 语言标准库。它包含了 C 语言中常用的标准函数，如 printf、malloc 等，是编译 C 语言程序时必不可少的部分。

4. 其他辅助工具：

    * 调试器（gdb）：用于调试可执行文件，帮助开发者定位和解决程序中的错误。
    * 性能分析工具（gprof）：用于分析程序的性能，帮助开发者优化代码。

### 工具链选型

在编译过程中，工具链中的软件组件对编译结果具有直接影响。具体而言，GCC、binutils 以及 glibc是其核心要素。glibc 作为 C 语言标准库，其选型通常与操作系统内核版本紧密绑定，不轻易进行更改。本工具链仅包含 GCC 和 binutils 两款软件来满足副版本编译需求。

当前最新的 GCC release 版本为 gcc-14.2.0，因此 openEuler GCC Toolset 工具链选型 的 GCC 的版本为gcc-14.2.0 。

至于 binutils，openEuler 24.09 的默认 binutils 为 2.41 版本，而最新的 GCC-14 官方推荐搭配 binutils-2.42 使用，因此本工具链的 binutils 的版本选择 binutils-2.42 。

基于此考量，openEuler GCC Toolset 副版本工具链引入 gcc-14.2.0 和 binutils-2.42，此举旨在确保编译环境的稳定性和效率，同时避免不必要的复杂性，力求在保障编译结果质量的同时，优化用户的使用体验。后期待 gcc-14.3.0 在上游社区 release 后，同步更新此工具链 GCC 版本。

### 架构设计

为区分于默认工具链安装，并防止 openEuler GCC Toolset 副版本编译工具链安装与默认编译工具链安装之间的依赖库冲突，将此工具链命名为 gcc-toolset-14 ，其软件包名均以前缀`gcc-toolset-14-`开头，后接原有工具链软件包名。同时，考虑到默认编译工具链安装路径为`/usr`，为避免路径重叠，特将 gcc-toolset-14 安装路径设定为`/opt/openEuler/gcc-toolset-14/`。为了与开源 GCC 做出区分，也便于后期合入更多 openEuler 社区特性，gcc-toolset-14-gcc 的版本设置为 14.2.1 。

副版本编译工具链 gcc-toolset-14 提供的应用程序和库不会取代系统默认GCC版本，其包含的应用程序和库旨在与系统默认编译工具链版本并存，而非取代或覆盖它们，亦不会自动设为默认或首选选项。此外，为了实现低成本切换编译工具链版本，便于版本切换与管理，本方案引入 scl-utils 版本切换工具，具体使用和切换方式见下文。

## 安装与部署

### 软件要求

* 操作系统：openEuler 24.09

### 硬件要求

* Aarch64 / X86_64

### 安装方式

默认 GCC 编译器 gcc-12.3.1，安装路径为 /usr ：

```shell
yum install -y gcc gcc-c++
```

副版本编译工具链 gcc-toolset-14 安装路径为 /opt/openEuler/gcc-toolset-14/root/usr/ ：

```shell
yum install -y gcc-toolset-14-gcc*
yum install -y gcc-toolset-14-binutils*
```

## 使用方式

本方案引入 SCL（Software Collections）工具进行不同版本编译工具链的管理。

## scl工具

SCL（Software Collections）是 Linux 系统中一个非常重要的工具，它为用户提供了一种方便、安全的安装和使用应用程序及运行时环境多个版本的方式，同时避免了系统混乱。

SCL 的主要用途包括：

1. 多版本共存：允许用户在同一系统上安装和使用多个版本的软件库、工具和语言运行环境，从而满足不同应用和开发需求。
2. 避免系统冲突：通过隔离不同版本的软件，避免了新版本软件与系统原有版本之间的冲突。
3. 提升开发效率：对于开发人员来说，SCL 提供了最新的开发工具链和运行时环境，从而提升了开发效率。

### 版本切换方式

**安装 SCL：**

```shell
yum install scl-utils scl-utils-build
```

**注册 gcc-toolset-14：**

```shell
## 注册 gcc-toolset-14
scl register /opt/openEuler/gcc-toolset-14/

## 取消注册 gcc-toolset-14
scl deregister gcc-toolset-14
```

使用 `scl list-collections` 显示 gcc-toolset-14 已经在 scl 中注册成功；

**切换 gcc-toolset-14：**

```shell
scl enable gcc-toolset-14 bash
```

此命令会启动一个新的 bash shell 会话，使用 gcc-toolset-14 内的工具版本，而不是系统默认版本。在新的 bash shell 会话中，无需再显式切换编译器版本和路径。
如果需要退出 gcc-toolset-14 的编译环境，输入 `exit` 退出 bash shell 会话，此时编译工具链的版本切换成系统默认版本。

SCL工具的本质就是自动设置不同工具版本的环境变量，具体可以参考 `/opt/openEuler/gcc-toolset-14/enable` 文档，gcc-toolset-14 的环境变量均在该文件中设置。若用户系统没有 SCL 工具，则可以使用以下方式进行工具链版本切换：

```shell
## 方案一：无 SCL 工具，使用脚本切换编译工具链
source /opt/openEuler/gcc-toolset-14/enable

## 方案二：有 SCL 工具，使用 SCL 工具切换编译工具链并激活运行环境
scl enable gcc-toolset-14 bash
```

## 使用约束

### 编译场景

主版本场景：正常编译使用系统默认的 gcc-12.3.1 进行构建；

副版本场景：需要使用 GCC-14 高版本特性构建相关应用，使用 SCL 工具将 bash 环境切换为 gcc-toolset-14 编译工具链的编译环境。

### 副版本GCC-14使用说明

1. openEuler GCC Toolset 14 副版本编译工具链提供如下两种使用方式：

    1）动态链接：默认场景下会自动添加选项 -lstdc++ 进行动态链接，此时会链接系统库动态库 /usr/lib64/libstdc++.so.6 和 GCC-14 副版本提供的 libstdc++_nonshared.a 静态库，此静态库是 GCC-14 相比于 GCC-12 新增的稳定 C++ 特性；     
    2）静态链接：用户使用选项 -static 进行静态链接，此时会链接 GCC-14 副版本提供的 libstdc++.a 全量特性静态库，该静态库路径为 `/opt/openEuler/gcc-toolset-14/root/usr/lib/gcc/aarch64-openEuler-linux/14/libstdc++.a`。

2. 用户默认构建使用动态链接，会链接新增的 libstdc++_nonshared.a 静态库，该库为了保持和系统兼容性，仅对 C++ 中正式标准特性进行封装。对于 -fmodules-ts ， -fmodule-header 等选项，属于 C++20 的模块特性，而该特性在 C++20 中仍属于实验性质，并未封装在 libstdc++_nonshared.a 中，若用户需要使用此类新增特性，建议直接使用静态链接的方式全量链接 GCC-14 副版本的静态库。
