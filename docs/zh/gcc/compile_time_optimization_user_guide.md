# 背景

为了提升openEuler软件包的编译效率，从而进一步提升门禁和开发者开发效率，我们在2503版本使用了打开PGO（Profile Guided Optimization）、LTO（Link Time Optimization）编译的GCC结合mold（A Modern Linker）链接器来缩短软件包中C/C++库的编译时间。

# 方案

## 优化技术原理

Mold：是一种对比其他链接器（ld、gold、lld）链接效率更高的开源链接器，其通过更高效的Identical Comdat Folding（ICF）算法、更好的并行编程，更高效的多线程库等技术来提升链接的效率，具体可参考：<https://github.com/rui314/mold/blob/main/docs/design.md>。

PGO：利用程序运行时的剖析信息来指导编译优化，以提高程序的性能，运行时的剖析信息包括函数调用频率，分支走向等。

LTO：链接时优化，可以进行更多的跨模块的优化，消除模块之间的冗余代码和不必要的函数调用，提升GCC编译其他应用时候执行效率。

## 使能方式

在GCC的编译过程取消了`--disable-bootstrap`，同时添加了`BUILD_CONFIG=bootstrap-lto profiledbootstrap`来打开PGO和LTO去编译GCC。

在openEuler-rpm-config的macro中我们通过检查软件包名字是否在白名单中来给LDFLAGS添加`-fuse-ld=mold`，使得软件包可使用mold进行编译。

## 使能范围

打开了PGO、LTO编译的GCC在2503版本上默认使能。

由于mold链接器本身存在一定的功能欠缺（对kernel的不支持，对linker script支持不完善）我们在2503上使用[白名单](https://gitee.com/src-openeuler/openEuler-rpm-config/blob/openEuler-25.03/0002-Enable-mold-links-through-whitelist.patch#L49)的方式使能mold。

同时我们提供了更加灵活的链接器配置方式，各软件包可以在spec内覆写`_ld_use`变量来切换链接器，如：

- `%define _ld_use %{nil}` 取消因为软件包在白名单中所添加的mold选项。

- `%define _ld_use -fuse-ld=xxx` 切换不同的链接器，注意当定义多个`-fuse-ld`选项时编译器会取最后一个配置项。

# 注意事项

1. 只有在构建环境上存在mold时，白名单内的软件包才会启用mold链接。
2. 当启用mold链接的时候需在GCC 12版本及以上进行构建。
