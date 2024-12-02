# math相关用例分析

# math/test-*-pow

定位是clang构建的pow(-0x1.000002p+0, 0xf.ffffffffffff8p+1020)触发了underflow和overflow浮点异常，但期望值是仅触发overflow浮点异常，导致用例失败。

整体加了fno-builtin进行构建，也没有效果，排除builtin函数导致

分析汇编差异，发现clang版本这部分进行了两次乘法，导致了两种浮点异常

![image.png](.images/image%202.png)

这段代码是这样的

```cpp
/* |y| is huge.
     2^-16495 = 1/2 of smallest representable value.
     If (1 - 1/131072)^y underflows, y > 1.4986e9 */
  if (iy > 0x401d654b)
    {
      /* if (1 - 2^-113)^y underflows, y > 1.1873e38 */
      if (iy > 0x407d654b)
	{// 关注这个控制流
	  if (ix <= 0x3ffeffff)
	    return (hy < 0) ? huge * huge : tiny * tiny;
	  if (ix >= 0x3fff0000)
	    return (hy > 0) ? huge * huge : tiny * tiny;
	}
      /* over/underflow if x is not close to one */
      if (ix < 0x3ffeffff)
	return (hy < 0) ? sgn * huge * huge : sgn * tiny * tiny;
      if (ix > 0x3fff0000)
	return (hy > 0) ? sgn * huge * huge : sgn * tiny * tiny;
    }
```

clang编译出来的汇编是这样

```cpp
  if (iy > 0x401d654b)
   214c4:	6b0802bf 	cmp	w21, w8
   214c8:	54000fa3 	b.cc	216bc <__powl_finite@GLIBC_2.17+0x45c>  // b.lo, b.ul, b.last
   214cc:	528ca988 	mov	w8, #0x654c                	// #25932
   214d0:	72a80fa8 	movk	w8, #0x407d, lsl #16
      if (iy > 0x407d654b)
   214d4:	6b0802bf 	cmp	w21, w8
   214d8:	54000ba3 	b.cc	2164c <__powl_finite@GLIBC_2.17+0x3ec>  // b.lo, b.ul, b.last
   214dc:	d00001a8 	adrp	x8, 57000 <_fini+0x43b0>
   214e0:	3dc36500 	ldr	q0, [x8, #3472]
   214e4:	4ea01c01 	mov	v1.16b, v0.16b
   214e8:	9400bcea 	bl	50890 <__multf3>
   214ec:	d00001a8 	adrp	x8, 57000 <_fini+0x43b0>
   214f0:	3d8027e0 	str	q0, [sp, #144]
   214f4:	3dc36100 	ldr	q0, [x8, #3456]
   214f8:	4ea01c01 	mov	v1.16b, v0.16b
   214fc:	9400bce5 	bl	50890 <__multf3> // 可以看到这里连续计算了两次__multf，分别是 huge * huge （触发overflow）和  tiny * tiny（触发underflow），计算完再根据控制流选择对应的结果
   21500:	12b80028 	mov	w8, #0x3ffeffff            	// #1073676287
	  if (ix <= 0x3ffeffff)
   21504:	6b08029f 	cmp	w20, w8
   21508:	54000c48 	b.hi	21690 <__powl_finite@GLIBC_2.17+0x430>  // b.pmore
	    return (hy < 0) ? huge * huge : tiny * tiny;
   2150c:	f100027f 	cmp	x19, #0x0
   21510:	54fff38a 	b.ge	21380 <__powl_finite@GLIBC_2.17+0x120>  // b.tcont
   21514:	14000061 	b	21698 <__powl_finite@GLIBC_2.17+0x438>
```

gcc是这样

```cpp
      if (iy > 0x407d654b)
   1c968:	54005789 	b.ls	1d458 <__powl_finite@GLIBC_2.17+0xd68>  // b.plast
	  if (ix <= 0x3ffeffff)
   1c96c:	6b00029f 	cmp	w20, w0
   1c970:	54005dc8 	b.hi	1d528 <__powl_finite@GLIBC_2.17+0xe38>  // b.pmore
	    return (hy < 0) ? huge * huge : tiny * tiny;
   1c974:	37f85df3 	tbnz	w19, #31, 1d530 <__powl_finite@GLIBC_2.17+0xe40>
   1c978:	d00001a0 	adrp	x0, 52000 <_fini+0xf50>
   1c97c:	91160000 	add	x0, x0, #0x580
   1c980:	3dc00001 	ldr	q1, [x0]
   1c984:	4ea11c20 	mov	v0.16b, v1.16b
   1c988:	9400c8da 	bl	4ecf0 <__multf3> // 这里只计算一次
   1c98c:	3d801be0 	str	q0, [sp, #96]
   1c990:	17ffffb0 	b	1c850 <__powl_finite@GLIBC_2.17+0x160>
```

