# GCC Base Performance Optimization Guide

## Overview

The optimization of compiler base performance is crucial to improving the development efficiency, running performance, and maintainability of applications. It is important in both computer science and software development. Based on the general compilation optimization capability, GCC for openEuler enhances middle- and back-end performance optimization technologies, including instruction optimization, vectorization enhancement, prefetch enhancement, and data flow analysis enhancement.

## Installation and Deployment

### Software Requirements

OS: openEuler 24.03 LTS SP3

### Hardware Requirements

AArch64 architecture

### Software Installation

Install GCC and related components as needed. The following uses GCC as an example:

```shell
yum install gcc
```

## How to Use

### Optimization for CRC

#### Description

GCC identifies cyclic redundancy check (CRC) code and generates efficient hardware instructions.

#### How to Use

Add `-floop-crc` during compilation.

Note: `-floop-crc` must be used together with `-O3 -march=armv8.1-a`.

### If-conversion Enhancement

#### Description

If-conversion optimization is enhanced by using more registers to reduce conflicts.

#### How to Use

This enhancement is part of Register Transfer Language (RTL) if-conversion optimization. Enable the enhancement by using the following options:

`-fifcvt-allow-complicated-cmps`

`--param=ifcvt-allow-register-renaming=[0,1,2]`, where the numbers are used to control the optimization scope

Note: This optimization requires the `-O2` optimization level and must be used together with `--param=max-rtl-if-conversion-unpredictable-cost=48` and `--param=max-rtl-if-conversion-predictable-cost=48`.

### Optimization for Multiplication

#### Description

Arm instructions are combined to convert low-order 32-bit multiplications into high-order 64-bit multiplication instructions.

#### How to Use

Use the `-fuaddsub-overflow-match-all` and `-fif-conversion-gimple` options.

Note: This optimization requires the `-O3` or higher optimization level.

### Optimization for CMLT Instruction Generation

#### Description

`cmlt` instructions are generated for some elementary arithmetic operations to reduce the number of instructions.

#### How to Use

Use the `-mcmlt-arith` option.

Note: This optimization requires the `-O3` or higher optimization level.

### Optimization for Vectorization

#### Description

Redundant instructions generated during vectorization are identified and simplified, and shorter arrays can be vectorized.

#### How to Use

Use `--param=vect-alias-flexible-segment-len=1`. The default value is `0`.

Note: This optimization requires the `-O3` or higher optimization level.

### Optimization for min max and uzp1/uzp2 Instructions

#### Description

The min max and uzp1/uzp2 instructions are optimized to reduce the total instructions and improve performance.

#### How to Use

Use the `-fconvert-minmax` option to enable `min max` optimization. `uzp1/uzp2` instruction optimization is enabled by default at a level higher than `-O3`.

Note: This optimization requires the `-O3` or higher optimization level.

### Optimization for LDP and STP

#### Description

Each LDP and STP instruction with poor performance is split into two LDR and STR instructions.

#### How to Use

Use the `-fsplit-ldp-stp` option. Use `--param=param-ldp-dependency-search-range=[1,32]` to control the search range. The default value is `16`.

Note: This optimization requires the `-O1` or higher optimization level.

### Optimization for AES Instruction

#### Description

The AES software instruction sequences are identified and accelerated using hardware instructions.

#### How to Use

Use the `-fcrypto-accel-aes` option.

Note: This optimization requires the `-O3` or higher optimization level.

### Optimization for Indirect Calls

#### Description

Indirect calls in programs are identified, analyzed, and then optimized into direct calls.

#### How to Use

Use the `-ficp -ficp-speculatively` option.

Note: This optimization must be used together with `-O2 -flto -flto-partition=one`.

### IPA-prefetch

#### Description

Indirect memory accesses in a loop are identified, and a prefetch instruction is inserted to reduce the delay.

#### How to Use

Use the `-fipa-prefetch -fipa-ic` option.

Note: This optimization must be used together with `-O3 -flto`.

### -fipa-struct-reorg

#### Description

This option optimizes memory layout. The structure members are rearranged in memory to improve the cache hit rate.

#### How to Use

Add `-O3 -flto -flto-partition=one -fipa-struct-reorg` to the option.

Note: The `-fipa-struct-reorg` option can be enabled only when `-O3 -flto -flto-partition=one` is enabled globally.

### -fipa-reorder-fields

#### Description

The memory space layout is optimized by arranging structure members from largest to smallest based on their size. This reduces padding caused by alignment boundaries, decreases overall memory usage, and improves the cache hit rate.

#### How to Use

Add `-O3 -flto -flto-partition=one -fipa-reorder-fields` to the option.

Note: The `-fipa-reorder-fields` option can be enabled only when `-O3 -flto -flto-partition=one` is enabled globally.

### -ftree-slp-transpose-vectorize

#### Description

In the loop splitting phase, temporary arrays are introduced to partition the loop, which enhances data-flow analysis for loops that read continuous memory. In the superword-level parallelism (SLP) vectorization phase, SLP analysis is performed for transposing grouped_stores.

#### How to Use

Add `-O3 -ftree-slp-transpose-vectorize` to the option.

Note: The `-ftree-slp-transpose-vectorize` option can be enabled only when `-O3` is enabled.

### LLC-prefetch

#### Description

