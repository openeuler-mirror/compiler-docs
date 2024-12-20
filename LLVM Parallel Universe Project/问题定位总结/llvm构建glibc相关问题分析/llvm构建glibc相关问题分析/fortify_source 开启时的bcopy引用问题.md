# fortify_source 开启时的bcopy引用问题

llvm会匹配内联函数名称，并替换为llvm内置函数。fortify_level >= 1时，这个替换优先于glibc提供的内联函数，会造成glibc相关运行时检查失效。

```

#include<string.h>
int main(){
        char buf[9];
        memmove (buf + 2, buf + 1, 9);
        bcopy (buf + 1, buf + 2, 9);
        return buf[5];
}
// clang bcopy.c -O2 -Wp,-U_FORTIFY_SOURCE,-D_FORTIFY_SOURCE=1 -S -emit-llvm
```

正常情况下应该调用glibc中的fortify内联实现，内联为___memmove_chk

```

#ifndef __STRINGS_FORTIFIED
# define __STRINGS_FORTIFIED 1
__fortify_function void
NTH (bcopy (const void *src, void *__dest, size_t __len))
{
  (void) builtin_memmove_chk (__dest, src, len,
                  __glibc_objsize0 (__dest));
}
__fortify_function void
NTH (bzero (void *dest, size_t __len))
{
  (void) builtin_memset_chk (__dest, '\0', __len,
                 __glibc_objsize0 (__dest));
}
#endif
```

查看ir可以发现bcopy被替换为 @llvm.memmove 即 __builtin_memmove

```

; ModuleID = 'bcopy.c'
source_filename = "bcopy.c"
target datalayout = "e-m:e-i8:8:32-i16:16:32-i64:64-i128:128-n32:64-S128"
target triple = "aarch64-unknown-linux-gnu"
; Function Attrs: nofree nounwind uwtable
define dso_local i32 @main() local_unnamed_addr #0 {
  %1 = alloca [9 x i8], align 1
  call void @llvm.lifetime.start.p0(i64 9, ptr nonnull %1) #4
  %2 = getelementptr inbounds i8, ptr %1, i64 2
  %3 = getelementptr inbounds i8, ptr %1, i64 1
  %4 = call ptr @__memmove_chk(ptr noundef nonnull %2, ptr noundef nonnull %3, i64 noundef 9, i64 noundef 7) #4
  call void @llvm.memmove.p0.p0.i64(ptr noundef nonnull align 1 dereferenceable(9) %2, ptr noundef nonnull align 1 dereferenceable(9) %3, i64 9, i1 false)
  %5 = getelementptr inbounds [9 x i8], ptr %1, i64 0, i64 5
  %6 = load i8, ptr %5, align 1, !tbaa !6
  %7 = zext i8 %6 to i32
  call void @llvm.lifetime.end.p0(i64 9, ptr nonnull %1) #4
  ret i32 %7
}
```

