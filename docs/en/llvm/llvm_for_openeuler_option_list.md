# LLVM for openEuler User Manual

The LLVM for openEuler compiler is deeply built upon the open-source LLVM software. It is a high-performance compiler tailored for high-computing power scenarios, such as the server internet industry, data center deployments, general-purpose computing plus AI workloads, and video encoding/decoding. Concurrently, it establishes a LLVM baseline within the openEuler community, providing a stable, highly innovative downstream release that delivers high performance and robust security. It currently supports mainstream programming languages (C/C++) and major chip architectures (including x86, AArch64, RISC-V, and LoongArch).

> Note: Unless otherwise specified, the options described in this document are strictly supported on the major version LLVM 17. Support for minor or subsequent versions (for example, LLVM 18, 19, or 20) will be explicitly stated.

# LLVM for openEuler Optimization Option List

LLVM for openEuler supports custom optimization options driven via the `-mllvm` flag. Because these optimizations are deeply tailored for the Kunpeng architecture, enabling these custom options typically requires specifying the corresponding Kunpeng CPU target, such as `-mcpu=tsv110`.

## `-mllvm -profile-summary-cutoff-hot-icp=<num>`

The Indirect Call Promotion (ICP) optimization in LLVM utilizes profile-guided feedback to transform hot indirect function calls into direct function calls, increasing potential opportunities for subsequent inline optimizations and significantly reducing function call overhead. This option is used to fine-tune the ICP optimization threshold, resolving the issue where the default threshold is too high for large-scale software.

Default value: `990000` (representing 99%)

Supported scope: It currently supports C/C++ for the frontend, and AArch64/x86 for the backend.

## `-mllvm -loop-versioning-overlap=<true|false>`

By identifying the characteristics of the source and destination pointers during memory copies, this optimization injects runtime checks to specialize the memory copy approach (such as generating specific `memset` or `memmove` calls).

Default value: `true` (depending on enabling the open-source option `-mllvm -enable-loop-versioning-licm`, which defaults to `false`)

Supported scope: It currently supports C/C++ for the frontend, and only `-mcpu=[tsv110|hip09|hip10c|hip11|hip12]` for the backend.

## `-mllvm -indirect-load-prefetch=<true|false>`

Includes support for the following optimization options:

`-mllvm -indirect-load-prefetch=<true|false>`

`-mllvm -outer-loop-prefetch=<true|false>`

`-mllvm -random-access-prefetch-only=<true|false>`

`-mllvm -disable-direct-prefetch=<true|false>`

`-mllvm -indirect-prefetch-skip-intermediate=<true|false>`

Enhances the compiler's prefetching capability by identifying multi-level indirect nested memory access patterns within applications. It automatically calculates the actual memory access address and inserts data prefetch instructions, thereby reducing data cache miss rates. This primarily includes:

- Indirect memory access prefetching

- CRC hash indirect memory access prefetching

The relationships between these options are detailed in the table below:

||inner loop direct load prefetch|inner loop indirect load prefetch|outer loop direct load prefetch|outer loop indirect load prefetch|
|-|-|-|-|-|
|default|√| | | |
|indirect-load-prefetch|√|√| | |
|outer-loop-prefetch|√| |√| |
|indirect-load-prefetch,outer-loop-prefetch|√|√|√|√|
|indirect-load-prefetch,outer-loop-prefetch,random-access-prefetch-only|√|√(hash indirect load type only)| |√ (hash indirect load type only)|
|indirect-load-prefetch,outer-loop-prefetch,disable-direct-prefetch| |√| |√|
|indirect-load-prefetch,outer-loop-prefetch,indirect-prefetch-skip-intermediate|√|√ (exclude B[i] prefetch in A[B[i]])|√|√ (exclude B[i] prefetch in A[B[i]])|

Default value: `false` for all of them

Supported scope: It currently supports C/C++ for the frontend, and AArch64 for the backend.

## `-mllvm -enable-aggressive-inline=<true|false>`

Ignores the `__attribute__((noinline))` restriction in the source code, forcing such functions to be treated as ordinary functions when evaluating whether to perform inline optimization.

Default value: `false`

