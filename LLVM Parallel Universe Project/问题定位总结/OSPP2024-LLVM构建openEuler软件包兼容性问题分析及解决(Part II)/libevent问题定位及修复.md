来源：https://gitee.com/src-openeuler/libevent (openEuler-24.03-LTS)

# 相关PR

openEuler：[Fix function undeclared,incompatible pointer and parameter lack in &apos;add-testcases-for-event.c-apis.patch&apos;,support clang build · Pull Request !74 · src-openEuler/libevent - Gitee.com](https://gitee.com/src-openeuler/libevent/pulls/74)

上游社区：不涉及

# 问题一：call to undeclared function问题

## 1、问题现象

`[  705s] test/regress.c:3689:12: error: call to undeclared function 'event_base_start_iocp_'; ISO C99 and later do not support implicit function declarations [-Wimplicit-function-declaration]`

`[  705s] test/regress.c:3691:2: error: call to undeclared function 'event_base_stop_iocp_'; ISO C99 and later do not support implicit function declarations [-Wimplicit-function-declaration]`

## 2、问题定位

### 2.1、问题定位分析

[add-testcases-for-event.c-apis.patch](https://gitee.com/src-openeuler/libevent/blob/6fd712438bafd9eafdb3cf3b3e9d66913378b2a3/add-testcases-for-event.c-apis.patch)是对regress.c的patch，该patch中调用了函数`event_base_start_iocp_`与`event_base_stop_iocp_`但忘记了#include其所在的头文件"iocp-internal.h"

### 2.2 gcc与clang对于调用未声明函数的差异处理

对于调用未声明的函数，c语言曾有一种叫隐式声明的特性，能规避编译构建的报错，然而该特性在C99以后已经被移除，clang17默认执行gnu17标准（见[官方文档](https://clang.llvm.org/docs/UsersManual.html#extensions-supported-by-clang)），因此不支持。

![gcc遇到未声明函数时的表现](https://foruda.gitee.com/images/1725034630272359183/577103f3_13034847.png "屏幕截图")

### 2.3、问题总结及根因确认

[add-testcases-for-event.c-apis.patch](https://gitee.com/src-openeuler/libevent/blob/6fd712438bafd9eafdb3cf3b3e9d66913378b2a3/add-testcases-for-event.c-apis.patch)是对regress.c的patch，该patch中调用了函数`event_base_start_iocp_`与`event_base_stop_iocp_`但忘记了#include其所在的头文件"iocp-internal.h"，gcc对于调用未声明函数会使用隐式声明，仅仅warning，而clang会报error

## 3、修改建议

补充打一个patch，在regress.c中#include "iocp-internal.h"

# 问题二：call, expected 2, have 1问题

## 1、问题现象

```c
[  142s] test/regress.c:3691:39: error:  call, expected 2, have 1
[  142s]  3691 |         int res = event_base_start_iocp_(base);
[  142s]       |                   ~~~~~~~~~~~~~~~~~~~~~~     ^
[  142s] ./iocp-internal.h:197:5: note: 'event_base_start_iocp_' declared here
[  142s]   197 | int event_base_start_iocp_(struct event_base *base, int n_cpus);
[  142s]       |     ^   
```

## 2、问题定位

### 2.1、问题定位分析

错误出处：`int res = event_base_start_iocp_(base);`

查找`event_base_start_iocp_`函数的定义，在event.c中：

```c
int
event_base_start_iocp_(struct event_base *base, int n_cpus)
{
#ifdef _WIN32
	if (base->iocp)
		return 0;
	base->iocp = event_iocp_port_launch_(n_cpus);
	if (!base->iocp) {
		event_warnx("%s: Couldn't launch IOCP", __func__);
		return -1;
	}
	return 0;
#else
	return -1;
#endif
}
```

可见，该函数要求两个参数，而该处错误只传入了一个参数

![输入图片说明](https://foruda.gitee.com/images/1725034687466247382/59733cb3_13034847.png "屏幕截图")

综合其他所有地方的用处来看，参数改为(base,0)是保险的修复方法

### 2.2、问题总结及根因确认

调用函数时，传入的参数的数量不符合预期，要求2，实际1

## 3、修改建议

使用合理的参数补全参数个数要求，即将int res = event_base_start_iocp_(base);改为int res = event_base_start_iocp_(base,0);

# 问题三：错误调用导致的incompatible function pointer types问题

## 1、问题现象

```c
[  140s] test/regress.c:3707:2: error: incompatible function pointer types initializing 'testcase_fn' (aka 'void (*)(void *)') with an expression of type 'void (void)' [-Wincompatible-function-pointer-types]
[  140s]  3707 |         BASIC(event_config_set_max_dispatch_interval, TT_FORK|TT_NEED_BASE),
[  140s]       |         ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[  140s] test/regress.h:105:11: note: expanded from macro 'BASIC'
[  140s]   105 |         { #name, test_## name, flags, &basic_setup, NULL }
[  140s]       |                  ^~~~~~~~~~~~
[  140s] <scratch space>:168:1: note: expanded from here
[  140s]   168 | test_event_config_set_max_dispatch_interval
[  140s]       | ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[  140s] test/regress.c:3708:2: error: incompatible function pointer types initializing 'testcase_fn' (aka 'void (*)(void *)') with an expression of type 'void (void)' [-Wincompatible-function-pointer-types]
[  140s]  3708 |         BASIC(event_config_set_num_cpus_hint, TT_FORK|TT_NEED_BASE),
[  140s]       |         ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[  140s] test/regress.h:105:11: note: expanded from macro 'BASIC'
[  140s]   105 |         { #name, test_## name, flags, &basic_setup, NULL }
[  140s]       |                  ^~~~~~~~~~~~
[  140s] <scratch space>:170:1: note: expanded from here
[  140s]   170 | test_event_config_set_num_cpus_hint
[  140s]       | ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[  140s] 2 errors generated.
```

## 2、问题定位

### 2.1、问题定位分析

错误定位：

```c
struct testcase_t main_testcases[] = {
+	/* event.c api tests */
+	BASIC(event_base_del_virtual_, TT_FORK|TT_NEED_BASE),
+	BASIC(event_deferred_cb_set_priority_, TT_FORK|TT_NEED_BASE),
+	BASIC(event_callback_init_, TT_FORK|TT_NEED_BASE),
+	BASIC(event_del_noblock, TT_FORK|TT_NEED_BASE),
+	BASIC(event_del_block, TT_FORK|TT_NEED_BASE),
+	BASIC(event_get_events, TT_FORK|TT_NEED_BASE|TT_NEED_SOCKETPAIR),
+	BASIC(event_config_set_max_dispatch_interval, TT_FORK|TT_NEED_BASE),
+	BASIC(event_config_set_num_cpus_hint, TT_FORK|TT_NEED_BASE),
+	BASIC(event_base_stop_iocp_, TT_FORK|TT_NEED_BASE),
```

main_testcases[]的元素类型为struct testcase_t，而struct testcase_t的定义为：

```c
/** A single test-case that you can run. */
struct testcase_t {
	const char *name; /**< An identifier for this case. */
	testcase_fn fn; /**< The function to run to implement this case. */
	unsigned long flags; /**< Bitfield of TT_* flags. */
	const struct testcase_setup_t *setup; /**< Optional setup/cleanup fns*/
	void *setup_data; /**< Extra data usable by setup function */
};
```

错误就出现在第二个元素testcase_fn fn，其类型定义为：

```c
typedef void (*testcase_fn)(void *);
```

也就是说，要求类型为void\*(void\*)

我们再来看看BASIC是什么，他是一个宏，定义如下，同时还需要注意，与他一同定义的LEGACY宏

```c
#define BASIC(name,flags)						\
	{ #name, test_## name, flags, &basic_setup, NULL }

#define LEGACY(name,flags)						\
	{ #name, run_legacy_test_fn, flags|TT_LEGACY, &legacy_setup,	\
	  test_## name }
```

我们再回来看一开始报错的地方

```c
+	BASIC(event_config_set_max_dispatch_interval, TT_FORK|TT_NEED_BASE),
+	BASIC(event_config_set_num_cpus_hint, TT_FORK|TT_NEED_BASE),
```

也就是说，问题出在函数test_event_config_set_max_dispatch_interval和test_event_config_set_num_cpus_hint的类型不符合void\*(void\*)的要求，那我们来看看这两个函数的定义：

```c
+static void
+test_event_config_set_max_dispatch_interval(void)
+{
+	struct event_config *cfg = NULL;
+	struct timeval tv;
+	int max_dispatch_cbs = 100;
+	int min_priority = 2;
+
+	evutil_timerclear(&tv);
+	tv.tv_sec = 3;
+	tv.tv_usec = 0;
+
+	cfg = event_config_new();
+	event_config_set_max_dispatch_interval(cfg, &tv, max_dispatch_cbs, min_priority);
+
+	tt_assert(max_dispatch_cbs == cfg->max_dispatch_callbacks);
+	tt_assert(min_priority == cfg->limit_callbacks_after_prio);
+	tt_assert(3 == cfg->max_dispatch_interval.tv_sec);
+	tt_assert(0 == cfg->max_dispatch_interval.tv_usec);
+end:
+	if (cfg)
+		event_config_free(cfg);
+}
+
+static void
+test_event_config_set_num_cpus_hint(void)
+{
+	struct event_config *cfg = NULL;
+	int n_cpus = 4;
+
+	cfg = event_config_new();
+	event_config_set_num_cpus_hint(cfg, 4);
+	tt_assert(4 == cfg->n_cpus_hint);
+end:
+	if (cfg)
+		event_config_free(cfg);
+}
```

可见他们的类型是void(void)

除了这两个函数之外，其他BASIC宏传入的函数（没有被报错的）的类型例如下：

```c
+static void
+test_event_del_noblock(void *ptr) {
+	struct basic_test_data *data = ptr;
+	struct event_base *base = data->base;
+	struct timeval tv;
+	struct event ev;
+	int count = 0;
+	int res_del = 0;
+
+	evutil_timerclear(&tv);
+	tv.tv_usec = 10000;
+
+	event_assign(&ev, base, -1, EV_TIMEOUT|EV_PERSIST,
+	    timeout_cb, &count);
+	event_add(&ev, &tv);
+
+	res_del = event_del_noblock(&ev);
+	tt_int_op(res_del, ==, 0);
+end:
+	;
+}
+
```

修改方法显然不能直接改参数要求，如果将void改为void*，那么凭空要求传入一个参数，显然不对，那么还有其他方法吗？我在上游社区没有发现类似的issue，然而，我们可以去找找看，对于void(void)类型的函数，上游社区是如何将其传入testcase的，我们大致看一部分：

```c
struct testcase_t main_testcases[] = {
	/* Some converted-over tests */
	{ "methods", test_methods, TT_FORK, NULL, NULL },
	{ "version", test_version, 0, NULL, NULL },
	BASIC(base_features, TT_FORK|TT_NO_LOGS),
	{ "base_environ", test_base_environ, TT_FORK, NULL, NULL },

	BASIC(event_base_new, TT_FORK|TT_NEED_SOCKETPAIR),
	BASIC(free_active_base, TT_FORK|TT_NEED_SOCKETPAIR),

	BASIC(manipulate_active_events, TT_FORK|TT_NEED_BASE),
	BASIC(event_new_selfarg, TT_FORK|TT_NEED_BASE),
	BASIC(event_assign_selfarg, TT_FORK|TT_NEED_BASE),
	BASIC(event_base_get_num_events, TT_FORK|TT_NEED_BASE),
	BASIC(event_base_get_max_events, TT_FORK|TT_NEED_BASE),
	BASIC(evmap_invalid_slots, TT_FORK|TT_NEED_BASE),

	BASIC(bad_assign, TT_FORK|TT_NEED_BASE|TT_NO_LOGS),
	BASIC(bad_reentrant, TT_FORK|TT_NEED_BASE|TT_NO_LOGS),
	BASIC(active_later, TT_FORK|TT_NEED_BASE|TT_NEED_SOCKETPAIR|TT_RETRIABLE),
	BASIC(event_remove_timeout, TT_FORK|TT_NEED_BASE|TT_NEED_SOCKETPAIR),

	/* These are still using the old API */
	LEGACY(persistent_timeout, TT_FORK|TT_NEED_BASE),
	{ "persistent_timeout_jump", test_persistent_timeout_jump, TT_FORK|TT_NEED_BASE, &basic_setup, NULL },
	{ "persistent_active_timeout", test_persistent_active_timeout,
	  TT_FORK|TT_NEED_BASE|TT_RETRIABLE, &basic_setup, NULL },
	LEGACY(priorities, TT_FORK|TT_NEED_BASE),
	BASIC(priority_active_inversion, TT_FORK|TT_NEED_BASE),
	{ "common_timeout", test_common_timeout, TT_FORK|TT_NEED_BASE|TT_RETRIABLE,
	  &basic_setup, NULL },

	/* These legacy tests may not all need all of these flags. */
	LEGACY(simpleread, TT_ISOLATED),
	LEGACY(simpleread_multiple, TT_ISOLATED),
	LEGACY(simplewrite, TT_ISOLATED),
	{ "simpleclose_rw", test_simpleclose_rw, TT_FORK, &basic_setup, NULL },
	/* simpleclose */
	{ "simpleclose_close", test_simpleclose,
	  TT_FORK|TT_NEED_SOCKETPAIR|TT_NEED_BASE,
	  &basic_setup, (void *)"close" },
	{ "simpleclose_shutdown", test_simpleclose,
	  TT_FORK|TT_NEED_SOCKETPAIR|TT_NEED_BASE,
	  &basic_setup, (void *)"shutdown" },
```

可见，除了BASIC，有时也用LEGACY，那么什么时候用BASIC，什么时候用LEGACY呢？本人查看了上文的所有testcase中的测试函数，发现BASIC用于传入void(void*)，而LEGACY用于传入void(void)，全部贴上来篇幅过长，因此此处只放一个例子：

```c
LEGACY(persistent_timeout, TT_FORK|TT_NEED_BASE)
```

```c
static void
test_persistent_timeout(void)
{
	struct timeval tv;
	struct event ev;
	int count = 0;

	evutil_timerclear(&tv);
	tv.tv_usec = 10000;

	event_assign(&ev, global_base, -1, EV_TIMEOUT|EV_PERSIST,
	    periodic_timeout_cb, &count);
	event_add(&ev, &tv);

	event_dispatch();

	event_del(&ev);
}
```

### 2.2、问题总结及根因确认

[add-testcases-for-event.c-apis.patch](https://gitee.com/src-openeuler/libevent/blob/6fd712438bafd9eafdb3cf3b3e9d66913378b2a3/add-testcases-for-event.c-apis.patch)该patch试图增加对一些函数的testcase，对于void(void)类型的函数，正确的方法是调用LEGACY宏将函数加入测试行列，错误在于调用了BASIC宏（其用于void(void*)类型的函数）

## 3、修改建议

把报错两处的BASIC改为LEGACY

# 问题四：autoreconf: command not found问题

## 1、问题现象

出自：https://gitee.com/src-openeuler/libevent (openEuler-24.03-LTS)

`[  119s] /var/tmp/rpm-tmp.PicZ3O: line 47: autoreconf: command not found`

## 2、问题成因

rpm打包时使用了autoreconf命令，但是在BuildRequires中没有写该命令的软件依赖

## 3、修改建议

在spec文件的BuildRequires中添加autoconf automake libtool