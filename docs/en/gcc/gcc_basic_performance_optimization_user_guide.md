# GCC Basic Performance Optimization User Guide

## Introduction

Compiler performance optimization plays a vital role in enhancing application development efficiency, runtime performance, and maintainability. It is a significant research area in computer science and a critical component of the software development process. GCC for openEuler extends its general compilation optimization capabilities by improving backend performance techniques, including instruction optimization, vectorization enhancements, prefetching improvements, and data flow analysis optimizations.

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

OS: openEuler 24.09

### Hardware Requirements

AArch64 architecture

### Software Installation

Install GCC and related components as required. For example, to install GCC:

```shell
yum install gcc
```

## Usage

### CRC Optimization

#### Description

Detects CRC software loop code and generates efficient hardware instructions.

#### Usage

Include the `-floop-crc` option during compilation.

Note: The `-floop-crc` option must be used alongside `-O3 -march=armv8.1-a`.

### If-Conversion Enhancement

#### Description

Improves If-Conversion optimization by leveraging additional registers to minimize conflicts.

#### Usage

This optimization is part of the RTL if-conversion process. Enable it using the following compilation options:

`-fifcvt-allow-complicated-cmps`

`--param=ifcvt-allow-register-renaming=[0,1,2]` (The value controls the optimization scope.)

Note: This optimization requires the `-O2` optimization level and should be used with `--param=max-rtl-if-conversion-unpredictable-cost=48` and `--param=max-rtl-if-conversion-predictable-cost=48`.

### Multiplication Calculation Optimization

#### Description

Optimizes Arm-related instruction merging to recognize 32-bit complex combinations of 64-bit integer multiplication logic and produce efficient 64-bit instructions.

#### Usage

Enable the optimization using the `-fuaddsub-overflow-match-all` and `-fif-conversion-gimple` options.

Note: This optimization requires `-O3` or higher optimization levels.

### cmlt Instruction Generation Optimization

#### Description

Generates `cmlt` instructions for specific arithmetic operations, reducing the instruction count.

#### Usage

Enable the optimization using the `-mcmlt-arith` option.

Note: This optimization requires `-O3` or higher optimization levels.

### Vectorization Optimization Enhancement

#### Description

Identifies and simplifies redundant instructions generated during vectorization, enabling shorter loops to undergo vectorization.

#### Usage

Enable the optimization using the parameter `--param=vect-alias-flexible-segment-len=1` (default is 0).

Note: This optimization requires `-O3` or higher optimization levels.

### Combined Optimization of min max and uzp1/uzp2 Instructions

#### Description

Identifies opportunities to optimize `min max` and `uzp1/uzp2` instructions together, reducing the instruction count to enhance performance.

#### Usage

Enable `min max` optimization with the `-fconvert-minmax` option. The `uzp1/uzp2` instruction optimization is automatically enabled at `-O3` or higher levels.

Note: This optimization requires `-O3` or higher optimization levels.

### ldp/stp Optimization

#### Description

Detects poorly performing `ldp/stp` instructions and splits them into two separate `ldr` and `str` instructions.

#### Usage

Enable the optimization using the `-fsplit-ldp-stp` option. Control the search range with the parameter `--param=param-ldp-dependency-search-range=[1,32]` (default is 16).

Note: This optimization requires `-O1` or higher optimization levels.

### AES Instruction Optimization

#### Description

Identifies AES software algorithm instruction sequences and replaces them with hardware instructions for acceleration.

#### Usage

Enable the optimization using the `-fcrypto-accel-aes` option.

Note: This optimization requires `-O3` or higher optimization levels.

### Indirect Call Optimization

#### Description

Analyzes and optimizes indirect calls in the program, converting them into direct calls where possible.

#### Usage

Enable the optimization using the `-ficp -ficp-speculatively` options.

Note: This optimization must be used with `-O2 -flto -flto-partition=one`.

### IPA-prefetch

#### Description

Detects indirect memory accesses in loops and inserts prefetch instructions to minimize latency.

#### Usage

Enable the optimization using the `-fipa-prefetch -fipa-ic` options.

Note: This optimization must be used with `-O3 -flto`.

### -fipa-struct-reorg

#### Description

Optimizes memory layout by reorganizing the arrangement of structure members to improve cache hit rates.

#### Usage

Add the options `-O3 -flto -flto-partition=one -fipa-struct-reorg` to enable the optimization.

Note: The `-fipa-struct-reorg` option requires `-O3 -flto -flto-partition=one` to be enabled globally.

### -fipa-reorder-fields

#### Description

Optimizes memory layout by reordering structure members from largest to smallest, reducing padding and improving cache hit rates.

#### Usage

Add the options `-O3 -flto -flto-partition=one -fipa-reorder-fields` to enable the optimization.

Note: The `-fipa-reorder-fields` option requires `-O3 -flto -flto-partition=one` to be enabled globally.

### -ftree-slp-transpose-vectorize

#### Description

