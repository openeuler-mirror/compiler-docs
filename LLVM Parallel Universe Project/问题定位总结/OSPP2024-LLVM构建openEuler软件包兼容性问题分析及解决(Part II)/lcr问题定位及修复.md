来源：https://gitee.com/src-openeuler/lcr (openEuler-24.03-LTS)

# 相关PR

openEuler：[backport(rm attribute &quot;visibility&quot; before struct),add -Wno-error,fix changelog ,support clang build · Pull Request !344 · src-openEuler/lcr - Gitee.com](https://gitee.com/src-openeuler/lcr/pulls/344)

上游社区：https://gitee.com/openeuler/lcr/pulls/332

# 问题一：attribute 'visibility' is ignored, place it after "struct"问题

## 1、问题现象

`[   98s] /home/abuild/rpmbuild/BUILD/lcr-v2.1.4/src/runtime/lcrcontainer.h:43:1: error: attribute 'visibility' is ignored, place it after "struct" to apply attribute to type declaration [-Werror,-Wignored-attributes]`

## 2、问题定位

### 2.1、问题复现

小例子复现：

#### 第一种：__attribute__((visibility("default"))) struct  A{...};

lcrcontainer_1.h

```c
__attribute__((visibility("default"))) struct A {
  int n;
};
```

main.c

```c
#include <stdio.h>
#include "lcrcontainer_1.h"

int main() {
    struct A a={1};
    printf("%d\n",a.n);
    return 0;
}
```

执行编译与运行命令，gcc正常，clang报错：

```c
[root@localhost lcr]# gcc main.c -fvisibility=hidden -Werror -o gcc1.out
[root@localhost lcr]# ./gcc1.out
1
[root@localhost lcr]# clang main.c -fvisibility=hidden -Werror -o clang1.out
In file included from main.c:2:
./lcrcontainer_1.h:1:16: error: attribute 'visibility' is ignored, place it after "struct" to apply attribute to type declaration [-Werror,-Wignored-attributes]
    1 | __attribute__((visibility("default"))) struct A {
      |                ^
1 error generated.
```

#### 第二种：struct __attribute__((visibility("default"))) A{...};

lcrcontainer_2.h

```c
struct __attribute__((visibility("default"))) A {
  int n;
};
```

执行编译与运行命令，gcc报错，clang正常：

```c
[root@localhost lcr]# gcc main.c -fvisibility=hidden -Werror -o gcc2.out
In file included from main.c:2:
lcrcontainer_2.h:3:1: 错误：‘visibility’属性在类型上被忽略 [-Werror=attributes]
    3 | };
      | ^
cc1：所有的警告都被当作是错误
[root@localhost lcr]# clang main.c -fvisibility=hidden -Werror -o clang2.out
[root@localhost lcr]# ./clang2.out
1
```

#### 第三种：删去__attribute__((visibility("default")))

lcrcontainer_3.h

```c
#include <stdio.h>
#include "lcrcontainer_3.h"

int main() {
    struct A a={1};
    printf("%d\n",a.n);
    return 0;
}
```

执行编译与运行命令，gcc与clang都正常：

```c
[root@localhost lcr]# gcc main.c -fvisibility=hidden -Werror -o gcc3.out
[root@localhost lcr]# ./gcc3.out
1
[root@localhost lcr]# clang main.c -fvisibility=hidden -Werror -o clang3.out
[root@localhost lcr]# ./clang3.out
1
```

### 2.2、关于attribute 'visibility'

可以通过查阅GNU（[Visibility - GCC Wiki (gnu.org)](https://gcc.gnu.org/wiki/Visibility)）以及LLVM官方文档（[LTO Visibility — Clang 20.0.0git documentation (llvm.org)](https://clang.llvm.org/docs/LTOVisibility.html)）去了解。

#### 2.2.1、visibility的作用

简单来说，Visibility在编译和链接过程中控制着符号（如函数、变量和类型信息）的可见性，即它们是否可以被其他编译单元或动态共享对象（DSO）引用。以下是Visibility的几个主要作用：

1. **优化加载时间**：通过减少不必要的符号导出，可以减少动态共享对象（DSO）的大小，从而加快加载时间。
2. **提高代码优化**：Visibility允许编译器生成更优化的代码。例如，通过避免全局偏移表（GOT）的间接寻址，可以减少处理器流水线停顿，提高代码执行速度。
3. **减少DSO大小**：通过减少导出的符号数量，可以显著减少DSO的大小，因为ELF的导出符号表格式占用较多空间。
4. **降低符号冲突**：减少不必要的符号导出可以降低不同库之间符号名称冲突的可能性。
5. **支持链接时优化（LTO）** ：Visibility对于使用链接时优化特性（如全程序虚拟表优化和控制流完整性检查）至关重要，因为这些特性需要在整个程序级别上可见的类层次结构。
6. **避免ODR违规**：Visibility属性需要在不同翻译单元之间保持一致，以避免违反C++的One Definition Rule（ODR），确保每个非内联函数或变量在整个程序中只有一个定义。
7. **跨平台兼容性**：Visibility提供了一种机制，使得在不同平台（如Windows和POSIX系统）上进行编译的代码可以有一致的行为，这对于开发可移植的应用程序非常重要。
8. **细粒度控制**：开发者可以细粒度地控制哪些符号应该公开，哪些应该隐藏，从而优化程序的性能和安全性。
9. **简化构建系统**：通过使用Visibility属性，可以简化构建系统，因为可以减少对链接器脚本的依赖，从而减少维护成本。

#### 2.2.2、关于attribute 'visibility'在类型(type)上的应用

参考[Type Attributes - Using the GNU Compiler Collection (GCC)](https://gcc.gnu.org/onlinedocs/gcc-4.7.2/gcc/Type-Attributes.html#Type-Attributes)可知，

`In C++, attribute visibility (see Function Attributes) can also be applied to class, struct, union and enum types. Unlike other type attributes, the attribute must appear between the initial keyword and the name of the type; it cannot appear after the body of the type.`

即，在C++中attribute 'visibility'可以应用于class，struct，union以及enum类型，并且用法是将其置于关键字和类型名称之间，例如`struct __attribute__((visibility("default"))) A{...};`

然而，经过上面的小例子测试可知，在C中，gcc并不支持将attribute 'visibility'应用于type，然而clang是支持的，这就是问题的根源

### 2.3、问题总结及根因确认

在C++中attribute 'visibility'可以应用于class，struct，union以及enum类型，并且用法是将其置于关键字和类型名称之间，在C中，gcc并不支持将attribute 'visibility'应用于type，然而clang是支持的。所以对于struct及attribute 'visibility'的顺序，无论是谁前谁后，必定有一个编译器报错，唯一的二者兼顾方法是删去__attribute__ ((visibility("default")))。

## 3、问题修改建议

想要兼顾gcc和clang，目前能想到的方法只能是C语言中不将attribute 'visibility'应用于type。参考：[https://gitee.com/openeuler/lcr/pulls/332](https://gitee.com/openeuler/lcr/pulls/332)

# 问题二：bad date in %changelog问题

## 1、问题现象

出自：https://gitee.com/src-openeuler/lcr (openEuler-24.03-LTS)


`bad date in %changelog: Tue June 11 2024 jikai<jikai11@huawei.com> - 2.1.4-8`

## 2、changelog日期写法规范

规范写法是：

星期(简写) 月(简写) 日 年 名称 邮箱 - 版本

例如：`Tue Jun 11 2024 jikai<jikai11@huawei.com> - 2.1.4-8`

以下是星期与月份的规范简写：

一月：January 简写:Jan
二月：February 简写:Feb
三月：March 简写:Mar
四月：April 简写:Apr
五月：May 简写:May
六月：June 简写:Jun
七月：July 简写:Jul
八月：August 简写:Aug
九月：September 简写:Sep
十月：October 简写:Oct
十一月：November 简写:Nov
十二月：December 简写:Dec

星期一：Monday 简写:Mon
星期二：Tuesday 简写:Tue
星期三：Wednesday 简写:Wed
星期四：Thursday 简写:Thu
星期五：Friday 简写:Fri
星期六：Saturday 简写:Sat
星期日：Sunday 简写:Sun

## 3、修改建议

June改为Jun

# LLVM兼容性问题总结

lcr包中的LLVM兼容性问题主要体现在问题二。

问题二是由于clang与gcc对于C++中的'visibility'特性用于TYPE（类型）时的支持标准不统一。解决方法为，在不影响功能的情况下，暂时对类型不使用该特性。