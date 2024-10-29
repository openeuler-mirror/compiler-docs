## accountsservice

### 问题描述

#### 报错信息

##### 1. aarch64 —— meson.build: ERROR: Assert failed: Do not know which filename to watch for wtmp changes

```
[   89s] Checking for function "setutxdb" : NO 
[   89s] Checking for function "fgetpwent" : YES 
[   89s] Header "utmpx.h" has symbol "WTMPX_FILENAME" : NO 
[   89s] Header "paths.h" has symbol "_PATH_WTMPX" : NO 
[   89s] 
[   89s] meson.build:107:2: ERROR: Assert failed: Do not know which filename to watch for wtmp changes
[   89s] 
[   89s] A full log can be found at /home/abuild/rpmbuild/BUILD/accountsservice-23.13.9/aarch64-openEuler-linux-gnu/meson-logs/meson-log.txt
```

##### 2. error: call to undeclared function 'print_indent'; ISO C99 and later do not support implicit function declarations [-Wimplicit-function-declaration]

```
...
[  131s] [75/109] clang -Isubprojects/mocklibc-1.0/src/mocklibc-debug-netgroup.p -Isubprojects/mocklibc-1.0/src -I../subprojects/mocklibc-1.0/src -fdiagnostics-color=always -D_FILE_OFFSET_BITS=64 -Wall -Winvalid-pch -O2 -g -grecord-gcc-switches -pipe -fstack-protector-strong -Wall -Werror=format-security -Wp,-D_FORTIFY_SOURCE=2 -Wp,-D_GLIBCXX_ASSERTIONS --config /usr/lib/rpm/generic-hardened-clang.cfg -m64 -mtune=generic -fasynchronous-unwind-tables -fstack-clash-protection -fsigned-char -MD -MQ subprojects/mocklibc-1.0/src/mocklibc-debug-netgroup.p/netgroup-debug.c.o -MF subprojects/mocklibc-1.0/src/mocklibc-debug-netgroup.p/netgroup-debug.c.o.d -o subprojects/mocklibc-1.0/src/mocklibc-debug-netgroup.p/netgroup-debug.c.o -c ../subprojects/mocklibc-1.0/src/netgroup-debug.c
[  131s] FAILED: subprojects/mocklibc-1.0/src/mocklibc-debug-netgroup.p/netgroup-debug.c.o 
[  131s] clang -Isubprojects/mocklibc-1.0/src/mocklibc-debug-netgroup.p -Isubprojects/mocklibc-1.0/src -I../subprojects/mocklibc-1.0/src -fdiagnostics-color=always -D_FILE_OFFSET_BITS=64 -Wall -Winvalid-pch -O2 -g -grecord-gcc-switches -pipe -fstack-protector-strong -Wall -Werror=format-security -Wp,-D_FORTIFY_SOURCE=2 -Wp,-D_GLIBCXX_ASSERTIONS --config /usr/lib/rpm/generic-hardened-clang.cfg -m64 -mtune=generic -fasynchronous-unwind-tables -fstack-clash-protection -fsigned-char -MD -MQ subprojects/mocklibc-1.0/src/mocklibc-debug-netgroup.p/netgroup-debug.c.o -MF subprojects/mocklibc-1.0/src/mocklibc-debug-netgroup.p/netgroup-debug.c.o.d -o subprojects/mocklibc-1.0/src/mocklibc-debug-netgroup.p/netgroup-debug.c.o -c ../subprojects/mocklibc-1.0/src/netgroup-debug.c
[  131s] ../subprojects/mocklibc-1.0/src/netgroup-debug.c:25:3: error: call to undeclared function 'print_indent'; ISO C99 and later do not support implicit function declarations [-Wimplicit-function-declaration]
[  131s]    25 |   print_indent(stream, indent);
[  131s]       |   ^
[  131s] ../subprojects/mocklibc-1.0/src/netgroup-debug.c:44:3: error: call to undeclared function 'print_indent'; ISO C99 and later do not support implicit function declarations [-Wimplicit-function-declaration]
[  131s]    44 |   print_indent(stream, indent);
[  131s]       |   ^
[  131s] ../subprojects/mocklibc-1.0/src/netgroup-debug.c:53:3: error: call to undeclared function 'print_indent'; ISO C99 and later do not support implicit function declarations [-Wimplicit-function-declaration]
[  131s]    53 |   print_indent(stream, indent);
[  131s]       |   ^
[  131s] 3 errors generated.
...
```

