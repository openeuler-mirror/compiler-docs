# vdo #

## 1、问题现象 ##

出自：https://gitee.com/src-openeuler/vdo
(openEuler-24.03-LTS)
```
[   77s] error: unknown warning option '-Wlogical-op'; did you mean '-Wlong-long'? [-Werror,-Wunknown-warning-option]
[   77s] make[2]: *** [Makefile:137: murmurhash3.o] Error 1
[   77s] make[2]: Leaving directory '/home/abuild/rpmbuild/BUILD/vdo-8.2.2.2/utils/uds'
[   77s] make[2]: Entering directory '/home/abuild/rpmbuild/BUILD/vdo-8.2.2.2/utils/uds'
[   77s] clang -O2 -g -grecord-gcc-switches -pipe -fstack-protector-strong  -Wall -Werror=format-security -Wp,-D_FORTIFY_SOURCE=2 -Wp,-D_GLIBCXX_ASSERTIONS --config /usr/lib/rpm/generic-hardened-clang.cfg -m64 -mtune=generic -fasynchronous-unwind-tables -fstack-clash-protection  -D_GNU_SOURCE -g -O3 -fno-omit-frame-pointer -Wall -Wcast-align -Werror -Wextra -Winit-self -Wlogical-op -Wmissing-include-dirs -Wpointer-arith -Wredundant-decls -Wunused -Wwrite-strings   -DCURRENT_VERSION='"8.2.2.2"'  -I. -std=gnu99 -pedantic -Wbad-function-cast -Wcast-qual -Wfloat-equal -Wformat=2 -Wmissing-declarations -Wmissing-format-attribute -Wmissing-prototypes -Wnested-externs -Wold-style-definition -Wswitch-default    -c -MD -MF .deps/buffer.d.new -MP -MT buffer.o buffer.c -o buffer.o
```
## 2、问题定位及根因分析 ##

通过搜索-Wlong-long关键字找到了两个文件中存在：

即utils/uds/Makefile文件和utils/vdo/Makefile文件中的-WARNS和-C_WARNS部分是clang不接受的形式
```
WARNS =	-Wall			\
	-Wcast-align		\
	-Werror			\
	-Wextra			\
	-Winit-self		\
	-Wlogical-op		\
	-Wmissing-include-dirs	\
	-Wpointer-arith		\
	-Wredundant-decls	\
	-Wunused		\
	-Wwrite-strings

C_WARNS =	-Wbad-function-cast		\
		-Wcast-qual			\
		-Wfloat-equal			\
		-Wformat=2			\
		-Wmissing-declarations		\
		-Wmissing-format-attribute	\
		-Wmissing-prototypes		\
		-Wnested-externs		\
		-Wold-style-definition		\
		-Wswitch-default
```
## 3.解决方案 ##

于是做了一个判断，如果编译器为gcc则执行-WARNS和-C_WARNS，如果是clang则不执行，成功修复

即
```
+# Define WARNS and C_WARNS only if using gcc
+CLANG_VERSION := $(shell $(CC) --version | grep -i clang)
+ifeq ($(findstring clang, $(CLANG_VERSION)),clang)
+WARNS =
+C_WARNS =
+else
```
## 4、相关PR ##
openEuler：https://gitee.com/src-openeuler/vdo/pulls/31
上游社区：https://github.com/dm-vdo/vdo/pull/72