In main execution paths of programs, memory-reuse patterns in loops are analyzed to identify and rank top hot data. The prefetch instruction is introduced to allocate the data to last-level cache (LLC), reducing LLC misses.

#### How to Use

Use the `-fllc-allocate` option. The `-O2` or higher optimization level is required.

Other related interfaces:

|  Option  | Default Value | Description |
|  ----  | ----  | ----  |
| --param=mem-access-ratio=[0,100]  | 20 | Ratio of the number of memory accesses in a loop to the number of instructions.|
| --param=mem-access-num=unsigned  | 3 | Number of memory accesses in a loop. |
| --param=outer-loop-nums=[1,10]  |  1 | Maximum number of outer loop layers that can be unrolled. |
| --param=filter-kernels=[0,1]  |  1 | Indicates whether to perform path series filtering on loops. |
| --param=branch-prob-threshold=[50,100]  |  80 | Probability threshold for a branch to be considered highly probable. |
| --param=prefetch-offset=[1,999999]  | 1024  |  Prefetch offset distance, where the value is a power of 2.|
| --param=issue-topn=unsigned  |  1 |  Number of prefetch instructions.|
| --param=force-issue=[0,1]  |  0 |  Indicates whether to perform forcible prefetch, that is, the static mode.|
| --param=llc-capacity-per-core=[0,999999]  |  107 | Average LLC capacity allocated to each core in multi-branch prefetch mode. |

### -fipa-struct-sfc

#### Description

This option is used to statically compress structure members to reduce the structure size and improve the cache hit rate.

#### How to Use

Add `-O3 -flto -flto-partition=one -fipa-reorder-fields -fipa-struct-sfc` to the option. You can use `-fipa-struct-sfc-bitfield` and `-fipa-struct-sfc-shadow` for further optimization.

Note: The `-fipa-struct-sfc` option can be enabled only when `-O3 -flto -flto-partition=one` is enabled globally and `-fipa-reorder-fields` or `-fipa-struct-reorg>=2` is enabled.

### -fipa-struct-dfc

#### Description

This option is used to dynamically compress structure members by cloning the program path and heuristically minimizing the structure size. At runtime, it improves the cache hit rate by checking execution paths and selecting the optimal one.

#### How to Use

Add `-O3 -flto -flto-partition=one -fipa-reorder-fields -fipa-struct-dfc` to the option. You can use `-fipa-struct-dfc-bitfield` and `-fipa-struct-dfc-shadow` for further optimization.

Note: The `-fipa-struct-dfc` option can be enabled only when `-O3 -flto -flto-partition=one` is enabled globally and `-fipa-reorder-fields` or `-fipa-struct-reorg>=2` is enabled.

### -fipa-alignment-propagation

#### Description

This option is used to analyze and propagate the address-alignment values for local variables, optimizing the bitwise AND operations.

#### How to Use

Add `-O3 -fipa-alignment-propagation` to the option.

Note: The `-fipa-alignment-propagation` option can be enabled only when `-O3` is enabled.

### -fipa-localize-array

#### Description

This option is used to convert the global pointer variables allocated by calloc to local variables.

#### How to Use

Add `-O3 -fipa-localize-array` to the option.

Note: The `-fipa-localize-array` option can be enabled only when `-O3` is enabled.

### -fipa-array-dse

#### Description

This option is used to analyze the transfer of arrays between functions and the usage of the arrays in the called functions, removing redundant array writes.

#### How to Use

Add `-O3 -fipa-array-dse` to the option.

Note: The `-fipa-array-dse` option can be enabled only when `-O3` is enabled.

### -ffind-with-sve

#### Description

This option is used to identify `std::find` function calls and attempt to optimize them using SVE instructions.

#### How to Use

Add `-ffind-with-sve` to the option.

### -floop-sve-mode-opt

#### Description

By analyzing static code characteristics, special scenarios can be identified. When the conditions are met, additional optimization opportunities leveraging the SVE instruction set are introduced, resulting in improved performance.

#### How to Use

Add `-O3 -floop-sve-mode-opt` to the option.

Note: The `-floop-sve-mode-opt` option can be enabled only when `-O3` is enabled and SVE is included in the `-march` setting.

### SVE Memcall Inline Optimization

#### Description

This optimization replaces `memcpy` and `memset` calls with inline SVE loops, reducing library call overhead and improving the performance of small memory operations. It does not support `memmove` and is unsuitable for workloads with frequent large memory operations.

#### How to Use

Add `-march=armv8-a+sve -maarch64-sve-memcall-inlining` to the compilation options.

The related options and parameters are as follows:

| Option or Parameter | Default | Description |
| --- | --- | --- |
| `-maarch64-sve-memcall-inlining` | Disabled | Enables SVE memcall inline optimization |
| `-mno-aarch64-sve-memcall-inlining` | — | Disables SVE memcall inline optimization |
| `--param=aarch64-sve-memcall-size-threshold=N` | 4096 | Sets the length threshold for SVE inlining, in bytes |
| `-maarch64-sve-memcall-runtime-check` | Enabled | When the main option is enabled, checks the run-time length and falls back to a library call when the length exceeds the threshold |
| `-mno-aarch64-sve-memcall-runtime-check` | — | Disables the run-time length check, directly inlining variable-length operations as SVE loops |

Note: Adjust the threshold based on the size distribution of `memcpy` and `memset` operations in the actual workload so that large memory operations fall back to library calls.
