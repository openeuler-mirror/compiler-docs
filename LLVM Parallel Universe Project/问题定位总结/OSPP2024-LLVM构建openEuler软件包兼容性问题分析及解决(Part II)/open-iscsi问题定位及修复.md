来源：https://gitee.com/src-openeuler/open-iscsi (openEuler-24.03-LTS)

# 相关PR

openEuler：[Backport:fix several bugs, support clang build · Pull Request !162 · src-openEuler/open-iscsi - Gitee.com](https://gitee.com/src-openeuler/open-iscsi/pulls/162)

上游社区：[fix 4 issues which are finded when building with clang 17 by yuncang123 · Pull Request #478 · open-iscsi/open-iscsi (github.com)](https://github.com/open-iscsi/open-iscsi/pull/478)

[fix: add usr/iscsid_req.h missinig underline (#431) by hanqingwu · Pull Request #436 · open-iscsi/open-iscsi (github.com)](https://github.com/open-iscsi/open-iscsi/pull/436)

# 问题一：'linker' input unused问题

## 1、问题现象

```c
[  102s] clang: error: -Wl,-z,relro: 'linker' input unused [-Werror,-Wunused-command-line-argument]
[  102s] clang: error: -Wl,-z,now: 'linker' input unused [-Werror,-Wunused-command-line-argument]
[  102s] clang: error: -lkmod: 'linker' input unused [-Werror,-Wunused-command-line-argument]
```

## 2、问题定位

open-iscsi.spec中：

`%make_build OPTFLAGS="%{optflags} %{?__global_ldflags} -DUSE_KMOD -lkmod" LIB_DIR=%{_libdir}`

在open-iscsi-2.1.5/Makefile中：

```c
# Compatibility: parse old OPTFLAGS argument
ifdef OPTFLAGS
CFLAGS = $(OPTFLAGS)
endif
```


也就是将`%{?__global_ldflags}`（即-Wl,-z,relro  -Wl,-z,now）、`-lkmod`等链接器（linker）选项设置给了编译器CFLAGS，这在clang看来是多余的、无用的

## 3、修改建议

当toolchain为clang时，将`%{?__global_ldflags}`（即-Wl,-z,relro  -Wl,-z,now）、`-lkmod`等链接器（linker）选项设置给LDFLAGS

即将

```c
%make_build OPTFLAGS="%{optflags} %{?__global_ldflags} -DUSE_KMOD -lkmod" LIB_DIR=%{_libdir}
```

改为：

```c
%if "%toolchain"=="clang"
CFLAGS="%{optflags} -DUSE_KMOD" \
LDFLAGS="%{?__global_ldflags} -lkmod" \
LIB_DIR=%{_libdir}
%make_build 
%else
%make_build OPTFLAGS="%{optflags} %{?__global_ldflags} -DUSE_KMOD -lkmod" LIB_DIR=%{_libdir}
%endif
```

# 问题二：header guard followed by #define of a different macro问题

## 1、问题现象

```c
[   89s] In file included from session_info.c:19:
[   89s] ./iscsid_req.h:21:9: error: 'ISCSID_REQ_H_' is used as a header guard here, followed by #define of a different macro [-Werror,-Wheader-guard]
[   89s]    21 | #ifndef ISCSID_REQ_H_
[   89s]       |         ^~~~~~~~~~~~~
[   89s] ./iscsid_req.h:22:9: note: 'ISCSID_REQ_H' is defined here; did you mean 'ISCSID_REQ_H_'?
[   89s]    22 | #define ISCSID_REQ_H
[   89s]       |         ^~~~~~~~~~~~
[   89s]       |         ISCSID_REQ_H_
[   89s] 1 error generated.
```

## 2、问题成因

前后名字不一致，少了个_，clang语法检测更加严格，因此发现问题

## 3、修改建议

回合上游commit：[fix: add usr/iscsid_req.h missinig underline (#431) · hanqingwu/open-iscsi@2989724 (github.com)](https://github.com/hanqingwu/open-iscsi/commit/29897249ab77739f6b233677d761e24705b33f6a)

# 问题三：declaration will not be visible outside of this function问题

## 1、问题现象

```c
[   95s] In file included from discoveryd.c:31:
[   95s] ./iface.h:61:29: error: declaration of 'struct iovec' will not be visible outside of this function [-Werror,-Wvisibility]
[   95s]    61 |                                   int iface_all, struct iovec *iovs);
[   95s]       |                                                         ^
[   95s] 1 error generated.
```

## 2、问题定位

iface.h

```c
extern int iface_build_net_config(struct iface_rec *iface_primary,
				  int iface_all, struct iovec *iovs);
```

经过查询，struct iovec的定义在系统库 <sys/uio.h>中，而iface.h中既没有对struct iovec的定义，也没有包含该头文件

## 3、修改建议

在iface.h头文件中#include<sys/uio.h>

# 问题三：attribute declaration must precede definition问题

## 1、问题现象

```c
[   93s] In file included from io.c:41:
[   93s] In file included from ./iface.h:23:
[   93s] In file included from ../libopeniscsiusr/libopeniscsiusr/libopeniscsiusr.h:30:
[   93s] ../libopeniscsiusr/libopeniscsiusr/libopeniscsiusr_common.h:75:8: error: attribute declaration must precede definition [-Werror,-Wignored-attributes]
[   93s]    75 | struct __DLL_EXPORT iscsi_session;
[   93s]       |        ^
[   93s] ../libopeniscsiusr/libopeniscsiusr/libopeniscsiusr_common.h:64:38: note: expanded from macro '__DLL_EXPORT'
[   93s]    64 | #define __DLL_EXPORT    __attribute__ ((visibility ("default")))
[   93s]       |                                         ^
[   93s] ./initiator.h:198:16: note: previous definition is here
[   93s]   198 | typedef struct iscsi_session {
[   93s]       |                ^
```

## 2、问题定位

### 2.1、问题定位分析

网上搜索该问题，发现问题出在头文件的顺序上

头文件"iface.h"中声明了struct iscsi_session且声明了__DLL_EXPORT（该宏展开为\_\_attribute\_\_ ((visibility ("default")))），"initiator.h"中定义了struct iscsi_session，clang要求"attribute declaration must precede definition"即带有attribute的声明要先于其定义

以io.c为例

```c
...
#include "types.h"
#include "iscsi_proto.h"
#include "iscsi_settings.h"
#include "initiator.h"
#include "iscsi_ipc.h"
#include "log.h"
#include "transport.h"
#include "idbm.h"
#include "iface.h"
#include "sysdeps.h"
...
```

问题就出在\#include "initiator.h"和\#include "iface.h"的顺序上

只要将\#include "iface.h"移动到\#include "initiator.h"之前就能解决报错

### 2.2、问题总结及根因确认

clang要求"attribute declaration must precede definition"即带有attribute的声明必须先于定义，在该问题情境中体现为头文件的顺序，\#include "iface.h"要先于\#include "initiator.h"

## 3、修改建议

通过移动头文件顺序的方式，使带有attribute的声明先于其定义

# 问题四：implicit truncation from 'int' to a one-bit wide bit-field changes value from 1 to -1问题

## 1、问题现象

```c
[   88s] mgmt_ipc.c:511:19: error: implicit truncation from 'int' to a one-bit wide bit-field changes value from 1 to -1 [-Werror,-Wsingle-bit-bitfield-constant-conversion]
[   88s]   511 |         qtask->allocated = 1;
[   88s]       |                          ^ ~
[   88s] 1 error generated.
```

## 2、问题定位

### 2.1、问题定位分析

qtask的类型为queue_task_t，而queue_task_t的定义在usr/initiator.h中：

```c
typedef struct queue_task {
	iscsi_conn_t *conn;
	iscsiadm_req_t req;
	iscsiadm_rsp_t rsp;
	int mgmt_ipc_fd;
	int allocated : 1;
	/* Newer request types include a
	 * variable-length payload */
	void *payload;
} queue_task_t;
```

这个错误是由于在代码中存在一个隐式截断问题。试图将一个 `int` 类型的值（这里是 `1`）赋给一个一位宽的位域（allocated），但由于位域的宽度为1位，这样的操作可能会导致值从`1`被截断为`-1`

### 2.2、关于位域（bit-field）

有些信息在存储时，并不需要占用一个完整的字节，而只需占几个或一个二进制位。例如在存放一个开关量时，只有 0 和 1 两种状态，用 1 位二进位即可。为了节省存储空间，并使处理简便，C 语言又提供了一种数据结构，称为"位域"或"位段"。

所谓"位域"是把一个字节中的二进位划分为几个不同的区域，并说明每个区域的位数。每个域有一个域名，允许在程序中按域名进行操作。这样就可以把几个不同的对象用一个字节的二进制位域来表示。

详见：[C 位域 | 菜鸟教程 (runoob.com)](https://www.runoob.com/cprogramming/c-bit-fields.html)

这里的allocated本意是利用位域来节省空间，只有0和1两种状态，但是错误地将其类型设定为了int，这就会出现隐式截断的问题，正确的做法是将其类型设定为unsigned int

### 2.3、问题总结及根因确认

clang比gcc检查更加严格，检测出了给位域赋值时可能出现的隐式截断问题

## 3、修改建议

方案一：

修改`int allocated : 1`为`unsigned int allocated : 1`

参考：

方案二：

改为使用bool类型赋值，而不是1

修改`qtask->allocated = 1;`为`qtask->allocated = true;`


方案一能从根源解决问题，因此方案一更优

# 问题五：unused-but-set-variable问题

## 1、问题现象

```c
[   93s] discovery.c:698:6: error: variable 'num_targets' set but not used [-Werror,-Wunused-but-set-variable]
[   93s]   698 |         int num_targets = 0;
[   93s]       |             ^
[   93s] 1 warning and 1 error generated.
```

## 2、问题定位

查找num_targets在全文中的用法可知，该变量在定义之后只有两处自增的操作，除此之外并没有被实际使用过

## 3、修改建议

方案一：

理论上，这里的num_targets只有自增操作，并没有被具体使用，所以是可以将这三处出现的地方删除的

方案二：

clang开启Werror将`unused-but-set-variable`当做error，所以可以添加-Wno-error=unused-but-set-variable将error降级为warning，使llvm能够构建