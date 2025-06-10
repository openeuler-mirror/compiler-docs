# Background

## Overview

OSs prioritize robustness by adopting time-tested, stable software versions rather than the latest releases. This strategy minimizes instability risks from version changes and maintains system stability throughout the LTS cycle. Consequently, openEuler has chosen GCC 12.3.1 as its baseline for the entire 24.03 LTS lifecycle.

This decision, however, introduces challenges. Many hardware features rely on the foundational GCC toolchain, and using an older GCC version delays the activation of new features in OS releases. Additionally, some users prefer the latest compiler versions to unlock new capabilities, which often deliver performance gains over older versions.

To enable diverse computational features and cater to varying user needs for hardware support, openEuler 24.09 introduces the openEuler GCC Toolset. This multi-version GCC compilation toolchain, tailored for openEuler, provides a secondary GCC version higher than the system primary version, offering users a more flexible and efficient compilation environment. With the openEuler GCC Toolset 14, users can seamlessly switch between GCC versions to leverage new hardware features and benefit from the latest GCC optimizations.

## Solution Design

### Compilation Toolchain Features

The GCC compilation toolchain, developed and maintained by GNU, is a collection of open source tools designed to translate high-level language code into machine language. Beyond GCC itself, it includes a range of auxiliary tools and libraries, collectively forming a comprehensive compilation environment.

1. GCC compiler (such as `gcc`, `g++`, and `gfortran`):

    - Role: The GCC compiler is the heart of the toolchain, handling preprocessing and compilation to transform source code into assembly or intermediate representation. For C++ code, `g++` acts as the C++ frontend, performing compilation and automatically linking C++ standard libraries.

2. Binutils toolset:

    - Tools: including the linker (`ld`), assembler (`as`), object file viewer (`readelf`), symbol viewer (`nm`), object file format converter (`objcopy`), disassembler (`objdump`), and size viewer (`size`)
    - Role: These tools support the compilation process by converting assembly to machine code, linking object files into executables, and inspecting file details.

3. glibc library:

    - Role: The GNU C Library (glibc) is the standard C library for GNU and Linux systems, providing essential functions like `printf` and `malloc` required for compiling C programs.

4. Other auxiliary tools:

    - Debugger (`gdb`): assists developers in debugging executables to identify and fix errors.
    - Performance Analysis Tool (`gprof`): helps analyze and optimize program performance.

### Toolchain Selection

The software components in the toolchain significantly influence compilation results, with GCC, binutils, and glibc being the core elements. Since glibc, the C standard library, is tightly coupled with the OS kernel version, it remains unchanged. This toolchain includes only GCC and binutils to fulfill the needs of secondary version compilation.

The latest GCC release, gcc-14.2.0, is selected for the openEuler GCC toolset. For binutils, while openEuler 24.09 defaults to version 2.41, the latest GCC 14 recommends binutils-2.42. Thus, binutils-2.42 is chosen for this toolchain.

The openEuler GCC toolset incorporates gcc-14.2.0 and binutils-2.42 as the secondary version toolchain to ensure compilation environment stability and efficiency while minimizing complexity. This approach balances compilation quality and user experience. The toolchain GCC version will be updated to gcc-14.3.0 upon its release by the upstream community.

### Architecture Design

To differentiate from the default toolchain and prevent conflicts, this toolchain is named gcc-toolset-14. Its package names begin with the prefix `gcc-toolset-14-`, followed by the original toolchain package name. To avoid path overlap with the default **/usr** installation path, gcc-toolset-14 is installed in **/opt/openEuler/gcc-toolset-14/**. Additionally, to distinguish it from open source GCC and enable future integration of openEuler community features, the version of gcc-toolset-14-gcc is set to 14.2.1.

The applications and libraries in gcc-toolset-14 coexist with the system default GCC version without replacing or overwriting it. They are not set as the default or preferred option. To simplify version switching and management, the scl-utils tool is introduced. Its usage and switching methods are outlined below.

## Installation and Deployment

### Software Requirements

- OS: openEuler 24.09

### Hardware Requirements

- AArch64 or X86_64

### Installation Methods

Install the default GCC compiler, gcc-12.3.1, in **/usr**:

