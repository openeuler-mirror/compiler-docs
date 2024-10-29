来源：https://gitee.com/src-openeuler/opensc (openEuler-24.03-LTS)

# 相关PR

openEuler：[backport 1 commit，fix clang build error · Pull Request !79 · src-openEuler/opensc - Gitee.com](https://gitee.com/src-openeuler/opensc/pulls/79)

上游社区：https://github.com/OpenSC/OpenSC/commit/3f485ff28cecf0cdddecef369442df2efed80a99

openEuler社区会默认将这个error降级为warning，实际情况中不影响构建。

# 问题一：unused-but-set-variable问题

## 1、问题现象

```c
[  117s] pkcs11-tool.c:7304:6: error: variable 'errors' set but not used [-Werror,-Wunused-but-set-variable]
[  117s]  7304 |         int errors = 0;
[  117s]       |             ^
[  117s] 1 error generated.
[  117s] make[3]: *** [Makefile:1356: pkcs11_tool-pkcs11-tool.o] Error 1
[  117s] make[3]: *** Waiting for unfinished jobs....
[  117s] cardos-tool.c:1146:6: error: variable 'action_count' set but not used [-Werror,-Wunused-but-set-variable]
[  117s]  1146 |         int action_count = 0;
[  117s]       |             ^
[  117s] 1 error generated.
```

## 2、问题根因分析

存在定义了但是未使用的变量，clang开启Werror将`unused-but-set-variable`当做error

## 3、修改建议

方案一：

删去没有被具体使用的变量，不会产生实际的影响

方案二：

可以添加-Wno-error=unused-but-set-variable将error降级为warning，使llvm能够构建

# 问题二：passing arguments to a function without a prototype问题

## 1、问题现象

```c
[  100s] card-openpgp.c:1164:7: error: passing arguments to a function without a prototype is deprecated in all versions of C and is not supported in C2x [-Werror,-Wdeprecated-non-prototype]
[  100s]  1164 |                 func(blob);
[  100s]       |                     ^
[  100s] 1 error generated.
```

## 2、问题定位

```c
/**
 * Internal: iterate through the blob tree, calling a function for each blob.
 */
static void
pgp_iterate_blobs(pgp_blob_t *blob, void (*func)())
{
	if (blob) {
		pgp_blob_t *child = blob->files;

		while (child != NULL) {
			pgp_blob_t *next = child->next;

			pgp_iterate_blobs(child, func);
			child = next;
		}
		func(blob);
	}
}
```

参数列表里func应该是一个不接受任何参数的函数的指针，而实际运用上传入了blob

上游commit：

[pgp: avoid calling functions without prototype · OpenSC/OpenSC@3f485ff (github.com)](https://github.com/OpenSC/OpenSC/commit/3f485ff28cecf0cdddecef369442df2efed80a99)

更改了代码逻辑，弃用`pgp_iterate_blobs`函数，改为使用`pgp_free_blobs`函数

## 3、修改建议

回合上游commit，消除错误：

[pgp: avoid calling functions without prototype · OpenSC/OpenSC@3f485ff (github.com)](https://github.com/OpenSC/OpenSC/commit/3f485ff28cecf0cdddecef369442df2efed80a99)