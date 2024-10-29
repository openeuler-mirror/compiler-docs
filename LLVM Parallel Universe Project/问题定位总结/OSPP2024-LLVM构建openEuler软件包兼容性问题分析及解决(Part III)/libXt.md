## libXt

### 问题描述

#### 报错信息

在修改前使用clang构建出现错误，软件包构建于check阶段失败：

```
+ make check
Making check in util
make[1]: Entering directory '/home/abuild/rpmbuild/BUILD/libXt-1.3.0/util'
make[1]: Nothing to be done for 'check'.
make[1]: Leaving directory '/home/abuild/rpmbuild/BUILD/libXt-1.3.0/util'
Making check in src
make[1]: Entering directory '/home/abuild/rpmbuild/BUILD/libXt-1.3.0/src'
make  check-am
make[2]: Entering directory '/home/abuild/rpmbuild/BUILD/libXt-1.3.0/src'
make[2]: Nothing to be done for 'check-am'.
make[2]: Leaving directory '/home/abuild/rpmbuild/BUILD/libXt-1.3.0/src'
make[1]: Leaving directory '/home/abuild/rpmbuild/BUILD/libXt-1.3.0/src'
Making check in include
make[1]: Entering directory '/home/abuild/rpmbuild/BUILD/libXt-1.3.0/include'
make[1]: Nothing to be done for 'check'.
make[1]: Leaving directory '/home/abuild/rpmbuild/BUILD/libXt-1.3.0/include'
Making check in man
make[1]: Entering directory '/home/abuild/rpmbuild/BUILD/libXt-1.3.0/man'
make[1]: Nothing to be done for 'check'.
make[1]: Leaving directory '/home/abuild/rpmbuild/BUILD/libXt-1.3.0/man'
Making check in specs
make[1]: Entering directory '/home/abuild/rpmbuild/BUILD/libXt-1.3.0/specs'
make[1]: Nothing to be done for 'check'.
make[1]: Leaving directory '/home/abuild/rpmbuild/BUILD/libXt-1.3.0/specs'
Making check in test
make[1]: Entering directory '/home/abuild/rpmbuild/BUILD/libXt-1.3.0/test'
make  Alloc Converters Event
make[2]: Entering directory '/home/abuild/rpmbuild/BUILD/libXt-1.3.0/test'
  CC       Alloc.o
  CCLD     Alloc
  CC       Converters.o
  CCLD     Converters
  CC       Event.o
Event.c:51:1: warning: function 'sigalrm' could be declared with attribute 'noreturn' [-Wmissing-noreturn]
   51 | {
      | ^
1 warning generated.
  CCLD     Event
make[2]: Leaving directory '/home/abuild/rpmbuild/BUILD/libXt-1.3.0/test'
make  check-TESTS
make[2]: Entering directory '/home/abuild/rpmbuild/BUILD/libXt-1.3.0/test'
make[3]: Entering directory '/home/abuild/rpmbuild/BUILD/libXt-1.3.0/test'
../test-driver: line 112:  9366 Aborted                 (core dumped) "$@" >> "$log_file" 2>&1
FAIL: Alloc
PASS: Converters
PASS: Event
============================================================================
Testsuite summary for libXt 1.3.0
============================================================================
# TOTAL: 3
# PASS:  2
# SKIP:  0
# XFAIL: 0
# FAIL:  1
# XPASS: 0
# ERROR: 0
============================================================================
See test/test-suite.log
Please report to https://gitlab.freedesktop.org/xorg/lib/libxt/-/issues/
```

其中，test/test-suite.log显示为：

