# GNU 兼容性用户指南

## 简介

相比 GNU 编译器，LLVM 前端 Clang 对语法的检查更严谨，严格匹配语言标准，Clang 的常见兼容性和可移植性问题，请参考[开源官方文档](https://clang.llvm.org/compatibility.html) 。本文主要列出一些 LLVM for openEuler 相对 GNU 编译器不支持的问题以及部分固有实现，以供用户参考。LLVM for openEuler 针对 GNU 编译器进行了一定程度的兼容，并提供针对 GNU 编译器的整体兼容选项 `-fGNU-compatibility`。

## -Werror 引入的问题

### 错误信息

在编译选项打开 `-Werror` 的情况下，编译器会将所有 warning 转化为 error，不同于 warning，error 会阻碍构建。此类 error 格式如下：

```shell
test.c:6:1: error: non-void function does not return a value [-Werror,-Wreturn-type]
```

### 问题介绍

因为 LLVM 编译器和 GNU 编译器对于 warning 的处理有差异，因此可能存在切换 LLVM 后，新增 warning 并被 `-Werror` 选项转化为 error，阻碍构建的情况。

### 解决方案

可以通过增加 `-Wno` 选项关闭对应的 warning，例如上例中 warning 提示告警来自 `-Wreturn-type` 选项，则可添加 `-Wno-return-type`（`W` 后添加 `no-`）选项关闭该告警，可以通过在源码根目录 `grep -Werror` 找到设置 `-Werror` 选项的位置，并在该处添加对应的 `-Wno` 选项。openEuler 中的 LLVM17 版本可通过打开 `-fGNU-compatibility` 选项，控制大部分与 GNU 编译器存在差异的 warning 不被 `-Werror` 选项升级，批量解决该问题。

## 选项兼容问题

### 问题介绍

因为 LLVM 编译器和 GNU 编译器支持的选项功能存在差异，因此可能存在切换 LLVM 后，部分编译选项不识别导致报错。

包括但不限于：

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

### 解决方式

1. 去除相关选项或使用 LLVM 支持的相关选项替代。

2. openEuler 中的 LLVM17 版本包含一系列选项兼容适配特性，使用部分选项编译时会自动适配 GNU 行为，避免生成阻塞性告警。

## Clang 不支持问题

### 不支持 print-multi-os-directory

#### 错误信息

```shell
ERROR: Problem encountered: Cannot find elf_aarch64_efi.lds
```

#### 问题介绍

GCC 使用 `print-multi-os-directory`，该选项返回 `../lib64`，而 Clang 不支持该选项，故无法组成完整的路径，因此找不到某些文件。

#### 解决方案

在编译时，对需要通过选项获取 lib 路径的代码进行硬编码。

### 不支持 __builtin_longjmp/__builtin_setjmp

#### 错误信息

```shell
error: __builtin_longjmp is not supported for the current target 
  __builtin_longjmp (buf, 1); 
  ^~~~~~~~~~~~~~~~~~~~~~~~~~ 
error: __builtin_setjmp is not supported for the current target 
  int r = __builtin_setjmp (buf); 
          ^~~~~~~~~~~~~~~~~~~~~~
```

#### 问题介绍

当前 AArch64 后端不支持内置函数 `__builtin_longjmp` 和 `__builtin_setjmp`。

#### 解决方案

- 使用标准库中的setjmp/longjmp。

- 编译器后续支持。

### 32 位下不支持 __uint128_t

#### 错误信息

```shell
clang: warning: unknown platform, assuming -mfloat-abi=soft 
error: unknown type name '__uint128_t'
```

#### 问题介绍

在 32 位模式下编译（例如添加了 `-m32` 参数）报错：`__uint128` 不被支持。

除此之外，还可能看到一个平台无法识别的 warning：使用 `-mfloat-abi=soft` 参数将浮点运算编译为软浮点。

#### 解决方案

- 去除 `-m32` 参数，使用 64 位模式编译；

- 检查 Makefile、configure 等构建脚本的平台判断流程，使用正确的平台选项编译。

### 不支持选项 -aux-info

#### 错误信息

无报错信息

#### 问题介绍

`-aux-info` 选项用于将编译单元中声明或定义的所有函数（包括头文件中的函数）输出到给定的输入文件。

该选项一般用于从 `*.c` 自动生成 `*.h` 文件。

Clang 不支持这个选项。

#### 解决方案

避免使用 Clang 不支持的选项。

如果确实需要这些信息，可以通过预处理器处理之后，用 `sed/aux` 等命令提取出想要的函数信息。

例如：

```shell
clang -E -xc test.c | sed -n 's/^extern * int *\(\w*\) *(.*$/\1/p'
```

### 不支持 __builtin___snprintf_chk

#### 错误信息

```shell
error: no member named '__builtin___snprintf_chk' in
```

#### 问题介绍

Clang 暂不支持检查 built-in 函数格式化输出，会报出 warning，当开启 `-Werror` 选项时升级为 error。

#### 解决方案

添加编译选项-Wno-builtin-memcpy-chk-size规避该告警

### 不支持 NEON 指令

#### 错误信息

```shell
error: unknown register name 'q0' in asm 
        : "memory", "cc", "q0" 
                          ^
```

#### 问题介绍

Clang 中不支持 NEON 指令 Q 寄存器设计

代码示例

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

#### 解决方案

修改 `qX` 寄存器为 `vX` 寄存器。

### 不支持部分运行库

#### 错误信息

```shell
undefined reference to `__muloti4'
```

#### 问题介绍

某符号不在 libgcc 中，但是在 compiler-rt 中，特别是使用 Clang 的 `__builtin_*_overflow` 家族的内联函数时。

#### 解决方案

使用 `--rtlib=compiler-rt` 来启用 compiler-rt ，注意目前并不支持所有平台。

如果使用 libc++ 或者 libc++abi，使用 compiler-rt 而不是 libgcc_s，通过在 cmake 中添加 `-DLIBCXX_USE_COMPILER_RT=YES` 和 `-DLIBCXXABI_USE_COMPILER_RT=YES` 实现。否则可能会链接两个运行时库，虽然不影响功能但是造成性能浪费。

参考 [LLVM 官方说明](https://clang.llvm.org/docs/Toolchain.html)。

### 不支持原子类型（atomic type）的类型转换

#### 错误信息

```shell
error: used type '_Atomic(int_fast32_t)' where arithmetic or pointer type is required 
                uint32_t(_Atomic(int_fast32_t))(1) 
                        ^
```

#### 问题介绍

Clang 拒绝对非标准类型的类型转换，并且目前不支持对原子类型（atomic type）的转换。所支持的类型在 `clang/include/clang/AST/BuiltinTypes.def` 中有列出，不包括原子类型。

#### 解决方案

避免对原子类型使用类型转换。

## 链接问题

### 指定 pic、pie

#### 错误信息

某些动态库在未使用 pic/pie 选项的情况下，会报符号表缺失的错误，比如：

```shell
undefined reference to `cmsPlugin'
```

#### 问题介绍

生成动态库与 pie 可执行文件编译或链接时未带对应 pic/pie 选项，导致符号表缺失，需要手动指定。

#### 代码示例

执行如下命令检查是否为 PIC 库，检索到 textrel 符号则表明是 PIC 库。

```shell
readelf -a libhello.so |grep -i textrel
```

检查是否为 PIE 共享文件，使用 `file` 命令查看，或者用 `size --format=sysv` 查看基地址是否在 0 附近。

#### 解决方案

编译选项增加 `-fPIC`，链接选项增加 `-pie`。

> [!WARNING]注意
> Clang 严格区分编译和链接选项，不能通过 cflags 将 `-pie` 传递给链接器，否则会报错：
> `clang: error: argument unused during compilation: '-pie' [-Werror,-Wunused-command-line-argument]`

### 给链接器参数加上 -Wl

#### 错误信息

```shell
clang: error: unsupported option `--whole-archive` 
clang: error: unsupported option `--no-whole-archive` 
clang: error: unknown argument: `-soname`
```

#### 问题介绍

部分应用构建脚本在编译器为 clang 时可能不会默认给需要传递给链接器加上 `-Wl`，这部分参数必须添加 `-Wl`，才能传递给链接器。

包括但不限于：

```shell
--whole-archive
--no-whole-archive
-soname
```

如果出现 `unknown argument` 或者 `unsupported option` ，并且该选项是应该传给链接器的，则需要加上 `-Wl`。

#### 解决方案

这些参数前面添加 `-Wl,`。例如：

```shell
-Wl,--whole-archive -Wl,--no-whole-archive -Wl,-soname
```

### Clang 不再默认传递 --build-id 到链接器

#### 错误信息

```shell
ERROR: No build ID note found in xxx.so
```

#### 问题介绍

为了避免额外链接器开销，Clang 不再默认传递 `--build-id` 到链接器。

#### 解决方案

编译目标源码时增加 `-Wl,--build-id` 选项可以临时传递。

### 系统 libstdc++ 库版本过低导致符号未定义或运行结果错误

#### 错误信息

无法找到高版本 C++ 标准库函数定义的接口：

```shell
undefined reference to `std::xxx`
```

#### 问题介绍

Clang 默认使用系统路径下的 `libstdc++.so` 动态库，过低的系统 `libstdc++.so` 库版本可能不支持用户代码中使用的高版本特性，导致链接时出现未定义符号或运行结果错误。

#### 解决方案

链接时加入 `-stdlib=libc++` 或 `-lc++` 选项，使用 Clang 提供的 `libc++.so` 库中提供的标准 C++ 库实现。

### 系统 binutils 版本过低导致 debug_info 段链接失败

#### 错误信息

```shell
unable to initializedecompress status for section .debug_info
```

#### 问题介绍

在 x86_64 环境 CentOS 7.6、Ubuntu 18.04 系统上运行时，如果系统默认的 binutils 版本低于 2.32，会由于低版本 binutils 对调试信息段的对齐处理有误，导致在链接时如果链接由高版本（版本 >2.32）binutils 生成的 debug_info 段链接失败。

#### 解决方案

1. 在链接时使用 `-fuse-ld=lld` 选项，选择 LLVM for openEuler 自带的链接器即可。

2. 如仍然需要 GNU 链接器，请升级系统链接器到 2.32 版本以上。

## 其他类兼容问题

### Clang 预处理器结果与 GCC 存在较大差异

#### 错误信息

- 格式错误：syntax error, unexpected IDENT

- 找不到头文件等

#### 问题介绍

Clang 的预处理器实现和 GCC 有比较大的不同，例如：

- Clang 会保留每行开头的空白符。

- Clang 会保留引入的头文件的绝对路径。

其它的不一一列举。

有一些程序会使用预处理器来处理源码文件，但是因为Clang和GCC的预处理器的行为有一些不同，可能会因此导致一些问题。

#### 解决方案

修改源码使得其能被 Clang 的预处理器正确处理。例如：

- 删除代码行前的空白符。

- 保证include的文件能被找到。

### Clang不支持在使用'-o'指定输出时直接添加头文件

#### 错误信息

```shell
clang test.h test.c -o test 
clang: error: cannot specify -o when generating multiple output files
```

#### 问题介绍

Clang 不支持在使用 `-o` 指定输出文件时直接添加头文件，但允许在编译命令中使用预编译头文件，以减少编译时间。

```shell
$ cat test.c
#include "test.h"
```

生成预编译头文件的命令：

```shell
clang -x c-header test.h -o test.h.pch
```

可以通过添加 `-include` 命令使用预编译头文件：

```shell
clang -include test.h test.c -o test
```

Clang 会先检查 `test.h` 对应的预编译头文件是否存在；如果存在，则会使用对应的预编译头文件对 `test.h` 进行处理，否则，Clang 会直接处理 `test.h` 的内容。

如果设法在 Clang 编译命令中保留头文件，则可以通过添加命令 `-include` 的方法使其通过编译。

[官方参考文档](https://clang.llvm.org/docs/UsersManual.html)

#### 解决方案

避免在 Clang 的编译命令中直接添加头文件，或者按照上述方法使用预编译功能。

### 不同编译器对 built-in includes 的实现不同

#### 错误信息

```shell
error: typedef redefinition with different types ('__uint64_t' (aka 'unsigned long') vs 'UINT64' (aka 'unsigned long long')) 
error: unknown type name 'wchar_t'
```

#### 问题介绍

某些头文件（例如 `stdatomic.h`、`stdint.h` ）是由编译器实现的，不同的编译器对于这些文件的实现存在差异，因此使用 GCC 头文件实现的程序，切换到 Clang 之后，用户自定义的代码可能会与 Clang 头文件发生冲突。

比如说重定义问题：在 Clang 的某些头文件中定义了一些在对应 GCC 的头文件中没有定义的变量，而用户在自己编写的或引入的其它库的头文件也定义了该变量，变量被重复定义，导致 redefinition 错误。

又或者：在 GCC 的 built-in 头文件中定义了一些变量，而 Clang 对应的头文件中没有定义该变量，用户在自己编写的代码中直接使用了该变量，结果就会导致 unknown type 的错误。

#### 解决方案

建议修改源码。

### 不同编译器链接的OpenMP运行时库不同

#### 错误信息

- 测试错误

- 无法加载OpenMP运行时库

#### 问题介绍

Clang 编译的可执行程序链接的 OpenMP 运行时库叫 `libomp.so`，GCC 链接的叫 `libgomp.so`。

#### 解决方案

确保能够找到 `libomp.so`。

- 将 `libomp.so` 所在的目录（例如 `{$INSTALLATION_HOME}/lib，INSTALLATION_HOME` 为安装根目录）添加到环境变量 `LD_LIBRARY_PATH`。

- 或者安装libomp：

    ```shell
    yum install libomp -y
    ```

### __builtin_prefetch 语义检查错误

#### 错误信息

```shell
 error: argument to '__builtin_prefetch' must be a constant integer 
    __builtin_prefetch(address, forWrite); 
    ^
```

#### 问题介绍

在这段代码中，因为 `__builtin_prefetch` 的第二个参数需要是常量，所以先用 `__builtin_constant_p` 检查 `forWrite` 是否是常量。但是，对于 Clang 而言，会出现语义检查错误。

#### 代码示例

```c
static void prefetchAddress(const void *address, bool forWrite) { 
    if (__builtin_constant_p(forWrite)) { 
        __builtin_prefetch(address, forWrite); 
    } 
}
```

#### 解决方案

将函数转换为宏函数：

```c
##define prefetchAddress(address,forWrite) do{\ 
  if (__builtin_constant_p(forWrite)) {      \ 
    __builtin_prefetch(address, forWrite);   \ 
  }                                          \ 
}while(0)
```

### 找不到符号 perl_tsa_mutex_lock

#### 错误信息

```shell
Can't load 'xxx.so' for module threads: xxx.so: undefined symbol: perl_tsa_mutex_lock at xxx
```

#### 问题介绍

在文件 `/usr/lib64/perl5/CORE/perl.h` 中有如下的定义：

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

由于针对 Clang 使用的 mutex 相关的符号是有线程安全标记的 `perl_tsa_*`，但是 `libperl.so` 并不包含这些符号，故而出现链接错误。

#### 解决方案

- 使用包含 `perl_tsa_*` 符号的 `libperl.so`（在编译 `libperl.so` 时，加上宏 `USE_ITHREADS` 和 `I_PTHREAD`）。

- 去除预定义宏 `__clang__`：

    ```c
    clang -U__clang__ ...
    ```

### Clang 宏问题

#### 问题介绍

程序代码逻辑使用了 `__GNUC__` 的宏作为判断依据，但是 GCC 与 Clang 中定义的宏内容不一致，可以使用如下命令确认 Clang 中宏定义的值。

```shell
clang -x c /dev/null -dM -E >clang.log;cat clang.log|grep '__GNUC__'
```

#### 解决方案

若宏内容不一致导致报错，可以在编译选项加入 `-D__GNUC__=x` 进行适配修改。

### 支持的 Attributes 集合

LLVM for openEuler 只支持 Clang 框架中的 attributes，请参考[clang文档](https://clang.llvm.org/docs/AttributeReference.html)

链接中未提到 attributes 暂不支持。

### -march 选项在架构扩展特性上的使用说明

Clang 使用架构扩展特性时，需在 `-march=<arch_name>` 后加上扩展特性的名称，包括架构默认支持的扩展特性。例如，DotProd 特性是 Armv8.4 架构默认支持的，可使用 `-march=armv8.4-a+dotprod` 使能该特性。

### -mgeneral-regs-only 选项的使用说明

使用该选项时将生成仅使用通用寄存器的代码，这会阻止编译器使用浮点或高级 SIMD 寄存器。因此当编译时加入该选项，编译器应避免使用浮点运算指令，如果程序中有浮点运算，LLVM for openEuler 将会调用 compiler-rt 中的库函数进行运算。因此，链接时要配合添加 `-rtlib=compiler-rt -l gcc_s` 选项。

### Hardware-assisted AddressSanitizer 仅支持在基于 Linux5.4 及以上内核版本的 OS 上运行

Hardware-assisted AddressSanitizer 可以通过 `-fsanitize=hwaddress -fuse-ld=lld` 使能，该特性依赖一部分 Linux5.4 及以上才支持的内核接口，若环境内核版本低于 5.4 则无法正常使能，这种情况下建议使用常规地址消毒。

### Neon Intrinsic

Neon Intrinsic 和编译器的具体实现相关，Clang 的neon Intrinsic 的功能与官方文档[《Arm Neon Intrinsics Reference》](https://developer.arm.com/documentation/ihi0073/g)（以下简称ANIR文档）一致。

但生成 ANIR 文档上指定的汇编指令，需指定优化级别大于 O0。

#### 使用举例

以内容如下的test.c为例：

```c
#include <arm_neon.h>
int32x2_t test_vsudot_lane_s32(int32x2_t r, int8x8_t a, uint8x8_t b) {
    return vsudot_lane_s32(r, a, b, 0);
}
```

表1 ANIR文档上vsudot_lane_s32的描述

| Intrinsic | Argument Preparation | Instruction | Result |Supported Architectures |
|-|-|-|-|-|
| int32x2_t vsudot_lane_s32(int32x2_t r, int8x8_t a, uint8x8_t b, const int lane) | r -\> Vd.2S <br /> a -\> Vn.8B <br /> b -\> Vm.4B <br /> 0 \<= lane \<= 1 | SUDOT Vd.2S,Vn.8B,Vm.4B[lane] | Vd.2S -\> result | A32/A64 |

使用命令 `clang -march=armv8.6-a+i8mm test.c -O0 -S` 生成的结果是以 mov、dup 和 usdot 等多条指令组合的形式。

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

使用命令 `clang -march=armv8.6-a+i8mm test.c -O1 -S` 生成结果与 ANIR 文档一致。

```asm
test_vsudot_lane_s32:                   // @test_vsudot_lane_s32
// %bb.0:                               // %entry
                                        // kill: def $d2 killed $d2 def $q2
        sudot   v0.2s, v1.8b, v2.4b[0]
        ret
```

## OpenMP兼容性

### linear 子句和 chucked dynamic schedule 子句共用不支持

当 `linear` 子句和 `chucked dynamic schedule` 子句一起使用时，OpenMP 不支持该场景的线程调度，导致运行后循环的线性变量的值不正确。

该场景已提交到[上游社区](https://github.com/llvm/llvm-project/issues/61230)，等待社区进行修复。

规避方案是可以使用其它子句来替换，或者不使用 `chucked dynamic schedule` 的线程调度方式。

### Omp atomic 特性依赖 10.3.0 及以上的系统 GCC 版本

Omp atomic 特性依赖系统 GCC 的 libgcc，低于 10.3.0 的系统 GCC 版本可能引发运行结果异常。如果需要使用请保证系统 GCC 版本满足要求。使用 Omp atomic 特性的具体用例如下：

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

## 问题反馈

在使用过程中遇到问题，需要技术支持时，请反馈问题信息至 openEuler 社区 [llvm-project](https://gitee.com/openeuler/llvm-project) 源码仓。