Supported scope: It currently supports C/C++ for the frontend, and AArch64/x86 for the back end.

## `-mllvm -aarch64-ldp-stp-noq=<true|false>`

Prevents the generation of `stp/ldp q1, q2, addr` instructions on the Kunpeng tsv110 backend, as these instructions exhibit poor performance. Set to `true` to enable this optimization.

Default value: `true`

Supported scope: It currently supports C/C++ for the frontend, and only `-mcpu=tsv110` for the backend.


## `-fno-plt`

Conventional dynamic library function calls require jumping through the Procedure Linkage Table (PLT), resulting in extra memory accesses and branch instructions. Enabling this option optimizes calls to directly use the function address in the Global Offset Table (GOT), eliminating PLT jump overhead.

Supported scope: It currently supports C/C++ for the frontend, and AArch64/x86 for the backend.

## `-mllvm -no-sink-prtadd-post-load=<true|false>`

During compiler optimization, iteration operations on pointer-type loop induction variables (such as a `ptr++` operation inside a loop) are typically sunk to the end of the loop. If a load operation exists at the start of the next loop iteration and depends on the updated value of that pointer, a data dependency occurs, causing instruction stalls. This optimization avoids data-dependency-induced instruction stalls by properly scheduling instruction positions.

Default value: `false`

Supported scope: It currently supports C/C++ for the frontend, and AArch64/x86 for the backend.

## `-mllvm -aarch64-endianness-opts=<true|false>`

In common database applications like MySQL, data is stored in big-endian byte order. When loaded into memory for use, it must first be converted to little-endian byte order for subsequent computation. This typically requires a cyclic sequence of "byte-by-byte load," "shift," and "bitwise OR" operations, which incurs performance overhead. This optimization efficiently leverages the AArch64 REV instruction series and increases the load width to reduce both the number of load operations and the required shifting and bitwise OR operations, thereby boosting application performance.

Default value: `false`

Supported scope: It currently supports C/C++ for the frontend, and AArch64 for the backend.

## `-mllvm -disable-aarch64-lit-all=<true|false>`

Loop idiom optimization is used to identify common loop patterns in various applications and convert them into optimized loop implementations using flexible instruction capabilities such as AArch64 SVE, significantly boosting loop execution efficiency. Set to `true` to disable this optimization.

Default value: `false`

Supported scope: It currently supports C/C++ for the frontend, and AArch64 for the backend.

## `-mllvm -enable-loop-vectorize-prepare=<true|false>`

Adds a preprocessing optimization pass prior to loop vectorization to guide the vectorization policy for specific loops. For instance, on an AArch64 machine that supports SVE but not SVE2, certain NEON instructions offer higher flexibility than SVE instructions. Consequently, NEON vectorization delivers better performance than SVE vectorization in certain scenarios. This optimization guides the generation of NEON vectorization schemes.

Default value: `true`

Supported scope: It currently supports C/C++ for the frontend, and AArch64/x86 for the backend.

## `-mllvm -inline-memcpy-threshold=<num>`

For `memcpy` function calls where the copy length is statically unknown, inlining is typically not performed. This optimization enables the inlining of `memcpy` functions and configures a threshold: at runtime, if the checked copy length is smaller than the threshold, the inlined version of the `memcpy` implementation is invoked; otherwise, the standard `memcpy` function call is used. Set to `0` to disable this optimization.

Default value: `0`

Supported scope: It currently supports C/C++ for the frontend, and AArch64/x86 for the backend.

# LLVM for openEuler Feature Option List

## `-fgcc-compatible`

Enables compatibility features in LLVM for openEuler with the GCC compiler. This includes, but is not limited to, downgrading the severity of errors to warnings for unrecognized GCC functional options that the LLVM compiler cannot resolve, and providing compatibility support for GCC extended syntax.

> Supported version: LLVM 17/19

## `clang-tidy --checks='-*,BSCompatibility*' --export-details=output.yaml demo.c`

Supports utilizing the clang-tidy tool to identify specific GCC extended syntaxes that are incompatible with LLVM, generating warnings and providing modification suggestions that are output to `output.yaml`.

