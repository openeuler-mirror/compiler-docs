## freerdp

### 问题描述

#### 报错信息

修改前 clang 构建在 `wlfreerdp.c` 出现报错：

```
/home/abuild/rpmbuild/BUILD/freerdp-2.11.7/client/Wayland/wlfreerdp.c:637:19: error: incompatible function pointer types assigning to 'OBJECT_NEW_FN' (aka 'void *(*)(void *)') from 'void *(const void *)' [-Wincompatible-function-pointer-types]
  637 |         obj->fnObjectNew = uwac_event_clone;
      |                          ^ ~~~~~~~~~~~~~~~~
```

#### 相关PR

- 原有上游社区 PR：[Fixes clang error by wangmingyu84 · Pull Request #9373 · FreeRDP/FreeRDP (github.com)](https://github.com/FreeRDP/FreeRDP/pull/9373)

- openEuler 仓库 PR：[Fix clang incompatible function pointer error · Pull Request !116 · src-openEuler/freerdp - Gitee.com](https://gitee.com/src-openeuler/freerdp/pulls/116)

### 修复过程

- 在已有上游社区 PR：[Fixes clang error by wangmingyu84 · Pull Request #9373 · FreeRDP/FreeRDP (github.com)](https://github.com/FreeRDP/FreeRDP/pull/9373)当中：

  - 由于PR作者未更新到最新版本而是针对落后版本的`collection.h`进行了修改，因此该PR被上游代码仓管理者关闭。修复时中间仓freerdp版本为2.11.7，同样使用了未更新的`collection.h`，但更改到该行代码的上游commit：[Cleaned up collections: · FreeRDP/FreeRDP@6e3c007 (github.com)](https://gitee.com/link?target=https%3A%2F%2Fgithub.com%2FFreeRDP%2FFreeRDP%2Fcommit%2F6e3c00725aae99d03a0baa65430eceddebd9dee8)与2.11.7的commit：[release-2.11.7 · FreeRDP/FreeRDP@7f6cc93 (github.com)](https://gitee.com/link?target=https%3A%2F%2Fgithub.com%2FFreeRDP%2FFreeRDP%2Fcommit%2F7f6cc93c21d7f0faad6daacca06f494f29ce882c)之间涉及到的文件过多，更改过大，选择仅按照上游社区的代码修改变更报错涉及的三处代码。

- 该报错的原因为 clang 对类型的检查比 gcc 更严格，该clang error在gcc中只是warning。相关代码行中`obj->fnObjectNew`的类型定义为：

  ```c
  typedef void* (*OBJECT_NEW_FN)(void* val);
  ```

  即`void *(*)(void *)`，不符合需求的`void *(const void *)`

  修改类型定义以及涉及的相应的函数参数类型即可修复，如：

  ```
  -	typedef void* (*OBJECT_NEW_FN)(void* val);
  +	typedef void* (*OBJECT_NEW_FN)(const void* val);
  ```

  ```
  -static void* rfx_decoder_tile_new(void* val)
  +static void* rfx_decoder_tile_new(const void* val)
  ```

