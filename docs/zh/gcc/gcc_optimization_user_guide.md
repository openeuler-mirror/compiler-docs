# GCC 基础性能优化用户指南

## 简介

编译器基础性能优化对于提高应用程序的开发效率、运行性能和可维护性都非常重要。它是计算机科学领域的一个重要研究方向，也是软件开发过程中的重要环节之一。GCC for openEuler 在通用编译优化能力的基础上，对中后端性能优化技术进行了增强，包括指令优化、向量化增强、预取增强、数据流分析增强等优化。

## 安装与部署

### 软件要求

操作系统：openEuler 25.03

### 硬件要求

aarch64 架构

### 安装软件

按需安装 GCC 和相关组件即可，以 GCC 为例。

```shell
yum install gcc
```

## 使用方法

### CRC 优化

#### 说明

识别 CRC 软件循环代码，生成高效硬件指令。

#### 使用方法

在编译时增加 -floop-crc 选项。

注：`-floop-crc`选项需要和`-O3 -march=armv8.1-a`一起使用。

### If-conversion 增强

#### 说明

增强 If conversion 优化，使用更多的寄存器以减少冲突。

#### 使用方法

本优化是 RTL 优化 if-conversion 的一部分，使用如下编译选项控制优化启用。

`-fifcvt-allow-complicated-cmps`

`--param=ifcvt-allow-register-renaming=[0,1,2]`数字用于控制优化范围。

注：此优化依赖`-O2`优化等级，以及与`--param=max-rtl-if-conversion-unpredictable-cost=48`、`--param=max-rtl-if-conversion-predictable-cost=48`共同使用。

### 乘法计算优化

#### 说明

Arm 相关指令合并优化，实现32位复杂组合的64位整形乘法逻辑的识别，并以高效的64位指令数输出。

#### 使用方法

使用`-fuaddsub-overflow-match-all`和`-fif-conversion-gimple`选项使能优化。

注：此优化需要`-O3`及以上优化等级。

### cmlt 指令生成优化

#### 说明

对一些四则运算生成`cmlt`指令，减少指令数。

#### 使用方法

使用选项`-mcmlt-arith`使能优化。

注：此优化需要`-O3`以上优化等级使用。

### 向量化优化增强

#### 说明

识别并简化向量化过程中生成的冗余指令，允许更短的循环进入向量化。

#### 使用方法

使用参数`--param=vect-alias-flexible-segment-len=1`使能，默认为0。

注：此优化需要`-O3`及以上优化等级。

### min max 和 uzp1/uzp2 指令联合优化

#### 说明

识别 min max 和 uzp1/uzp2 指令联合优化机会，减少指令数从而提升性能。

#### 使用方法

使用`-fconvert-minmax`选项使能`min max`优化，`uzp1/uzp2`指令优化在`-O3`以上等级默认使能。

注：依赖`-O3`及以上优化等级。

### ldp/stp 优化

#### 说明

识别某些性能表现差的 ldp/stp，将其拆分成2个 ldr 和 str。

#### 使用方法

使用`-fsplit-ldp-stp`选项使能优化，使用参数`--param=param-ldp-dependency-search-range=[1,32]`控制搜索范围，默认16。

注：依赖`-O1`及以上优化等级。

### AES指令优化

#### 说明

识别 AES 软件算法指令序列，使用硬件指令加速。

#### 使用方法

使用`-fcrypto-accel-aes`选项使能优化。

注：依赖`-O3`及以上优化等级。

### 间接调用提升

#### 说明

识别和分析程序中的间接调用，尝试将其优化为直接调用。

#### 使用方法

使用选项`-ficp -ficp-speculatively`使能优化。

注：此优化需要和`-O2 -flto -flto-partition=one`共同使用。

### IPA-prefetch

#### 说明

识别循环中的间接访存，插入预取指令，从而减少间接访存的延迟。

#### 使用方法

通过选项`-fipa-prefetch -fipa-ic`使能优化。

注：此优化需要和`-O3 -flto`共同使用。

### -fipa-struct-reorg

#### 说明

内存空间布局优化，将结构体成员在内存中的排布进行新的排列组合，来提高 cache 的命中率。

#### 使用方法

在选项中加入`-O3 -flto -flto-partition=one -fipa-struct-reorg`即可。

注：`-fipa-struct-reorg`选项，需要在`-O3 -flto -flto-partition=one`全局同时开启的基础上才使能。

### -fipa-reorder-fields

#### 说明

内存空间布局优化，根据结构体中成员的占用空间大小，将成员从大到小排列，以减少边界对齐引入的 padding，来减少结构体整体占用的内存大小，以提高 cache 的命中率。

#### 使用方法

在选项中加入`-O3 -flto -flto-partition=one -fipa-reorder-fields`即可。

注：`-fipa-reorder-fields`选项，需要在`-O3 -flto -flto-partition=one`全局同时开启的基础上才使能。

### -ftree-slp-transpose-vectorize

#### 说明

该选项在循环拆分阶段，增强对存在连续访存读的循环的数据流分析能力，通过插入临时数组拆分循环；SLP 矢量化阶段，新增对 grouped_stores 进行转置的 SLP 分析。

#### 使用方法

在选项中加入`-O3 -ftree-slp-transpose-vectorize`即可。

注：`-ftree-slp-transpose-vectorize`选项，需要在`-O3`开启的基础上才使能。

### LLC-prefetch

#### 说明

通过分析程序中主要的执行路径，对主路径上的循环进行访存的复用分析，计算排序出 TOP 的热数据，并插入预取指令将数据先分配至 LLC 中，减少 LLC miss。

#### 使用方法

使能 LLC 特性，需开启 `-O2` 及以上优化等级，同时使用编译选项`-fllc-allocate`。

其他相关接口：

|  选项   | 默认值  | 说明  |
|  ----  | ----  | ----  |
| --param=mem-access-ratio=[0,100]  | 20 | 循环内访存数对指令数的占比。|
| --param=mem-access-num=unsigned  | 3 | 循环内访存数量。  |
| --param=outer-loop-nums=[1,10]  |  1 | 允许扩展的外层循环的最大层数。  |
| --param=filter-kernels=[0,1]  |  1 | 是否针对循环做路径串联筛选。  |
| --param=branch-prob-threshold=[50,100]  |  80 | 高概率执行分支的概率阈值。  |
| --param=prefetch-offset=[1,999999]  | 1024  |  预取偏移距离，一般为2的次幂。 |
| --param=issue-topn=unsigned  |  1 |  预取指令个数。 |
| --param=force-issue=[0,1]  |  0 |  是否执行强制预取，即静态模式。 |
| --param=llc-capacity-per-core=[0,999999]  |  107 | 多分支预取下每个核平均分配的 LLC 容量。  |

### -fipa-struct-sfc

#### 说明

静态压缩结构体成员，从而减小结构体整体占用的内存大小，以提高 cache 命中率。

#### 使用方法

在选项中加入`-O3 -flto -flto-partition=one -fipa-reorder-fields -fipa-struct-sfc`即可，可在此基础上使用`-fipa-struct-sfc-bitfield`、`-fipa-struct-sfc-shadow`开启额外优化。

注：`-fipa-struct-sfc`选项，需要在`-O3 -flto -flto-partition=one`全局同时开启以及开启`-fipa-reorder-fields`或`-fipa-struct-reorg>=2`的基础上才能使能。
