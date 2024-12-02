# fortify相关问题分析

里面提到了一些c库函数clang不支持的原因[clang-style FORTIFY (google.com)](https://docs.google.com/document/u/0/d/1DFfZDICTbL7RqS74wJVIJ-YnjQOj1SaoqfhbgddFYSM/mobilebasic?_immersive_translate_auto_translate=1#heading=h.s80bbcz9m5p8)

第一个原因是使用了gcc的inline策略

> While this is an entirely viable approach on GCC, it’s much less so on clang. This is because by the time clang (LLVM, rather) gets to inlining and optimizing, there’s minimal reliable source and type-level information available. So, **much of the functionality that GCC provides simply can’t be reasonably emulated in clang**.
> 
> 
> 虽然这在 GCC 上是完全可行的方法，但在 clang 上却不太可行。这是因为当 clang（更确切地说是 LLVM）开始内联和优化时，可用的可靠源和类型级信息很少。因此， **GCC 提供的许多功能根本无法在 clang 中合理地模拟**。
> 

使用如下源码编译

```c
#include <unistd.h>

extern __inline __attribute__ ((__always_inline__)) __attribute__ ((__gnu_inline__)) __attribute__ ((__artificial__)) __attribute__ ((__warn_unused_result__)) ssize_t
iread (int __fd, void *__buf, size_t __nbytes)
{
  if (__builtin_object_size (__buf, 0) != (size_t) -1)
    {
      if (!__builtin_constant_p (__nbytes))
 return __read_chk (__fd, __buf, __nbytes, __builtin_object_size (__buf, 0));

      if (__nbytes > __builtin_object_size (__buf, 0))
 return __read_chk_warn (__fd, __buf, __nbytes,
    __builtin_object_size (__buf, 0));
    }
  return __read_alias (__fd, __buf, __nbytes);
}

int main(){
        int someFD;

        char buf[2];

        ssize_t result = iread(someFD, buf, 10); //
}
```

```cpp
# gcc gcc_read.c -D_FORTIFY_SOURCE=2 -O2                
In function 'iread',
    inlined from 'main' at gcc_read.c:45:27:
gcc_read.c:33:14: warning: call to '__read_chk_warn' declared with attribute warning: read called with bigger length than size of the destination buffer [-Wattribute-warning]
   33 |       return __read_chk_warn (__fd, __buf, __nbytes, __builtin_object_size (__buf, 0));
      |              ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# gcc gcc_read.c -D_FORTIFY_SOURCE=2 -O2 -fdump-tree-all
```

发现在einline这个pass后，__builtin_object_size/__builtin_constant_p已经inline成常量，源码变成了这样

```cpp
main ()
{
  ssize_t D.4471;
  ssize_t result;
  char buf[255];
  int someFD;
  int _6;
  long unsigned int _7;
  int _8;
  long int _9;
  long int _10;
  long int _11;
  long int _12;

  <bb 2> :
  _7 = 255;
  if (_7 != 18446744073709551615)
    goto <bb 3>; [79.76%]
  else
    goto <bb 7>; [20.24%]

  <bb 3> :
  _8 = 1;
  if (_8 == 0)
    goto <bb 4>; [20.24%]
  else
    goto <bb 5>; [79.76%]

  <bb 4> :
  _9 = __read_chk (someFD_2(D), &buf, 255, _7);
  goto <bb 8>; [100.00%]

  <bb 5> :
  if (_7 <= 254)
    goto <bb 6>; [20.24%]
  else
    goto <bb 7>; [79.76%]

  <bb 6> :
  _10 = __read_chk_warn (someFD_2(D), &buf, 255, _7);
  goto <bb 8>; [100.00%]

  <bb 7> :
  _11 = __read_alias (someFD_2(D), &buf, 255);

  <bb 8> :
  # _12 = PHI <_9(4), _10(6), _11(7)>
  _17 = _12;
  result_4 = _17;
  buf ={v} {CLOBBER};
  _6 = 0;
  return _6;

}
```

```cpp
;; Function main (main, funcdef_no=13, decl_uid=4504, cgraph_uid=14, symbol_order=13) (executed once)

int main ()
{
  char buf[2];
  int someFD;

  <bb 2> [local count: 1073741824]:
  __read_chk_warn (someFD_2(D), &buf, 10, 2);
  buf ={v} {CLOBBER(eol)};
  return 0;

}
```

而clang编译的情况下，__builtin_object_size (__buf, 0)被inline处理的时候已经优化成了这样

```cpp
*** IR Dump After SimplifyCFGPass on main ***
; Function Attrs: nounwind uwtable
define dso_local i32 @main() local_unnamed_addr #0 {
  %1 = alloca [2 x i8], align 1
  call void @llvm.lifetime.start.p0(i64 2, ptr nonnull %1) #6
  %2 = call i64 @llvm.objectsize.i64.p0(ptr nonnull %1, i1 false, i1 true, i1 false)
  %3 = icmp ne i64 %2, -1
  %4 = icmp ult i64 %2, 10
  %5 = and i1 %3, %4
  br i1 %5, label %6, label %8

6:                                                ; preds = %0
  %7 = call i64 @__read_chk(i32 noundef undef, ptr noundef nonnull %1, i64 noundef 10, i64 noundef %2) #6
  br label %10

8:                                                ; preds = %0
  %9 = call i64 @read(i32 noundef undef, ptr noundef nonnull %1, i64 noundef 10) #6
  br label %10

10:                                               ; preds = %6, %8
  call void @llvm.lifetime.end.p0(i64 2, ptr nonnull %1) #6
  ret i32 0
}
*** IR Dump After InstCombinePass on main ***
; Function Attrs: nounwind uwtable
define dso_local i32 @main() local_unnamed_addr #0 {
  %1 = alloca [2 x i8], align 1
  call void @llvm.lifetime.start.p0(i64 2, ptr nonnull %1) #6
  br i1 true, label %2, label %4

2:                                                ; preds = %0
  %3 = call i64 @__read_chk(i32 noundef undef, ptr noundef nonnull %1, i64 noundef 10, i64 noundef 2) #6
  br label %5

4:                                                ; preds = %0
  br label %5

5:                                                ; preds = %2, %4
  call void @llvm.lifetime.end.p0(i64 2, ptr nonnull %1) #6
  ret i32 0
}
```

差异主要是clang的inline策略为先处理普通函数或者符号重定向，后处理builtin函数；而gcc正相反

这个其实不影响最终调用到read或者read_chk，而是glibc中的__read_chk_warn的实现clang没法支持

```c
extern ssize_t __REDIRECT (__read_chk_warn,
                            (int __fd, void *__buf, size_t __nbytes,
                             size_t __buflen), __read_chk)
      __wur __warnattr ("read called with bigger length than size of "
                        "the destination buffer");
// 宏展开后是这样的
extern ssize_t __read_chk_warn (int __fd, void *__buf, size_t __nbytes, size_t __buflen)
__asm__ ("" "__read_chk")
__attribute__((__warning__ (
		"read called with bigger length than size of " "the destination buffer"
		)));
```

实际上__read_chk_warn在重定向到__read_chk的时候就应该报这个warning，而clang并没有报warning，用下面这个例子试下

```c
int main(){
        int someFD;

        char buf[2];
        ssize_t result1 = __read_chk_warn (someFD, buf, 10, __builtin_object_size (buf, 0));
        // ssize_t result2 = iread(someFD, buf, 10); //
}

$ clang gcc_read.c -D_FORTIFY_SOURCE=2 -O2
$ gcc gcc_read.c -D_FORTIFY_SOURCE=2 -O2
gcc_read.c: In function 'main':
gcc_read.c:21:27: warning: call to '__read_chk_warn' declared with attribute warning: read called with bigger length than size of the destination buffer [-Wattribute-warning]
   21 |         ssize_t result1 = __read_chk_warn (someFD, buf, 10, __builtin_object_size (buf, 0));
      |                           ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
```

但是如果使用兼容clang的写法，会由于clang过早的__REDIRECT到__read_chk导致warning误报，以下面这个为例

```c
extern ssize_t __read_chk_warn1 (int __fd, void *__buf, size_t __nbytes, size_t __buflen)
__asm__ ("" "__read_chk") // __REDIRECT的宏展开是这样的
__attribute__((diagnose_if(1 ,"The given object can’t handle n characters!", "warning"))) // clang支持diagnose_if写法
__attribute__((__warning__ (
		"read called with bigger length than size of " "the destination buffer"
		))); // gcc的写法

extern __inline __attribute__ ((__always_inline__)) __attribute__ ((__gnu_inline__)) __attribute__ ((__artificial__)) __attribute__ ((__warn_unused_result__)) ssize_t
iread (int __fd, void *__buf, size_t __nbytes)
{
  if (__builtin_object_size (__buf, 0) != (size_t) -1)
  {
    if (!__builtin_constant_p (__nbytes))
      return __read_chk (__fd, __buf, __nbytes, __builtin_object_size (__buf, 0));

    if (__nbytes > __builtin_object_size (__buf, 0)) 
      return __read_chk_warn1 (__fd, __buf, __nbytes, __builtin_object_size (__buf, 0));
  }
  return __read_alias (__fd, __buf, __nbytes);
}

int main(){
  int someFD;

  char buf[2];
  ssize_t result2 = iread(someFD, buf, 2); // __nbytes没有超过__buflen
}

# clang gcc_read.c -D_FORTIFY_SOURCE=2 -O2
gcc_read.c:30:14: warning: The given object can’t handle n characters! [-Wuser-defined-warnings]
   30 |       return __read_chk_warn1 (__fd, __buf, __nbytes, __builtin_object_size (__buf, 0));
      |              ^
gcc_read.c:19:16: note: from 'diagnose_if' attribute on '__read_chk_warn1':
   19 | __attribute__((diagnose_if(1 ,"The given object can’t handle n characters!", "warning")));
      |                ^           ~
1 warning generated.
# gcc gcc_read.c -D_FORTIFY_SOURCE=2 -O2 -Wno-attributes
// 修改为iread(someFD, buf, 10)
# gcc gcc_read.c -D_FORTIFY_SOURCE=2 -O2 -Wno-attributes
In function 'iread',
    inlined from 'main' at gcc_read.c:45:27:
gcc_read.c:33:14: warning: call to '__read_chk_warn1' declared with attribute warning: read called with bigger length than size of the destination buffer [-Wattribute-warning]
   33 |       return __read_chk_warn1 (__fd, __buf, __nbytes, __builtin_object_size (__buf, 0));
      |              ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
```

最终linaro的解决方案是为gcc和clang提供了两套不同的实现，具体实现中的问题可以参考开头链接中的文章，这里只介绍glibc原先写法中不兼容llvm的原因。