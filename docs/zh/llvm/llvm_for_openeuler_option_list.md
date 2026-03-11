# LLVM for openEuler 优化选项列表

LLVM for openEuler 支持通过 `-mllvm` 驱动的自定义优化选项，由于基于鲲鹏架构优化，使能自定义优化选项通常需要指定鲲鹏架构，如 `-mcpu=tsv110`

## `-mllvm -profile-summary-cutoff-hot-icp=<num>`

LLVM 的 Indirect Call Promotion (ICP) 优化通过反馈信息将 hot 的间接函数调用优化为直接函数调用，增加潜在的内联优化机会，并降低函数调用开销。该选项用于设置 ICP 优化阈值，从而解决软件规模过大时优化阈值过高的问题。

默认值：990000，即 99%

支持范围：当前语言支持 C/C++，后端支持 AArch64/X86

## `-mllvm -loop-versioning-overlap=<true|false>`

通过识别内存拷贝时源指针和目标指针特征，增加运行时检查对内存拷贝方式进行特化优化（如生成 memset、memmove 等）。

默认值：true，依赖使能开源选项 `-mllvm -enable-loop-versioning-licm`（默认值为 false）

支持范围：当前语言支持 C/C++，后端仅支持 `-mcpu=[tsv110|hip09|hip10c|hip11|hip12]`

## `-mllvm -indirect-load-prefetch=<true|false>`

包含以下优化选项支持：

`-mllvm -indirect-load-prefetch=<true|false>`

`-mllvm -outer-loop-prefetch=<true|false>`

`-mllvm -random-access-prefetch-only=<true|false>`

`-mllvm -disable-direct-prefetch=<true|false>`

`-mllvm -indirect-prefetch-skip-intermediate=<true|false>`

提升编译器预取能力，识别应用中多层间接嵌套访存场景，自动计算实际访存地址并插入数据预取指令，降低数据缓存未命中的概率，主要包括：

- 间接内存访问预取

- CRC哈希间接内存访问

选项关系如下表所示

||inner loop direct load prefetch|inner loop indirect load prefetch|outer loop direct load prefetch|outer loop indirect load prefetch|
|-|-|-|-|-|
|default|√| | | |
|indirect-load-prefetch|√|√| | |
|outer-loop-prefetch|√| |√| |
|indirect-load-prefetch,outer-loop-prefetch|√|√|√|√|
|indirect-load-prefetch,outer-loop-prefetch,random-access-prefetch-only|√|√(hash indirect load type only)| |√ (hash indirect load type only)|
|indirect-load-prefetch,outer-loop-prefetch,disable-direct-prefetch| |√| |√|
|indirect-load-prefetch,outer-loop-prefetch,indirect-prefetch-skip-intermediate|√|√ (exclude B[i] prefetch in A[B[i]])|√|√ (exclude B[i] prefetch in A[B[i]])|

默认值：都为 false

支持范围：当前语言支持 C/C++，后端支持 AArch64

## `-mllvm -enable-aggressive-inline=<true|false>`

不考虑源码中的 `__attribute__((noinline))` 限制，强制将该类函数视为普通函数来判断是否进行 inline 优化

默认值：false

支持范围：当前语言支持 C/C++，后端支持 AArch64/X86

## `-mllvm -aarch64-ldp-stp-noq=<true|false>`

禁止在鲲鹏 tsv110 后端下生成 `stp/ldp q1, q2, addr` 形式的指令，此形式的指令性能较差。设置为 true 开启该优化。

默认值：true

支持范围：当前语言支持 C/C++，后端仅支持 `-mcpu=tsv110`

## `-fno-plt`

传统动态库函数调用需通过 PLT （过程链接表）跳转，导致额外的内存访问和跳转指令。使能该选项后优化为直接使用 GOT （全局偏移表）中的函数地址调用，消除 PLT 跳转开销。

支持范围：当前语言支持 C/C++，后端支持 AArch64/X86

## `-mllvm -no-sink-prtadd-post-load=<true|false>`

在编译器优化过程中，通常会把指针类型的循环迭代变量的迭代操作（如循环内 "ptr++" 操作）下沉到循环末尾，如果循环下一迭代开始存在 load 操作并依赖该指针变化后的值时，就会出现数据依赖而造成指令 stall。该优化通过合理调度指令位置来避免因数据依赖而造成的指令 stall。

默认值：false

支持范围：当前语言支持 C/C++，后端支持 AArch64/X86

## `-mllvm -aarch64-endianness-opts=<true|false>`

在常见数据库如 MySQL 等应用中，数据是以大端字节序进行存储，在加载到内存中使用的时候，需要先转换为小端字节序然后进行后续计算，这通常需要"按 Byte 进行 load"-"移位"-"按位或"步骤循环操作，带来性能开销。该优化高效使能 AArch64 REV 系列指令，加大 load 位宽以减少 load 次数以及减少移位和按位或操作，提升应用性能。

默认值：false

支持范围：当前语言支持 C/C++，后端支持 AArch64

# LLVM for openEuler 功能选项列表

## `-fgcc-compatible`

开启 LLVM for openEuler 对 GCC 编译器的兼容性特性，包括但不限于将 LLVM 编译器不识别的 GCC 功能性选项的告警严重程度从 error 降级至 warning、兼容 GCC 扩展语法等。

## `clang-tidy --checks='-*,BSCompatibility*' --export-details=output.yaml demo.c`

支持使用 clang-tidy 工具识别部分 LLVM 不兼容的 GCC 拓展语法，产生告警并给出修改建议，输出到 output.yaml 中。

同时提供 share/clang/clang-tidy-stats.py 脚本能够将修改建议输出为表格，

`clang-tidy-stats.py --checks='-*,BSCompatibility*' --export-stats=stats.xlsx demo.c`

该脚本可以识别并告警，将代码信息和修改建议直接输出到 stats.xlsx 表格中；

`clang-tidy-stats.py --args -export-stats=stats.xlsx -export-stats-input=output.yaml`

该脚本也可以将 output.yaml 转化为 stats.xlsx 的表格形式。

## `-mcpu=<CPU-name>`

通过 `-mcpu=<CPU-name>` 选项指定当前的 CPU 型号，使能该 CPU 所有默认支持的硬件特性。在使能对应的微架构亲和选项后，LLVM for openEuler 会依据对应微架构的指令特征进行指令流水线调优，提升性能。

鲲鹏硬件平台与该选项的配置项对应关系如下：

|硬件平台|配置项|
|-|-|
|鲲鹏920|tsv110|
|HiSilicon HIP09|hip09|
|HiSilicon HIP10c|hip10c|
|HiSilicon HIP11|hip11|
|鲲鹏950|hip12|
