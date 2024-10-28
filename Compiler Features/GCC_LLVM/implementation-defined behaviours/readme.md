llvm和gcc是当下主流的C、C++编译器。应用使用不同编译器编译构建，可能遇到编译或者运行的兼容性问题。产生问题的主要原因有二种：1. 代码存在未定义行为，未遵循C/C++语言标准；2. 编译器实现的差别，又分为unspecified behavior 和 implementation defined behavior两种。 unspecified behavior：2种及以上的实现方式，编译器可以选择一种实现； implementation defined behavior 没有明确规定的实现方式，由编译器自定义实现。 

C/C++语言标准里已明确定义和列举了undefined behavior、unspecified behavior、implementation defined behavior，明确llvm和gcc在此类行为方面的不同，就能提前预警编译器间的兼容性问题。



测试环境为Ubuntu 22.04.4 LTS，GCC版本为11.4.0，LLVM版本为14.0.0，Clang版本为14.0.0-1ubuntu1.1，glibc版本为2.35。

对于implementation-defined behaviours而言，有很多是由the implementation of the C library以及ABI决定的，在本梳理中选择跳过处理。

测试C所用的`Makefile`如下：

```makefile
TESTFILE = ./test.c

TARGET_GCC = target_gcc
TARGET_CLANG = target_clang
OUTPUT_GCC = output_gcc
OUTPUT_CLANG = output_clang

CAT = cat
GXX = gcc
CLANG = clang
CFLAGS = --std=c11 #-fsanitize=shift
FOLLOW_OPTION = #-lm
PRE_PROCESS_CFLAGS = $(CFLAGS) -E

gcc:$(TESTFILE)
	$(GXX) $(CFLAGS) $(TESTFILE) -o $(TARGET_GCC) $(FOLLOW_OPTION)
	echo //$(TARGET_GCC) > $(OUTPUT_GCC)
	./$(TARGET_GCC)>>$(OUTPUT_GCC)

clang:$(TESTFILE)
	$(CLANG) $(CFLAGS) $(TESTFILE) -o $(TARGET_CLANG) $(FOLLOW_OPTION)
	echo //$(TARGET_CLANG) > $(OUTPUT_CLANG)
	./$(TARGET_CLANG)>>$(OUTPUT_CLANG)

preprocess_gcc:$(TESTFILE)
	$(GXX) $(PRE_PROCESS_CFLAGS) $(TESTFILE) -o $(TARGET_GCC) $(FOLLOW_OPTION)
	$(CAT) $(TARGET_GCC) > $(OUTPUT_GCC)

preprocess_clang:$(TESTFILE)
	$(CLANG) $(PRE_PROCESS_CFLAGS) $(TESTFILE) -o $(TARGET_CLANG) $(FOLLOW_OPTION)
	$(CAT) $(TARGET_CLANG) > $(OUTPUT_CLANG)

clean:
	rm -f $(TARGET_GCC) $(TARGET_CLANG) $(OUTPUT_GCC) $(OUTPUT_CLANG)

test: clean gcc clang
preprocess_test: preprocess_gcc preprocess_clang

.PHONY: clean test preprocess_test
```

测试CPP所用的`Makefile`如下：


```makefile
TESTFILE = ./test.c

TARGET_GCC = target_gcc
TARGET_CLANG = target_clang
OUTPUT_GCC = output_gcc
OUTPUT_CLANG = output_clang

CAT = cat
GXX = g++
CLANG = clang++
CFLAGS = --std=c++11 -Wall -Wno-varargs
FOLLOW_OPTION = #-lm
PRE_PROCESS_CFLAGS = $(CFLAGS) -E

gcc:$(TESTFILE)
	$(GXX) $(CFLAGS) $(TESTFILE) -o $(TARGET_GCC) $(FOLLOW_OPTION)
	echo //$(TARGET_GCC) > $(OUTPUT_GCC)
	./$(TARGET_GCC)>>$(OUTPUT_GCC)

clang:$(TESTFILE)
	$(CLANG) $(CFLAGS) $(TESTFILE) -o $(TARGET_CLANG) $(FOLLOW_OPTION)
	echo //$(TARGET_CLANG) > $(OUTPUT_CLANG)
	./$(TARGET_CLANG)>>$(OUTPUT_CLANG)

preprocess_gcc:$(TESTFILE)
	$(GXX) $(PRE_PROCESS_CFLAGS) $(TESTFILE) -o $(TARGET_GCC) $(FOLLOW_OPTION)
	$(CAT) $(TARGET_GCC) > $(OUTPUT_GCC)

preprocess_clang:$(TESTFILE)
	$(CLANG) $(PRE_PROCESS_CFLAGS) $(TESTFILE) -o $(TARGET_CLANG) $(FOLLOW_OPTION)
	$(CAT) $(TARGET_CLANG) > $(OUTPUT_CLANG)

clean:
	rm -f $(TARGET_GCC) $(TARGET_CLANG) $(OUTPUT_GCC) $(OUTPUT_CLANG)

test: clean gcc clang
preprocess_test: preprocess_gcc preprocess_clang

.PHONY: clean test preprocess_test
```

