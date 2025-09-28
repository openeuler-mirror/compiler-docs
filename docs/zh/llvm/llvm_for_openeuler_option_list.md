# LLVM for openEuler 优化选项列表

LLVM for openEuler 支持通过 `-mllvm` 驱动的自定义优化选项，由于基于鲲鹏架构优化，使能自定义优化选项通常需要指定鲲鹏架构，如 `-mcpu=tsv110`

`-mllvm -profile-summary-cutoff-hot-icp=<num>`

LLVM 的 Indirect Call Promotion (ICP) 优化通过反馈信息将间接函数调用优化为直接函数调用，增加潜在的内联优化机会，并降低函数调用开销。该选项用于设置 ICP 优化阈值，从而解决软件规模过大时优化阈值过高的问题。

当前语言支持 C/C++，后端支持 AArch64。

`-mllvm -loop-versioning-overlap=<true|false>`

通过识别内存拷贝时源指针和目标指针特征，增加运行时检查对内存拷贝方式进行特化优化（如生成 memset、memmove 等）。默认开启，依赖开源选项 `-mllvm -enable-loop-versioning-licm` 的能力。

当前语言支持 C/C++，后端支持 AArch64。

`-mllvm -indirect-load-prefetch=<true|false>`

`-mllvm -outer-loop-prefetch=<true|false>`

`-mllvm -random-access-prefetch-only=<true|false>`

`-mllvm -disable-direct-prefetch=<true|false>`

`-mllvm -indirect-prefetch-skip-intermediate=<true|false>`

提升编译器预取能力，识别应用中多层间接嵌套访存场景，自动计算实际访存地址并插入数据预取指令，降低数据缓存未命中的概率，主要包括：

- 间接内存访问预取

- Crc哈希间接内存访问

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

当前语言支持 C/C++，后端支持 AArch64。

`-mllvm -enable-aggressive-inline=<true|false>`

不考虑源码中的 `__attribute__((noinline))` 限制，强制将该类函数视为普通函数来判断是否进行 inline 优化，设置为 true 开启该优化，默认关闭。

当前语言支持 C/C++，后端支持 AArch64。

`-mllvm -aarch64-ldp-stp-noq=<true|false>`

禁止在鲲鹏 tsv110 后端下生成 `stp/ldp q1, q2, addr` 形式的指令，此形式的指令性能较差。设置为 true 开启该优化，默认开启。

当前语言支持 C/C++，后端仅支持 tsv110。

`-fno-plt`

传统动态库函数调用需通过 PLT （过程链接表）跳转，导致额外的内存访问和跳转指令。使能该选项后优化为直接使用 GOT （全局偏移表）中的函数地址调用，消除 PLT 跳转开销。

当前语言支持 C/C++。

# LLVM for openEuler 功能选项列表

`-fgcc-compatible`

开启 LLVM for openEuler 对 GCC 编译器的兼容性特性，包括但不限于对编译器不识别的 GCC 功能性选项的告警严重程度降级至 warning。