最终计算结果是相同的，只是针对控制流处理方式不同，clang多计算了一次tiny * tiny，导致多触发了一种浮点异常。

同时预计算控制流两侧的优化为Conditional Select

看下glibc或者c标准对于数学库浮点异常抛出的要求是什么样的，感觉clang倾向于不针对可能出现浮点异常的情况处理。

# math/test-fexcept-traps用例x86失败

打开用例发现是stmxcsr 0xc(%rsp)指令执行时产生signal SIGFPE

`stmxcsr 0xc(%rsp)` 表示将当前的 MXCSR 寄存器的值存储到栈指针 (`%rsp`) 偏移 0xC 的内存地址处。

看了下signal SIGFPE的原因有这些

> 在使用 `stmxcsr` 指令时出现 `signal SIGFPE` 异常，通常有以下几个原因：
> 
> 1. **地址未对齐**：`stmxcsr` 指令要求目标内存地址是 4 字节对齐的。如果 `0xc(%rsp)` 计算出的地址不是 4 字节对齐的（例如 `rsp + 0xc` 不是 4 的倍数），会导致未对齐的内存访问，从而触发 `SIGFPE`。
> 2. **无效内存地址或不可写**：如果 `0xc(%rsp)` 计算出的地址无效（比如指向未分配的内存区域），或不可写（例如只读的内存区域），则会在执行时发生错误，引发 `SIGFPE`。
> 3. **编译器/汇编器或系统的设置问题**：在某些情况下，浮点控制寄存器的访问可能被系统限制，尤其是在特定的受限环境（例如一些嵌入式系统或沙箱中）。如果没有权限执行 `stmxcsr`，也可能会产生异常。

gdb排除前两种原因

```cpp
(gdb) p/x $rsp+0xc
$3 = 0x7fffffffdf8c
(gdb) info proc mappings
process 1031
Mapped address spaces:

          Start Addr           End Addr       Size     Offset  Perms  objfile
      0x555555554000     0x555555559000     0x5000        0x0  r-xp   /root/rpmbuild/BUILD/glibc-2.38/build-x86_64-openEuler-linux/math/test-fexcept-traps
      0x555555559000     0x55555555a000     0x1000     0x4000  r--p   /root/rpmbuild/BUILD/glibc-2.38/build-x86_64-openEuler-linux/math/test-fexcept-traps
      0x55555555a000     0x55555555b000     0x1000     0x5000  rw-p   /root/rpmbuild/BUILD/glibc-2.38/build-x86_64-openEuler-linux/math/test-fexcept-traps
      0x7ffff7d10000     0x7ffff7d11000     0x1000        0x0  rw-s   /dev/zero (deleted)
      0x7ffff7d11000     0x7ffff7d13000     0x2000        0x0  rw-p   
      0x7ffff7d13000     0x7ffff7ed8000   0x1c5000        0x0  r-xp   /root/rpmbuild/BUILD/glibc-2.38/build-x86_64-openEuler-linux/libc.so
      0x7ffff7ed8000     0x7ffff7edc000     0x4000   0x1c4000  r--p   /root/rpmbuild/BUILD/glibc-2.38/build-x86_64-openEuler-linux/libc.so
      0x7ffff7edc000     0x7ffff7ede000     0x2000   0x1c8000  rw-p   /root/rpmbuild/BUILD/glibc-2.38/build-x86_64-openEuler-linux/libc.so
      0x7ffff7ede000     0x7ffff7eeb000     0xd000        0x0  rw-p   
      0x7ffff7eeb000     0x7ffff7fc5000    0xda000        0x0  r-xp   /root/rpmbuild/BUILD/glibc-2.38/build-x86_64-openEuler-linux/math/libm.so
      0x7ffff7fc5000     0x7ffff7fc6000     0x1000    0xd9000  r--p   /root/rpmbuild/BUILD/glibc-2.38/build-x86_64-openEuler-linux/math/libm.so
      0x7ffff7fc6000     0x7ffff7fc7000     0x1000    0xda000  rw-p   /root/rpmbuild/BUILD/glibc-2.38/build-x86_64-openEuler-linux/math/libm.so
      0x7ffff7fc7000     0x7ffff7fc9000     0x2000        0x0  rw-p   
      0x7ffff7fc9000     0x7ffff7fcd000     0x4000        0x0  r--p   [vvar]
      0x7ffff7fcd000     0x7ffff7fcf000     0x2000        0x0  r-xp   [vdso]
      0x7ffff7fcf000     0x7ffff7ffb000    0x2c000        0x0  r-xp   /usr/lib64/ld-linux-x86-64.so.2
      0x7ffff7ffb000     0x7ffff7ffd000     0x2000    0x2c000  r--p   /usr/lib64/ld-linux-x86-64.so.2
      0x7ffff7ffd000     0x7ffff7fff000     0x2000    0x2e000  rw-p   /usr/lib64/ld-linux-x86-64.so.2
      0x7ffffffde000     0x7ffffffff000    0x21000        0x0  rw-p   [stack]
(gdb) 
```

