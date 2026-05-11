# GNU Compatibility User Guide

## Overview

Compared with the GNU compiler, the LLVM frontend Clang performs stricter syntax checking and more rigorously adheres to language standards. For common compatibility and portability issues with Clang, see the [open-source official document](https://clang.llvm.org/compatibility.html). This document lists some of the features LLVM for openEuler does not support compared with the GNU compiler, as well as some inherent implementations for your reference. LLVM for openEuler is compatible with the GNU compiler to some extent and provides the overall compatibility option `-fGNU-compatibility` for the GNU compiler.

## Issue Introduced by -Werror

### Error Message

When the `-Werror` compilation option is enabled, the compiler converts all warnings into errors. Unlike warnings, errors will halt the build process. The format of such errors is as follows:

```shell
test.c:6:1: error: non-void function does not return a value [-Werror,-Wreturn-type]
```

### Description

The LLVM and GNU compilers handle warnings differently. Therefore, after the LLVM compiler is used, new warnings may be added and converted into errors by the `-Werror` option, which halts the build process.

### Solution

You can add the `-Wno` option to disable the corresponding warnings. For instance, the warning in the preceding example is caused by the `-Wreturn-type` option. You can add the `-Wno-return-type` option (add `no-` after `W`) to disable the warning. You can find the location where the `-Werror` option is set in the root directory `grep -Werror` of the source code and add the corresponding `-Wno` option. In LLVM 17 of openEuler, you can enable the `-fGNU-compatibility` option to prevent most warnings that differ from those of the GNU compiler from being promoted by the `-Werror` option. This allows you to resolve this issue in batches.

## Option Compatibility Issue

### Description

The LLVM and GNU compilers support different options. Therefore, after LLVM is used, some compilation options may not be recognized, resulting in errors.

This includes but is not limited to:

```shell
-znow
# error: unknown argument: `-znow`

-mabi=lp64
# error: unknown target ABI 'lp64'

clang -std=c++11
# error: invalid argument '-std=c++11' not allowed with 'C'

-Wa,--generate-missing-build-notes=yes
# clang: error: unsupported argument '--generate-missing-build-notes=yes' to option '-Wa,'
```

### Solution

1. Remove related options or use options supported by LLVM.

2. LLVM 17 of openEuler contains a series of option compatibility features. When some options are used for compilation, the behavior of GNU is automatically adapted to avoid generating blocking alarms.

## Clang Incompatibility Issues

### print-multi-os-directory Not Supported

#### Error Message

```shell
ERROR: Problem encountered: Cannot find elf_aarch64_efi.lds
```

#### Description

Clang does not support the `print-multi-os-directory` option that GCC uses to return `../lib64`. Because Clang is unable to construct the full path, it cannot find certain files.

#### Solution

Hardcode the **lib** path into the code that requires it.

### `__builtin_longjmp` and `__builtin_setjmp` Not Supported

#### Error Message

```shell
error: __builtin_longjmp is not supported for the current target 
  __builtin_longjmp (buf, 1); 
  ^~~~~~~~~~~~~~~~~~~~~~~~~~ 
error: __builtin_setjmp is not supported for the current target 
  int r = __builtin_setjmp (buf); 
          ^~~~~~~~~~~~~~~~~~~~~~
```

#### Description

Currently, the AArch64 backend does not support built-in functions `__builtin_longjmp` and `__builtin_setjmp`.

#### Solution

- Use **setjmp** and **longjmp** in the standard library.

- The compiler will support this in the future.

### __uint128_t Not Supported in 32-bit Mode

#### Error Message

```shell
clang: warning: unknown platform, assuming -mfloat-abi=soft 
error: unknown type name '__uint128_t'
```

#### Description

An error is reported during compilation in 32-bit mode (for example, `-m32` is added), indicating that `__uint128` is not supported.

In addition, a warning that cannot be identified by the platform may be displayed: `-mfloat-abi=soft` is used to compile floating-point operations into soft floating-point operations.

#### Solution

- Remove the `-m32` option and use the 64-bit mode for compilation.

- Check the platform determination process of build scripts such as **Makefile** and **configure**, and use the correct platform options for compilation.

### -aux-info Not Supported

#### Error Message

No error message is reported.

#### Description

The `-aux-info` option is used to output all functions declared or defined in a compilation unit (including functions in header files) to a specified input file.

This option is generally used to automatically generate `*.h` files from `*.c` files.

Clang does not support this option.

#### Solution

Do not use options that are not supported by Clang.

If the information is required, you can use the preprocessor to process the information and then use commands such as `sed` or `aux` to extract the desired function information.

For example:

```shell
clang -E -xc test.c | sed -n 's/^extern * int *\(\w*\) *(.*$/\1/p'
```

### __builtin___snprintf_chk Not Supported

#### Error Message

```shell
error: no member named '__builtin___snprintf_chk' in
```

#### Description

Currently, Clang cannot check the formatted output of built-in functions. A warning is reported. When the `-Werror` option is enabled, the warning is promoted to an error.

#### Solution

Add the compilation option **-Wno-builtin-memcpy-chk-size** to avoid this warning.

### NEON Instructions Not Supported

#### Error Message

```shell
error: unknown register name 'q0' in asm 
        : "memory", "cc", "q0" 
                          ^
```

#### Description

Clang does not support the Q register design of NEON instructions.

#### Sample Code

```shell
$ cat bar.c 
int foo(void) { 
    __asm__("":::"q0"); 
    return 0; 
} 
$ clang bar.c  
bar.c:2:16: error: unknown register name 'q0' in asm 
    __asm__("":::"q0"); 
               ^ 
1 error generated.
```

#### Solution

Change the `q*X*` register to the `v*X*` register.

### Some Runtime Libraries Not Supported

#### Error Message

```shell
undefined reference to `__muloti4'
```

#### Description

A symbol is in compiler-rt instead of libgcc, especially when an inline function in the `__builtin_*_overflow` family of Clang is used.

#### Solution

Use `--rtlib=compiler-rt` to enable compiler-rt. This solution is applicable only to some platforms.

If libc++ or libc++abi is used, use compiler-rt instead of libgcc_s by adding `-DLIBCXX_USE_COMPILER_RT=YES` and `-DLIBCXXABI_USE_COMPILER_RT=YES` to CMake configurations. Otherwise, two runtime libraries may be linked, which does not affect functionality but causes performance waste.

For details, see the [LLVM official documentation](https://clang.llvm.org/docs/Toolchain.html).

### Type Conversion Not Supported for Atomic Types

#### Error Message

```shell
error: used type '_Atomic(int_fast32_t)' where arithmetic or pointer type is required 
                uint32_t(_Atomic(int_fast32_t))(1) 
                        ^
```

#### Description

Clang rejects conversion for non-standard types and currently does not support conversion for atomic types. The supported types are listed in `clang/include/clang/AST/BuiltinTypes.def` and do not include atomic types.

#### Solution

Do not perform type conversion for atomic types.

## Link Issues

### Specifying pic or pie

#### Error Message

When the **pic** or **pie** option is not used for some dynamic libraries, an error indicating that the symbol table is missing is reported. For example:

```shell
undefined reference to `cmsPlugin'
```

#### Description

The **pic** or **pie** option is not specified during the compilation or linking of the generated dynamic library or **pie** executable file. As a result, the symbol table is missing. In this case, you need to manually specify the option.

#### Sample Code

Run the following command to check whether the library is a PIC library. If the **textrel** symbol is found, the library is a PIC library.

```shell
readelf -a libhello.so |grep -i textrel
```

Run the `file` command to check whether the file is a PIE shared file, or run the `size --format=sysv` command to check whether the base address is around **0**.

#### Solution

Add `-fPIC` to compiler options and `-pie` to linker options.

> ![WARNING](./public_sys_resources/icon_warning.gif)
> 
> Clang strictly distinguishes compiler and linker options. Because of this, `-pie` cannot be transferred to the linker through **cflags**. Otherwise, the following error is reported:
> `clang: error: argument unused during compilation: '-pie' [-Werror,-Wunused-command-line-argument]`

### Adding -Wl to Linker Options

#### Error Message

```shell
clang: error: unsupported option `--whole-archive` 
clang: error: unsupported option `--no-whole-archive` 
clang: error: unknown argument: `-soname`
```

#### Description

Some application build scripts may not add `-Wl` to the options that need to be transferred to the linker by default when the compiler is Clang. These options must be added with `-Wl` before being transferred to the linker.

This includes but is not limited to:

```shell
--whole-archive
--no-whole-archive
-soname
```

If `unknown argument` or `unsupported option` is displayed and the option is to be transferred to the linker, `-Wl` must be added to the option.

#### Solution

Add `-Wl,` before these options. For example:

```shell
-Wl,--whole-archive -Wl,--no-whole-archive -Wl,-soname
```

### Clang No Longer Transfers --build-id to the Linker by Default

#### Error Message

```shell
ERROR: No build ID note found in xxx.so
```

#### Description

Clang no longer transfers `--build-id` to the linker by default to avoid extra linker overhead.

#### Solution

Add `-Wl` during compilation of the target source code to transfer the `--build-id` option temporarily.

### Undefined Symbols or Incorrect Running Results Due to an Outdated libstdc++ Library Version

#### Error Message

The interfaces defined by the C++ standard library functions of a later version cannot be found:

```shell
undefined reference to `std::xxx`
```

#### Description

By default, Clang uses the `libstdc++.so` dynamic library in the system path. If the `libstdc++.so` library version is outdated, it may not support the features of a later version used in the user code. As a result, undefined symbols occur during linking or the running result is incorrect.

#### Solution

Add `-stdlib=libc++` or `-lc++` during linking and use the standard C++ library provided by the `libc++.so` library provided by Clang.

### Linking Error in the debug_info Section Due to an Outdated binutils Version

#### Error Message

```shell
unable to initializedecompress status for section .debug_info
```

#### Description

When running on CentOS 7.6 or Ubuntu 18.04 in the x86_64 environment, if the default binutils version is earlier than 2.32, the alignment of the **debug_info** section is incorrect. As a result, the **debug_info** section generated by binutils of a later version (2.32 or later) fails to be linked.

#### Solution

1. Use `-fuse-ld=lld` during linking and select the linker provided by LLVM for openEuler.

2. If you still need the GNU linker, upgrade the system linker to a version later than 2.32.

## Other Compatibility Issues

### The Preprocessor Result of Clang Differs Greatly from That of GCC

#### Error Message

- Format errors: **syntax error**, **unexpected IDENT**

- Header files cannot be found, and more

#### Description

The preprocessor implementation of Clang differs greatly from that of GCC. For example:

- Clang retains the space characters at the beginning of each line.

- Clang retains the absolute path of the imported header file.

Other differences are not listed here.

Some programs use the preprocessor to process source code files. However, because the behavior of the Clang preprocessor differs from that of the GCC preprocessor, some issues may occur.

#### Solution

Modify the source code so that it can be correctly processed by the Clang preprocessor. For example:

- Delete the space characters at the beginning of code lines.

- Ensure that files in the **include** directory can be found.

### Clang Does Not Allow Header Files to Be Directly Added When -o Is Used to Specify the Output

#### Error Message

```shell
clang test.h test.c -o test 
clang: error: cannot specify -o when generating multiple output files
```

#### Description

Clang does not allow header files to be directly added when `-o` is used to specify the output file. However, it allows the use of precompiled header files in compilation commands to reduce the compile time.

```shell
$ cat test.c
#include "test.h"
```

The following is the command for generating a precompiled header file:

```shell
clang -x c-header test.h -o test.h.pch
```

You can use the precompiled header file by adding the `-include` command:

```shell
clang -include test.h test.c -o test
```

Clang first checks whether the precompiled header file corresponding to `test.h` exists. If it exists, Clang uses the precompiled header file to process `test.h`. Otherwise, it directly processes the content of `test.h`.

If you want to retain the header file in the Clang compile command, you can add the `-include` command to make it compile successfully.

[Official documentation](https://clang.llvm.org/docs/UsersManual.html)

#### Solution

Do not directly add header files to Clang compilation commands. Alternatively, use the precompilation function as described above.

### The Implementation of built-in includes Varies According to Compilers

#### Error Message

```shell
error: typedef redefinition with different types ('__uint64_t' (aka 'unsigned long') vs 'UINT64' (aka 'unsigned long long')) 
error: unknown type name 'wchar_t'
```

#### Description

Some header files (such as `stdatomic.h` and `stdint.h`) are implemented by compilers. Different compilers implement these files differently. Therefore, for programs that use GCC header files, if Clang is used, the user-defined code may conflict with the Clang header files.

For example, a redefinition error occurs when some variables defined in certain Clang header files are not defined in the corresponding GCC header files, but the same variables are also defined in the header files of other libraries that are written or imported by the user. As a result, the variables are repeatedly defined, leading to a redefinition error.

Alternatively, some variables are defined in the built-in header files of GCC, but the corresponding Clang header files do not define these variables. If the user directly uses these variables in their own code, an **unknown type** error occurs.

#### Solution

You are advised to modify the source code.

### Different Compilers Link to Different OpenMP Runtime Libraries

#### Error Message

- Test error.

- The OpenMP runtime library fails to be loaded.

#### Description

Executable programs compiled with Clang link to OpenMP runtime library `libomp.so`, while those compiled with GCC link to `libgomp.so`.

#### Solution

Ensure that `libomp.so` can be found.

- Add the directory where `libomp.so` is located (for example, `{$INSTALLATION_HOME}/lib, where INSTALLATION_HOME` is the root installation directory) to environment variable `LD_LIBRARY_PATH`.

- Alternatively, install **libomp**:

    ```shell
    yum install libomp -y
    ```

### Semantic Check Error of __builtin_prefetch

#### Error Message

```shell
 error: argument to '__builtin_prefetch' must be a constant integer 
    __builtin_prefetch(address, forWrite); 
    ^
```

#### Description

In this code snippet, the second argument of `__builtin_prefetch` must be a constant. Therefore, `__builtin_constant_p` is used to check whether `forWrite` is a constant. However, for Clang, a semantic check error occurs.

#### Sample Code

```c
static void prefetchAddress(const void *address, bool forWrite) { 
    if (__builtin_constant_p(forWrite)) { 
        __builtin_prefetch(address, forWrite); 
    } 
}
```

#### Solution

Convert the function into a macro function:

```c
##define prefetchAddress(address,forWrite) do{\ 
  if (__builtin_constant_p(forWrite)) {      \ 
    __builtin_prefetch(address, forWrite);   \ 
  }                                          \ 
}while(0)
```

### Symbol perl_tsa_mutex_lock Cannot Be Found

#### Error Message

```shell
Can't load 'xxx.so' for module threads: xxx.so: undefined symbol: perl_tsa_mutex_lock at xxx
```

#### Description

The following definition exists in the `/usr/lib64/perl5/CORE/perl.h` file:

```c
##if ... 
    defined(__clang__) 
    ... 
##  define PERL_TSA__(x)   __attribute__((x)) 
##  define PERL_TSA_ACTIVE 
##else 
##  define PERL_TSA__(x)   /* No TSA, make TSA attributes no-ops. */ 
##  undef PERL_TSA_ACTIVE 
##endif 
 
##ifdef PERL_TSA_ACTIVE 
EXTERN_C int perl_tsa_mutex_lock(perl_mutex* mutex) 
  PERL_TSA_ACQUIRE(*mutex) 
  PERL_TSA_NO_TSA; 
EXTERN_C int perl_tsa_mutex_unlock(perl_mutex* mutex) 
  PERL_TSA_RELEASE(*mutex) 
  PERL_TSA_NO_TSA; 
##endif##endif
```

The mutex-related symbols used by Clang are marked with `perl_tsa_*` for thread safety. However, `libperl.so` does not contain these symbols, resulting in a linking error.

#### Solution

- Use `libperl.so` that contains the `perl_tsa_*` symbols (add the `USE_ITHREADS` and `I_PTHREAD` macros when compiling `libperl.so`).

- Remove the predefined macro `__clang__`:

    ```c
    clang -U__clang__ ...
    ```

### Clang Macro Issue

#### Description

The program code logic uses the `__GNUC__` macro as the judgment basis. However, the macro content defined in GCC is different from that defined in Clang. You can run the following command to check the value defined by the macro in Clang:

```shell
clang -x c /dev/null -dM -E >clang.log;cat clang.log|grep '__GNUC__'
```

#### Solution

If the error is reported due to inconsistent macro content, you can add `-D__GNUC__=x` to the compilation options for adaptation.

### Supported Attributes

LLVM for openEuler supports only the attributes in the Clang framework. For details, see [Clang documentation](https://clang.llvm.org/docs/AttributeReference.html).

Attributes that are not mentioned in the link are not supported currently.

### Usage of -march for Extended Features of the Architecture

When Clang uses extended features of the architecture, the names of the extended features must be added next to `-march=<arch_name>`, including the names of the extended features supported by the architecture by default. For example, the DotProd feature is supported by the Armv8.4 architecture by default. You can use `-march=armv8.4-a+dotprod` to enable this feature.

### Usage of the -mgeneral-regs-only Option

If this option is used, the compiler only generates code for general-purpose registers. This prevents the compiler from using floating-point or advanced SIMD registers. Therefore, when this option is added during compilation, the compiler should avoid floating-point operation instructions. If floating-point operations exist in the program, LLVM for openEuler calls the library functions in compiler-rt for calculation. In this case, the `-rtlib=compiler-rt -l gcc_s` option needs to be added during linking.

### Hardware-assisted AddressSanitizer Can Run Only on the OS Based on Linux 5.4 or Later Kernel Versions

Hardware-assisted AddressSanitizer can be enabled by specifying `-fsanitize=hwaddress -fuse-ld=lld`. This feature depends on some kernel interfaces supported only by Linux 5.4 or later versions. If the kernel version is earlier than Linux 5.4, this feature cannot be enabled. In this case, you are advised to use a common AddressSanitizer.

### Neon Intrinsic

NEON intrinsics depend on the implementation of the compiler. The functions of NEON intrinsics in Clang are the same as those in the official documentation [Arm Neon Intrinsics Reference](https://developer.arm.com/documentation/ihi0073/g) (ANIR documentation for short).

However, the optimization level must be set to a level higher than O0 to generate the assembly instructions specified in the ANIR documentation.

#### Example

Take the following **test.c** as an example:

```c
#include <arm_neon.h>
int32x2_t test_vsudot_lane_s32(int32x2_t r, int8x8_t a, uint8x8_t b) {
    return vsudot_lane_s32(r, a, b, 0);
}
```

Table 1 Description of vsudot_lane_s32 in the ANIR documentation

| Intrinsic | Argument Preparation | Instruction | Result |Supported Architectures |
|-|-|-|-|-|
| int32x2_t vsudot_lane_s32(int32x2_t r, int8x8_t a, uint8x8_t b, const int lane) | r -\> Vd.2S <br> a -\> Vn.8B <br> b -\> Vm.4B <br> 0 \<= lane \<= 1 | SUDOT Vd.2S,Vn.8B,Vm.4B[lane] | Vd.2S -\> result | A32/A64 |

The `clang -march=armv8.6-a+i8mm test.c -O0 -S` command output is a combination of multiple instructions, such as **mov**, **dup**, and **usdot** instructions.

```asm
test_vsudot_lane_s32:                   // @test_vsudot_lane_s32
// %bb.0:                               // %entry
        sub     sp, sp, #112            // =112
        str     d0, [sp, #72]
        str     d1, [sp, #64]
        str     d2, [sp, #56]
        ldr     d0, [sp, #72]
        str     d0, [sp, #48]
        ldr     d0, [sp, #64]
        str     d0, [sp, #40]
        ldr     d0, [sp, #56]
        str     d0, [sp, #32]
        ldr     d0, [sp, #32]
        str     d0, [sp, #16]
        ldr     d0, [sp, #48]
        ldr     d1, [sp, #16]
                                        // implicit-def: $q3
        mov     v3.16b, v1.16b
        dup     v1.2s, v3.s[0]
        ldr     d2, [sp, #40]
        str     d0, [sp, #104]
        str     d1, [sp, #96]
        str     d2, [sp, #88]
        ldr     d0, [sp, #104]
        ldr     d1, [sp, #96]
        ldr     d2, [sp, #88]
        usdot   v0.2s, v1.8b, v2.8b
        str     d0, [sp, #80]
        ldr     d0, [sp, #80]
        str     d0, [sp, #24]
        ldr     d0, [sp, #24]
        str     d0, [sp, #8]
        ldr     d0, [sp, #8]
        add     sp, sp, #112            // =112
        ret
```

The `clang -march=armv8.6-a+i8mm test.c -O1 -S` command output is the same as that in the ANIR documentation.

```asm
test_vsudot_lane_s32:                   // @test_vsudot_lane_s32
// %bb.0:                               // %entry
                                        // kill: def $d2 killed $d2 def $q2
        sudot   v0.2s, v1.8b, v2.4b[0]
        ret
```

## OpenMP Compatibility

### The linear Clause and chucked dynamic schedule Clause Cannot Be Used Together

When the `linear` clause and `chucked dynamic schedule` clause are used together, OpenMP does not support thread scheduling in this scenario. As a result, the value of the linear variable of the loop after running is incorrect.

This scenario has been pulled to the [upstream community](https://github.com/llvm/llvm-project/issues/61230) and is waiting for the community to rectify the fault.

The workaround is to use other clauses or do not use `chucked dynamic schedule`.

### The Omp atomic Feature Depends on GCC 10.3.0 or Later

The Omp atomic feature depends on libgcc of GCC. If the GCC version is earlier than 10.3.0, the running result may be abnormal. If you need to use this feature, ensure that the GCC version meets the requirements. The following is an example of using the Omp atomic feature:

```c
void foo(double *Ptr, double M, double N) {
    double sum = 0;
    #pragma omp parallel for
    for (int i = 0; i < 100; ++i){
        Ptr[i] = i+(M*2 + N);
        #pragma omp atomic
        sum += Ptr[i];
    }
}
```

## Feedback

If you encounter any problem and need technical support, send the problem information to the [llvm-project](https://gitee.com/openeuler/llvm-project) source code repository in the openEuler community.
