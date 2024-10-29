来源：https://gitee.com/src-openeuler/libreswan (openEuler-24.03-LTS)

# 相关PR

openEuler：[Fix prototype deprecated and expression result unused,as well rewrite spec to support clang build · Pull Request !99 · src-openEuler/libreswan - Gitee.com](https://gitee.com/src-openeuler/libreswan/pulls/99)

上游社区：https://github.com/libreswan/libreswan/pull/1791

# 问题一：function definition without a prototype is deprecated问题

## 1、问题现象

`a function definition without a prototype is deprecated in all versions of C and is not supported in C2x [-Werror,-Wdeprecated-non-prototype]`

## 2、问题定位

有两处使用的函数原型定义方式（形参的类型单独在括号外说明）已经被弃用，一处在上游有commit

[building: prototype maskcount() definition; clang-16 · libreswan/libreswan@e52a0af (github.com)](https://github.com/libreswan/libreswan/commit/e52a0af61990754b26a2825fb775fd5d1680112b)

另一处找不到上游对应的文件，但也可以用同样的改法

```c
size_t                          /* length required for full conversion */
ultot(n, base, dst, dstlen)
unsigned long n;
int base;
char *dst;                      /* need not be valid if dstlen is 0 */
size_t dstlen;
{
	...
}

```

改为

```c
size_t                          /* length required for full conversion */
ultot(unsigned long n, int base, char *dst, size_t dstlen)
                     /* char *dst need not be valid if dstlen is 0 */
{
	...
}
```

## 3、修改建议

将函数原型定义方式改为现在支持的形式。

# 问题二：语句误用","而非";"结尾而引起的expression result unused问题

## 1、问题现象


```c
[   97s] /home/abuild/rpmbuild/BUILD/libreswan-4.15/lib/libswan/ckaid.c:85:2: error: expression result unused [-Werror,-Wunused-value]
[   97s]    85 |         pexpect(ckaid.len == nss_ckaid_len);
[   97s]       |         ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[   97s] ../../include/lswlog.h:489:29: note: expanded from macro 'pexpect'
[   97s]   489 | #define pexpect(ASSERTION)  PEXPECT(&global_logger, ASSERTION)
[   97s]       |                             ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[   97s] ../../include/lswlog.h:486:3: note: expanded from macro 'PEXPECT'
[   97s]   486 |                 assertion__; /* result */                               \
[   97s]       |                 ^~~~~~~~~~~
[   97s] 1 error generated.
```

## 2、问题定位

### 2.1、问题定位分析

* 问题定位：

```c
ckaid_t ckaid_from_secitem(const SECItem *const nss_ckaid)
{
	size_t nss_ckaid_len = nss_ckaid->len;
	/* ckaid = { .len = min(...), } barfs with gcc 11.2.1 */
	ckaid_t ckaid = {0};
	/* should not be truncated but can be */
	ckaid.len = min(nss_ckaid_len, sizeof(ckaid.ptr/*array*/)),
	pexpect(ckaid.len == nss_ckaid_len);
	memmove(ckaid.ptr, nss_ckaid->data, ckaid.len);
	return ckaid;
}
```

发现其实是因为ckaid.len = min(nss_ckaid_len, sizeof(ckaid.ptr/\*array *** /)),结尾是逗号，而不是分号

在上游提交了pr：https://github.com/libreswan/libreswan/pull/1791修正了问题

### 2.2、问题总结及根因确认

语句误用","而非";"结尾，编译器没有识别出这个错误，而是错误地将其识别成了expression result unused错误，gcc对于这种类型的错误可能较为宽松，导致gcc编译时没有发现这个错误。

## 3、修改建议

ckaid.len = min(nss_ckaid_len, sizeof(ckaid.ptr/\*array *** /)),结尾的","改为";"

# 问题三：unknown warning option问题

## 1、问题现象


```c
[  537s] error: unknown warning option '-Wno-maybe-uninitialized'; did you mean '-Wno-uninitialized'? [-Werror,-Wunknown-warning-option]
[  537s] error: unknown warning option '-Wno-lto-type-mismatch'; did you mean '-Wno-selector-type-mismatch'? [-Werror,-Wunknown-warning-option]
```

## 2、问题分析

`-Wno-maybe-uninitialized`和`-Wno-lto-type-mismatch`是gcc的编译器选项，而查阅资料得知，clang没有这两个选项

## 3、修改建议

clang构建时，不设置这两个选项