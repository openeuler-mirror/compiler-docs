# pkcs11-helper #

## 1、问题现象 ##

出自：https://gitee.com/src-openeuler/pkcs11-helper(openEuler-24.03-LTS)
```
[  101s] pkcs11h-openssl.c:1108:3: error: incompatible function pointer types passing 'int (CRYPTO_EX_DATA *, const CRYPTO_EX_DATA *, void *, int, long, void *)' (aka 'int (struct crypto_ex_data_st *, const struct crypto_ex_data_st *, void *, int, long, void *)') to parameter of type 'CRYPTO_EX_dup *' (aka 'int (*)(struct crypto_ex_data_st *, const struct crypto_ex_data_st *, void **, int, long, void *)') [-Wincompatible-function-pointer-types]
[  101s]  1108 |                 __pkcs11h_openssl_ex_data_dup,
[  101s]       |                 ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[  101s] /usr/include/openssl/rsa.h:444:62: note: expanded from macro 'RSA_get_ex_new_index'
[  101s]   444 |     CRYPTO_get_ex_new_index(CRYPTO_EX_INDEX_RSA, l, p, newf, dupf, freef)
[  101s]       |                                                              ^~~~
```
## 2、问题定位及根因分析 ##

这个错误表明在 pkcs11h-openssl.c 文件的第 1108 行，传递的函数指针类型与期望的类型不兼容。
```
__pkcs11h_openssl_ex_data_dup,
```
具体来说，正在尝试将函数 __pkcs11h_openssl_ex_data_dup 的指针传递给一个期望类型为` CRYPTO_EX_dup *（即 int (*)(struct crypto_ex_data_st *, const struct crypto_ex_data_st *, void **, int, long, void *)）`的参数，但 __pkcs11h_openssl_ex_data_dup 的类型是` int (CRYPTO_EX_DATA *, const CRYPTO_EX_DATA *, void *, int, long, void *)（等同于 int (struct crypto_ex_data_st *, const struct crypto_ex_data_st *, void *, int, long, void *)）`。
```
__pkcs11h_openssl_ex_data_dup (
	CRYPTO_EX_DATA *to,
	CRYPTO_EX_DATA *from,
	void *from_d,
	int idx,
	long argl,
	void *argp
)
```
即__pkcs11h_openssl_ex_data_dup函数定义的第三个参数有问题：
需要的是void **
结果传递的是void *

小例子复现：

main.c

```
 #include <stdio.h>

typedef struct {
    int data;
} CRYPTO_EX_DATA;

typedef int (*CRYPTO_EX_dup)(CRYPTO_EX_DATA *to, const CRYPTO_EX_DATA *from, void **from_d, int idx, long argl, void *argp);

int my_dup_function(CRYPTO_EX_DATA *to, const CRYPTO_EX_DATA *from, void *from_d, int idx, long argl, void *argp) {
    // 这里是函数的实现
    return 1;
}

void set_dup_function(CRYPTO_EX_dup dup_func) {
    // 这里是设置函数指针的实现
}

int main() {
    set_dup_function(my_dup_function); // 这里会产生编译错误
    return 0;
}
```

分析原因
在C语言中，void *和void **是不同的类型。void *是一个指向任意类型的指针，而void **是一个指向void *类型的指针。编译器在处理这些类型时可能会有不同的行为。

在你的例子中，GCC可能对类型转换更宽松，而Clang则更严格。因此，当你将void *from_d改为void **from_d时，Clang能够正确识别并处理类型，而GCC则可能忽略了这个问题。

## 3、修改建议 ##

在lib/pkcs11h-openssl.c文件中将__pkcs11h_openssl_ex_data_dup函数定义中的void *from_d,
改为void **from_d,
因为有两个该函数，是在判断条件下执行不同的函数，所以要改两次
```
__pkcs11h_openssl_ex_data_dup (
	CRYPTO_EX_DATA *to,
	CRYPTO_EX_DATA *from,
	void **from_d,
	int idx,
	long argl,
	void *argp
)
```
## 4、相关PR ##
openEuler：https://gitee.com/src-openeuler/pkcs11-helper/pulls/11
上游社区：不涉及