#### 相关PR

- 上游社区PR：
  - [Draft: Make checking for valid wtmp path optional, for musl. (!97) · Merge requests · accountsservice / accountsservice · GitLab](https://gitlab.freedesktop.org/accountsservice/accountsservice/-/merge_requests/97)
  - [Fix compiler warnings (!131) · Merge requests · accountsservice / accountsservice · GitLab](https://gitee.com/link?target=https%3A%2F%2Fgitlab.freedesktop.org%2Faccountsservice%2Faccountsservice%2F-%2Fmerge_requests%2F131)
- openEuler仓库PR：
  - [Remove wtmp check to supprt clang build in aarch64 · Pull Request !30 · src-openEuler/accountsservice - Gitee.com](https://gitee.com/src-openeuler/accountsservice/pulls/30)
  - [Backport fix compiler warning/error in mocklibc to support clang build · Pull Request !29 · src-openEuler/accountsservice - Gitee.com](https://gitee.com/src-openeuler/accountsservice/pulls/29)

### 修复过程

#### 1. Assert failed: Do not know which filename to watch for wtmp changes

- 参考了上游社区的草稿PR：[Draft: Make checking for valid wtmp path optional, for musl. (!97) · Merge requests · accountsservice / accountsservice · GitLab](https://gitlab.freedesktop.org/accountsservice/accountsservice/-/merge_requests/97)

    - 作者是为了支持musl而提出的pr，他表示：musl根本不支持wtmp，只是将其实现为不做任何事情的stub，参见[这个链接](https://wiki.musl-libc.org/faq.html#Q:_Why_is_the_utmp/wtmp_functionality_only_implemented_as_stubs?) ，当使用`meson setup`配置accountsservice时，musl会失败，因为musl没有定义到wtmp的路径。作者维护的作者维护的使用到accountsservice的项目Gentoo中有多人报告了这个bug：[762442 – sys-apps/accountsservice-23.13.9 - meson.build:85:2: ERROR: Assert failed: Do not know which filename to watch for wtmp changes (on musl) (gentoo.org)](https://bugs.gentoo.org/762442)。虽然主要是为了支持musl，但是使用arm64 clang构建也能复现这个报错。

    - 在上游pr中，作者一开始将wtmp该检查成设为可选的，使其适用于musl，并添加了musl需要的fgetspent_r的替换。后来在项目所有者的意见下改为将wtmp检查直接删除了。

    - 目前该pr已经一年没有更新。但Alpine以及Gentoo中都采用了该pr的修改方式。且按照项目所有者的说法，他认为直接删除wtmp的检查而不进行选择是OK的。因此选择了继续采用该pr的修复方式。

  - 由于本次修复目标是aarch64的构建问题，对于musl是否支持并没有检验，又因为经过测试，仅删除wtmp的检查即可构建成功，不需要添加fgetspent_r的替换，所以目前仅搬运了删除wtmp检查的部分。


#### 2. call to undeclared function 'print_indent'

- 该报错位于项目使用的一个子项目`mocklibc`，报错原因在于`print_indent`在`mocklibc`中的声明和定义位于`src/netgroup.c`中，而在`src/netgroup-debug.c`中未声明就进行了调用。

- 在上游社区查找到相关bug报告：https://bugs.freedesktop.org/show_bug.cgi?id=95342

- 在上游社区找到相关已合并PR和commit，其中选择的解决方案为在`subprojects/mocklibc.wrap`中加入一个`src/netgroup-debug.c`的补丁，使得`src/netgroup-debug.c`中加入`print_indent`的声明：

  - PR：[Fix compiler warnings (!131) · Merge requests · accountsservice / accountsservice · GitLab](https://gitee.com/link?target=https%3A%2F%2Fgitlab.freedesktop.org%2Faccountsservice%2Faccountsservice%2F-%2Fmerge_requests%2F131)

  - commit：[mocklibc: Fix compiler warning (da65bee1) · Commits · accountsservice / accountsservice · GitLab](https://gitee.com/link?target=https%3A%2F%2Fgitlab.freedesktop.org%2Faccountsservice%2Faccountsservice%2F-%2Fcommit%2Fda65bee12d9118fe1a49c8718d428fe61d232339)

- 引入该commit的patch修改后，clang构建“call to undeclared function 'print_indent'”报错消失。
