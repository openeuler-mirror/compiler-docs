测试环境为 Gentoo Linux，GCC 版本为 14.2.1，LLVM 的版本为 18.1.8，glibc 的版本为 2.40。

期间使用 `make` 简化测试流程，具体的 Makefile 如下所示:

```Makefile
SRC = ./57_lrint.c

TARGET_GCC = test_gcc
TARGET_CLANG = test_clang

CFLAGS = --std=c11 -ggdb
# CFLAGS +=  -Wall -Wextra -Wpedantic -Werror

.PHONY: clean gcc clang g++ clang++ cmp

gcc: $(SRC)
	gcc $(CFLAGS) $(SRC) -o $(TARGET_GCC)
	./$(TARGET_GCC) > gcc_out
clang: $(SRC)
	clang $(CFLAGS) $(SRC) -o $(TARGET_CLANG)
	./$(TARGET_CLANG) > clang_out

clean:
	rm $(TARGET_GCC) $(TARGET_CLANG)

cmp: gcc clang
	./compare.sh
```

`make cmp` 会调用 **./compare.sh** 对 GCC 和 LLVM 的输出结果进行比对，**./compare.sh** 的内容是:

```sh
#!/usr/bin/bash

output=$(diff gcc_out clang_out)
if [ -z "$output" ]; then
  echo "GCC 和 LLVM 输出一致"
  cat gcc_out
else
  echo "GCC 和 LLVM 输出不一致"
  cat gcc_out
  echo ""
  cat clang_out
fi
```

C++ 14 的基本和上面这个一致，只是使用的编译器和 `CXXFLAGS` 不一样而已。