自己编写的小例子编译运行ok

```cpp
typedef struct
  {
    unsigned short int __control_word;
    unsigned short int __glibc_reserved1;
    unsigned short int __status_word;
    unsigned short int __glibc_reserved2;
    unsigned short int __tags;
    unsigned short int __glibc_reserved3;
    unsigned int __eip;
    unsigned short int __cs_selector;
    unsigned int __opcode:11;
    unsigned int __glibc_reserved4:5;
    unsigned int __data_offset;
    unsigned short int __data_selector;
    unsigned short int __glibc_reserved5;
#ifdef __x86_64__
    unsigned int __mxcsr;
#endif
  }
fenv_t;

int foo(int a) {
    fenv_t temp;
  unsigned int mxcsr;
    __asm__ ("fnstenv %0" : "=m" (*&temp));
    temp.__status_word &= ~(a & (0x20 | 0x04 | 0x10 | 0x08 | 0x01));
    temp.__status_word |= a & a & (0x20 | 0x04 | 0x10 | 0x08 | 0x01);
    __asm__ ("fldenv %0" : : "m" (*&temp));
    __asm__ ("stmxcsr %0" : "=m" (*&mxcsr));
    mxcsr &= ~(a & (0x20 | 0x04 | 0x10 | 0x08 | 0x01));
    mxcsr |= a & a & (0x20 | 0x04 | 0x10 | 0x08 | 0x01);
    __asm__ ("ldmxcsr %0" : : "m" (*&mxcsr));
    return 0;
}

int main()
{
		foo(61);
}
```

相关glibc二进制的构建指令

```cpp
clang ../sysdeps/x86_64/fpu/fsetexcptflg.c -c -std=gnu11 -fgnu89-inline  -O2 -g -DNDEBUG -fPIC -fPIE -fstack-protector-strong -Wp,-D_GLIBCXX_ASSERTIONS -m64 -mtune=generic -fasynchronous-unwind-tables -fstack-clash-protection -fgcc-compatible -Wall -Wwrite-strings -Wundef -Werror -fmerge-all-constants -frounding-math -ftrapping-math -fstack-protector-strong -fno-common -Wp,-U_FORTIFY_SOURCE -Wstrict-prototypes -Wold-style-definition -fno-math-errno    -fPIE   -fcf-protection        -I../include -I/root/rpmbuild/BUILD/glibc-2.38/build-x86_64-openEuler-linux/math  -I/root/rpmbuild/BUILD/glibc-2.38/build-x86_64-openEuler-linux  -I../sysdeps/unix/sysv/linux/x86_64/64  -I../sysdeps/unix/sysv/linux/x86_64  -I../sysdeps/unix/sysv/linux/x86/include -I../sysdeps/unix/sysv/linux/x86  -I../sysdeps/x86/nptl  -I../sysdeps/unix/sysv/linux/wordsize-64  -I../sysdeps/x86_64/nptl  -I../sysdeps/unix/sysv/linux/include -I../sysdeps/unix/sysv/linux  -I../sysdeps/nptl  -I../sysdeps/pthread  -I../sysdeps/gnu  -I../sysdeps/unix/inet  -I../sysdeps/unix/sysv  -I../sysdeps/unix/x86_64  -I../sysdeps/unix  -I../sysdeps/posix  -I../sysdeps/x86_64/64  -I../sysdeps/x86_64/fpu/multiarch  -I../sysdeps/x86_64/fpu  -I../sysdeps/x86/fpu  -I../sysdeps/x86_64/multiarch  -I../sysdeps/x86_64  -I../sysdeps/x86/include -I../sysdeps/x86  -I../sysdeps/ieee754/float128  -I../sysdeps/ieee754/ldbl-96/include -I../sysdeps/ieee754/ldbl-96  -I../sysdeps/ieee754/dbl-64  -I../sysdeps/ieee754/flt-32  -I../sysdeps/wordsize-64  -I../sysdeps/ieee754  -I../sysdeps/generic  -I.. -I../libio -I. -nostdinc -isystem /usr/bin/../lib64/clang/17/include -isystem /usr/include -D_LIBC_REENTRANT -include /root/rpmbuild/BUILD/glibc-2.38/build-x86_64-openEuler-linux/libc-modules.h -DMODULE_NAME=libm -include ../include/libc-symbols.h  -DPIC     -DTOP_NAMESPACE=glibc -I../soft-fp -o /root/rpmbuild/BUILD/glibc-2.38/build-x86_64-openEuler-linux/math/fsetexcptflg.o 
```