```
======================================
   libXt 1.3.0: test/test-suite.log
======================================

# TOTAL: 3
# PASS:  2
# SKIP:  0
# XFAIL: 0
# FAIL:  1
# XPASS: 0
# ERROR: 0

.. contents:: :depth: 2

FAIL: Alloc
===========

TAP version 13
# random seed: R02S8d0db6758044940514b2e6e038c63bd3
1..18
# Start of Alloc tests
# Start of XtAsprintf tests
ok 1 /Alloc/XtAsprintf/short
ok 2 /Alloc/XtAsprintf/long
# End of XtAsprintf tests
# Start of XtMalloc tests
ok 3 /Alloc/XtMalloc/normal
ok 4 /Alloc/XtMalloc/zero
Warning: This program is an suid-root program or is being run by the root user.
The full text of the error or warning message cannot be safely formatted
in this environment. You may get a more descriptive message by running the
program as a non-root user or by removing the suid bit on the executable.
Caught Error: Cannot perform %s
ok 5 /Alloc/XtMalloc/oversize
ok 6 /Alloc/XtMalloc/overflow # SKIP overflow not possible in this config
# End of XtMalloc tests
# Start of XtCalloc tests
ok 7 /Alloc/XtCalloc/normal
ok 8 /Alloc/XtCalloc/zero
Warning: This program is an suid-root program or is being run by the root user.
The full text of the error or warning message cannot be safely formatted
in this environment. You may get a more descriptive message by running the
program as a non-root user or by removing the suid bit on the executable.
Caught Error: Cannot perform %s
ok 9 /Alloc/XtCalloc/oversize
ok 10 /Alloc/XtCalloc/overflow # SKIP overflow not possible in this config
# End of XtCalloc tests
# Start of XtRealloc tests
ok 11 /Alloc/XtRealloc/normal
Warning: This program is an suid-root program or is being run by the root user.
The full text of the error or warning message cannot be safely formatted
in this environment. You may get a more descriptive message by running the
program as a non-root user or by removing the suid bit on the executable.
Caught Error: Cannot perform %s
ok 12 /Alloc/XtRealloc/zero
ok 13 /Alloc/XtRealloc/overflow # SKIP overflow not possible in this config
free(): double free detected in tcache 2
FAIL Alloc (exit status: 134)
```

#### 相关PR

- 上游社区PR：https://gitlab.freedesktop.org/xorg/util/macros/-/merge_requests/8
- openEuler仓库pr：https://gitee.com/src-openeuler/libXt/pulls/31

### 修复过程

