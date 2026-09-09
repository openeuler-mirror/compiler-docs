# GCC Base Performance Optimization Guide

## Overview

The optimization of compiler base performance is crucial to improving the development efficiency, running performance, and maintainability of applications. It is important in both computer science and software development. Based on the general compilation optimization capability, GCC for openEuler enhances middle- and back-end performance optimization technologies, including instruction optimization, vectorization enhancement, prefetch enhancement, and data flow analysis enhancement.

## Toolchain Applicability

The features in this guide apply to the system GCC (gcc 12.3.1) and to gcc-toolset-14 with the openEuler optimization stack merged (see [Alternative GCC 14 User Guide](gcc_14_secondary_version_compilation_toolchain_user_guide.md)), with the following exceptions.

Supported by the system GCC only:

- `-mcmlt-arith`
- `-fconvert-minmax`
- `-fsplit-ldp-stp` and `--param=param-ldp-dependency-search-range`
- `-fllc-allocate` and its associated parameters
- `-ftree-slp-transpose-vectorize`
- `--param=vect-alias-flexible-segment-len`

The following system GCC options are retired in gcc-toolset-14. Their positive forms are rejected with an error; remove or replace them in build scripts:

| System GCC option | Behavior in gcc-toolset-14 |
| ---- | ---- |
| `-falias-analysis-expand-ssa` | Rejected; the non-loop alias disambiguation it enabled is built in and always active |
| `-fchrec-mul-fold-strict-overflow` | Rejected; the chain-of-recurrences multiplication folding it enabled is built in and unconditional |
| `-fmerge-mull` | Rejected; use `-mmul-widen128` |
| `-fftz` | Rejected; use `-mdaz-ftz` |
| `-fp-model=` | Rejected; use `-ffp-model=` |

Note: The `-fno-` forms of the first two options are silently accepted.

## Installation and Deployment

### Software Requirements

OS: openEuler 25.03

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

Note: The `-floop-sve-mode-opt` option can be enabled only when `-O3` is enabled and SVE is included in the `-march` setting. The optimization takes effect on AArch64 only.

### -fif-split

#### Description

Splits a composite condition of the form "variable compared with a constant || other condition" and clones the branch body, so IPA constant propagation can propagate the constant into function calls.

#### How to Use

Add the `-fif-split` option.

Note: The `-fif-split` option requires `-O3` to be enabled.

### Vectorization Analysis Parameters

#### Description

Three independent vectorization analysis switches.

#### How to Use

| Option | Default | Description |
| ---- | ---- | ---- |
| `--param=vect-swap-operands=[0,1]` | 0 | Allows swapping operands of commutative operations in vectorization analysis. |
| `--param=addr-expand-for-alias-check=[0,1]` | 0 | Expands data reference addresses for alias checks. |
| `--param=vect-register-size-check=[0,1]` | 0 | Checks whether a group of interleaved memory accesses exceeds vector register capacity. |

### -ftree-slp-late

#### Description

Runs an additional SLP vectorization pass after reassociation.

#### How to Use

Add the `-ftree-slp-late` option. Disabled by default.

### `-floop-elim`

#### Description

Eliminates redundant loops.

#### How to Use

Add the `-floop-elim` option.

Note: Runs as part of phiopt and depends on `-fssa-phiopt` (enabled by default at `-O1` and higher).

### -farray-widen-compare

#### Description

Rewrites qualifying byte-wise compare loops to compare eight bytes at a time.

#### How to Use

Add the `-farray-widen-compare` option.

Note: Requires `-O3`; effective only on little-endian targets; only the LP64 data model is supported. The compared arrays must remain readable up to the loop bound: the widened 8-byte loads may read bytes past an early exit that the byte-wise loop would never access, so violating this precondition risks out-of-bounds reads.

### `-fbuiltin-will-return`

#### Description

Treats built-ins that cannot loop, throw, or exit (currently only `__builtin_prefetch`) as calls that always return.

#### How to Use

Add the `-fbuiltin-will-return` option.

### `-mmul-widen128`

#### Description

Recognizes 64-bit to 128-bit widening multiplication idioms and generates efficient instruction sequences.

#### How to Use

Add the `-mmul-widen128` option.

Note: AArch64 only. The corresponding system GCC option is `-fmerge-mull`.

### Vector Math Library and Floating-Point Control

#### Description

Vector math library support and floating-point behavior control.

#### How to Use

- `-fsimdmath`: declares vector variants of math functions (`_ZGV*`) and injects libmathlib at link time.
- `-ffp-model=`: floating-point model control; values: normal/fast/precise/except/strict.
- `-mdaz-ftz`: sets the FTZ/DAZ flags in the floating-point control register at startup, flushing subnormal values to zero.

Note: The corresponding system GCC spellings of `-ffp-model=` and `-mdaz-ftz` are `-fp-model=` and `-fftz`.

### CFGO / CSPGO

#### Description

CFGO enables a bundle of openEuler optimizations alongside PGO profile use; CSPGO instruments a second time after inlining so profile counts carry call context. gcc-toolset-14 provides the same capability.

#### How to Use

For the options and the full optimization flow, see the [CFGO User Guide](cfgo_user_guide.md). In addition, `-fcfgo-csprofile-dir=<dir>` sets the CSPGO profile directory on its own.

Note: The CFGO bundle does not include `-fselective-scheduling`. The option is still accepted, but combining it with profile instrumentation (`-fcfgo-profile-generate`/`-fcfgo-csprofile-generate`) carries a known miscompilation risk (statements observed silently dropped on AArch64 with no diagnostic); do not combine them.

### oeAware Co-Optimization

#### Description

Emits an ELF section read by the oeAware runtime tuning framework.

#### How to Use

Add the `-foeaware-policy` option; `-foeaware-policy=[1,7]` selects the optimization policy (default: 1).

### AutoBOLT

#### Description

Writes function and branch information (derived from AutoFDO/PGO profiles) into an ELF section at compile time; after linking, bolt-plugin collects it and performs BOLT binary optimization.

#### How to Use

Add the `-fauto-bolt` option; `-fauto-bolt=<dir>` specifies the profile data directory (default: current directory).

Note: AArch64 only. Not supported with `-flto` (compilation error) and mutually exclusive with `-fbolt-use`; it therefore cannot be combined with the Struct-Reorg family options in this guide, which require `-flto`.
