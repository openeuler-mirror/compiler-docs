# openssh.1 #

## 1、问题现象 ##
出自：https://gitee.com/src-openeuler/openssh(openEuler-24.03-LTS)

```
[  188s] ssh-ecdsa.c:267:6: error: call to undeclared function 'is_ecdsa_pkcs11'; ISO C99 and later do not support implicit function declarations [-Wimplicit-function-declaration]
[  188s]   267 |         if (is_ecdsa_pkcs11(key->ecdsa)) {          ^
```
## 2、问题定位及根因分析 ##

源代码中没有出现问题代码
问题出在openssh-9.3p1-merged-openssl-evp.patch中
is_ecdsa_pkcs11函数调用了一个未声明的函数，而这个函数在ssh-pkcs11.h中定义了，这就是导致这个问题的原因
`if (key->ecdsa != NULL && is_ecdsa_pkcs11(key->ecdsa)) {`

小例子复现：

complex_header.h

```
 #ifndef COMPLEX_HEADER_H
 #define COMPLEX_HEADER_H

 #include <iostream>
 #include <vector>

// 声明一个复杂的类
class ComplexClass {
public:
    ComplexClass();
    void complexMethod();
    int complexFunction(int a, int b);
};

// 声明一个函数
int declaredFunction(int x);

 #endif
```

main.c

```
 #include "complex_header.h"

// 定义 ComplexClass 的构造函数
ComplexClass::ComplexClass() {
    std::cout << "ComplexClass 构造函数调用" << std::endl;
}

// 定义 ComplexClass 的方法
void ComplexClass::complexMethod() {
    std::cout << "ComplexMethod 调用" << std::endl;
}

// 定义 ComplexClass 的函数
int ComplexClass::complexFunction(int a, int b) {
    return a + b;
}

// 定义 declaredFunction
int declaredFunction(int x) {
    return x * x;
}

int main() {
    ComplexClass obj;
    obj.complexMethod();
    std::cout << "ComplexFunction 结果: " << obj.complexFunction(3, 4) << std::endl;
    std::cout << "DeclaredFunction 结果: " << declaredFunction(5) << std::endl;

    // 调用未声明的函数
    std::cout << "UndeclaredFunction 结果: " << undeclaredFunction(10) << std::endl;

    return 0;
}
```

错误信息
当使用gcc编译时正确，而clang 编译上述代码时，会出现隐式函数声明错误：

```
$ clang main.c -o main
main.cpp:28:52: error: call to undeclared function 'undeclaredFunction'; ISO C99 and later do not support implicit function declarations [-Wimplicit-function-declaration]
    std::cout << "UndeclaredFunction 结果: " << undeclaredFunction(10) << std::endl;
                                                   ^
1 error generated.
```
## 3、修改建议 ##

在ssh-rsa.c和ssh-ecdsa.c中加入头文件
即`#include "ssh-pkcs11.h"`
就能消除这个错误
## 4、相关PR ##
openEuler：https://gitee.com/src-openeuler/openssh/pulls/307
上游社区：不涉及