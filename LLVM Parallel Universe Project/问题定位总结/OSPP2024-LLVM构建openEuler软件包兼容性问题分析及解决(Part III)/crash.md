## crash

### 问题描述

#### 报错信息

修改前 clang 构建出现多个`error: integer value -1 is outside the valid range of values`报错，比如：

```
./../gdbsupport/enum-flags.h:85:52: error: integer value -1 is outside the valid range of values [0, 15] for the enumeration type 'ui_out_flag' [-Wenum-constexpr-conversion]
   85 |     integer_for_size<sizeof (T), static_cast<bool>(T (-1) < T (0))>::type
      |                                                    ^
In file included from cp-valprint.c:20:
In file included from ./defs.h:65:
./../gdbsupport/enum-flags.h:85:52: error: integer value -1 is outside the valid range of values [0, 15] for the enumeration type 'ui_out_flag' [-Wenum-constexpr-conversion]
   85 |     integer_for_size<sizeof (T), static_cast<bool>(T (-1) < T (0))>::type
      |                                                    ^
In file included from d-lang.c:20:
In file included from ./defs.h:65:
./../gdbsupport/enum-flags.h:85:52: error: integer value -1 is outside the valid range of values [0, 15] for the enumeration type 'ui_out_flag' [-Wenum-constexpr-conversion]
   85 |     integer_for_size<sizeof (T), static_cast<bool>(T (-1) < T (0))>::type
      |                                                    ^
In file included from cp-namespace.c:21:
In file included from ./defs.h:65:
./../gdbsupport/enum-flags.h:85:52: error: integer value -1 is outside the valid range of values [0, 15] for the enumeration type 'ui_out_flag' [-Wenum-constexpr-conversion]
   85 |     integer_for_size<sizeof (T), static_cast<bool>(T (-1) < T (0))>::type
      |                                                    ^
In file included from d-namespace.c:20:
In file included from ./defs.h:65:
./../gdbsupport/enum-flags.h:85:52: error: integer value -1 is outside the valid range of values [0, 15] for the enumeration type 'ui_out_flag' [-Wenum-constexpr-conversion]
   85 |     integer_for_size<sizeof (T), static_cast<bool>(T (-1) < T (0))>::type
      |                                                    ^
In file included from ctfread.c:78:
In file included from ./defs.h:65:
./../gdbsupport/enum-flags.h:85:52: error: integer value -1 is outside the valid range of values [0, 15] for the enumeration type 'ui_out_flag' [-Wenum-constexpr-conversion]
   85 |     integer_for_size<sizeof (T), static_cast<bool>(T (-1) < T (0))>::type
      |                                                    ^
In file included from d-valprint.c:20:
In file included from ./defs.h:65:
./../gdbsupport/enum-flags.h:85:52: error: integer value -1 is outside the valid range of values [0, 15] for the enumeration type 'ui_out_flag' [-Wenum-constexpr-conversion]
   85 |     integer_for_size<sizeof (T), static_cast<bool>(T (-1) < T (0))>::type
      |                                                    ^
./../gdbsupport/enum-flags.h:85:52: error: integer value -1 is outside the valid range of values [0, 3] for the enumeration type 'innermost_block_tracker_type' [-Wenum-constexpr-conversion]
./../gdbsupport/enum-flags.h:85:52: error: integer value -1 is outside the valid range of values [0, 3] for the enumeration type 'innermost_block_tracker_type' [-Wenum-constexpr-conversion]
./../gdbsupport/enum-flags.h:85:52: error: integer value -1 is outside the valid range of values [0, 1] for the enumeration type 'btrace_insn_flag' [-Wenum-constexpr-conversion]
./../gdbsupport/enum-flags.h:85:52: error: integer value -1 is outside the valid range of values [0, 3] for the enumeration type 'btrace_function_flag' [-Wenum-constexpr-conversion]
./../gdbsupport/enum-flags.h:85:52: error: integer value -1 is outside the valid range of values [0, 1] for the enumeration type 'btrace_insn_flag' [-Wenum-constexpr-conversion]
./../gdbsupport/enum-flags.h:85:52: error: integer value -1 is outside the valid range of values [0, 3] for the enumeration type 'btrace_function_flag' [-Wenum-constexpr-conversion]
In file included from dbxread.c:33:
In file included from ./defs.h:65:
./../gdbsupport/enum-flags.h:85:52: error: integer value -1 is outside the valid range of values [0, 15] for the enumeration type 'ui_out_flag' [-Wenum-constexpr-conversion]
   85 |     integer_for_size<sizeof (T), static_cast<bool>(T (-1) < T (0))>::type
      |                                                    ^
...
```

#### 相关PR

- 在上游社区crash提出PR，链接为：[Backport gdb patch to fix integer value -1 outside valid range error in clang 16+ by yanyir · Pull Request #188 · crash-utility/crash (github.com)](https://github.com/crash-utility/crash/pull/188)
- openEuler仓库PR：[Backport a gdb patch to fix clang build "integer value -1 is outside the valid range of values" error · Pull Request !101 · src-openEuler/crash - Gitee.com](https://gitee.com/src-openeuler/crash/pulls/101)

### 修复过程

- 这些报错都位于crash使用的gdb-10.2包当中，crash社区选择了对gdb 10.2进行patch的方式对gdb代码进行迭代，所有更新全部位于`gdb-10.2.patch`中。但并非所有gdb的更新都被其采用，能够解决本次pr涉及的报错的patch在之前并未被添加。GDB由于需要使用代码中的功能且未找到其他实现方式，故对该报错的解决方式为忽略`enum-flags.h`中的`-Wenum-constexpr-conversion`警告。

  对应的 GDB commit：
  8cbde735e40551a694d4ee5cfd4d525d3d77ff7e ("gdbsupport: ignore -Wenum-constexpr-conversion in enum-flags.h")
  gdb仓库地址：https://git.linaro.org/toolchain/binutils-gdb.git

- 不影响原来的构建行为。修改后clang构建成功，OBS验证工程：
  https://build.tarsier-infra.isrc.ac.cn/package/show/home:yy:branches:Mega-LLVM:crash
