# CFGO User Guide

## Overview

Feedback-directed optimization (FDO), also known as `profile-guided optimization (PGO)`, is a compilation technique that optimizes program performance by collecting runtime information during program execution.

`Continuous Feature Guided Optimization (CFGO)` in `GCC for openEuler` and `BiSheng Compiler` refers to continuous feedback-directed optimization for multimodal files (source code and binaries) and the full lifecycle (compilation, linking, post-linking, runtime, OS, and libraries).

Major optimizations:

* Code layout optimization: Techniques such as basic block reordering, function rearrangement, and hot/cold separation are used to optimize the binary layout of the target program, improving I-cache and I-TLB hit rates.
* Advanced compiler optimization: Techniques such as inlining, loop unrolling, vectorization, and indirect calls enable the compiler to make more accurate optimization decisions.

This section mainly introduces the static feedback optimization technologies in CFGO.

## Installation and Deployment

### Software Requirements

* OS: openEuler 24.03 LTS SP1
* Compiler: GCC 12.3.1-47 or later

## How to Use

CFGO consists of three optimization techniques: `CFGO-PGO`, `CFGO-CSPGO`, and `CFGO-BOLT`. `CFGO-PGO` is primarily used to enable optimizations such as inlining, constant propagation, and devirtualization. `CFGO-CSPGO` is built on `CFGO-PGO` and uses the updated profiles to perform optimizations such as basic block reordering and register allocation. `CFGO-BOLT` is used for post-link binary code layout optimization.

To further enhance the optimization, you are advised to add the `-flto=auto` compilation option during `CFGO-PGO` and `CFGO-CSPGO` processes.

To achieve the optimal optimization, the techniques should be applied in the following order: `CFGO-PGO` -> `CFGO-CSPGO` -> `CFGO-BOLT`.

### 1. `CFGO-PGO`

Feature description: Based on open-source PGO, `AI4C` is used to tune some optimization options and parameters, further improving performance.

Option: `-fcfgo-profile-generate=${path}`

Description: Enables compilation and instrumentation to generate an instrumented executable program. If `${path}` is specified, the profiling data files of the program are written to the specified path. Otherwise, they will be generated in the current directory.

Option: `-fcfgo-profile-use=${path}`

Description: Uses the profiling data of the program to perform compilation optimization. If `${path}` is specified, the profiling data files will be read from the specified path. Otherwise, they will be read from the current directory.

Typical workflow:

```bash
// 1. Compilation and instrumentation
gcc -fcfgo-profile-generate=${path} test.c -o test

// 2. Profiling
./test

// 3. Using the profile
gcc -fcfgo-profile-use=${path} test.c -o test
```

### 2. `CFGO-CSPGO`

Feature description: The profile in conventional PGO is context-insensitive, which may result in suboptimal optimization. By adding an additional `CFGO-CSPGO` instrumentation phase after PGO, runtime information from the inlined program is collected. This provides more accurate execution data for code layout and register optimizations, leading to enhanced performance.

Option: `-fcfgo-csprofile-generate=${path}`

Description: Enables post-inline compilation and instrumentation. This option must be used together with `-fcfgo-profile-use`. `${path}` must be specified and cannot be the same as the path used in the `CFGO-PGO` phase.

Option: `-fcfgo-csprofile-use=${path}`

Description: Uses the post-inline profile to optimize the program. This option must be used together with `-fcfgo-profile-use`. `${path}` must be specified and be the same as the path specified in `-fcfgo-csprofile-generate=${path}`.

Typical workflow:

```bash
// 1. Compilation and instrumentation
gcc -fcfgo-profile-use={path_1} -fcfgo-csprofile-generate=${path_2} test.c -o test

// 2. Profiling
./test

// 3. Using the profile
gcc -fcfgo-profile-use={path_1} -fcfgo-csprofile-use=${path_2} test.c -o test
```

### 3. `CFGO-BOLT`

Feature description: BOLT is a post-link binary code layout optimizer. This feature uses advanced binary instrumentation capabilities to collect more accurate runtime information than sampling-based methods, enabling optimized binary code layout and improving overall performance.

Option: `-instrument`

Description: Enables software instrumentation to collect runtime information of the program. This option requires that the binary contains relocation information (for example, added `-Wl,-q` during compilation).

In addition to this option, BOLT instrumentation supports other related options. For details, see `BOLT instrumentation options` displayed by `llvm-bolt --help`.

Typical workflow:

```bash
// 1. Retaining relocation information during program compilation
gcc -Wl,-q test.c -o test

// 2. Compilation and BOLT instrumentation
llvm-bolt ./test -instrument -o test.inst -instrumentation-file=${test.fdata} --instrumentation-wait-forks --instrumentation-sleep-time=2 --instrumentation-no-counters-clear

// 3. Profiling
./test.inst

// 4. Using the profile
llvm-bolt ./test -o test.opt -data=test.fdata
```

## Compatibility

This section describes the compatibility issues in some special scenarios. This project is in continuous iteration and issues will be fixed as soon as possible. Developers are welcome to join this project.

* Currently, `CFGO-CSPGO` does not support dynamic library optimization.
