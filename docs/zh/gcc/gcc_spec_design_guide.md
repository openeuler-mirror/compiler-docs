# GCC SPEC 设计指导

## 1. 概述

本文档详细描述了多版本 GCC 14 RPM 打包规格文件（[gcc\-14.spec](https://gitcode.com/src-openeuler/gcc-14/blob/openEuler-24.03-LTS-SP3/gcc-14.spec)）的设计架构和各个功能模块。该 SPEC 文件用于构建 GCC14 （GNU Compiler Collection 第14版本）的 RPM 软件包，支持多种架构（x86、ARM、RISC\-V、PowerPC 等）和多语言（C、C++、Fortran、Objective\-C 等）。同时可用于提供其他多版本GCC的SPEC设计参考。

***

## 2. 架构总览

```scss
┌─────────────────────────────────────────────────────────────────┐
│                    GCC 14 SPEC 架构                             │
├─────────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐             │
│  │  全局配置    │  │  条件编译   │  │  补丁管理   │             │
│  │  (Macros)   │  │  (Condition)│  │  (Patches)  │             │
│  └─────────────┘  └─────────────┘  └─────────────┘             │
├─────────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐             │
│  │  软件包定义  │  │  构建流程   │  │  安装规则   │             │
│  │ (Packages)  │  │   (Build)   │  │  (Install)  │             │
│  └─────────────┘  └─────────────┘  └─────────────┘             │
├─────────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐                              │
│  │  文件列表   │  │  脚本与触发 │                              │
│  │  (Files)    │  │  (Scripts)  │                              │
│  └─────────────┘  └─────────────┘                              │
└─────────────────────────────────────────────────────────────────┘
```

***

## 3. 模块详细说明

### 3.1 全局配置模块 \(Global Macros\)

**位置**: 文件开头到第 ~300 行

**功能**: 定义整个 SPEC 文件使用的全局变量和宏，控制编译行为和特性开关。

**关键配置项**:

| 宏名称       | 说明                     | 典型值     | 说明                 |
| ------------ | ------------------------ | ---------- | -------------------- |
| `sys_gcc_14` | 系统 GCC 开关            | 条件启用   | 是否编译为系统 gcc   |
| `scl`        | Software Collection 模式 | 0/1        | 是否编译为副版本 gcc |
| `vendor`     | 供应商名称               | openEuler  |                      |
| `gcc_major`  | GCC 主版本号             | 14         |                      |
| `gcc_ver`    | GCC 完整版本             | 空/14      |                      |
| `binsuffix`  | 二进制后缀               | 空         |                      |
| `build_ada`  | Ada 支持                 | 0 \(禁用\) |                      |
| `build_objc` | Objective\-C 支持        | 条件启用   |                      |
| `build_go`   | Go 支持                  | 0 \(禁用\) |                      |
| `build_d`    | D 语言支持               | 0 \(禁用\) |                      |

**架构特定配置**:

```rpm-spec
# 各架构支持的功能库
build_libquadmath   # x86/x86_64/PowerPC 支持
build_libasan       # 地址消毒器
build_libhwasan     # 硬件辅助地址消毒器
build_libtsan       # 线程消毒器
build_liblsan       # 内存泄漏消毒器
build_libubsan      # 未定义行为消毒器
build_libatomic     # 原子操作库
build_libitm        # 事务内存库
```

**路径配置 \(SCL 模式\)**:

```rpm-spec
_scl_prefix   = gcc-toolset-14-
_scl_path     = /opt/openEuler/gcc-toolset-14
_scl_root     = /opt/openEuler/gcc-toolset-14/root
_prefix       = /opt/openEuler/gcc-toolset-14/root/usr
```

***

### 3.2 条件编译模块 \(Conditional Compilation\)

**位置**: 分散在宏定义区域 \(~100\-300 行\)

**功能**: 根据目标架构和构建条件启用/禁用特定功能。

**主要条件块**:

```rpm-spec
# SCL 模式条件
%if 0%{?scl:1}
  # SCL 特定配置
%else
  # 非 SCL 配置
%endif

# 架构特定条件
%ifarch x86_64
  # x86_64 特定配置
%endif

%ifarch aarch64
  # ARM64 特定配置
%endif

%ifarch riscv64
  # RISC-V 特定配置
%endif
```

**多架构支持表**:

| 架构          | 多库支持  | 特殊配置            |
| ------------- | --------- | ------------------- |
| x86\_64       | 32/64位   | i686 multilib       |
| aarch64       | 仅64位    | lp64 ABI            |
| riscv64       | lp64d ABI | disable libquadmath |
| ppc64/ppc64le | 32/64位   | PowerPC multilib    |
| s390x         | 31/64位   | s390 multilib       |

***

### 3.3 补丁管理模块 \(Patch Management\)

**位置**: ~350\-700 行

**功能**: 定义应用到上游 GCC 源代码的补丁文件，用于添加功能、修复bug或适配特定平台。

**补丁分类**:

```rpm-spec
# 核心功能补丁 (1000-1100)
Patch1001: GCC14-1001-libstdc++-compat.patch
Patch1002: GCC14-1002-change-gcc-version.patch
Patch1004: GCC14-1004-riscv-lib64.patch
Patch1005: GCC14-1005-libstdc-compat-Update-symbol-list-for-RISC-V-64.patch
Patch1006: GCC14-1006-Add-multi-version-lto-symbol-parse-cross-lto-units-i.patch
Patch1007: GCC14-1007-Add-hip09-machine-discribtion.patch

# RISC-V 支持补丁 (1008-1100)
Patch1008: 0001-RISC-V-Add-basic-Zaamo-and-Zalrsc-support.patch
Patch1009: 0002-RISC-V-Use-widening-shift-for-scatter-gather-if-appl.patch
...
Patch1100: 0093-RISC-V-Update-testsuite-to-use-b.patch

# x86 架构优化补丁 (1101-1121)
Patch1109: 0109-i386-Remove-CLDEMOTE-for-clients.patch
Patch1110: 0110-i386-Remove-KEYLOCKER-related-feature-since-Panther-.patch
...
Patch1121: 0121-i386-Support-C-template-parameters-in-AMX-intrinsics.patch
```

**补丁应用 \(在 %prep 段\)**:

```rpm-spec
%prep
%autosetup -p1 -n gcc-14.3.0
```

***

### 3.4 软件包定义模块 \(Package Definitions\)

**位置**: ~750\-1200 行

**功能**: 定义 RPM 构建过程中生成的各个子软件包，包括依赖关系、描述信息等。

**主要软件包结构**:

```scss
gcc-14 (主包)
├── gcc-c++-14 (C++ 编译器)
├── cpp-14 (C 预处理器)
├── libgcc-14 (GCC 运行时库)
├── gcc-gfortran-14 (Fortran 编译器)
├── gcc-objc-14 (Objective-C 编译器)
├── gcc-objc++-14 (Objective-C++ 编译器)
├── libstdc++-14 (C++ 标准库)
├── libgfortran-14 (Fortran 运行时库)
├── libobjc-14 (Objective-C 运行时库)
├── libgomp-14 (OpenMP 运行时库)
├── libquadmath-14 (四精度数学库)
├── libitm-14 (事务内存库)
├── libatomic-14 (原子操作库)
├── libasan-14 (地址消毒器)
├── libtsan-14 (线程消毒器)
├── libubsan-14 (未定义行为消毒器)
├── liblsan-14 (内存泄漏消毒器)
├── libhwasan-14 (硬件辅助地址消毒器)
├── libgccjit-14 (GCC JIT 编译器)
├── gcc-plugin-14-devel (GCC 插件开发)
└── gdb-plugin (GDB 插件)
```

**典型软件包定义示例**:

```rpm-spec
%package -n %{?_scl_prefix}gcc-c++%{gcc_ver}
Summary: C++ support for GCC
Requires: %{?_scl_prefix}gcc%{gcc_ver} = %{version}-%{release}
Requires: %{?_scl_prefix}libstdc++%{gcc_ver} = %{version}-%{release}
Requires: %{?_scl_prefix}libstdc++%{gcc_ver}-devel = %{version}-%{release}
Provides: %{?_scl_prefix}gcc-g++%{gcc_ver} = %{version}-%{release}
Provides: %{?_scl_prefix}g++%{gcc_ver} = %{version}-%{release}
Autoreq: true

%description -n %{?_scl_prefix}gcc-c++%{gcc_ver}
This package adds C++ support to the GNU Compiler Collection
version 14.  It includes support for most of the current C++ specification
and a lot of support for the upcoming C++ specification.
```

软件包名确保加上 gcc\-toolset\-xx\- 前缀，%package 和 %description 默认会加上主包名作为前缀，如果带了 \-n 选项，则必须加上 \_scl\_prefix

***

### 3.5 构建流程模块 \(Build Section\)

**位置**: ~1200\-1700 行

**功能**: 定义 GCC 的完整编译过程，包括配置、编译和测试。

**构建流程结构**:

```rpm-spec
%build
# 1. 环境准备
export CONFIG_SITE=NONE
CC=gcc
CXX=g++

# 2. 优化标志处理
OPT_FLAGS="%{optflags}"
# 移除不需要的优化标志
OPT_FLAGS=`echo $OPT_FLAGS|sed -e 's/-Wp,-U_FORTIFY_SOURCE,-D_FORTIFY_SOURCE=[123]//g'`
...

# 3. 安全配置
OPT_FLAGS="$OPT_FLAGS -O2 -Wp,-D_FORTIFY_SOURCE=2 -fstack-protector-strong -fPIE -Wl,-z,relro,-z,now"

# 4. 创建构建目录
rm -rf obj-%{gcc_target_platform}
mkdir obj-%{gcc_target_platform}
cd obj-%{gcc_target_platform}

# 5. 配置选项
CONFIGURE_OPTS="\
    --prefix=%{_prefix} \
    --mandir=%{_mandir} \
    --infodir=%{_infodir} \
    --enable-shared \
    --enable-threads=posix \
    --enable-checking=release \
    ..."
    
# 6. 运行配置
../configure --disable-bootstrap --without-cloog \
    --enable-languages=c,c++,fortran${enablelobjc}${enablelada}${enablelgo}${enableld},lto \
    $CONFIGURE_OPTS

# 7. 编译
make %{?_smp_mflags} BOOT_CFLAGS="$OPT_FLAGS" LDFLAGS_FOR_TARGET=-Wl,-z,relro,-z,now

# 8. 构建 libgccjit (独立构建)
mkdir objlibgccjit
cd objlibgccjit
...
```

**关键配置选项说明**:

| 选项                        | 说明                   |
| --------------------------- | ---------------------- |
| `--disable-bootstrap`       | 禁用自举编译，加快构建 |
| `--enable-languages`        | 启用支持的语言         |
| `--enable-shared`           | 构建共享库             |
| `--enable-threads=posix`    | 使用 POSIX 线程        |
| `--enable-checking=release` | 发布版检查级别         |
| `--with-system-zlib`        | 使用系统 zlib          |
| `--enable-multilib`         | 多库支持 \(aarch64\)   |
| `--enable-cet`              | 控制流执行技术 \(x86\) |

**副版本新增文件：**

需要创建 c89/c99/enable/sudo 四个文件；

**文件说明：**

c89：默认添加 \-std=c89 的 gcc 脚本

c99：默认添加 \-std=c99 的 gcc 脚本

enable：scl 环境使能文件

sudo：sudo 包装器脚本，用于在 sudo 环境中正确启用 scl

```perl
cat > %{buildroot}%{_prefix}/bin/c89 <<"EOF"
#!/bin/sh
fl="-std=c89"
for opt; do
  case "$opt" in
    -ansi|-std=c89|-std=iso9899:1990) fl="";;
    -std=*) echo "`basename $0` called with non ANSI/ISO C option $opt" >&2
        exit 1;;
  esac
done
exec gcc $fl ${1+"$@"}
EOF

cat > %{buildroot}%{_prefix}/bin/c99 <<"EOF"
#!/bin/sh
fl="-std=c99"
for opt; do
  case "$opt" in
    -std=c99|-std=iso9899:1999) fl="";;
    -std=*) echo "`basename $0` called with non ISO C99 option $opt" >&2
        exit 1;;
  esac
done
exec gcc $fl ${1+"$@"}
EOF
chmod 755 %{buildroot}%{_prefix}/bin/c?9

%if 0%{?scl:1}
cat <<EOF > %{buildroot}%{_scl_path}/enable
# General environment variables
export PATH=%{_bindir}\${PATH:+:\${PATH}}
export MANPATH=%{_mandir}:\${MANPATH}
export INFOPATH=%{_infodir}\${INFOPATH:+:\${INFOPATH}}
export PCP_DIR=%{_scl_root}

# bz847911 workaround:
# we need to evaluate rpm's installed run-time % { _libdir }, not rpmbuild time
# or else /etc/ld.so.conf.d files?
rpmlibdir=\$(rpm --eval "%%{_libdir}")
# bz1017604: On 64-bit hosts, we should include also the 32-bit library path.
if [ "\$rpmlibdir" != "\${rpmlibdir/lib64/}" ]; then
  rpmlibdir32=":%{_scl_root}\${rpmlibdir/lib64/lib}"
fi
export LD_LIBRARY_PATH=%{_scl_root}\$rpmlibdir\$rpmlibdir32\${LD_LIBRARY_PATH:+:\${LD_LIBRARY_PATH}}
export LD_LIBRARY_PATH=%{_scl_root}\$rpmlibdir\$rpmlibdir32:%{_scl_root}\$rpmlibdir/dyninst\$rpmlibdir32/dyninst\${LD_LIBRARY_PATH:+:\${LD_LIBRARY_PATH}}

EOF
chmod 755 %{buildroot}%{_scl_path}/enable

cat <<EOF > %{buildroot}%{_bindir}/sudo
#! /bin/sh
# TODO: parse & pass-through sudo options from \$@
sudo_options="-E"

for arg in "\$@"
do
   case "\$arg" in
    *\'*)
      arg=`echo "\$arg" | sed "s/'/'\\\\\\\\''/g"` ;;
   esac
   cmd_options="\$cmd_options '\$arg'"
done
exec /usr/bin/sudo \$sudo_options LD_LIBRARY_PATH=\$LD_LIBRARY_PATH PATH=\$PATH scl enable %{scl} "\$cmd_options"
EOF
chmod 775 %{buildroot}%{_bindir}/sudo
%endif
```

***

### 3.6 安装规则模块 \(Install Section\)

**位置**: ~1700\-2500 行

**功能**: 定义编译完成后如何将文件安装到构建根目录，包括库文件、头文件、二进制文件等的组织。

**安装流程结构**:

```rpm-spec
%install
# 1. 清理并创建构建根目录
rm -rf %{buildroot}
mkdir -p %{buildroot}

# 2. RISC-V 特殊处理 (lp64d ABI)
%ifarch riscv64
for d in %{buildroot}%{_libdir} ...; do
  mkdir -p $d
  (cd $d && ln -sf . lp64d)
done
%endif

# 3. 执行安装
cd obj-%{gcc_target_platform}
make prefix=%{buildroot}%{_prefix} mandir=%{buildroot}%{_mandir} \
  infodir=%{buildroot}%{_infodir} install

FULLPATH=%{buildroot}%{_prefix}/lib/gcc/%{gcc_target_platform}/%{gcc_major}
...

# 4. 库文件和链接处理
# 创建符号链接
ln -sf gcc %{buildroot}%{_prefix}/bin/cc
ln -sf gfortran %{buildroot}%{_prefix}/bin/f95
...

# 5. 头文件处理
# 处理多架构 c++config.h
...

# 6. 库脚本创建
# 创建 GNU ld 脚本
echo '/* GNU ld script */...' > libstdc++.so
...

# 7. 清理和优化
# 删除不必要的文件
rm -f ...
```

**关键安装操作**:

| 操作            | 说明                        |
| --------------- | --------------------------- |
| 符号链接创建    | gcc → cc, gfortran → f95 等 |
| ld 脚本生成     | 用于库版本控制和多重定位    |
| 多架构支持      | 32/64位库的组织             |
| 头文件整理      | 统一放置到标准位置          |
| Python 字节编译 | 对 GDB 插件进行编译         |

***

### 3.7 文件列表模块 \(Files Sections\)

**位置**: ~2500\-2682 行 \(及后续\)

**功能**: 定义每个子软件包包含的文件和目录。

**文件列表结构示例**:

```rpm-spec
%files
# 主 gcc 包文件
%if 0%{?scl:1}
%{_scl_path}/enable
%{_prefix}/bin/sudo
%endif
%{_prefix}/bin/cc
%{_prefix}/bin/c89
%{_prefix}/bin/c99
%{_prefix}/bin/gcc
...
%doc gcc/README* rpm.doc/changelogs/gcc/ChangeLog* gcc/COPYING* COPYING.RUNTIME

%files -n %{?_scl_prefix}cpp%{gcc_ver}
%{_prefix}/lib/cpp
%{_prefix}/bin/cpp
%{_mandir}/man1/cpp.1*
...
```

**文件列表分类**:

| 类别         | 说明                        |
| ------------ | --------------------------- |
| scl 相关文件 | enable、sudo                |
| 二进制文件   | /usr/bin 下的编译器、工具   |
| 库文件       | /usr/lib 下的共享库和静态库 |
| 头文件       | /usr/include 下的开发头文件 |
| 文档         | man 手册、info 文档、README |
| 许可证       | COPYING 等许可证文件        |
| 配置文件     | 架构特定的配置文件          |

**注意事项：**

构建完成后，未打包的文件或不存在的文件会报错，可参考其他副版本 spec 对报错的文件进行调整

***

### 3.8 脚本与触发器模块 \(Scripts &amp; Triggers\)

**位置**: 分散在文件列表之后

**功能**: 定义软件包安装、卸载时执行的脚本，以及触发器。

**脚本类型**:

```rpm-spec
# 安装后脚本 (post)
%post -n %{?_scl_prefix}libgcc%{gcc_ver} -p <lua>
if posix.access ("/sbin/ldconfig", "x") then
  local pid = posix.fork ()
  if pid == 0 then
    posix.exec ("/sbin/ldconfig")
  elseif pid ~= -1 then
    posix.wait (pid)
  end
end

# 卸载后脚本 (postun)
%postun -n %{?_scl_prefix}libgcc%{gcc_ver} -p <lua>
...

# ldconfig 脚本 (自动更新动态链接器缓存)
%ldconfig_scriptlets -n %{?_scl_prefix}libstdc++%{gcc_ver}-devel
%ldconfig_scriptlets -n %{?_scl_prefix}libobjc%{gcc_ver}
...

```

***

## 4. 扩展功能模块

### 4.1 高版本 libstdc++ 符号隐藏

**功能：**将高版本 libstdc++ 相对于系统 libstdc++ 新增的符号打包成静态库，提高兼容性

**关键补丁：**libstdc++\-compat.patch

**补丁来源：**参考并扩展上游社区patch，如 centos 等社区 gcc\-toolset 源码包中获取 [mirror.stream.centos.org](https://mirror.stream.centos.org/9-stream/AppStream/source/tree/Packages/)

**配置修改：**参考现有 spec 文件或 centos 社区的 spec 文件

**注意事项：**如补丁构建遇到 hidden symbol isn't defined 问题，需要在 libstdc++\-compat.patch 中注释掉对应符号的 hidden 声明。

### 4.2 安全检查与加固

**位置**: 构建段

**功能**: 应用安全编译选项，包括栈保护、PIE、RELRO 等。

**安全配置**:

```rpm-spec
# 添加安全编译选项
OPT_FLAGS="$OPT_FLAGS -O2 -Wp,-D_FORTIFY_SOURCE=2 -fstack-protector-strong -fPIE"
OPT_LDFLAGS="$OPT_LDFLAGS -Wl,-z,relro,-z,now"

# 导出到编译环境
export FCFLAGS="$OPT_FLAGS"
CFLAGS="$OPT_FLAGS"
CXXFLAGS="..."
LDFLAGS="$OPT_LDFLAGS"

```

***

## 5. 构建流程图

```scss
┌─────────────────────────────────────────────────────────────┐
│                      构建流程                                 │
└─────────────────────────────────────────────────────────────┘

  ┌──────────────┐
  │    %prep     │  ← 解压源码，应用补丁
  │   (准备)     │
  └──────┬───────┘
         ▼
  ┌──────────────┐
  │   环境设置    │  ← 设置编译器、优化标志
  │   优化处理    │
  └──────┬───────┘
         ▼
  ┌──────────────┐
  │    配置      │  ← 运行 configure 脚本
  │  (Configure) │
  └──────┬───────┘
         ▼
  ┌──────────────┐
  │    编译      │  ← make 编译
  │   (Make)     │
  └──────┬───────┘
         ▼
  ┌──────────────┐
  │  libgccjit   │  ← 单独构建 JIT 库
  │  单独构建    │
  └──────┬───────┘
         ▼
  ┌──────────────┐
  │   %install   │  ← 安装到构建根目录
  │   (安装)     │
  └──────┬───────┘
         ▼
  ┌──────────────┐
  │  库文件处理   │  ← 创建符号链接、ld 脚本
  │  头文件整理   │
  └──────┬───────┘
         ▼
  ┌──────────────┐
  │   %check     │  ← 运行测试 (可选)
  │   (测试)     │
  └──────┬───────┘
         ▼
  ┌──────────────┐
  │   生成 RPM   │  ← 打包为 RPM 文件
  │   (Package)  │
  └──────────────┘

```

***

## 6. 总结

本 SPEC 文件是一个功能完善、架构复杂的 GCC 打包定义，具有以下特点：

1. **多架构支持**: 支持 x86、ARM、RISC\-V、PowerPC、S390 等多种 CPU 架构

2. **多语言支持**: C、C++、Fortran、Objective\-C 等

3. **安全加固**: 集成栈保护、PIE、RELRO 等安全特性

4. **模块化设计**: 将不同功能拆分为独立子软件包

5. **SCL 支持**: 支持 Software Collection 模式，允许并行安装多版本

6. **丰富的运行时库**: 包含各种 Sanitizer、OpenMP、事务内存等高级功能

该 SPEC 文件体现了企业级 Linux 发行版对编译器工具链的完整支持需求。
