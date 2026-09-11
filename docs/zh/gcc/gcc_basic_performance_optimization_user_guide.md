# GCC 基础性能优化用户指南

## 简介

编译器基础性能优化对于提高应用程序的开发效率、运行性能和可维护性都非常重要。它是计算机科学领域的一个重要研究方向，也是软件开发过程中的重要环节之一。GCC for openEuler 在通用编译优化能力的基础上，对中后端性能优化技术进行了增强，包括指令优化、向量化增强、预取增强、数据流分析增强等优化。

## 工具链适用性

本文特性适用于系统 GCC（gcc 12.3.1），亦适用于已合入 openEuler 优化栈的 gcc-toolset-14（见[GCC14副版本编译工具链](gcc_14_secondary_version_compilation_toolchain_user_guide.md)），例外如下。

仅系统 GCC 支持：

- `-mcmlt-arith`
- `-fconvert-minmax`
- `-fsplit-ldp-stp` 及 `--param=param-ldp-dependency-search-range`
- `-fllc-allocate` 及其关联参数
- `-ftree-slp-transpose-vectorize`
- `--param=vect-alias-flexible-segment-len`

以下系统 GCC 选项在 gcc-toolset-14 中已废弃，正向形式报错，需从构建脚本移除或替换：

| 系统 GCC 选项 | gcc-toolset-14 行为 |
| ---- | ---- |
| `-falias-analysis-expand-ssa` | 拒绝；对应的非循环别名消歧能力已内置且默认生效 |
| `-fchrec-mul-fold-strict-overflow` | 拒绝；对应的标量演化乘法折叠已内置且无条件执行 |
| `-fmerge-mull` | 拒绝；改用 `-mmul-widen128` |
| `-fftz` | 拒绝；改用 `-mdaz-ftz` |
| `-fp-model=` | 拒绝；改用 `-ffp-model=` |

注：前两项的 `-fno-` 形式被静默接受。

## 安装与部署

### 软件要求

操作系统：openEuler 24.03

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

### -fipa-struct-dfc

#### 说明

动态压缩结构体成员，克隆程序路径并启发式压缩结构体大小，根据运行时检查选择运行路径，以提高 cache 命中率。

#### 使用方法

在选项中加入`-O3 -flto -flto-partition=one -fipa-reorder-fields -fipa-struct-dfc`即可，可在此基础上使用`-fipa-struct-dfc-bitfield`、`-fipa-struct-dfc-shadow`开启额外优化。

注：`-fipa-struct-dfc`选项，需要在`-O3 -flto -flto-partition=one`全局同时开启以及开启`-fipa-reorder-fields`或`-fipa-struct-reorg>=2`的基础上才能使能。

### -fipa-alignment-propagation

#### 说明

分析并传播局部变量地址对齐值，优化按位与运算。

#### 使用方法

在选项中加入`-O3 -fipa-alignment-propagation`即可。

注：`-fipa-alignment-propagation`选项，需要在`-O3`开启的基础上才使能。

### -fipa-localize-array

#### 说明

将由 calloc 分配的全局指针变量转换为局部变量。

#### 使用方法

在选项中加入`-O3 -fipa-localize-array`即可。

注：`-fipa-localize-array`选项，需要在`-O3`开启的基础上才使能。

### -fipa-array-dse

#### 说明

分析数组在函数之间的传递情况以及在被调用函数中的使用情况，消除冗余的数组写入。

#### 使用方法

在选项中加入`-O3 -fipa-array-dse`即可。

注：`-fipa-array-dse`选项，需要在`-O3`开启的基础上才使能。

### -ffind-with-sve

#### 说明

识别 `std::find` 函数调用，尝试使用 SVE 指令优化。

#### 使用方法

在选项中加入 `-ffind-with-sve` 即可。

### -floop-sve-mode-opt

#### 说明

通过静态代码特征分析，识别特殊场景。并在确认满足条件时新增SVE指令集优化机会，从而获得性能提升。

#### 使用方法

在选项中加入`-O3 -floop-sve-mode-opt`即可。

注：`-floop-sve-mode-opt`选项，需要在`-O3`开启以及`-march`中加入sve的基础上才使能；该优化仅在 aarch64 上生效。

### -fif-split

#### 说明

拆分形如"变量与常量比较 || 其他条件"的复合条件并克隆分支体，使 IPA 常量传播可将该常量传入函数调用。

#### 使用方法

在选项中加入`-fif-split`即可。

注：`-fif-split`选项，需要在`-O3`开启的基础上才使能。

### 向量化分析参数

#### 说明

三个独立的向量化分析开关。

#### 使用方法

| 选项 | 默认值 | 说明 |
| ---- | ---- | ---- |
| `--param=vect-swap-operands=[0,1]` | 0 | 向量化分析中允许交换可交换运算的操作数。 |
| `--param=addr-expand-for-alias-check=[0,1]` | 0 | 别名检查时展开数据引用地址。 |
| `--param=vect-register-size-check=[0,1]` | 0 | 检查交错访存组是否超出向量寄存器容量。 |