发现这个指令构建出来的汇编会多出来一句fwait指令

```cpp
# objdump -dS math/fsetexcptflg.o |grep "<fesetexceptflag>:" -A100
0000000000000000 <fesetexceptflag>:
#include <fenv.h>
#include <math.h>

int
fesetexceptflag (const fexcept_t *flagp, int excepts)
{
   0:   f3 0f 1e fa             endbr64
   4:   48 83 ec 38             sub    $0x38,%rsp
   8:   64 48 8b 04 25 28 00    mov    %fs:0x28,%rax
   f:   00 00 
  11:   48 89 44 24 30          mov    %rax,0x30(%rsp)
  /* XXX: Do we really need to set both the exception in both units?
     Shouldn't it be enough to set only the SSE unit?  */

  /* Get the current x87 FPU environment.  We have to do this since we
     cannot separately set the status word.  */
  __asm__ ("fnstenv %0" : "=m" (*&temp));
  16:   d9 74 24 10             fnstenv 0x10(%rsp)
  1a:   9b                      fwait

  temp.__status_word &= ~(excepts & FE_ALL_EXCEPT);
  1b:   89 f0                   mov    %esi,%eax
  1d:   83 e0 3d                and    $0x3d,%eax
  20:   89 c1                   mov    %eax,%ecx
  22:   f7 d1                   not    %ecx
  24:   0f b7 54 24 14          movzwl 0x14(%rsp),%edx
  temp.__status_word |= *flagp & excepts & FE_ALL_EXCEPT;
  29:   66 23 37                and    (%rdi),%si
  temp.__status_word &= ~(excepts & FE_ALL_EXCEPT);
  2c:   66 21 ca                and    %cx,%dx
  temp.__status_word |= *flagp & excepts & FE_ALL_EXCEPT;
  2f:   83 e6 3d                and    $0x3d,%esi
  32:   09 d6                   or     %edx,%esi
  34:   66 89 74 24 14          mov    %si,0x14(%rsp)

  /* Store the new status word (along with the rest of the environment.
     Possibly new exceptions are set but they won't get executed unless
     the next floating-point instruction.  */
  __asm__ ("fldenv %0" : : "m" (*&temp));
  39:   d9 64 24 10             fldenv 0x10(%rsp)

  /* And now the same for SSE.  */
  __asm__ ("stmxcsr %0" : "=m" (*&mxcsr));
  3d:   0f ae 5c 24 0c          stmxcsr 0xc(%rsp)
  42:   9b                      fwait

  mxcsr &= ~(excepts & FE_ALL_EXCEPT);
  43:   23 4c 24 0c             and    0xc(%rsp),%ecx
  mxcsr |= *flagp & excepts & FE_ALL_EXCEPT;
  47:   0f b7 17                movzwl (%rdi),%edx
  4a:   21 c2                   and    %eax,%edx
  4c:   09 ca                   or     %ecx,%edx
  4e:   89 54 24 0c             mov    %edx,0xc(%rsp)

  __asm__ ("ldmxcsr %0" : : "m" (*&mxcsr));
  52:   0f ae 54 24 0c          ldmxcsr 0xc(%rsp)
  57:   9b                      fwait
```

怀疑是fwait导致异常触发，因为查看gcc和clang构建的这个函数执行到stmxcsr指令时mxcsr的值都是0x100，fwait执行之后就报了异常

验证了下是-ftrapping-math导致clang插入了这条指令，但是clang20之后似乎不插入这条指令了，修复了？

[https://godbolt.org/z/7qGf86Wex](https://godbolt.org/z/7qGf86Wex)

找下相关提交，应该是这个

[https://github.com/llvm/llvm-project/pull/101686](https://github.com/llvm/llvm-project/pull/101686)

期望这个能解决这个问题