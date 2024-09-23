## alsa-tools

### 问题描述

#### 报错信息

修改前 clang 构建在 `echomixer.c` 出现多个类似下方片段的报错：

```
echomixer.c:2108:7: error: incompatible function pointer types passing 'void (GtkWidget *, gpointer)' (aka 'void (struct _GtkWidget *, void *)') to parameter of type 'GCallback' (aka 'void (*)(void)') [-Wincompatible-function-pointer-types]
  2108 |       gtk_signal_connect(GTK_OBJECT(menuitem), "activate", Digital_mode_activate, (gpointer)(long)i);
       |       ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
```

#### 相关PR

- 已有上游社区 PR：[Fix build failure on Clang 16 by mazunki · Pull Request #17 · alsa-project/alsa-tools (github.com)](https://github.com/alsa-project/alsa-tools/pull/17)；
- 已有上游社区 issue：[Build failure with upcoming Clang 16 (-Wincompatible-function-pointer-types) · Issue #12 · alsa-project/alsa-tools (github.com)](https://github.com/alsa-project/alsa-tools/issues/12)
- openEuler 仓库 PR：[Backport fix for clang build · Pull Request !32 · src-openEuler/alsa-tools - Gitee.com](https://gitee.com/src-openeuler/alsa-tools/pulls/32)

### 修复过程

- 回合上游社区PR。
- 修复内容为：以上文中片段为例，该报错的原因为`Digital_mode_activate`的类型并没有显式转换为`GCallback`，Clang 在类型检查方面相对于 GCC 更加严格，对于这种函数指针类型不兼容，GCC 不会默认处理为 error。解决方法则是将其显式转换为目标类型，比如：

  ```
  - gtk_signal_connect（GTK_OBJECT（menuitem）， “activate”， Digital_mode_activate， （gPointer）（long）i）;
  + gtk_signal_connect（GTK_OBJECT（menuitem）， “activate”， G_CALLBACK（Digital_mode_activate）， （gpointer）（long）i）;
  ```

