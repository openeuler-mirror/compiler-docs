## jemalloc

### 问题描述

#### 报错信息

修改前使用clang构建出现如下错误，导致软件包构建于check阶段失败：

```
  test_alignment_errors:test/integration/aligned_alloc.c:24: Failed assertion: (p != ((void*)0) || get_errno() != 22) == (0) --> true != false: Expected error for invalid alignment 0
  test_alignment_errors:test/integration/aligned_alloc.c:32: Failed assertion: (p != ((void*)0) || get_errno() != 22) == (0) --> true != false: Expected error for invalid alignment 9
  test_alignment_errors:test/integration/aligned_alloc.c:32: Failed assertion: (p != ((void*)0) || get_errno() != 22) == (0) --> true != false: Expected error for invalid alignment 17
  test_alignment_errors:test/integration/aligned_alloc.c:32: Failed assertion: (p != ((void*)0) || get_errno() != 22) == (0) --> true != false: Expected error for invalid alignment 33
  test_alignment_errors:test/integration/aligned_alloc.c:32: Failed assertion: (p != ((void*)0) || get_errno() != 22) == (0) --> true != false: Expected error for invalid alignment 65
  test_alignment_errors:test/integration/aligned_alloc.c:32: Failed assertion: (p != ((void*)0) || get_errno() != 22) == (0) --> true != false: Expected error for invalid alignment 129
  test_alignment_errors:test/integration/aligned_alloc.c:32: Failed assertion: (p != ((void*)0) || get_errno() != 22) == (0) --> true != false: Expected error for invalid alignment 257
  test_alignment_errors:test/integration/aligned_alloc.c:32: Failed assertion: (p != ((void*)0) || get_errno() != 22) == (0) --> true != false: Expected error for invalid alignment 513
  test_alignment_errors:test/integration/aligned_alloc.c:32: Failed assertion: (p != ((void*)0) || get_errno() != 22) == (0) --> true != false: Expected error for invalid alignment 1025
  test_alignment_errors:test/integration/aligned_alloc.c:32: Failed assertion: (p != ((void*)0) || get_errno() != 22) == (0) --> true != false: Expected error for invalid alignment 2049
  test_alignment_errors:test/integration/aligned_alloc.c:32: Failed assertion: (p != ((void*)0) || get_errno() != 22) == (0) --> true != false: Expected error for invalid alignment 4097
  test_alignment_errors:test/integration/aligned_alloc.c:32: Failed assertion: (p != ((void*)0) || get_errno() != 22) == (0) --> true != false: Expected error for invalid alignment 8193
  test_alignment_errors:test/integration/aligned_alloc.c:32: Failed assertion: (p != ((void*)0) || get_errno() != 22) == (0) --> true != false: Expected error for invalid alignment 16385
  test_alignment_errors:test/integration/aligned_alloc.c:32: Failed assertion: (p != ((void*)0) || get_errno() != 22) == (0) --> true != false: Expected error for invalid alignment 32769
  test_alignment_errors:test/integration/aligned_alloc.c:32: Failed assertion: (p != ((void*)0) || get_errno() != 22) == (0) --> true != false: Expected error for invalid alignment 65537
  test_alignment_errors:test/integration/aligned_alloc.c:32: Failed assertion: (p != ((void*)0) || get_errno() != 22) == (0) --> true != false: Expected error for invalid alignment 131073
  test_alignment_errors:test/integration/aligned_alloc.c:32: Failed assertion: (p != ((void*)0) || get_errno() != 22) == (0) --> true != false: Expected error for invalid alignment 262145
  test_alignment_errors:test/integration/aligned_alloc.c:32: Failed assertion: (p != ((void*)0) || get_errno() != 22) == (0) --> true != false: Expected error for invalid alignment 524289
  test_alignment_errors:test/integration/aligned_alloc.c:32: Failed assertion: (p != ((void*)0) || get_errno() != 22) == (0) --> true != false: Expected error for invalid alignment 1048577
  test_alignment_errors:test/integration/aligned_alloc.c:32: Failed assertion: (p != ((void*)0) || get_errno() != 22) == (0) --> true != false: Expected error for invalid alignment 2097153
  test_alignment_errors:test/integration/aligned_alloc.c:32: Failed assertion: (p != ((void*)0) || get_errno() != 22) == (0) --> true != false: Expected error for invalid alignment 4194305
  test_alignment_errors (non-reentrant): fail
  test_alignment_errors:test/integration/aligned_alloc.c:24: Failed assertion: (p != ((void*)0) || get_errno() != 22) == (0) --> true != false: Expected error for invalid alignment 0
  test_alignment_errors:test/integration/aligned_alloc.c:32: Failed assertion: (p != ((void*)0) || get_errno() != 22) == (0) --> true != false: Expected error for invalid alignment 9
  test_alignment_errors:test/integration/aligned_alloc.c:32: Failed assertion: (p != ((void*)0) || get_errno() != 22) == (0) --> true != false: Expected error for invalid alignment 17
  test_alignment_errors:test/integration/aligned_alloc.c:32: Failed assertion: (p != ((void*)0) || get_errno() != 22) == (0) --> true != false: Expected error for invalid alignment 33
  test_alignment_errors:test/integration/aligned_alloc.c:32: Failed assertion: (p != ((void*)0) || get_errno() != 22) == (0) --> true != false: Expected error for invalid alignment 65
  test_alignment_errors:test/integration/aligned_alloc.c:32: Failed assertion: (p != ((void*)0) || get_errno() != 22) == (0) --> true != false: Expected error for invalid alignment 129
  test_alignment_errors:test/integration/aligned_alloc.c:32: Failed assertion: (p != ((void*)0) || get_errno() != 22) == (0) --> true != false: Expected error for invalid alignment 257
  test_alignment_errors:test/integration/aligned_alloc.c:32: Failed assertion: (p != ((void*)0) || get_errno() != 22) == (0) --> true != false: Expected error for invalid alignment 513
  test_alignment_errors:test/integration/aligned_alloc.c:32: Failed assertion: (p != ((void*)0) || get_errno() != 22) == (0) --> true != false: Expected error for invalid alignment 1025
  test_alignment_errors:test/integration/aligned_alloc.c:32: Failed assertion: (p != ((void*)0) || get_errno() != 22) == (0) --> true != false: Expected error for invalid alignment 2049
  test_alignment_errors:test/integration/aligned_alloc.c:32: Failed assertion: (p != ((void*)0) || get_errno() != 22) == (0) --> true != false: Expected error for invalid alignment 4097
  test_alignment_errors:test/integration/aligned_alloc.c:32: Failed assertion: (p != ((void*)0) || get_errno() != 22) == (0) --> true != false: Expected error for invalid alignment 8193
  test_alignment_errors:test/integration/aligned_alloc.c:32: Failed assertion: (p != ((void*)0) || get_errno() != 22) == (0) --> true != false: Expected error for invalid alignment 16385
  test_alignment_errors:test/integration/aligned_alloc.c:32: Failed assertion: (p != ((void*)0) || get_errno() != 22) == (0) --> true != false: Expected error for invalid alignment 32769
  test_alignment_errors:test/integration/aligned_alloc.c:32: Failed assertion: (p != ((void*)0) || get_errno() != 22) == (0) --> true != false: Expected error for invalid alignment 65537
  test_alignment_errors:test/integration/aligned_alloc.c:32: Failed assertion: (p != ((void*)0) || get_errno() != 22) == (0) --> true != false: Expected error for invalid alignment 131073
  test_alignment_errors:test/integration/aligned_alloc.c:32: Failed assertion: (p != ((void*)0) || get_errno() != 22) == (0) --> true != false: Expected error for invalid alignment 262145
  test_alignment_errors:test/integration/aligned_alloc.c:32: Failed assertion: (p != ((void*)0) || get_errno() != 22) == (0) --> true != false: Expected error for invalid alignment 524289
  test_alignment_errors:test/integration/aligned_alloc.c:32: Failed assertion: (p != ((void*)0) || get_errno() != 22) == (0) --> true != false: Expected error for invalid alignment 1048577
  test_alignment_errors:test/integration/aligned_alloc.c:32: Failed assertion: (p != ((void*)0) || get_errno() != 22) == (0) --> true != false: Expected error for invalid alignment 2097153
  test_alignment_errors:test/integration/aligned_alloc.c:32: Failed assertion: (p != ((void*)0) || get_errno() != 22) == (0) --> true != false: Expected error for invalid alignment 4194305
  test_alignment_errors (libc-reentrant): fail
  test_alignment_errors:test/integration/aligned_alloc.c:24: Failed assertion: (p != ((void*)0) || get_errno() != 22) == (0) --> true != false: Expected error for invalid alignment 0
  test_alignment_errors:test/integration/aligned_alloc.c:32: Failed assertion: (p != ((void*)0) || get_errno() != 22) == (0) --> true != false: Expected error for invalid alignment 9
  test_alignment_errors:test/integration/aligned_alloc.c:32: Failed assertion: (p != ((void*)0) || get_errno() != 22) == (0) --> true != false: Expected error for invalid alignment 17
  test_alignment_errors:test/integration/aligned_alloc.c:32: Failed assertion: (p != ((void*)0) || get_errno() != 22) == (0) --> true != false: Expected error for invalid alignment 33
  test_alignment_errors:test/integration/aligned_alloc.c:32: Failed assertion: (p != ((void*)0) || get_errno() != 22) == (0) --> true != false: Expected error for invalid alignment 65
  test_alignment_errors:test/integration/aligned_alloc.c:32: Failed assertion: (p != ((void*)0) || get_errno() != 22) == (0) --> true != false: Expected error for invalid alignment 129
  test_alignment_errors:test/integration/aligned_alloc.c:32: Failed assertion: (p != ((void*)0) || get_errno() != 22) == (0) --> true != false: Expected error for invalid alignment 257
  test_alignment_errors:test/integration/aligned_alloc.c:32: Failed assertion: (p != ((void*)0) || get_errno() != 22) == (0) --> true != false: Expected error for invalid alignment 513
  test_alignment_errors:test/integration/aligned_alloc.c:32: Failed assertion: (p != ((void*)0) || get_errno() != 22) == (0) --> true != false: Expected error for invalid alignment 1025
  test_alignment_errors:test/integration/aligned_alloc.c:32: Failed assertion: (p != ((void*)0) || get_errno() != 22) == (0) --> true != false: Expected error for invalid alignment 2049
  test_alignment_errors:test/integration/aligned_alloc.c:32: Failed assertion: (p != ((void*)0) || get_errno() != 22) == (0) --> true != false: Expected error for invalid alignment 4097
  test_alignment_errors:test/integration/aligned_alloc.c:32: Failed assertion: (p != ((void*)0) || get_errno() != 22) == (0) --> true != false: Expected error for invalid alignment 8193
  test_alignment_errors:test/integration/aligned_alloc.c:32: Failed assertion: (p != ((void*)0) || get_errno() != 22) == (0) --> true != false: Expected error for invalid alignment 16385
  test_alignment_errors:test/integration/aligned_alloc.c:32: Failed assertion: (p != ((void*)0) || get_errno() != 22) == (0) --> true != false: Expected error for invalid alignment 32769
  test_alignment_errors:test/integration/aligned_alloc.c:32: Failed assertion: (p != ((void*)0) || get_errno() != 22) == (0) --> true != false: Expected error for invalid alignment 65537
  test_alignment_errors:test/integration/aligned_alloc.c:32: Failed assertion: (p != ((void*)0) || get_errno() != 22) == (0) --> true != false: Expected error for invalid alignment 131073
  test_alignment_errors:test/integration/aligned_alloc.c:32: Failed assertion: (p != ((void*)0) || get_errno() != 22) == (0) --> true != false: Expected error for invalid alignment 262145
  test_alignment_errors:test/integration/aligned_alloc.c:32: Failed assertion: (p != ((void*)0) || get_errno() != 22) == (0) --> true != false: Expected error for invalid alignment 524289
  test_alignment_errors:test/integration/aligned_alloc.c:32: Failed assertion: (p != ((void*)0) || get_errno() != 22) == (0) --> true != false: Expected error for invalid alignment 1048577
  test_alignment_errors:test/integration/aligned_alloc.c:32: Failed assertion: (p != ((void*)0) || get_errno() != 22) == (0) --> true != false: Expected error for invalid alignment 2097153
  test_alignment_errors:test/integration/aligned_alloc.c:32: Failed assertion: (p != ((void*)0) || get_errno() != 22) == (0) --> true != false: Expected error for invalid alignment 4194305
  test_alignment_errors (arena_new-reentrant): fail
  test_oom_errors:test/integration/aligned_alloc.c:63: Failed assertion: (p != ((void*)0) || get_errno() != 12) == (0) --> true != false: Expected error for aligned_alloc(9223372036854775808, 9223372036854775808)
  test_oom_errors:test/integration/aligned_alloc.c:76: Failed assertion: (p != ((void*)0) || get_errno() != 12) == (0) --> true != false: Expected error for aligned_alloc(4611686018427387904, 13835058055282163713)
  test_oom_errors:test/integration/aligned_alloc.c:88: Failed assertion: (p != ((void*)0) || get_errno() != 12) == (0) --> true != false: Expected error for aligned_alloc(&p, 16, 18446744073709551600)
  test_oom_errors (non-reentrant): fail
  test_oom_errors:test/integration/aligned_alloc.c:63: Failed assertion: (p != ((void*)0) || get_errno() != 12) == (0) --> true != false: Expected error for aligned_alloc(9223372036854775808, 9223372036854775808)
  test_oom_errors:test/integration/aligned_alloc.c:76: Failed assertion: (p != ((void*)0) || get_errno() != 12) == (0) --> true != false: Expected error for aligned_alloc(4611686018427387904, 13835058055282163713)
  test_oom_errors:test/integration/aligned_alloc.c:88: Failed assertion: (p != ((void*)0) || get_errno() != 12) == (0) --> true != false: Expected error for aligned_alloc(&p, 16, 18446744073709551600)
  test_oom_errors (libc-reentrant): fail
  test_oom_errors:test/integration/aligned_alloc.c:63: Failed assertion: (p != ((void*)0) || get_errno() != 12) == (0) --> true != false: Expected error for aligned_alloc(9223372036854775808, 9223372036854775808)
  test_oom_errors:test/integration/aligned_alloc.c:76: Failed assertion: (p != ((void*)0) || get_errno() != 12) == (0) --> true != false: Expected error for aligned_alloc(4611686018427387904, 13835058055282163713)
  test_oom_errors:test/integration/aligned_alloc.c:88: Failed assertion: (p != ((void*)0) || get_errno() != 12) == (0) --> true != false: Expected error for aligned_alloc(&p, 16, 18446744073709551600)
  test_oom_errors (arena_new-reentrant): fail
```

