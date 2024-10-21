# “expected parameter declarator”问题定位及修复指导
## 1、问题现象
24.03 LTS分支上的`crash`、`dhcp`、`cyrus-sasl`软件包切换为Clang构建后出现类似编译错误，如：
```abap
[ 4289s] dhclient.c:52:12: error: expected parameter declarator
[ 4289s] extern int asprintf(char **strp, const char *fmt, ...);
[ 4289s]            ^
[ 4289s] /usr/include/bits/stdio2.h:207:24: note: expanded from macro 'asprintf'
[ 4289s]   __asprintf_chk (ptr, __USE_FORTIFY_LEVEL - 1, __VA_ARGS__)
[ 4289s]                        ^
[ 4289s] /usr/include/features.h:424:31: note: expanded from macro '__USE_FORTIFY_LEVEL'
[ 4289s] #  define __USE_FORTIFY_LEVEL 2
```
## 2、问题根因分析
### 2.1、问题复现
selfdef.h
```abap
extern int dprintf(int lvl, const char *fmt, ...);
```
selfdef.c
```abap
#include <stdio.h>
#include "selfdef.h"

int dprintf(int lvl, const char *fmt, ...) {
    printf("call dprintf from selfdef\n");
    return 10;
}
```
main.c
```abap
#include <stdio.h>
#include "selfdef.h"

int main() {
    dprintf(2, "call dprintf from stdio\n");
    return 0;
}
```
执行命令
```abap
$ gcc main.c selfdef.c -O2
$ ./a.out
call dprintf from selfdef

$ gcc main.c selfdef.c -O2 -Wp,-D_FORTIFY_SOURCE=2
$ ./a.out
call dprintf from stdio

$ clang main.c selfdef.c -O2
$ ./a.out
call dprintf from selfdef

clang main.c selfdef.c -O2 -Wp,-D_FORTIFY_SOURCE=2
$ ./a.out
In file included from main.c:2:
./selfdef.h:1:12: error: expected parameter declarator
extern int dprintf(int lvl, const char *fmt, ...);
           ^
/usr/include/bits/stdio2.h:149:22: note: expanded from macro 'dprintf'
  __dprintf_chk (fd, __USE_FORTIFY_LEVEL - 1, __VA_ARGS__)
                     ^
/usr/include/features.h:385:31: note: expanded from macro '__USE_FORTIFY_LEVEL'
#  define __USE_FORTIFY_LEVEL 2
                              ^
```
### 2.2、dprintf的定义
通过在系统中搜索dprintf的定义，发现有两处定义，分别是：
（1）/usr/include/stdio.h
```abap
extern int dprintf (int __fd, const char *__restrict __fmt, ...)
     __attribute__ ((__format__ (__printf__, 2, 3)));
```
(2) /usr/include/bits/stdio2.h
```abap
#  ifdef __va_arg_pack
__fortify_function int
dprintf (int __fd, const char *__restrict __fmt, ...)
{
  return __dprintf_chk (__fd, __USE_FORTIFY_LEVEL - 1, __fmt,
                        __va_arg_pack ());
}
#  elif !defined __cplusplus
#   define dprintf(fd, ...) \
  __dprintf_chk (fd, __USE_FORTIFY_LEVEL - 1, __VA_ARGS__)
#  endif

```
* 为什么存在两个原型定义呢？
  * 这个和`_FORTIFY_SOURCE`有关
* 为什么第（2）个定义中有两种情况呢？
  * 这个和`__va_arg_pack` 的定义有关

### 2.2.1、-D_FORTIFY_SOURCE=2
[FORTIFY_SOURCE](https://www.redhat.com/en/blog/security-technologies-fortifysource)为一些内存和字符串函数提供了轻量级编译和运行时保护。在GCC中，`FORTIFY_SOURCE`通常通过将一些字符串和内存函数替换为它们的`*_chk`对应函数（内置函数）来工作。这些函数进行必要的计算以确定溢出。如果发现溢出，程序将中止；否则控制被传递到相应的字符串或存储器操作函数。
在编译过程中，需要设置`D_FORTIFY_SOURCE`宏才能启用`FORTIFY_SOURCE`。实现了两个级别的检查。将宏设置为1将启用一些检查（手册页上说“执行不应改变符合程序行为的检查”），而将其设置为2将增加一些检查。

### 2.2.2、__va_arg_pack
这个是GCC处理变长参数函数时[引入的内建函数](https://gcc.gnu.org/onlinedocs/gcc-4.7.2/gcc/Constructing-Calls.html)，LLVM社区没有接纳对[这个GNU扩展的支持]（https://reviews.llvm.org/D57635）。

### 2.2.3、问题总结及根因确认
（1）系统对`dprintf`的定义有两种。
（2）根据`/usr/include/bits/stdio2.h`中的定义，如果一个编译器已定义`__va_arg_pack`，则`dprintf`被定义为一个函数，而如果编译器未定义`__va_arg_pack`，则`dprintf`被定义为一个宏。

根因：
当用如下构建命令时：
```abap
clang main.c selfdef.c -O2 -Wp,-D_FORTIFY_SOURCE=2
```
系统中已经有一个`dprintf`宏定义，和`./selfdef.h`文件中的声明冲突，导致编译错误。

## 3、问题修改建议
由以上分析可以想到，需要根据应用中实际情况来修改，确保修前后应用仍然能链接正确的定义。

有两种修改方法：
* 重命名自定义的dprintf
详见：https://github.com/cyrusimap/cyrus-sasl/commit/b0a587383e0d4e9852ec973accc0854a10b0f128?diff=unified&w=0

* undef系统定义宏。
详见：https://gitee.com/src-openeuler/dhcp/pulls/110
