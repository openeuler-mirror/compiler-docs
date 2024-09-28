## Implementation-defined Behaviors

## 1. Conditionally-Supported Behavior

Each implementation shall include documentation that identifies all conditionally-supported constructs that it does not support.

故为了本文档与GCC文档保持一致，仅实验GCC文档中提及的implementation-defined Behaviors.

### 1.1 Whether an argument of class type with a non-trivial copy constructor or destructor can be passed to ...(C++0x 5.2.2).

- 具有非平凡复制构造函数或析构函数的类类型的参数是否可以传递给...

GCC可以，LLVM不行，与GCC不一致

```cpp
#include <iostream>
#include <cstdarg>

class NonTrivialClass {
        public:
        int value;

        NonTrivialClass(int v) : value(v) { std::cout << "Constructor called\n"; }

        // 自定义复制构造函数，使类成为非平凡类
        NonTrivialClass(const NonTrivialClass &other) {
            value = other.value;
            std::cout << "Copy constructor called\n";
        }

        // 自定义析构函数
        ~NonTrivialClass() {
            std::cout << "Destructor called\n";
        }
};

void variadicFunction(int num, ...) {
    va_list args;
    va_start(args, num);

    for (int i = 0; i < num; ++i) {
        // 尝试提取 NonTrivialClass 类型的参数
        NonTrivialClass obj = va_arg(args, NonTrivialClass);
        std::cout << "Extracted value: " << obj.value << "\n";
    }

    va_end(args);
}

int main() {
    NonTrivialClass obj1(1);
    variadicFunction(1, obj1);
    return 0;
}
```

```bash
$ make gcc
g++ --std=c++11 -Wall -Wno-varargs ./test.cpp -o target_gcc
echo //target_gcc > output_gcc
./target_gcc>>output_gcc

$ make clang
clang++ --std=c++11 -Wall -Wnon-pod-varargs ./test.cpp -o target_clang
./test.cpp:28:44: error: second argument to 'va_arg' is of non-POD type 'NonTrivialClass' [-Wnon-pod-varargs]
        NonTrivialClass obj = va_arg(args, NonTrivialClass);
                                           ^~~~~~~~~~~~~~~
/usr/lib/llvm-14/lib/clang/14.0.0/include/stdarg.h:19:50: note: expanded from macro 'va_arg'
#define va_arg(ap, type)    __builtin_va_arg(ap, type)
                                                 ^~~~
./test.cpp:37:25: error: cannot pass object of non-trivial type 'NonTrivialClass' through variadic function; call will abort at runtime [-Wnon-pod-varargs]
    variadicFunction(1, obj1);
                        ^
2 errors generated.
make: *** [Makefile:21: clang] Error 1
```

```cpp
//target_gcc
Constructor called
Copy constructor called
Copy constructor called
Extracted value: 1
Destructor called
Destructor called
Destructor called
```

在本实验环境中，GCC可以，LLVM不行，与GCC一致。

## 2. Exception Handling

### 1. In the situation where no matching handler is found, it is implementation-defined whether or not the stack is unwound before std::terminate() is called.(C++98 15.5.1).

- 在未找到匹配的处理程序的情况下，在调用 std::terminate() 之前是否展开堆栈由实现定义。

```cpp
#include <iostream>
#include <exception>

class TestClass {
        public:
        TestClass() {
            std::cout << "TestClass Constructor\n";
        }
        ~TestClass() {
            std::cout << "TestClass Destructor\n";
        }
};

void throwException() {
    TestClass obj;
    std::cout << "Throwing exception...\n";
    throw std::runtime_error("Uncaught exception");
}

int main() {
    try {
        throwException();
    } catch (int e) {
        std::cout << "Caught an integer exception\n";
    }
    // No catch for std::runtime_error, std::terminate will be called
    return 0;
}
```

```bash
$ make gcc
g++ -std=c++11 -Wall  ./test.cpp -o target_gcc
echo //target_gcc > output_gcc
./target_gcc>>output_gcc
terminate called after throwing an instance of 'std::runtime_error'
  what():  Uncaught exception
Aborted
make: *** [Makefile:18: gcc] Error 134

$ make clang
clang++ -std=c++11 -Wall  ./test.cpp -o target_clang
./echo //target_clang > output_clang
./target_clang>>output_clang
terminate called after throwing an instance of 'std::runtime_error'
  what():  Uncaught exception
Aborted
make: *** [Makefile:23: clang] Error 134

$ ./target_gcc
TestClass Constructor
Throwing exception...
terminate called after throwing an instance of 'std::runtime_error'
  what():  Uncaught exception
Aborted

$ ./target_clang
TestClass Constructor
Throwing exception...
terminate called after throwing an instance of 'std::runtime_error'
  what():  Uncaught exception
Aborted
```

栈不会展开，和GCC一致。