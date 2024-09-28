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