lvm社区的[这个提交](https://reviews.llvm.org/D71082)通过检测名称为内置名称但也具有头文件提供的实现的函数，解决上述问题。

[⚙ D71082 Allow system header to provide their own implementation of some builtin](https://reviews.llvm.org/D71082)

但是bcopy为例外，由于没有对应的__builtin_bcopy，fortify_level >= 1的情况下依然会被转化为__builtin_memmove，而不是glibc内联函数。

llvm18通过[添加__builtin_bcopy](https://github.com/llvm/llvm-project/pull/67130)解决了这个问题。

[https://github.com/llvm/llvm-project/pull/67130](https://github.com/llvm/llvm-project/pull/67130)

总之是编译器问题，而非clang构建glibc导致的问题，如果需要规避可以在测试用例构建时添加-fno-builtin

已经回合到社区clang17

- 复现命令
    
    ```cpp
    clang /root/rpmbuild1/BUILD/glibc-2.38/build-aarch64-openEuler-linux/debug/tst-fortify-c-default-1-def.c -c -std=gnu11 -fgnu89-inline  -O2 -g -DNDEBUG -fPIC -fPIE -fstack-protector-strong -mno-outline-atomics -Wp,-D_GLIBCXX_ASSERTIONS -fasynchronous-unwind-tables -fstack-clash-protection -fgcc-compatible -Wall -Wwrite-strings -Wundef -Werror -fmerge-all-constants -frounding-math -ftrapping-math -fstack-protector-strong -fno-common -Wp,-U_FORTIFY_SOURCE -Wstrict-prototypes -Wold-style-definition -fmath-errno    -fPIE   -Wp,-U_FORTIFY_SOURCE,-D_FORTIFY_SOURCE=1 -Wno-format -Wno-deprecated-declarations -Wno-error       -I../include -I/root/rpmbuild1/BUILD/glibc-2.38/build-aarch64-openEuler-linux/debug  -I/root/rpmbuild1/BUILD/glibc-2.38/build-aarch64-openEuler-linux  -I../sysdeps/unix/sysv/linux/aarch64  -I../sysdeps/aarch64/nptl  -I../sysdeps/unix/sysv/linux/wordsize-64  -I../sysdeps/unix/sysv/linux/include -I../sysdeps/unix/sysv/linux  -I../sysdeps/nptl  -I../sysdeps/pthread  -I../sysdeps/gnu  -I../sysdeps/unix/inet  -I../sysdeps/unix/sysv  -I../sysdeps/unix  -I../sysdeps/posix  -I../sysdeps/aarch64/fpu  -I../sysdeps/aarch64/multiarch  -I../sysdeps/aarch64  -I../sysdeps/wordsize-64  -I../sysdeps/ieee754/ldbl-128  -I../sysdeps/ieee754/dbl-64  -I../sysdeps/ieee754/flt-32  -I../sysdeps/ieee754  -I../sysdeps/generic  -I.. -I../libio -I. -nostdinc -isystem /usr/bin/../lib64/clang/17/include -isystem /usr/include -D_LIBC_REENTRANT -include /root/rpmbuild1/BUILD/glibc-2.38/build-aarch64-openEuler-linux/libc-modules.h -DMODULE_NAME=testsuite -include ../include/libc-symbols.h  -DPIC     -DTOP_NAMESPACE=glibc -o /root/rpmbuild1/BUILD/glibc-2.38/build-aarch64-openEuler-linux/debug/tst-fortify-c-default-1-def.o -MD -MP -MF /root/rpmbuild1/BUILD/glibc-2.38/build-aarch64-openEuler-linux/debug/tst-fortify-c-default-1-def.o.dt -MT /root/rpmbuild1/BUILD/glibc-2.38/build-aarch64-openEuler-linux/debug/tst-fortify-c-default-1-def.o
    clang -o /root/rpmbuild1/BUILD/glibc-2.38/build-aarch64-openEuler-linux/debug/tst-fortify-c-default-1-def -pie  -Wl,-O1 -nostdlib -nostartfiles    -Wl,-z,relro  /root/rpmbuild1/BUILD/glibc-2.38/build-aarch64-openEuler-linux/csu/Scrt1.o /root/rpmbuild1/BUILD/glibc-2.38/build-aarch64-openEuler-linux/csu/crti.o `clang  --print-file-name=crtbeginS.o` /root/rpmbuild1/BUILD/glibc-2.38/build-aarch64-openEuler-linux/debug/tst-fortify-c-default-1-def.o /root/rpmbuild1/BUILD/glibc-2.38/build-aarch64-openEuler-linux/support/libsupport_nonshared.a  -Wl,-dynamic-linker=/lib/ld-linux-aarch64.so.1 -Wl,-rpath-link=/root/rpmbuild1/BUILD/glibc-2.38/build-aarch64-openEuler-linux:/root/rpmbuild1/BUILD/glibc-2.38/build-aarch64-openEuler-linux/math:/root/rpmbuild1/BUILD/glibc-2.38/build-aarch64-openEuler-linux/elf:/root/rpmbuild1/BUILD/glibc-2.38/build-aarch64-openEuler-linux/dlfcn:/root/rpmbuild1/BUILD/glibc-2.38/build-aarch64-openEuler-linux/nss:/root/rpmbuild1/BUILD/glibc-2.38/build-aarch64-openEuler-linux/nis:/root/rpmbuild1/BUILD/glibc-2.38/build-aarch64-openEuler-linux/rt:/root/rpmbuild1/BUILD/glibc-2.38/build-aarch64-openEuler-linux/resolv:/root/rpmbuild1/BUILD/glibc-2.38/build-aarch64-openEuler-linux/mathvec:/root/rpmbuild1/BUILD/glibc-2.38/build-aarch64-openEuler-linux/support:/root/rpmbuild1/BUILD/glibc-2.38/build-aarch64-openEuler-linux/nptl -lgcc -Wl,--as-needed -lgcc_s  -Wl,--no-as-needed /root/rpmbuild1/BUILD/glibc-2.38/build-aarch64-openEuler-linux/libc.so.6 /root/rpmbuild1/BUILD/glibc-2.38/build-aarch64-openEuler-linux/libc_nonshared.a -Wl,--as-needed /root/rpmbuild1/BUILD/glibc-2.38/build-aarch64-openEuler-linux/elf/ld.so -Wl,--no-as-needed -lgcc -Wl,--as-needed -lgcc_s  -Wl,--no-as-needed `clang  --print-file-name=crtendS.o` /root/rpmbuild1/BUILD/glibc-2.38/build-aarch64-openEuler-linux/csu/crtn.o
    env GCONV_PATH=/root/rpmbuild1/BUILD/glibc-2.38/build-aarch64-openEuler-linux/iconvdata LOCPATH=/root/rpmbuild1/BUILD/glibc-2.38/build-aarch64-openEuler-linux/localedata LC_ALL=C   /root/rpmbuild1/BUILD/glibc-2.38/build-aarch64-openEuler-linux/elf/ld-linux-aarch64.so.1 --library-path /root/rpmbuild1/BUILD/glibc-2.38/build-aarch64-openEuler-linux:/root/rpmbuild1/BUILD/glibc-2.38/build-aarch64-openEuler-linux/math:/root/rpmbuild1/BUILD/glibc-2.38/build-aarch64-openEuler-linux/elf:/root/rpmbuild1/BUILD/glibc-2.38/build-aarch64-openEuler-linux/dlfcn:/root/rpmbuild1/BUILD/glibc-2.38/build-aarch64-openEuler-linux/nss:/root/rpmbuild1/BUILD/glibc-2.38/build-aarch64-openEuler-linux/nis:/root/rpmbuild1/BUILD/glibc-2.38/build-aarch64-openEuler-linux/rt:/root/rpmbuild1/BUILD/glibc-2.38/build-aarch64-openEuler-linux/resolv:/root/rpmbuild1/BUILD/glibc-2.38/build-aarch64-openEuler-linux/mathvec:/root/rpmbuild1/BUILD/glibc-2.38/build-aarch64-openEuler-linux/support:/root/rpmbuild1/BUILD/glibc-2.38/build-aarch64-openEuler-linux/nptl /root/rpmbuild1/BUILD/glibc-2.38/build-aarch64-openEuler-linux/debug/tst-fortify-c-default-1-def
    ```