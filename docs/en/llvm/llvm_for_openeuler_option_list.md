# LLVM for openEuler Optimization Options

LLVM for openEuler supports the user-defined optimization option driven by `-mllvm`. Because the optimization is based on the Kunpeng architecture, the Kunpeng architecture needs to be specified to enable the user-defined optimization option, for example, set `-mcpu` to `tsv110`.

`-mllvm -profile-summary-cutoff-hot-icp=<num>`

The indirect call promotion (ICP) optimization of LLVM converts indirect function calls into direct function calls based on feedback information. This increases potential inlining opportunities and reduces function call overhead. This option is used to set the ICP optimization threshold to solve the problem that the optimization threshold is too high when the software scale is too large.

Currently, the C and C++ languages and the AArch64 backend are supported.

`-mllvm -loop-versioning-overlap=<true|false>`

By recognizing the characteristics of the source and destination pointers during memory copy, this feature adds runtime checks to optimize the memory copy method (such as generating memset and memmove intrinsics). This feature is enabled by default and depends on the `-mllvm -enable-loop-versioning-licm` capability of the open-source option.

Currently, the C and C++ languages and the AArch64 backend are supported.

`-mllvm -indirect-load-prefetch=<true|false>`

`-mllvm -outer-loop-prefetch=<true|false>`

`-mllvm -random-access-prefetch-only=<true|false>`

`-mllvm -disable-direct-prefetch=<true|false>`

`-mllvm -indirect-prefetch-skip-intermediate=<true|false>`

This feature improves the prefetch capability of the compiler, identifies multi-layer indirect nested memory access scenarios in applications, automatically calculates actual memory access addresses, and inserts data prefetch instructions, thereby reducing the probability of data cache misses. This includes:

- Indirect memory access prefetch

- CRC hash indirect memory access

The following table lists the option relationships.

||inner loop direct load prefetch|inner loop indirect load prefetch|outer loop direct load prefetch|outer loop indirect load prefetch|
|-|-|-|-|-|
|default|√| | | |
|indirect-load-prefetch|√|√| | |
|outer-loop-prefetch|√| |√| |
|indirect-load-prefetch,outer-loop-prefetch|√|√|√|√|
|indirect-load-prefetch,outer-loop-prefetch,random-access-prefetch-only|√|√(hash indirect load type only)| |√ (hash indirect load type only)|
|indirect-load-prefetch,outer-loop-prefetch,disable-direct-prefetch| |√| |√|
|indirect-load-prefetch,outer-loop-prefetch,indirect-prefetch-skip-intermediate|√|√ (exclude B[i] prefetch in A[B[i]])|√|√ (exclude B[i] prefetch in A[B[i]])|

Currently, the C and C++ languages and the AArch64 backend are supported.

`-mllvm -enable-aggressive-inline=<true|false>`

The `__attribute__((noinline))` restriction in the source code is ignored. This type of function is forcibly regarded as a common function to determine whether to perform inline optimization. If this option is set to **true**, the optimization is enabled. By default, the optimization is disabled.

Currently, the C and C++ languages and the AArch64 backend are supported.

`-mllvm -aarch64-ldp-stp-noq=<true|false>`

Do not generate `stp/ldp q1`, `q2`, or `addr` instructions at the Kunpeng tsv110 backend because the performance of these instructions is not ideal. The value **true** indicates that the optimization is enabled. By default, the optimization is enabled.

Currently, the C and C++ languages and only the tsv110 backend are supported.

`-fno-plt`

Conventional dynamic library function calls require jumping through the procedure linkage table (PLT), leading to extra memory access and jump instructions. After this option is enabled, function addresses in the global offset table (GOT) are directly used for calls, eliminating the overhead of PLT jumps.

Currently, the C and C++ languages are supported.

# LLVM for openEuler Function Options

`-fgcc-compatible`

This option enables the compatibility between LLVM for openEuler and the GCC compiler, including but not limited to degrading the severity of alarms for the compiler failing to identify GCC functional options to warning.