Enhances data flow analysis for loops with consecutive memory reads by inserting temporary arrays during loop splitting. During SLP vectorization, it introduces transposition analysis for `grouped_stores`.

#### Usage

Add the options `-O3 -ftree-slp-transpose-vectorize` to enable the optimization.

Note: The `-ftree-slp-transpose-vectorize` option requires `-O3` to be enabled.

### -fif-split

#### Description

Splits a composite condition of the form "variable compared with a constant || other condition" and clones the branch body, so IPA constant propagation can propagate the constant into function calls.

#### Usage

Add the `-fif-split` option.

Note: The `-fif-split` option requires `-O3` to be enabled.

### Vectorization Analysis Parameters

#### Description

Three independent vectorization analysis switches.

#### Usage

| Option | Default | Description |
| ---- | ---- | ---- |
| `--param=vect-swap-operands=[0,1]` | 0 | Allows swapping operands of commutative operations in vectorization analysis. |
| `--param=addr-expand-for-alias-check=[0,1]` | 0 | Expands data reference addresses for alias checks. |
| `--param=vect-register-size-check=[0,1]` | 0 | Checks whether a group of interleaved memory accesses exceeds vector register capacity. |

### -ftree-slp-late

#### Description

Runs an additional SLP vectorization pass after reassociation.

#### Usage

Add the `-ftree-slp-late` option. Disabled by default.

### `-floop-elim`

#### Description

Eliminates redundant loops.

#### Usage

Add the `-floop-elim` option.

Note: Runs as part of phiopt and depends on `-fssa-phiopt` (enabled by default at `-O1` and higher).

### -floop-sve-mode-opt

#### Description

Vectorizes certain loops in SVE mode.

#### Usage

Add the `-floop-sve-mode-opt` option.

Note: Effective only on AArch64 with SVE enabled.

### -farray-widen-compare

#### Description

Rewrites qualifying byte-wise compare loops to compare eight bytes at a time.

#### Usage

Add the `-farray-widen-compare` option.

Note: Requires `-O3`; effective only on little-endian targets; only the LP64 data model is supported. The compared arrays must remain readable up to the loop bound: the widened 8-byte loads may read bytes past an early exit that the byte-wise loop would never access, so violating this precondition risks out-of-bounds reads.

### `-fbuiltin-will-return`

#### Description

Treats built-ins that cannot loop, throw, or exit (currently only `__builtin_prefetch`) as calls that always return.

#### Usage

Add the `-fbuiltin-will-return` option.

### `-mmul-widen128`

#### Description

Recognizes 64-bit to 128-bit widening multiplication idioms and generates efficient instruction sequences.

#### Usage

Add the `-mmul-widen128` option.

Note: AArch64 only. The corresponding system GCC option is `-fmerge-mull`.

### Vector Math Library and Floating-Point Control

#### Description

Vector math library support and floating-point behavior control.

#### Usage

- `-fsimdmath`: declares vector variants of math functions (`_ZGV*`) and injects libmathlib at link time.
- `-ffp-model=`: floating-point model control; values: normal/fast/precise/except/strict.
- `-mdaz-ftz`: sets the FTZ/DAZ flags in the floating-point control register at startup, flushing subnormal values to zero.

Note: The corresponding system GCC spellings of `-ffp-model=` and `-mdaz-ftz` are `-fp-model=` and `-fftz`.

### CFGO / CSPGO

#### Description

CFGO enables a bundle of openEuler optimizations alongside PGO profile use; CSPGO instruments a second time after inlining so profile counts carry call context.

#### Usage

- CFGO instrumentation: `-fcfgo-profile-generate[=<dir>]`; use: `-fcfgo-profile-use[=<dir>]`.
- CSPGO instrumentation: `-fcfgo-csprofile-generate[=<dir>]`; use: `-fcfgo-csprofile-use[=<dir>]`; directory: `-fcfgo-csprofile-dir=<dir>`.

Note: The CFGO bundle does not include `-fselective-scheduling`. The option is still accepted, but combining it with profile instrumentation (`-fcfgo-profile-generate`/`-fcfgo-csprofile-generate`) carries a known miscompilation risk (statements observed silently dropped on AArch64 with no diagnostic); do not combine them.

### oeAware Co-Optimization

#### Description

Emits an ELF section read by the oeAware runtime tuning framework.

#### Usage

Add the `-foeaware-policy` option; `-foeaware-policy=[1,7]` selects the optimization policy (default: 1).

### AutoBOLT

#### Description

Writes function and branch information (derived from AutoFDO/PGO profiles) into an ELF section at compile time; after linking, bolt-plugin collects it and performs BOLT binary optimization.

#### Usage

Add the `-fauto-bolt` option; `-fauto-bolt=<dir>` specifies the profile data directory (default: current directory).

Note: AArch64 only. Not supported with `-flto` (compilation error) and mutually exclusive with `-fbolt-use`; it therefore cannot be combined with the Struct-Reorg family options in this guide, which require `-flto`.
