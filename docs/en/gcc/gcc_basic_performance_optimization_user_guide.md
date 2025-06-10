# GCC Basic Performance Optimization User Guide

## Introduction

Compiler performance optimization plays a vital role in enhancing application development efficiency, runtime performance, and maintainability. It is a significant research area in computer science and a critical component of the software development process. GCC for openEuler extends its general compilation optimization capabilities by improving backend performance techniques, including instruction optimization, vectorization enhancements, prefetching improvements, and data flow analysis optimizations.

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