```shell
yum install -y gcc gcc-c++
```

Install the secondary version compilation toolchain, gcc-toolset-14, in **/opt/openEuler/gcc-toolset-14/root/usr/**:

```shell
yum install -y gcc-toolset-14-gcc*
yum install -y gcc-toolset-14-binutils*
```

## Usage

This solution uses the SCL (Software Collections) tool to manage different versions of the compilation toolchain.

### SCL

SCL is a vital Linux tool that enables users to install and use multiple versions of applications and runtime environments safely and conveniently, preventing system conflicts.

Key benefits of SCL include:

1. Multi-version coexistence: allows installing and using multiple versions of software libraries, tools, and runtime environments on the same system to meet diverse needs.
2. Avoiding system conflicts: isolates different software versions to prevent conflicts with the system default version.
3. Enhancing development efficiency: provides developers with the latest toolchains and runtime environments, improving productivity.

### Version Switching Methods

**Install SCL:**

```shell
yum install scl-utils scl-utils-build
```

**Register gcc-toolset-14:**

```shell
## Register gcc-toolset-14.
scl register /opt/openEuler/gcc-toolset-14/

## Deregister gcc-toolset-14.
scl deregister gcc-toolset-14
```

Use `scl list-collections` to verify that gcc-toolset-14 is successfully registered.

**Switch to gcc-toolset-14:**

```shell
scl enable gcc-toolset-14 bash
```

This command launches a new bash shell session with tools from gcc-toolset-14, replacing the system defaults. In this session, there is no need to manually switch compiler versions or paths. To exit the gcc-toolset-14 environment, type `exit` to return to the system default version.

SCL works by automatically setting environment variables for different tool versions. For details, check the **/opt/openEuler/gcc-toolset-14/enable** file, which contains all environment variable configurations for gcc-toolset-14. If SCL is unavailable, use the following methods to switch toolchain versions:

```shell
## Option 1: Without SCL, use a script to switch the compilation toolchain.
source /opt/openEuler/gcc-toolset-14/enable

## Option 2: With SCL, use SCL to switch the toolchain and activate the runtime environment.
scl enable gcc-toolset-14 bash
```

## Usage Constraints

### Compilation Scenarios

- **Primary version**: Use the system default gcc-12.3.1 for standard compilation and building.
- **Secondary version**: When the advanced features of GCC 14 are needed for application building, use SCL to switch the bash environment to the gcc-toolset-14 compilation environment.

### GCC 14 Secondary Version Usage Instructions

1. The openEuler GCC toolset 14 secondary compilation toolchain offers two usage methods:
    1) **Dynamic linking**: By default, the `-lstdc++` option is automatically included for dynamic linking. This links the system dynamic library **/usr/lib64/libstdc++.so.6** and the **libstdc++_nonshared.a** static library provided by the GCC 14 secondary version. This static library contains stable C++ features introduced in GCC 14 compared to GCC 12.
    2) **Static linking**: You can use the `-static` option for static linking, which links the full-feature **libstdc++.a** static library provided by the GCC 14 secondary version. The path to this library is **/opt/openEuler/gcc-toolset-14/root/usr/lib/gcc/aarch64-openEuler-linux/14/libstdc++.a**.

2. By default, builds use dynamic linking, which links the **libstdc++_nonshared.a** static library. To ensure system compatibility, this library only includes officially standardized C++ features. Experimental features like `-fmodules-ts` and `-fmodule-header`, which are part of C++20 module capabilities, are not included in **libstdc++_nonshared.a**. If you need these features, you should use static linking to fully link the GCC 14 secondary version static library.
