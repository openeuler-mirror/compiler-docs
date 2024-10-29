## iSulad

#### 相关PR

- **上游社区pr**：[Fix clang build error · Pull Request !2500 · openEuler/iSulad - Gitee.com](https://gitee.com/openeuler/iSulad/pulls/2500)
- **openEuler中间仓pr**：[Fix clang build error · Pull Request !729 · src-openEuler/iSulad - Gitee.com](https://gitee.com/src-openeuler/iSulad/pulls/729)

### 报错信息一

修改前 clang 构建在 `stream_server.h` 出现报错`error: 'cri_stream_server_url' has C-linkage specified, but returns user-defined type 'url::URLDatum' which is incompatible with C [-Werror,-Wreturn-type-c-linkage]`：

```
In file included from /home/abuild/rpmbuild/BUILD/iSulad-v2.1.5/src/daemon/entry/connect/grpc/cri/cri_service.cc:19:
/home/abuild/rpmbuild/BUILD/iSulad-v2.1.5/src/daemon/entry/cri/streams/stream_server.h:30:15: error: 'cri_stream_server_url' has C-linkage specified, but returns user-defined type 'url::URLDatum' which is incompatible with C [-Werror,-Wreturn-type-c-linkage]
   30 | url::URLDatum cri_stream_server_url(void);
      |               ^
1 error generated.
```

#### 修复过程

- **根因分析**：

  原本`url::URLDatum cri_stream_server_url(void);`被包裹在`extern "C" {}`中：

  ```c
  #ifdef __cplusplus
  extern "C" {
  #endif
  
  ...
  url::URLDatum cri_stream_server_url(void);
  
  #ifdef __cplusplus
  }
  #endif
  ```

  `extern "C"`是C++特有的一个关键字，用来告诉编译器函数或者变量是以C语言的方式进行链接的。使用`extern "C"`来声明函数或变量时，是在告诉编译器虽然其中的内容是在C++代码中定义的，但应该用C语言的方式来进行链接。但`url::URLDatum`一个用户定义的类型，在C语言中是无法直接处理的。

- 参考了GitHub上对于该报错的修改方案，在移除了`stream_server.h`中的

  ```
  #ifdef __cplusplus
  extern "C" {
  #endif
  
  #ifdef __cplusplus
  }
  #endif
  ```

  后该报错消失。

- 在maintainer的建议下，发现没必要全部删除`extern "C"`，将涉及的代码移出`extern "C"`块更为合适。如：

  ```c
  #ifdef __cplusplus
  extern "C" {
  #endif
  
  ...
  
  #ifdef __cplusplus
  }
  #endif
  
  url::URLDatum cri_stream_server_url(void);
  ```

  

### 报错信息二

`error: unqualified call to 'std::move' [-Werror,-Wunqualified-std-cast-call]`

```
/root/rpmbuild/BUILD/iSulad-v2.1.5/src/daemon/entry/cri/v1/v1_cri_image_manager_service_impl.cc:152:26: error: unqualified call to 'std::move' [-Werror,-Wunqualified-std-cast-call]
  152 |         images.push_back(move(image));
      |                          ^
      |                          std::
...
/root/rpmbuild/BUILD/iSulad-v2.1.5/src/daemon/entry/cri/v1/v1_cri_container_manager_service.cc:693:30: error: unqualified call to 'std::move' [-Werror,-Wunqualified-std-cast-call]
  693 |         containers.push_back(move(container));
      |                              ^
      |                              std::
...
/root/rpmbuild/BUILD/iSulad-v2.1.5/src/daemon/entry/cri/v1/v1_cri_pod_sandbox_manager_service.cc:1173:16: error: unqualified call to 'std::move' [-Werror,-Wunqualified-std-cast-call]
 1173 |     podStats = move(podStatsPtr);
      |                ^
      |                std::
...
/root/rpmbuild/BUILD/iSulad-v2.1.5/src/daemon/entry/cri/v1/v1_cri_pod_sandbox_manager_service.cc:1323:29: error: unqualified call to 'std::move' [-Werror,-Wunqualified-std-cast-call]
 1323 |         podsStats.push_back(move(podStats));
      |                             ^
      |                             std::
...
/root/rpmbuild/BUILD/iSulad-v2.1.5/src/daemon/entry/cri/v1alpha/cri_image_manager_service_impl.cc:152:26:
 error: unqualified call to 'std::move' [-Werror,-Wunqualified-std-cast-call]
...
/root/rpmbuild/BUILD/iSulad-v2.1.5/src/daemon/entry/cri/v1/v1_cri_container_manager_service.cc:693:30: error: unqualified call to 'std::move' [-Werror,-Wunqualified-std-cast-call]
...
/root/rpmbuild/BUILD/iSulad-v2.1.5/src/daemon/entry/cri/v1/v1_cri_pod_sandbox_manager_service.cc:1173:16:
 error: unqualified call to 'std::move' [-Werror,-Wunqualified-std-cast-call]
...
/root/rpmbuild/BUILD/iSulad-v2.1.5/src/daemon/entry/cri/v1/v1_cri_pod_sandbox_manager_service.cc:1323:29:
 error: unqualified call to 'std::move' [-Werror,-Wunqualified-std-cast-call]
...
```

#### 修复过程

- 报错原因：代码中一些地方将应以`std::move`调用的`std::move`写成了`move`，如：

  ```
  containers.push_back(move(container));
  ```

- 修复方式：将对应的`move`改为`std::move`后解决

  ```
  containers.push_back(std::move(container));
  ```



### 报错信息三

`error: suggest braces around initialization of subobject [-Werror,-Wmissing-braces]`

```
/root/rpmbuild/BUILD/iSulad-v2.1.5/src/daemon/entry/cri/v1/v1_cri_pod_sandbox_manager_service.cc:1182:38: error: suggest braces around initialization of subobject [-Werror,-Wmissing-braces]
 1182 |     cgroup_metrics_t cgroupMetrics { 0 };
      |                                      ^
      |                                      {}
...
/root/rpmbuild/BUILD/iSulad-v2.1.5/src/daemon/entry/cri/v1alpha/cri_pod_sandbox_manager_service.cc:1574:38: error: suggest braces around initialization of subobject [-Werror,-Wmissing-braces]
 1574 |     cgroup_metrics_t cgroupMetrics { 0 };
      |                                      ^
      |                                      {}
```

#### 修复过程

- 报错原因：对象初始化的时候缺失花括号，如：

  ```
  cgroup_metrics_t cgroupMetrics { 0 };
  ```

- 修复方式：将初始化中的`{ 0 }`改为`{{ 0 }}`

  ```
  cgroup_metrics_t cgroupMetrics {{ 0 }};
  ```

  

### 报错信息四

`error: format string is not a string literal (potentially insecure) [-Werror,-Wformat-security]`

```
/root/rpmbuild/BUILD/iSulad-v2.1.5/src/daemon/entry/cri/v1/v1_cri_pod_sandbox_manager_service.cc:508:15: error: format string is not a string literal (potentially insecure) [-Werror,-Wformat-security]
  508 |         ERROR(tmp_errmsg.c_str());
      |               ^~~~~~~~~~~~~~~~~~
      /usr/include/isula_libutils/log.h:356:22: note: expanded from macro 'ERROR'
  356 |         LXC_ERROR(&locinfo, format, ##__VA_ARGS__);                     \
      |                             ^~~~~~
/root/rpmbuild/BUILD/iSulad-v2.1.5/src/daemon/entry/cri/v1/v1_cri_pod_sandbox_manager_service.cc:508:15: note: treat the string as an argument to avoid this
  508 |         ERROR(tmp_errmsg.c_str());
      |               ^
      |               "%s", 
/usr/include/isula_libutils/log.h:356:22: note: expanded from macro 'ERROR'
  356 |         LXC_ERROR(&locinfo, format, ##__VA_ARGS__);                     \
      |                             ^
/root/rpmbuild/BUILD/iSulad-v2.1.5/src/daemon/entry/cri/v1/v1_cri_pod_sandbox_manager_service.cc:523:19: error: format string is not a string literal (potentially insecure) [-Werror,-Wformat-security]
  523 |             ERROR(list_response->errmsg);
      |                   ^~~~~~~~~~~~~~~~~~~~~
/usr/include/isula_libutils/log.h:356:22: note: expanded from macro 'ERROR'
  356 |         LXC_ERROR(&locinfo, format, ##__VA_ARGS__);                     \
      |                             ^~~~~~
/root/rpmbuild/BUILD/iSulad-v2.1.5/src/daemon/entry/cri/v1/v1_cri_pod_sandbox_manager_service.cc:523:19: note: treat the string as an argument to avoid this
  523 |             ERROR(list_response->errmsg);
      |                   ^
      |                   "%s", 
/usr/include/isula_libutils/log.h:356:22: note: expanded from macro 'ERROR'
  356 |         LXC_ERROR(&locinfo, format, ##__VA_ARGS__);                     \
      |                             ^
/root/rpmbuild/BUILD/iSulad-v2.1.5/src/daemon/entry/cri/v1/v1_cri_pod_sandbox_manager_service.cc:523:19: error: format string is not a string literal (potentially insecure) [-Werror,-Wformat-security]
  523 |             ERROR(list_response->errmsg);
      |                   ^~~~~~~~~~~~~~~~~~~~~
/usr/include/isula_libutils/log.h:356:22: note: expanded from macro 'ERROR'
  356 |         LXC_ERROR(&locinfo, format, ##__VA_ARGS__);                     \
      |                             ^~~~~~
/root/rpmbuild/BUILD/iSulad-v2.1.5/src/daemon/entry/cri/v1/v1_cri_pod_sandbox_manager_service.cc:523:19: note: treat the string as an argument to avoid this
  523 |             ERROR(list_response->errmsg);
      |                   ^
      |                   "%s", 
/usr/include/isula_libutils/log.h:356:22: note: expanded from macro 'ERROR'
  356 |         LXC_ERROR(&locinfo, format, ##__VA_ARGS__);                     \
      |                             ^
```

#### 修复过程

- 报错原因：两处传入字符串时未进行格式化，如：

  ```
  ERROR(tmp_errmsg.c_str());
  ```

- 修复方式：使用`%s`进行格式化：

  ```
  ERROR("%s", tmp_errmsg.c_str());
  ```



### 报错信息五

`error: cannot pass non-trivial object of type 'std::string' (aka 'basic_string<char>') to variadic function; expected type from format string was 'char *' [-Wnon-pod-varargs]`

```
/root/rpmbuild/BUILD/iSulad-v2.1.5/src/daemon/common/cri/cri_helpers.cc:528:92: error: cannot pass non-trivial object of type 'std::string' (aka 'basic_string<char>') to variadic function; expected type from format string was 'char *' [-Wnon-pod-varargs]
  528 |             SYSERROR("Failed to remove container %s log symlink %s.", containerID.c_str(), path);
      |                                                                 ~~                         ^~~~
/usr/include/isula_libutils/log.h:414:32: note: expanded from macro 'SYSERROR'
  414 |                 ERROR("%s - " format, ptr, ##__VA_ARGS__); \
      |                               ~~~~~~         ^~~~~~~~~~~
/usr/include/isula_libutils/log.h:356:32: note: expanded from macro 'ERROR'
  356 |         LXC_ERROR(&locinfo, format, ##__VA_ARGS__);                     \
      |                             ~~~~~~    ^~~~~~~~~~~
/root/rpmbuild/BUILD/iSulad-v2.1.5/src/daemon/common/cri/cri_helpers.cc:528:92: note: did you mean to call the c_str() method?
/root/rpmbuild/BUILD/iSulad-v2.1.5/src/daemon/common/cri/cri_helpers.cc:529:96: error: cannot pass object of non-trivial type 'std::string' (aka 'basic_string<char>') through variadic method; call will abort at runtime [-Wnon-pod-varargs]
  529 |             error.Errorf("Failed to remove container %s log symlink %s.", containerID.c_str(), path);
      |                                                                                                ^
```

#### 修复过程

- 报错原因：在尝试将 `std::string` 类型的对象作为可变参数列表传递给函数时遇到了报错。在c++中，`std::string` 是一个复杂的类类型，它包含内部状态和方法，因此不能直接作为变长参数传递给函数，除非该函数明确地知道如何处理 `std::string` 对象。对应代码比如：

  ```
  SYSERROR("Failed to remove container %s log symlink %s.", containerID.c_str(), path);
  ```

- 修复方式：加入`.c_str()`调用，如：

  ```
  SYSERROR("Failed to remove container %s log symlink %s.", containerID.c_str(), path.c_str());
  ```

  

### 报错信息六

`error: moving a temporary object prevents copy elision [-Werror,-Wpessimizing-move]`

```
/root/rpmbuild/BUILD/iSulad-v2.1.5/src/daemon/sandbox/sandbox_ops.cc:75:19: error: moving a temporary object prevents copy elision [-Werror,-Wpessimizing-move]
   75 |     params.spec = std::move(std::unique_ptr<std::string>(new std::string(oci_spec)));
      |                   ^
/root/rpmbuild/BUILD/iSulad-v2.1.5/src/daemon/sandbox/sandbox_ops.cc:75:19: note: remove std::move call here
   75 |     params.spec = std::move(std::unique_ptr<std::string>(new std::string(oci_spec)));
      |                   ^~~~~~~~~~~
```

#### 修复过程

- 报错原因：代码中试图移动一个临时对象，建议删除`std::move`，关联代码：

  ```
  params.spec = std::move(std::unique_ptr<std::string>(new std::string(oci_spec)));
  ```

- 修复方式：将此处的`std::move`移除：

  ```
  params.spec = std::unique_ptr<std::string>(new std::string(oci_spec));
  ```

  

### 报错信息七

`error: more '%' conversions than data arguments [-Werror,-Wformat-insufficient-args]`

```
/root/rpmbuild/BUILD/iSulad-v2.1.5/src/daemon/sandbox/sandbox.cc:850:41: error: more '%' conversions than data arguments [-Werror,-Wformat-insufficient-args]
  850 |         SYSERROR("Failed to write file %s");
      |                                        ~^
/usr/include/isula_libutils/log.h:414:17: note: expanded from macro 'SYSERROR'
  414 |                 ERROR("%s - " format, ptr, ##__VA_ARGS__); \
      |                               ^~~~~~
/usr/include/isula_libutils/log.h:356:22: note: expanded from macro 'ERROR'
  356 |         LXC_ERROR(&locinfo, format, ##__VA_ARGS__);                     \
      |                             ^~~~~~
```

#### 修复过程

- 报错原因：代码中出现多余的`%s`，具体代码为：

  ```
  SYSERROR("Failed to write file %s");
  ```

- 修复方式：根据代码语义，加入`%s`的对应字符串`path.c_str()`：

  ```
  SYSERROR("Failed to write file %s", path.c_str());
  ```

  

### 报错信息八

`error: private field 'm_enablePodEvents' is not used [-Werror,-Wunused-private-field]`

```
/root/rpmbuild/BUILD/iSulad-v2.1.5/src/daemon/entry/cri/v1/v1_cri_runtime_service_impl.h:107:10: error: private field 'm_enablePodEvents' is not used [-Werror,-Wunused-private-field]
  107 |     bool m_enablePodEvents;
      |          ^
1 error generated.
```

#### 修复过程

- 报错原因：私有变量`m_enablePodEvents`被认为是没被用到的，但实际上并不是完全没有用到。

- 修复方式：参考StackOverflow的帖子（[c++ - Good way to fix warning "field a is not used" if field is unused in configuration - Stack Overflow](https://stackoverflow.com/questions/48192418/good-way-to-fix-warning-field-a-is-not-used-if-field-is-unused-in-configuratio/48192493#48192493)），在声明之前添加了`[[maybe_unused]]`

  ```
  [[maybe_unused]] bool m_enablePodEvents;
  ```

  

