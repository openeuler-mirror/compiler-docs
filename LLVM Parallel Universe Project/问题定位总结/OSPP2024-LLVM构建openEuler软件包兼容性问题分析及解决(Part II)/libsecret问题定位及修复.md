来源：https://gitee.com/src-openeuler/libsecret (openEuler-24.03-LTS)

# 相关PR

openEuler：[fix `incompatible function pointer types passing`errors in test-vala-unstable.vala and test-vala-lang.vala · Pull Request !23 · src-openEuler/libsecret - Gitee.com](https://gitee.com/src-openeuler/libsecret/pulls/23)

上游社区：https://gitlab.gnome.org/GNOME/libsecret/-/issues/95

# 问题一：错误调用导致的incompatible function pointer types问题

## 1、问题现象

```c
[  103s] libsecret/test-vala-unstable.p/test-vala-unstable.c:129:59: error: incompatible function pointer types passing 'void (gpointer)' (aka 'void (void *)') to parameter of type 'GTestDataFunc' (aka 'void (*)(const void *)') [-Wincompatible-function-pointer-types]
```

## 2.1、问题定位

在test-vala-unstable.vala和test-vala-lang.vala中，错误性质相同，这里以test-vala-unstable.vala为例：

```vala
private void test_read_alias () {
	try {
		var service = Secret.Service.get_sync(Secret.ServiceFlags.NONE);
		var path = service.read_alias_dbus_path_sync("default", null);
		GLib.assert (path != null);
	} catch ( GLib.Error e ) {
		GLib.error (e.message);
	}
}

private static int main (string[] args) {
	GLib.Test.init (ref args);

	try {
		MockService.start ("mock-service-normal.py");
	} catch ( GLib.Error e ) {
		GLib.error ("Unable to start mock service: %s", e.message);
	}

	GLib.Test.add_data_func ("/vala/unstable/read-alias", test_read_alias);

	var res = GLib.Test.run ();
	Secret.Service.disconnect ();
	MockService.stop ();
	return res;
}
```

而GLib.Test.add_data_func实际上是调用glib库中的：

```c
void    g_test_add_data_func            (const char     *testpath,
                                         gconstpointer   test_data,
                                         GTestDataFunc   test_func);
```

GTestDataFunc的类型定义为：

```c
void (*)(gconstpointer)
```

而GLib.Test.add_func调用的是：

```c
void    g_test_add_func                 (const char     *testpath,
                                         GTestFunc       test_func);
```

GTestFunc的类型定义为：

```c
void (*)()
```

## 2.2、问题总结及根因确认

使用GLib.Test.add\_data\_func时，参数要求为(const char \*testpath,gconstpointer test\_data,GTestDataFunc test\_func)，实际为(const char \*testpath,GTestFunc test\_func)，参数不匹配。

## 3、修改建议

改为使用GLib.Test.add\_func，其参数要求为(const char \*testpath,GTestFunc test\_func)，符合实际传入的参数