- 上游社区没有相关issue。问题的根源是软件包中的文件都使用了-O2的编译优化，当使用clang构建时，-O1及以上的优化水平会导致clang将malloc优化掉，从而在`configure`脚本中的这一段：

  ```
  # Check whether --enable-malloc0returnsnull was given.
  if test ${enable_malloc0returnsnull+y}
  then :
    enableval=$enable_malloc0returnsnull; MALLOC_ZERO_RETURNS_NULL=$enableval
  else $as_nop
    MALLOC_ZERO_RETURNS_NULL=auto
  fi
  
  
  { printf "%s\n" "$as_me:${as_lineno-$LINENO}: checking whether malloc(0) returns NULL" >&5
  printf %s "checking whether malloc(0) returns NULL... " >&6; }
  if test "x$MALLOC_ZERO_RETURNS_NULL" = xauto; then
  if test ${xorg_cv_malloc0_returns_null+y}
  then :
    printf %s "(cached) " >&6
  else $as_nop
    if test "$cross_compiling" = yes
  then :
    printf %s "(cached) " >&6
  else $as_nop
    if test "$cross_compiling" = yes
  then :
    { { printf "%s\n" "$as_me:${as_lineno-$LINENO}: error: in \`$ac_pwd':" >&5
  printf "%s\n" "$as_me: error: in \`$ac_pwd':" >&2;}
  as_fn_error $? "cannot run test program while cross compiling
  See \`config.log' for more details" "$LINENO" 5; }
  else $as_nop
    cat confdefs.h - <<_ACEOF >conftest.$ac_ext
  /* end confdefs.h.  */
  
  #include <stdlib.h>
  
  int
  main (void)
  {
  
      char *m0, *r0, *c0, *p;
      m0 = malloc(0);
      p = malloc(10);
      r0 = realloc(p,0);
      c0 = calloc(0,10);
      exit((m0 == 0 || r0 == 0 || c0 == 0) ? 0 : 1);
  
    ;
    return 0;
  }
  _ACEOF
  if ac_fn_c_try_run "$LINENO"
  then :
    xorg_cv_malloc0_returns_null=yes
  else $as_nop
    xorg_cv_malloc0_returns_null=no
  fi
  rm -f core *.core core.conftest.* gmon.out bb.out conftest$ac_exeext \
    conftest.$ac_objext conftest.beam conftest.$ac_ext
  fi
  
  fi
  
  MALLOC_ZERO_RETURNS_NULL=$xorg_cv_malloc0_returns_null
  fi
  { printf "%s\n" "$as_me:${as_lineno-$LINENO}: result: $MALLOC_ZERO_RETURNS_NULL" >&5
  printf "%s\n" "$MALLOC_ZERO_RETURNS_NULL" >&6; }
  
  if test "x$MALLOC_ZERO_RETURNS_NULL" = xyes; then
          MALLOC_ZERO_CFLAGS="-DMALLOC_0_RETURNS_NULL"
          XMALLOC_ZERO_CFLAGS=$MALLOC_ZERO_CFLAGS
          XTMALLOC_ZERO_CFLAGS="$MALLOC_ZERO_CFLAGS -DXTMALLOC_BC"
  else
          MALLOC_ZERO_CFLAGS=""
          XMALLOC_ZERO_CFLAGS=""
          XTMALLOC_ZERO_CFLAGS=""
  fi
  ```

  当中的conftest获得了与gcc不同的结果（若编译时采用-O0，clang的脚本执行结果与gcc相同），从而本应执行的`MALLOC_ZERO_CFLAGS="-DMALLOC_0_RETURNS_NULL"`变为了`MALLOC_ZERO_CFLAGS=""`，从而导致在`src/Alloc.c`中以下函数执行时，

  ```
  char *
  XtRealloc(char *ptr, unsigned size)
  {
      if (ptr == NULL) {
  #ifdef MALLOC_0_RETURNS_NULL
          if (!size)
              size = 1;
  #endif
          return (XtMalloc(size));
      }
      else if ((ptr = Xrealloc(ptr, size)) == NULL
  #ifdef MALLOC_0_RETURNS_NULL
               && size
  #endif
          )
          _XtAllocError("realloc");
  
      return (ptr);
  }
  ```

  由于没有定义`MALLOC_0_RETURNS_NULL`所以不会判断`&& size`，做出错误的行为。但是`configure`脚本通过`configure.ac`生成，而`configure.ac`中使用的是`xorg-macros`（[xorg-macros.m4.in · master · xorg / util / macros · GitLab](https://gitee.com/link?target=https%3A%2F%2Fgitlab.freedesktop.org%2Fxorg%2Futil%2Fmacros%2F-%2Fblob%2Fmaster%2Fxorg-macros.m4.in%3Fref_type%3Dheads)）的`XORG_CHECK_MALLOC_ZERO`来产生上文提到的片段：

  ```
  # Require X.Org macros 1.16 or later for XORG_MEMORY_CHECK_FLAGS
  m4_ifndef([XORG_MACROS_VERSION],
  	  [m4_fatal([must install xorg-macros 1.16 or later before running autoconf/autogen])])
  XORG_MACROS_VERSION(1.16)
  XORG_DEFAULT_OPTIONS
  XORG_CHECK_MALLOC_ZERO
  XORG_ENABLE_SPECS
  XORG_WITH_XMLTO(0.0.20)
  XORG_WITH_FOP
  XORG_WITH_XSLTPROC
  XORG_CHECK_SGML_DOCTOOLS(1.01)
  XORG_PROG_RAWCPP
  ```

- 修复方式：目前采用的是直接修改spec文件，当使用clang构建时传入`--enable-malloc0returnsnull`的configure参数的办法来强制指定的简单方案。在xorg-macros无法提pr，提的issue得到了回复，问题在https://gitlab.freedesktop.org/xorg/util/macros/-/merge_requests/8得到了解决，但可能需要等待更新。
