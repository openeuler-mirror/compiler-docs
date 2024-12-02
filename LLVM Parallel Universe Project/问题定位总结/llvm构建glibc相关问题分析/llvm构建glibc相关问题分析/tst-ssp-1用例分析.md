# tst-ssp-1用例分析

- 复现指令
    
    ```cpp
    clang tst-ssp-1.c -c -std=gnu11 -fgnu89-inline  -O2 -g -DNDEBUG -fPIC -fPIE -fstack-protector-strong -mno-outline-atomics -Wp,-D_GLIBCXX_ASSERTIONS -fasynchronous-unwind-tables -fstack-clash-protection -fgcc-compatible -Wall -Wwrite-strings -Wundef -Werror -fmerge-all-constants -frounding-math -fstack-protector-strong -fno-common -Wp,-U_FORTIFY_SOURCE -Wstrict-prototypes -Wold-style-definition -fmath-errno    -fPIE   -fstack-protector-all       -I../include -I/root/rpmbuild/BUILD/glibc-2.38/build-aarch64-openEuler-linux/debug  -I/root/rpmbuild/BUILD/glibc-2.38/build-aarch64-openEuler-linux  -I../sysdeps/unix/sysv/linux/aarch64  -I../sysdeps/aarch64/nptl  -I../sysdeps/unix/sysv/linux/wordsize-64  -I../sysdeps/unix/sysv/linux/include -I../sysdeps/unix/sysv/linux  -I../sysdeps/nptl  -I../sysdeps/pthread  -I../sysdeps/gnu  -I../sysdeps/unix/inet  -I../sysdeps/unix/sysv  -I../sysdeps/unix  -I../sysdeps/posix  -I../sysdeps/aarch64/fpu  -I../sysdeps/aarch64/multiarch  -I../sysdeps/aarch64  -I../sysdeps/wordsize-64  -I../sysdeps/ieee754/ldbl-128  -I../sysdeps/ieee754/dbl-64  -I../sysdeps/ieee754/flt-32  -I../sysdeps/ieee754  -I../sysdeps/generic  -I.. -I../libio -I. -nostdinc -isystem /usr/bin/../lib64/clang/17/include -isystem /usr/include -D_LIBC_REENTRANT -include /root/rpmbuild/BUILD/glibc-2.38/build-aarch64-openEuler-linux/libc-modules.h -DMODULE_NAME=testsuite -include ../include/libc-symbols.h  -DPIC     -DTOP_NAMESPACE=glibc -o /root/rpmbuild/BUILD/glibc-2.38/build-aarch64-openEuler-linux/debug/tst-ssp-1.o -MD -MP -MF /root/rpmbuild/BUILD/glibc-2.38/build-aarch64-openEuler-linux/debug/tst-ssp-1.o.dt -MT /root/rpmbuild/BUILD/glibc-2.38/build-aarch64-openEuler-linux/debug/tst-ssp-1.o
    
    clang -o /root/rpmbuild/BUILD/glibc-2.38/build-aarch64-openEuler-linux/debug/tst-ssp-1 -pie  -Wl,-O1 -nostdlib -nostartfiles    -Wl,-z,relro  /root/rpmbuild/BUILD/glibc-2.38/build-aarch64-openEuler-linux/csu/Scrt1.o /root/rpmbuild/BUILD/glibc-2.38/build-aarch64-openEuler-linux/csu/crti.o `clang  --print-file-name=crtbeginS.o` /root/rpmbuild/BUILD/glibc-2.38/build-aarch64-openEuler-linux/debug/tst-ssp-1.o /root/rpmbuild/BUILD/glibc-2.38/build-aarch64-openEuler-linux/support/libsupport_nonshared.a  -Wl,-dynamic-linker=/lib/ld-linux-aarch64.so.1 -Wl,-rpath-link=/root/rpmbuild/BUILD/glibc-2.38/build-aarch64-openEuler-linux:/root/rpmbuild/BUILD/glibc-2.38/build-aarch64-openEuler-linux/math:/root/rpmbuild/BUILD/glibc-2.38/build-aarch64-openEuler-linux/elf:/root/rpmbuild/BUILD/glibc-2.38/build-aarch64-openEuler-linux/dlfcn:/root/rpmbuild/BUILD/glibc-2.38/build-aarch64-openEuler-linux/nss:/root/rpmbuild/BUILD/glibc-2.38/build-aarch64-openEuler-linux/nis:/root/rpmbuild/BUILD/glibc-2.38/build-aarch64-openEuler-linux/rt:/root/rpmbuild/BUILD/glibc-2.38/build-aarch64-openEuler-linux/resolv:/root/rpmbuild/BUILD/glibc-2.38/build-aarch64-openEuler-linux/mathvec:/root/rpmbuild/BUILD/glibc-2.38/build-aarch64-openEuler-linux/support:/root/rpmbuild/BUILD/glibc-2.38/build-aarch64-openEuler-linux/nptl -lgcc -Wl,--as-needed -lgcc_s  -Wl,--no-as-needed /root/rpmbuild/BUILD/glibc-2.38/build-aarch64-openEuler-linux/libc.so.6 /root/rpmbuild/BUILD/glibc-2.38/build-aarch64-openEuler-linux/libc_nonshared.a -Wl,--as-needed /root/rpmbuild/BUILD/glibc-2.38/build-aarch64-openEuler-linux/elf/ld.so -Wl,--no-as-needed -lgcc -Wl,--as-needed -lgcc_s  -Wl,--no-as-needed `clang  --print-file-name=crtendS.o` /root/rpmbuild/BUILD/glibc-2.38/build-aarch64-openEuler-linux/csu/crtn.o
    
    /root/rpmbuild/BUILD/glibc-2.38/build-aarch64-openEuler-linux/elf/ld-linux-aarch64.so.1 --library-path /root/rpmbuild/BUILD/glibc-2.38/build-aarch64-openEuler-linux:/root/rpmbuild/BUILD/glibc-2.38/build-aarch64-openEuler-linux/math:/root/rpmbuild/BUILD/glibc-2.38/build-aarch64-openEuler-linux/elf:/root/rpmbuild/BUILD/glibc-2.38/build-aarch64-openEuler-linux/dlfcn:/root/rpmbuild/BUILD/glibc-2.38/build-aarch64-openEuler-linux/nss:/root/rpmbuild/BUILD/glibc-2.38/build-aarch64-openEuler-linux/nis:/root/rpmbuild/BUILD/glibc-2.38/build-aarch64-openEuler-linux/rt:/root/rpmbuild/BUILD/glibc-2.38/build-aarch64-openEuler-linux/resolv:/root/rpmbuild/BUILD/glibc-2.38/build-aarch64-openEuler-linux/mathvec:/root/rpmbuild/BUILD/glibc-2.38/build-aarch64-openEuler-linux/support:/root/rpmbuild/BUILD/glibc-2.38/build-aarch64-openEuler-linux/nptl /root/rpmbuild/BUILD/glibc-2.38/build-aarch64-openEuler-linux/debug/tst-ssp-1
    
    # 可以直接执行下面这个
    
    /root/rpmbuild/BUILD/glibc-2.38/build-aarch64-openEuler-linux/debug/tst-ssp-1
    ```
    

分析用例执行过程，是在以下用例中添加`-fstack-protector-strong`和`-fstack-protector-all` ，检查相关检测是否生效

```cpp
static void
__attribute__ ((noinline)) __attribute_noclone__
test (char *foo)
{
  int i;

  /* smash stack */
  for (i = 0; i <= 400; i++)
    foo[i] = 42;
}

static int
do_test (void)
{
  char foo[30];

  test (foo);

  return 1; /* fail */
}
```

预计输出是这样的，返回abort

```cpp
*** stack smashing detected ***: terminated
Aborted (core dumped)
```

但是实际上没有输出，排查发现是添加了O2之后的问题，O2去除之后就能成功执行

godbolt例子https://godbolt.org/z/GaMo6d594