Additionally, the `share/clang/clang-tidy-stats.py` script is provided to export these modification suggestions into a tabular format.

`clang-tidy-stats.py --checks='-*,BSCompatibility*' --export-stats=stats.xlsx demo.c`

This script can directly perform detection and emit warnings, outputting the code information and modification suggestions straight into a `stats.xlsx` spreadsheet.

`clang-tidy-stats.py --args -export-stats=stats.xlsx -export-stats-input=output.yaml`

Alternatively, this script can convert an existing `output.yaml` file into a `stats.xlsx` spreadsheet.

> Supported version: LLVM 17/19

## `-mcpu=<CPU-name>`

Specifies the current CPU model via the `-mcpu=<CPU-name>` option to enable all hardware features supported by that CPU by default. After enabling the microarchitecture-affinity options, LLVM for openEuler tunes the instruction pipeline based on the instruction characteristics of the corresponding microarchitecture to boost performance.

The mapping between Kunpeng hardware platforms and their corresponding option configurations is as follows:

|Hardware Platform|Configuration Option|Supported Version|
|-|-|-|
|Kunpeng 920|tsv110|LLVM 17/18/19/20|
|HiSilicon HIP09|hip09|LLVM 17|
|HiSilicon HIP10c|hip10c|LLVM 17|
|HiSilicon HIP11|hip11|LLVM 17|
|950|hip12|**LLVM 17/18/19/20**|

## `export LLVM_PROFILE_RESET_SIGNUM=[40, 64]`

You can customize a signal via the `LLVM_PROFILE_RESET_SIGNUM` environment variable. When a running program that has profile-guided optimization (PGO) instrumentation enabled receives this signal, it clears the currently collected PGO data and restarts the collection process. This allows you to precisely control when the PGO data collection begins. The supported signal range is the closed interval `[40, 64]`.

> Supported version: LLVM 19

## `-fprofile-use-dir=<PATH>`

After collecting profile feedback data using PGO instrumentation, one or more profile data files are typically generated. Under normal workflows, an additional step using the `llvm-profdata merge` command is required to merge and convert these files into the format needed for LLVM PGO. This option bypasses that manual step. By simply specifying the profile directory, it automatically handles profile file merging and format conversion, streamlining the optimization workflow.

> Supported version: LLVM 19

## Compilation Time Optimization

ThinLTO (enabled via `-flto=thin`) is a widely used compilation optimization technique in the LLVM for openEuler compiler. While a conventional compilation workflow compiles each source file independently into a separate object file before linking them into an executable, it leaves information completely isolated between source files, preventing cross-file optimizations. In contrast, under ThinLTO mode, source files are first compiled into LLVM IR files. During linking, these files are aggregated to undergo deep cross-module optimization, mimicking a scenario where all source code resides within a single file, resulting in more comprehensive and thorough optimization.

However, ThinLTO drastically increases compilation time, an issue that becomes particularly acute when building large-scale applications. ThinLTO split addresses this by partitioning compilation units into smaller blocks during the build process to enable parallel compilation, thereby accelerating build efficiency.

`-flto=thin -fuse-ld=lld -Wl,-mllvm,-thinlto-split=true -Wl,-mllvm,-thinlto-split-partitions=<num>`

- `-fuse-ld=lld`: The ThinLTO Split feature requires the lld linker.
- `-Wl,-mllvm,-thinlto-split=<true|false>`: Enables the ThinLTO Split feature. Defaults to `false`.
- `-Wl,-mllvm,-thinlto-split-partitions=<num>`: Sets the number of parallel partitions. When this option is omitted, the compiler automatically determines an optimal number of partitions.

Enabling the ThinLTO Split feature causes debug information (Debuginfo) to bloat several-fold, which may lead to addressing overflow during the linking stage. In such cases, the dwarfutils tool can be utilized to eliminate redundant Debuginfo and reduce binary size.

`llvm-dwarfutils --garbage-collection <INPUT_BINARY> <OUTPUT_BINARY>`

> Note: The `--garbage-collection option` is used to eliminate redundant Debuginfo.

Supported scope: It currently supports C/C++ for the frontend, and AArch64/x86 for the backend.
