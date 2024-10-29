来源：https://gitee.com/src-openeuler/proftpd (openEuler-24.03-LTS)

# 相关PR

openEuler：[给函数pr_ctrls_recv_request加上 __attribute__((optnone))，使得LLVM能够成功构建 · Pull Request !70 · src-openEuler/proftpd - Gitee.com](https://gitee.com/src-openeuler/proftpd/pulls/70)

上游社区：不涉及（上游社区已经弃用有问题的代码，但是由于别的原因弃用的）

# 问题：编译优化选项导致即使if条件为真但不执行

## 1、问题现象

clang不通过原因是，一处test结果FAILURE（没有按照预期处理特定情况）：

```c
[  254s] api/ctrls.c:883:F:base:ctrls_recv_request_actions_test:0: Failed to handle too-long reqarg
[  254s] -------------------------------------------------
[  254s]  FAILED 1 test
[  254s] 
[  254s]  Please send email to:
[  254s] 
[  254s]    proftp-devel@lists.sourceforge.net
[  254s] 
[  254s]  containing the `api-tests.log' file (in the tests/ directory)
[  254s]  and the output from running `proftpd -V'
```

## 2、问题定位与分析

### 2.1、问题定位

api/ctrls.c具体报错代码片段，位于START_TEST (ctrls_recv_request_actions_test){...}中：

```c
mark_point();
fd = reset_fd(fd);
if (fd < 0) {
  return;
}
clear_array(cl->cl_ctrls);
cl->cl_fd = fd;

reqarglen = 500;
(void) write(fd, &status, sizeof(status));
(void) write(fd, &nreqargs, sizeof(nreqargs));
(void) write(fd, &actionlen, sizeof(actionlen));
(void) write(fd, action, actionlen);
(void) write(fd, &reqarglen, sizeof(reqarglen));
rewind_fd(fd);
res = pr_ctrls_recv_request(cl);
ck_assert_msg(res < 0, "Failed to handle too-long reqarg");
ck_assert_msg(errno == ENOMEM, "Expected ENOMEM (%d), got %s (%d)", ENOMEM,
  strerror(errno), errno);
ck_assert_msg(cl->cl_ctrls->nelts == 0, "Expected 0 ctrl, got %d",
  cl->cl_ctrls->nelts);
```

ck_assert_msg断言了res<0，否则就会提示"Failed to handle too-long reqarg"，与此同时，还断言了errno==ENOMEM（12）。


而pr_ctrls_recv_request函数体内能让errno==ENOMEM的只有三处：

```c
  if (nreqargs > CTRLS_MAX_NREQARGS) {
    (void) pr_trace_msg(trace_channel, 3,
      "nreqargs (%u) exceeds max (%u), rejecting", nreqargs,
      CTRLS_MAX_NREQARGS);
    pr_signals_unblock();
    errno = ENOMEM;
    return -1;
  }
```

```c
  if (reqarglen >= sizeof(reqaction)) {
    pr_signals_unblock();
    errno = ENOMEM;
    return -1;
  }
```

```c
    if (reqarglen > CTRLS_MAX_REQARGLEN) {
      (void) pr_trace_msg(trace_channel, 3,
        "reqarglen (#%u) of %u bytes exceeds max (%u bytes), rejecting",
        i+1, reqarglen, CTRLS_MAX_REQARGLEN);
      pr_signals_unblock();
      errno = ENOMEM;
      return -1;
    }
```

我利用ck_assert_msg进行了如下测试：

#### 测试一：具体是哪一行出错的？

```c
+  errno=0;
+  res=10;
   reqarglen = 500;
   (void) write(fd, &status, sizeof(status));
   (void) write(fd, &nreqargs, sizeof(nreqargs));
@@ -880,12 +882,13 @@ START_TEST (ctrls_recv_request_actions_test) {
   (void) write(fd, &reqarglen, sizeof(reqarglen));
   rewind_fd(fd);
+  ck_assert_msg(res < 0, "Failed to handle too-long reqarg,res:%d,errno:%d(%s)",res,errno,strerror(errno));
   res = pr_ctrls_recv_request(cl);
-  ck_assert_msg(res < 0, "Failed to handle too-long reqarg");
   ck_assert_msg(errno == ENOMEM, "Expected ENOMEM (%d), got %s (%d)", ENOMEM,
     strerror(errno), errno);
   ck_assert_msg(cl->cl_ctrls->nelts == 0, "Expected 0 ctrl, got %d",
     cl->cl_ctrls->nelts);
```

```c
[  216s] api/ctrls.c:884:F:base:ctrls_recv_request_actions_test:0: Failed to handle too-long reqarg,res:10,errno:0(Success)
```

将断言放到`res = pr_ctrls_recv_request(cl);`之前一句，errno和res都还是我手动初始化的值，这说明了错误就只是在`res = pr_ctrls_recv_request(cl);`产生的。

#### 测试二：出错时res和errno分别为什么值？

让ck_assert_msg额外打印断言失败时，res的值与errno的值，发现执行完`res = pr_ctrls_recv_request(cl);`后res为0,而非预期的<0，errno为1(Operation not permitted)，而非预期的12（ENOMEM）

**这说明了：代码并没有运行进入上面那三处能让errno==ENOMEM的控制语句内，不然res一定返回-1；**

#### 测试三：为什么没有进入控制语句内？

经过一些调试，我最终将问题范围缩小到了if (reqarglen > CTRLS_MAX_REQARGLEN){...}这一语句块内

* 测试在if (reqarglen > CTRLS_MAX_REQARGLEN) 之前reqarglen的值，**结果：500**

```c
+    return reqarglen;
     if (reqarglen > CTRLS_MAX_REQARGLEN) {
```

* 测试在if (reqarglen > CTRLS_MAX_REQARGLEN) 之前CTRLS_MAX_REQARGLE，**结果：256**

```c
+    return CTRLS_MAX_REQARGLEN;
     if (reqarglen > CTRLS_MAX_REQARGLEN) {
```

* 测试if (reqarglen > CTRLS_MAX_REQARGLEN)之前reqarglen > CTRLS_MAX_REQARGLEN的值，**结果：1**

```c
+    return reqarglen > CTRLS_MAX_REQARGLEN;
     if (reqarglen > CTRLS_MAX_REQARGLEN) {
```

* 如果进入了控制语句，则令其立即返回(reqarglen > CTRLS_MAX_REQARGLEN)+1000，**预期结果为1001，实际结果为0**，已知函数pr_ctrls_recv_request如果没有进入任何if控制语句内，最后会返回0，这说明即使在上述已知reqarglen > CTRLS_MAX_REQARGLEN条件下，依然没有进入if控制语句内。

```c
+  
     if (reqarglen > CTRLS_MAX_REQARGLEN) {
-      (void) pr_trace_msg(trace_channel, 3,
-        "reqarglen (#%u) of %u bytes exceeds max (%u bytes), rejecting",
-        i+1, reqarglen, CTRLS_MAX_REQARGLEN);
-      pr_signals_unblock();
+      return (reqarglen > CTRLS_MAX_REQARGLEN)+1000;
+      //(void) pr_trace_msg(trace_channel, 3,
+      //  "reqarglen (#%u) of %u bytes exceeds max (%u bytes), rejecting",
+      //  i+1, reqarglen, CTRLS_MAX_REQARGLEN);
+      //  return CTRLS_MAX_REQARGLEN+1000;
+      //pr_signals_unblock();
       errno = ENOMEM;
       return -1;
     }
```

```c
[  206s] api/ctrls.c:885:F:base:ctrls_recv_request_actions_test:0: Failed to handle too-long reqarg,res:0,errno:1(Operation not permitted)
```

### 2.2、问题分析

#### 经过查阅资料发现，该问题很可能为优化选项导致的

尝试给函数pr_ctrls_recv_request加上 __attribute__((optnone))，即改为`int __attribute__((optnone)) pr_ctrls_recv_request(pr_ctrls_cl_t *cl) {...}`,加上之后成功构建！

#### 进一步分析

一般这种优化导致的问题把编译命令拿出来看下汇编IR会比较清晰，如果想要进一步分析具体是哪个优化选项使得if控制失效，还需要检查每个优化pass之后的IR。由于目前本人在该方面的技术与知识并不熟悉，因此将进一步分析的工作暂时搁置。

### 2.3、问题总结及根因确认

LLVM优化选项导致函数pr_ctrls_recv_request中的一处if控制语句失效，即在即使满足条件的情况下，依然未执行相应的语句。给函数pr_ctrls_recv_request加上 __attribute__((optnone))，即改为`int __attribute__((optnone)) pr_ctrls_recv_request(pr_ctrls_cl_t *cl) {...}`之后成功构建。

## 3、修改建议

给函数pr_ctrls_recv_request加上 __attribute__((optnone))，即改为`int __attribute__((optnone)) pr_ctrls_recv_request(pr_ctrls_cl_t *cl) {...}`。