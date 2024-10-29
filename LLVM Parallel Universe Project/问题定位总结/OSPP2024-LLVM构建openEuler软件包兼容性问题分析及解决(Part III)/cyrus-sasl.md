## cyrus-sasl

### 问题描述

#### 报错信息

修改前 clang 构建在 `saslutil.c` 出现报错：

```
saslutil.c:280:3: error: call to undeclared function 'time'; ISO C99 and later do not support implicit function declarations [-Wimplicit-function-declaration]
  280 |   time(&now);
      |   ^
saslutil.c:364:34: error: call to undeclared function 'clock'; ISO C99 and later do not support implicit function declarations [-Wimplicit-function-declaration]
  364 |             ret[1] ^= (unsigned short) (clock() & 0xFFFF);
      |                                         ^
saslutil.c:373:22: error: call to undeclared function 'time'; ISO C99 and later do not support implicit function declarations [-Wimplicit-function-declaration]
  373 |     curtime = (long) time(NULL); /* better be at least 32 bits */
      |                      ^
saslutil.c:377:33: error: call to undeclared function 'clock'; ISO C99 and later do not support implicit function declarations [-Wimplicit-function-declaration]
  377 |     ret[2] ^= (unsigned short) (clock() & 0xFFFF);
      |                                 ^
saslutil.c:563:42: warning: comparison of integers of different signs: 'unsigned long' and 'int' [-Wsign-compare]
  563 |         || strlen (result->ai_canonname) > namelen -1) {
      |            ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ ^ ~~~~~~~~~~
```

#### 相关PR

- 上游社区 PR：[Fix check by thesamesam · Pull Request #709 · cyrusimap/cyrus-sasl (github.com)](https://github.com/cyrusimap/cyrus-sasl/pull/709)
- openEuler 仓库 PR：[Backport fix for time.h check for clang compilation · Pull Request !43 · src-openEuler/cyrus-sasl - Gitee.com](https://gitee.com/src-openeuler/cyrus-sasl/pulls/43)

### 修复过程

- 回合上游已合并的PR。
- 报错为调用未声明的函数，C99 标准及其以后的标准，隐式函数声明是不允许的，在调用函数之前，必须先声明或定义该函数。在clang16后，该warning升级为了error（[New warnings and errors in Clang 16 ](https://www.redhat.com/en/blog/new-warnings-and-errors-clang-16)）。本软件包中相关报错全部来源于`time.h`中的各种定义。backport的上游PR中，添加对`time.h`的include及头文件检查。

