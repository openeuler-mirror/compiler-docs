来源：https://gitee.com/src-openeuler/lxc (openEuler-24.03-LTS)

# 相关PR

openEuler：[backport:fix clang build error on AARCH64 · Pull Request !531 · src-openEuler/lxc - Gitee.com](https://gitee.com/src-openeuler/lxc/pulls/531)

上游社区：https://github.com/lxc/lxc/pull/4481

# 问题一：meson.build中cc.compiles returns unexpected false问题

## 1、问题现象

```c
[  112s] clang: warning: argument unused during compilation: '-fstack-clash-protection' [-Wunused-command-line-argument]
[  112s] In file included from ../src/lxc/error.c:10:
[  112s] ../src/lxc/log.h:310:6: error: conflicting types for 'strerror_r'
[  112s]   310 |         int strerror_r(int errnum, char *buf, size_t buflen);
[  112s]       |             ^
[  112s] /usr/include/string.h:444:14: note: previous declaration is here
[  112s]   444 | extern char *strerror_r (int __errnum, char *__buf, size_t __buflen)
[  112s]       |              ^
[  112s] 1 error generated.
```

## 2、问题定位

### 2.1、问题定位分析

已同步提交上游PR：[fix possible clang compile error on AARCH by yuncang123 · Pull Request #4479 · lxc/lxc (github.com)](https://github.com/lxc/lxc/pull/4479)

* 报错的地方在log.h

```c
#if HAVE_STRERROR_R
	#if STRERROR_R_CHAR_P
	char *strerror_r(int errnum, char *buf, size_t buflen);
	#else
	int strerror_r(int errnum, char *buf, size_t buflen);
	#endif
```

STRERROR_R_CHAR_P的取值定义在下段代码中：

```c
have_func_strerror_r = cc.has_function('strerror_r', prefix: '#include <string.h>', args: '-D_GNU_SOURCE')
srcconf.set10('HAVE_STRERROR_R', have_func_strerror_r)

have_func_strerror_r_char_p = false

if have_func_strerror_r
    code = '''
#define _GNU_SOURCE
#include <string.h>
int func (void) {
    char error_string[256];
    char *ptr = strerror_r (-2, error_string, 256);
    char c = *strerror_r (-2, error_string, 256);
    return c != 0 && ptr != (void*) 0L;
}
'''

have_func_strerror_r_char_p = cc.compiles(code, name : 'strerror_r() returns char *')
endif

srcconf.set10('STRERROR_R_CHAR_P', have_func_strerror_r_char_p)
```

问题关键：

```c
[   94s] Checking if "strerror_r() returns char *" compiles: NO 
```


而这个结果本该是YES，因为openEuler-24.03-LTS中strerror_r的返回值类型就是char *没错。

问题出在cc.compiles结果出错：

```c
cc.compiles(code, name : 'strerror_r() returns char *')
```

最后发现是由于`'-fstack-clash-protection'`在aarch中暂时并未支持，再加上默认开启了‘-Werror’，因此导致-Wunused-command-line-argument升级为error，导致编译结果为false

### 2.2、关于meson中的Compiler properties

详见：https://mesonbuild.com/Compiler-properties.html

这里只简单介绍一下`compiler.compiles`

```c
compiler = meson.get_compiler('c')
code = '''#include<stdio.h>
void func() { printf("Compile me.\n"); }
'''
result = compiler.compiles(code, name : 'basic check')
```

`compiler = meson.get_compiler('c')`是获取c语言的编译器

`result = compiler.compiles(code, name : 'basic check')`如果编译器编译code通过，则返回true，反之返回false

`name : 'basic check'`是可选参数选项

此外，还可以加入`args:`参数，这个参数的作用是给这一次`compiler.compiles`添加编译器选项

例如本修改法中，就是当编译器为clang时，将`cc.compiles(code, name : 'strerror_r() returns char *')`改为`cc.compiles(code, args : '-Wno-error=unused-command-line-argument', name : 'strerror_r() returns char *')`从而避免`'-fstack-clash-protection'`导致的error

### 2.3、问题总结及根因确认

meson中的`compiler.compiles(code, name : 'basic check')`函数是用来测试code是否能够编译成功，本例使用该函数来测试strerror_r函数在当前系统中的返回值类型，然而由于aarch中没有实现`'-fstack-clash-protection'`，该编译器选项在Werror的影响下由warning升级成为了error，从而使`compiler.compiles`返回结果为false（这与测试strerror_r函数在当前系统中的返回值类型的本意是违背的）。

## 3、修改建议

当编译器为clang时，将`cc.compiles(code, name : 'strerror_r() returns char *')`改为`cc.compiles(code, args : '-Wno-error=unused-command-line-argument', name : 'strerror_r() returns char *')`从而避免`'-fstack-clash-protection'`导致的error