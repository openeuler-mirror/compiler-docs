## cups

### 问题描述

#### 报错信息

修改前 clang 构建在 `testhttp.c` 出现报错：

```
testhttp.c:395:32: error: incompatible integer to pointer conversion passing 'int' to parameter of type 'const char *' [-Wint-conversion]
  395 |     addrlist = httpAddrGetList(gethostname(localhostname, sizeof(localhostname)), AF_UNSPEC, NULL);
      |                                ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
../cups/http.h:526:53: note: passing argument to parameter 'hostname' here
  526 | extern http_addrlist_t  *httpAddrGetList(const char *hostname, int family, const char *service) _CUPS_API_1_2;
      |                                                      ^
1 error generated.
```

#### 相关PR

- openEuler 仓库 PR：[Fix clang build incompatible integer to pointer conversion error · Pull Request !158 · src-openEuler/cups - Gitee.com](https://gitee.com/src-openeuler/cups/pulls/158)

### 修复过程

- 该报错由openEuler中间仓的Patch: `fix-httpAddrGetList-test-case-fail.patch` 引入。`-Wint-conversion`在clang中默认为error，而在gcc中为warning，故当使用gcc构建时，不会报上述错误。该Patch对应PR为：[fix build error · Pull Request !103](https://gitee.com/src-openeuler/cups/pulls/103)。上游社区中报错这行代码至今仍未改变，为：

  ```c
  addrlist = httpAddrGetList(hostname, AF_UNSPEC, NULL);
  ```

  若将该Patch删除可以用clang构建成功，但目前版本（v2.4.7）中仍能复现[fix build error · Pull Request !103](https://gitee.com/src-openeuler/cups/pulls/103)中的构建失败问题，故选择在`fix-httpAddrGetList-test-case-fail.patch`的基础上修改，使得修改后使用clang也能构建成功。

- 错误信息表明试图将一个整数（`int`）类型的值传递给需要字符指针（`const char *`）类型的参数的函数。`gethostname`函数的参数和返回值要求如下：

  ```c
  int gethostname(char *name, size_t len)；
  ```

  `httpAddrGetList`需要的参数则是：

  ```c
  extern http_addrlist_t *httpAddrGetList(const char *hostname, int family, const char *service) _CUPS_PUBLIC;
  ```

  `gethostname` 函数会将当前主机的名字复制到不超过 `len` 字节的`name` 指向的缓冲区中，如果 `len` 足够大，能够容纳完整的主机名以及末尾的空字符终止符，那么 `name` 缓冲区中的字符串将以空字符终止。返回的int值代表是否成功。

- 修改建议：按照原本的含义来说，传入`httpAddrGetList`的应该是`hostname`而非`gethostname`的返回值。因此应当将代码改为

  ```c
  gethostname(localhostname, sizeof(localhostname));
  addrlist = httpAddrGetList(localhostname, AF_UNSPEC, NULL);
  ```


- 
