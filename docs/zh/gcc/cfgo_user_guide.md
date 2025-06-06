# CFGO静态反馈优化特性用户指南

## 简介

反馈编译优化，通常被称为`Profile-Guided Optimization（PGO）`，是一种编译技术，它通过收集程序执行期间的运行时`（runtime）`信息来优化程序性能。

`CFGO（Continuous Feature Guided Optimization）`是`GCC for openEuler`和`毕昇编译器`的反馈优化品牌，指多模态（源代码、二进制）、全生命周期（编译、链接、链接后、运行时、OS、库）的持续反馈优化。

主要优化点：

* 代码布局优化：通过基本块重排、函数重排、冷热分区等技术，优化目标程序的二进制布局，提升i-cache和i-TLB命中率。
* 高级编译器优化：内联、循环展开、向量化、间接调用等提升编译优化技术受益于反馈信息，能够使编译器执行更精确的优化决策。

本文档主要介绍CFGO中静态反馈优化相关技术。

## 安装与部署

### 软件要求

* 操作系统：openEuler 24.03 LTS SP1
* 编译器：GCC-12.3.1-47及以上版本

## 使用方法

CFGO静态反馈优化典型优化流程共包含三种优化技术`CFGO-PGO`、`CFGO-CSPGO`和`CFGO-BOLT`。`CFGO-PGO`优化主要用于启用inline、常量传播、去虚化等优化，`CFGO-CSPGO`用于在`CFGO-PGO`的基础上利用更新后的profile执行基本块重排、寄存器分配等优化，`CFGO-BOLT`用于post-link的二进制代码布局优化。

为了进一步提升优化效果，建议CFGO系列优化与链接时优化搭配使用，即在`CFGO-PGO`、`CFGO-CSPGO`优化过程中增加`-flto=auto`编译选项。

若要获取最佳优化效果，使用时可按照`CFGO-PGO`->`CFGO-CSPGO`->`CFGO-BOLT`顺序依次执行。

### 1. `CFGO-PGO`特性

特性介绍：在开源PGO的基础上，利用`AI4C`对部分优化选项和参数进行调优，进一步提升性能。

选项：`-fcfgo-profile-generate=${path}`

说明：启用编译插桩功能用于生成插桩可执行程序。如果指定`${path}`，程序的执行信息文件将写入该路径，否则将生成至当前路径。

选项：`-fcfgo-profile-use=${path}`

说明：利用程序的执行信息执行编译优化。如果指定`${path}`，程序的执行信息文件将从该路径读取，否则从当前路径读取。

典型使用流程：

```bash
// 1. 编译插桩
gcc -fcfgo-profile-generate=${path} test.c -o test

// 2. profiling
./test

// 3. 使用profile
gcc -fcfgo-profile-use=${path} test.c -o test
```

### 2. `CFGO-CSPGO`特性

特性介绍：PGO的profile对上下文不敏感，可能导致次优的优化效果。通过在PGO后增加一次`CFGO-CSPGO`插桩优化流程，从而收集inline后的程序运行信息，为重排和寄存器优化提供更准确的执行信息，最终进一步提升性能。

选项：`-fcfgo-csprofile-generate=${path}`

说明：启用post-inline的编译插桩功能，必须和`-fcfgo-profile-use`共同使用。必须指定`${path}`，且路径不能与`CFGO-PGO`阶段的路径相同。

选项：`-fcfgo-csprofile-use=${path}`

说明：利用post-inline的profile优化程序，必须和`-fcfgo-profile-use`共同使用。必须指定`${path}`，路径与`-fcfgo-csprofile-generate=${path}`的路径相同。

典型使用流程：

```bash
// 1. 编译插桩
gcc -fcfgo-profile-use={path_1} -fcfgo-csprofile-generate=${path_2} test.c -o test

// 2. profiling
./test

// 3. 使用profile
gcc -fcfgo-profile-use={path_1} -fcfgo-csprofile-use=${path_2} test.c -o test
```

### 3. `CFGO-BOLT`特性

特性介绍：BOLT是链接后二进制代码布局优化器，本特性回合高版本的二进制插桩能力，从而收集相比采样更准确的运行信息指导二进制优化代码布局来提升性能。

选项：`-instrument`

说明：启用软件插桩功能收集程序运行时信息，依赖二进制中包含重定位信息（如编译时增加`-Wl,-q`）。

除了该选项外，BOLT插桩功能还包含一些其他选项，具体可参考`llvm-bolt --help`中的`BOLT instrumentation options`章节。

典型使用流程：

```bash
// 1. 编译程序时保留重定位信息
gcc -Wl,-q test.c -o test

// 2. 编译BOLT插桩
llvm-bolt ./test -instrument -o test.inst -instrumentation-file=${test.fdata} --instrumentation-wait-forks --instrumentation-sleep-time=2 --instrumentation-no-counters-clear

// 3. profiling
./test.inst

// 4. 使用profile
llvm-bolt ./test -o test.opt -data=test.fdata
```

## 兼容性说明

此节主要列出当前一些特殊场景下的兼容性问题。本项目持续迭代中，会尽快进行修复，也欢迎广大开发者加入。

* `CFGO-CSPGO`暂当前不支持对动态库优化。