### -ftree-slp-late

#### 说明

在 reassociation 之后追加一次 SLP 向量化 pass。

#### 使用方法

在选项中加入`-ftree-slp-late`即可，默认关闭。

### `-floop-elim`

#### 说明

冗余循环消除。

#### 使用方法

在选项中加入`-floop-elim`即可。

注：作为 phiopt 的一部分执行，依赖`-fssa-phiopt`（`-O1`及以上默认开启）。

### -farray-widen-compare

#### 说明

将符合条件的逐字节比较循环改写为按8字节比较。

#### 使用方法

在选项中加入`-farray-widen-compare`即可。

注：需要在`-O3`开启的基础上才使能，仅小端目标生效，且仅支持 LP64 数据模型。要求参与比较的数组在循环上界内始终可读：合并后的 8 字节载入可能读取逐字节循环因提前退出而不会访问的后续字节，不满足该前提时存在越界访问风险。

### `-fbuiltin-will-return`

#### 说明

将不会循环、抛异常或退出的内建函数（当前仅`__builtin_prefetch`）视为必然返回的调用。

#### 使用方法

在选项中加入`-fbuiltin-will-return`即可。

### `-mmul-widen128`

#### 说明

识别64位到128位加宽乘法惯用法，生成高效指令序列。

#### 使用方法

在选项中加入`-mmul-widen128`即可。

注：仅 aarch64。系统 GCC 中对应选项为`-fmerge-mull`。

### 向量数学库与浮点控制

#### 说明

向量数学库支持与浮点行为控制。

#### 使用方法

- `-fsimdmath`：声明数学函数的向量变体（`_ZGV*`）并在链接时注入 libmathlib。
- `-ffp-model=`：浮点模型控制，取值 normal/fast/precise/except/strict。
- `-mdaz-ftz`：程序启动时设置浮点控制寄存器的 FTZ/DAZ 标志，非规格化数清零。

注：系统 GCC 中`-ffp-model=`、`-mdaz-ftz`对应拼写为`-fp-model=`、`-fftz`。

### CFGO / CSPGO

#### 说明

CFGO 在使用 PGO profile 的基础上联动使能一组 openEuler 优化；CSPGO 在内联后二次插桩，使 profile 计数携带调用上下文。gcc-toolset-14 同样提供该能力。

#### 使用方法

选项与完整优化流程见[CFGO特性](cfgo_user_guide.md)。此外可用`-fcfgo-csprofile-dir=<dir>`单独指定 CSPGO 的 profile 目录。

注：CFGO 联动选项集不包含`-fselective-scheduling`。该选项仍会被编译器接受，但与 profile 插桩（`-fcfgo-profile-generate`/`-fcfgo-csprofile-generate`）组合存在已知错编译风险（aarch64 上曾观测到语句被静默删除且无诊断），不应组合使用。

### oeAware 协同优化

#### 说明

生成供 oeAware 运行时调优框架读取的 ELF section。

#### 使用方法

在选项中加入`-foeaware-policy`即可；`-foeaware-policy=[1,7]`选择优化策略，默认1。

### AutoBOLT

#### 说明

编译时将函数与分支信息（源自 AutoFDO/PGO profile）写入 ELF section，链接后由 bolt-plugin 收集并执行 BOLT 二进制优化。

#### 使用方法

在选项中加入`-fauto-bolt`即可；`-fauto-bolt=<dir>`指定 profile 数据目录，默认当前目录。

注：仅 aarch64。不支持与`-flto`同时使用（编译报错），与`-fbolt-use`互斥；因此不可与本文中依赖`-flto`的 Struct-Reorg 系列选项组合。

### SVE memcall inline优化

#### 说明

将`memcpy`和`memset`函数调用内联展开为SVE循环，减少库函数调用开销，提升小尺寸内存操作的性能。该优化不支持`memmove`，不适用于大尺寸内存操作频繁的场景。

#### 使用方法

在编译选项中加入`-march=armv8-a+sve -maarch64-sve-memcall-inlining`。

相关选项和参数如下：

| 选项或参数 | 默认值 | 说明 |
| --- | --- | --- |
| `-maarch64-sve-memcall-inlining` | 关闭 | 启用SVE memcall inline优化 |
| `-mno-aarch64-sve-memcall-inlining` | — | 关闭SVE memcall inline优化 |
| `--param=aarch64-sve-memcall-size-threshold=N` | 4096 | 设置SVE内联的长度阈值，单位为字节 |
| `-maarch64-sve-memcall-runtime-check` | 开启 | 主开关开启后检查运行时长度，超过阈值时回退为库函数调用 |
| `-mno-aarch64-sve-memcall-runtime-check` | — | 关闭运行时长度检查，变量长度操作直接内联展开为SVE循环 |

注：建议根据实际负载中`memcpy`和`memset`的长度分布调整阈值，使大尺寸内存操作回退为库函数调用。