#### 相关PR

- 上游社区pr：[Add attribute to compile some tests in aligned_alloc.c with no optimization by yanyir · Pull Request #2704 · jemalloc/jemalloc (github.com)](https://github.com/jemalloc/jemalloc/pull/2704)

- openEuler中间仓pr：[Support clang build -- Add attribute to compile some tests in aligned_alloc.c with no optimization · Pull Request !43 · src-openEuler/jemalloc - Gitee.com](https://gitee.com/src-openeuler/jemalloc/pulls/43)

### 修复过程

- 上游社区已有对应issue：[Test failure on macOS Sonoma · Issue #2540 · jemalloc/jemalloc (github.com)](https://gitee.com/link?target=https%3A%2F%2Fgithub.com%2Fjemalloc%2Fjemalloc%2Fissues%2F2540)，issue作者使用apple的clang进行测试，遇到该问题，指出问题的根源在于clang将代码优化后关键代码消失，故不能产生预期输出。该作者向Homebrew社区提了issue并得到合并：[jemalloc: fix build with Xcode 15 by fxcoudert · Pull Request #146688 · Homebrew/homebrew-core (github.com)](https://gitee.com/link?target=https%3A%2F%2Fgithub.com%2FHomebrew%2Fhomebrew-core%2Fpull%2F146688)。但jemalloc的作者并未对此问题做出修正，issue作者的解决方案为在clang版本大于等于15时只执行`make`而非`make check`。

- 更好的做法应该是对这些特定的测试用例不做编译优化，比如使用`__attribute__((optnone))`

- **修复建议**：在出问题的两个测试函数之前加上

  ```c
  #if __clang__
  __attribute__((optnone))
  #endif
  ```

