## dpdk

#### 相关PR

- 不涉及上游，openEuler仓库PR：[Support clang build (modify 2 meson.build and add -fgcc-compatible to cflags) · Pull Request !619 · src-openEuler/dpdk - Gitee.com](https://gitee.com/src-openeuler/dpdk/pulls/619)

### 报错信息一

在修改前使用clang构建多次出现如下错误，软件包构建失败：

```
FAILED: drivers/libtmp_rte_common_mlx5.a.p/common_mlx5_linux_mlx5_glue.c.o
clang -Idrivers/libtmp_rte_common_mlx5.a.p -Idrivers -I../drivers -Idrivers/common/mlx5 -I../drivers/common/mlx5 -Idrivers/common/mlx5/linux -I../drivers/common/mlx5/linux -Ilib/hash -I../lib/hash -I. -I.. -Iconfig -I../config -Ilib/eal/include -I../lib/eal/include -Ilib/eal/linux/include -I../lib/eal/linux/include -Ilib/eal/x86/include -I../lib/eal/x86/include -Ilib/eal/common -I../lib/eal/common -Ilib/eal -I../lib/eal -Ilib/kvargs -I../lib/kvargs -Ilib/log -I../lib/log -Ilib/metrics -I../lib/metrics -Ilib/telemetry -I../lib/telemetry -Ilib/net -I../lib/net -Ilib/mbuf -I../lib/mbuf -Ilib/mempool -I../lib/mempool -Ilib/ring -I../lib/ring -Ilib/rcu -I../lib/rcu -Ilib/pci -I../lib/pci -Idrivers/bus/pci -I../drivers/bus/pci -I../drivers/bus/pci/linux -Idrivers/bus/auxiliary -I../drivers/bus/auxiliary -I/usr/include/libnl3 -fdiagnostics-color=always -D_FILE_OFFSET_BITS=64 -Wall -Winvalid-pch -Wextra -std=c11 -include rte_config.h -Wcast-qual -Wdeprecated -Wformat -Wformat-nonliteral -Wformat-security -Wmissing-declarations -Wmissing-prototypes -Wnested-externs -Wold-style-definition -Wpointer-arith -Wsign-compare -Wstrict-prototypes -Wundef -Wwrite-strings -Wno-address-of-packed-member -Wno-missing-field-initializers -D_GNU_SOURCE -O2 -g -grecord-gcc-switches -pipe -fstack-protector-strong -Wall -Werror=format-security -Wp,-D_FORTIFY_SOURCE=2 -Wp,-D_GLIBCXX_ASSERTIONS --config /usr/lib/rpm/generic-hardened-clang.cfg -m64 -mtune=generic -fasynchronous-unwind-tables -fstack-clash-protection -fcommon -fPIC -march=corei7 -mrtm -DALLOW_EXPERIMENTAL_API -DALLOW_INTERNAL_API -fPIE -pie -fPIC -fstack-protector-strong -D_FORTIFY_SOURCE=2 -O2 -Wall -Wl,-z,relro,-z,now,-z,noexecstack -Wtrampolines -std=c11 -Wno-strict-prototypes -D_BSD_SOURCE -D_DEFAULT_SOURCE -D_XOPEN_SOURCE=600 -UPEDANTIC -DRTE_LOG_DEFAULT_LOGTYPE=pmd.common.mlx5 -MD -MQ drivers/libtmp_rte_common_mlx5.a.p/common_mlx5_linux_mlx5_glue.c.o -MF drivers/libtmp_rte_common_mlx5.a.p/common_mlx5_linux_mlx5_glue.c.o.d -o drivers/libtmp_rte_common_mlx5.a.p/common_mlx5_linux_mlx5_glue.c.o -c ../drivers/common/mlx5/linux/mlx5_glue.c
clang: warning: -Wl,-z,relro,-z,now,-z,noexecstack: 'linker' input unused [-Wunused-command-line-argument]
clang: warning: argument unused during compilation: '-pie' [-Wunused-command-line-argument]
warning: unknown warning option '-Wtrampolines' [-Wunknown-warning-option]
In file included from ../drivers/common/mlx5/linux/mlx5_glue.c:17:
../drivers/common/mlx5/linux/mlx5_glue.h:58:6: error: redefinition of 'mlx5_ib_uapi_flow_action_packet_reformat_type'
   58 | enum mlx5dv_flow_action_packet_reformat_type { packet_reformat_type = 0, };
      |      ^
/usr/include/infiniband/mlx5_api.h:46:50: note: expanded from macro 'mlx5dv_flow_action_packet_reformat_type'
   46 | #define mlx5dv_flow_action_packet_reformat_type         mlx5_ib_uapi_flow_action_packet_reformat_type
      |                                                         ^
/usr/include/infiniband/mlx5_user_ioctl_verbs.h:50:6: note: previous definition is here
   50 | enum mlx5_ib_uapi_flow_action_packet_reformat_type {
      |      ^
In file included from ../drivers/common/mlx5/linux/mlx5_glue.c:17:
../drivers/common/mlx5/linux/mlx5_glue.h:59:6: error: redefinition of 'mlx5_ib_uapi_flow_table_type'
   59 | enum mlx5dv_flow_table_type { flow_table_type = 0, };
      |      ^
/usr/include/infiniband/mlx5_api.h:40:35: note: expanded from macro 'mlx5dv_flow_table_type'
   40 | #define mlx5dv_flow_table_type                          mlx5_ib_uapi_flow_table_type
      |                                                         ^
/usr/include/infiniband/mlx5_user_ioctl_verbs.h:42:6: note: previous definition is here
   42 | enum mlx5_ib_uapi_flow_table_type {
      |      ^
In file included from ../drivers/common/mlx5/linux/mlx5_glue.c:17:
../drivers/common/mlx5/linux/mlx5_glue.h:68:8: error: redefinition of 'mlx5dv_devx_umem'
   68 | struct mlx5dv_devx_umem { uint32_t umem_id; };
      |        ^
/usr/include/infiniband/mlx5dv.h:1746:8: note: previous definition is here
 1746 | struct mlx5dv_devx_umem {
      |        ^
In file included from ../drivers/common/mlx5/linux/mlx5_glue.c:17:
../drivers/common/mlx5/linux/mlx5_glue.h:69:8: error: redefinition of 'mlx5dv_devx_uar'
   69 | struct mlx5dv_devx_uar { void *reg_addr; void *base_addr; uint32_t page_id; };
      |        ^
/usr/include/infiniband/mlx5dv.h:1771:8: note: previous definition is here
 1771 | struct mlx5dv_devx_uar {
      |        ^
In file included from ../drivers/common/mlx5/linux/mlx5_glue.c:17:
../drivers/common/mlx5/linux/mlx5_glue.h:78:7: error: redefinition of 'mlx5dv_dr_domain_type'
   78 | enum  mlx5dv_dr_domain_type { unused, };
      |       ^
/usr/include/infiniband/mlx5dv.h:1933:6: note: previous definition is here
 1933 | enum mlx5dv_dr_domain_type {
      |      ^
In file included from ../drivers/common/mlx5/linux/mlx5_glue.c:17:
../drivers/common/mlx5/linux/mlx5_glue.h:109:8: error: redefinition of 'mlx5dv_dr_flow_sampler_attr'
  109 | struct mlx5dv_dr_flow_sampler_attr {
      |        ^
/usr/include/infiniband/mlx5dv.h:1953:8: note: previous definition is here
 1953 | struct mlx5dv_dr_flow_sampler_attr {
      |        ^
In file included from ../drivers/common/mlx5/linux/mlx5_glue.c:17:
../drivers/common/mlx5/linux/mlx5_glue.h:119:6: error: redefinition of 'mlx5dv_dr_action_dest_type'
  119 | enum mlx5dv_dr_action_dest_type {
      |      ^
/usr/include/infiniband/mlx5dv.h:2030:6: note: previous definition is here
 2030 | enum mlx5dv_dr_action_dest_type {
      |      ^
In file included from ../drivers/common/mlx5/linux/mlx5_glue.c:17:
../drivers/common/mlx5/linux/mlx5_glue.h:120:2: error: redefinition of enumerator 'MLX5DV_DR_ACTION_DEST'
  120 |         MLX5DV_DR_ACTION_DEST,
      |         ^
/usr/include/infiniband/mlx5dv.h:2031:2: note: previous definition is here
 2031 |         MLX5DV_DR_ACTION_DEST,
      |         ^
In file included from ../drivers/common/mlx5/linux/mlx5_glue.c:17:
../drivers/common/mlx5/linux/mlx5_glue.h:121:2: error: redefinition of enumerator 'MLX5DV_DR_ACTION_DEST_REFORMAT'
  121 |         MLX5DV_DR_ACTION_DEST_REFORMAT,
      |         ^
/usr/include/infiniband/mlx5dv.h:2032:2: note: previous definition is here
 2032 |         MLX5DV_DR_ACTION_DEST_REFORMAT,
      |         ^
In file included from ../drivers/common/mlx5/linux/mlx5_glue.c:17:
../drivers/common/mlx5/linux/mlx5_glue.h:123:8: error: redefinition of 'mlx5dv_dr_action_dest_reformat'
  123 | struct mlx5dv_dr_action_dest_reformat {
      |        ^
/usr/include/infiniband/mlx5dv.h:2035:8: note: previous definition is here
 2035 | struct mlx5dv_dr_action_dest_reformat {
      |        ^
In file included from ../drivers/common/mlx5/linux/mlx5_glue.c:17:
../drivers/common/mlx5/linux/mlx5_glue.h:127:8: error: redefinition of 'mlx5dv_dr_action_dest_attr'
  127 | struct mlx5dv_dr_action_dest_attr {
      |        ^
/usr/include/infiniband/mlx5dv.h:2040:8: note: previous definition is here
 2040 | struct mlx5dv_dr_action_dest_attr {
      |        ^
In file included from ../drivers/common/mlx5/linux/mlx5_glue.c:17:
../drivers/common/mlx5/linux/mlx5_glue.h:137:8: error: redefinition of 'mlx5dv_devx_event_channel'
  137 | struct mlx5dv_devx_event_channel { int fd; };
      |        ^
/usr/include/infiniband/mlx5dv.h:1837:8: note: previous definition is here
 1837 | struct mlx5dv_devx_event_channel {
      |        ^
In file included from ../drivers/common/mlx5/linux/mlx5_glue.c:17:
../drivers/common/mlx5/linux/mlx5_glue.h:139:9: warning: 'MLX5DV_DEVX_CREATE_EVENT_CHANNEL_FLAGS_OMIT_EV_DATA' macro redefined [-Wmacro-redefined]
  139 | #define MLX5DV_DEVX_CREATE_EVENT_CHANNEL_FLAGS_OMIT_EV_DATA 1
      |         ^
/usr/include/infiniband/mlx5_api.h:60:9: note: previous definition is here
   60 | #define MLX5DV_DEVX_CREATE_EVENT_CHANNEL_FLAGS_OMIT_EV_DATA MLX5_IB_UAPI_DEVX_CR_EV_CH_FLAGS_OMIT_DATA
      |         ^
In file included from ../drivers/common/mlx5/linux/mlx5_glue.c:17:
../drivers/common/mlx5/linux/mlx5_glue.h:143:8: error: redefinition of 'mlx5dv_var'
  143 | struct mlx5dv_var { uint32_t page_id; uint32_t length; off_t mmap_off;
      |        ^
/usr/include/infiniband/mlx5dv.h:1784:8: note: previous definition is here
 1784 | struct mlx5dv_var {
      |        ^
In file included from ../drivers/common/mlx5/linux/mlx5_glue.c:17:
../drivers/common/mlx5/linux/mlx5_glue.h:152:8: error: redefinition of 'mlx5dv_steering_anchor_attr'
  152 | struct mlx5dv_steering_anchor_attr {
      |        ^
/usr/include/infiniband/mlx5dv.h:748:8: note: previous definition is here
  748 | struct mlx5dv_steering_anchor_attr {
      |        ^
In file included from ../drivers/common/mlx5/linux/mlx5_glue.c:17:
../drivers/common/mlx5/linux/mlx5_glue.h:158:8: error: redefinition of 'mlx5dv_steering_anchor'
  158 | struct mlx5dv_steering_anchor {
      |        ^
/usr/include/infiniband/mlx5dv.h:754:8: note: previous definition is here
  754 | struct mlx5dv_steering_anchor {
      |        ^
2 warnings and 15 errors generated.
```

#### 修复过程

- 上述错误的根源是，原本`0002-dpdk-add-secure-compile-option-and-fPIC-option.patch`使得在`drivers/meson.build`中增加如下代码：

  ```
  default_cflags += ['-fPIE', '-pie', '-fPIC', '-fstack-protector-strong', '-D_FORTIFY_SOURCE=2', '-O2', '-Wall']
  default_cflags += ['-Wl,-z,relro,-z,now,-z,noexecstack', '-Wtrampolines']
  ```

  于是构建时的`c_args`会添加上上述选项。而clang中链接选项需要传给链接器而不是以编译器参数传入，且不支持`-pie`和`-Wtrampolines`选项。

  这导致了在执行`drivers/common/mlx5/meson.build`中的

  ```
  configure_file(output: 'mlx5_autoconf.h', configuration: mlx5_config)
  ```

  时，在检测`infiniband/mlx5dv.h`等中的定义时，gcc对测试代码能成功编译并检测出定义情况，以`mlx5dv_devx_obj_create`为例：

  > gcc构建时：`build-gcc/meson-logs/meson-log.txt`片段：
  >
  > ```
  > Running compile:
  > Working directory:  /home/abuild/rpmbuild/BUILD/dpdk-23.11/build-gcc/meson-private/tmpbopy1m2o
  > Code:
  >  
  >         #include <infiniband/mlx5dv.h>
  >         int main(void) {
  >             /* If it's not defined as a macro, try to use as a symbol */
  >             #ifndef mlx5dv_devx_obj_create
  >                 mlx5dv_devx_obj_create;
  >             #endif
  >             return 0;
  >         }
  > -----------
  > Command line: `/usr/bin/gcc -I/usr/include/libnl3 /home/abuild/rpmbuild/BUILD/dpdk-23.11/build-gcc/meson-private/tmpbopy1m2o/testfile.c -o /home/abuild/rpmbuild/BUILD/dpdk-23.11/build-gcc/meson-private/tmpbopy1m2o/output.obj -c -D_FILE_OFFSET_BITS=64 -O0 -std=c11 -march=native -mno-avx512f -mrtm -DALLOW_EXPERIMENTAL_API -DALLOW_INTERNAL_API -fPIE -pie -fPIC -fstack-protector-strong -D_FORTIFY_SOURCE=2 -O2 -Wall -Wl,-z,relro,-z,now,-z,noexecstack -Wtrampolines -Wno-format-truncation -std=c11 -Wno-strict-prototypes -D_BSD_SOURCE -D_DEFAULT_SOURCE -D_XOPEN_SOURCE=600 -UPEDANTIC` -> 0
  > stderr:
  > /home/abuild/rpmbuild/BUILD/dpdk-23.11/build-gcc/meson-private/tmpbopy1m2o/testfile.c: In function 'main':
  > /home/abuild/rpmbuild/BUILD/dpdk-23.11/build-gcc/meson-private/tmpbopy1m2o/testfile.c:6:17: warning: statement with no effect [-Wunused-value]
  >     6 |                 mlx5dv_devx_obj_create;
  >       |                 ^~~~~~~~~~~~~~~~~~~~~~
  > -----------
  > Header "infiniband/mlx5dv.h" has symbol "mlx5dv_devx_obj_create" with dependencies libmlx5, libibverbs: YES
  > ```

  而clang编译由于命令行使用了`-Werror=unknown-warning-option -Werror=unused-command-line-argument`两个参数，所以会报错`clang: error: -Wl,-z,relro,-z,now,-z,noexecstack: 'linker' input unused [-Werror,-Wunused-command-line-argument]`从而导致检测失败（同理，`-pie`和`-Wtrampolines`也会导致clang error）。

  > clang构建时：`build-clang/meson-logs/meson-log.txt`片段：
  >
  > ```
  > Running compile:
  > Working directory:  /home/abuild/rpmbuild/BUILD/dpdk-23.11/build-clang/meson-private/tmp399v6yuc
  > Code:
  >  
  >         #include <infiniband/mlx5dv.h>
  >         int main(void) {
  >             /* If it's not defined as a macro, try to use as a symbol */
  >             #ifndef mlx5dv_devx_obj_create
  >                 mlx5dv_devx_obj_create;
  >             #endif
  >             return 0;
  >         }
  > -----------
  > Command line: `/usr/bin/clang -I/usr/include/libnl3 /home/abuild/rpmbuild/BUILD/dpdk-23.11/build-clang/meson-private/tmp399v6yuc/testfile.c -o /home/abuild/rpmbuild/BUILD/dpdk-23.11/build-clang/meson-private/tmp399v6yuc/output.obj -c -D_FILE_OFFSET_BITS=64 -O0 -Werror=implicit-function-declaration -Werror=unknown-warning-option -Werror=unused-command-line-argument -Werror=ignored-optimization-argument -std=c11 -march=native -mrtm -DALLOW_EXPERIMENTAL_API -DALLOW_INTERNAL_API -fPIE -pie -fPIC -fstack-protector-strong -D_FORTIFY_SOURCE=2 -O2 -Wall -Wl,-z,relro,-z,now,-z,noexecstack -Wtrampolines -std=c11 -Wno-strict-prototypes -D_BSD_SOURCE -D_DEFAULT_SOURCE -D_XOPEN_SOURCE=600 -UPEDANTIC` -> 1
  > stderr:
  > clang: error: -Wl,-z,relro,-z,now,-z,noexecstack: 'linker' input unused [-Werror,-Wunused-command-line-argument]
  > -----------
  > Header "infiniband/mlx5dv.h" has symbol "mlx5dv_devx_obj_create" with dependencies libmlx5, libibverbs: NO 
  > ```

  又因为在`drivers/common/mlx5/linux/meson.build`中使用了数组来确定是否定义宏：

  ```
  # input array for meson member search:
  # [ "MACRO to define if found", "header for the search",
  #   "symbol to search", "struct member to search" ]
  has_sym_args = [
  	...
      [ 'HAVE_IBV_DEVX_OBJ', 'infiniband/mlx5dv.h',
      'mlx5dv_devx_obj_create' ],
      ...
  ]
  ```

  从而，对于`build-clang/drivers/common/mlx5/mlx5_autoconf.h`，会对应地得到：

  ```
  #undef HAVE_IBV_DEVX_OBJ
  ```

  而`build-gcc/drivers/common/mlx5/mlx5_autoconf.h`中则是：

  ```
  #define HAVE_IBV_DEVX_OBJ
  ```

  而在`drivers/common/mlx5/linux/mlx5_glue.h`中通过该宏是否定义来决定是否定义一些结构体：

  ```
  #ifndef HAVE_IBV_DEVX_OBJ
  struct mlx5dv_devx_obj;
  struct mlx5dv_devx_umem { uint32_t umem_id; };
  struct mlx5dv_devx_uar { void *reg_addr; void *base_addr; uint32_t page_id; };
  #endif
  ```

  由于上述原因clang构建时并没有定义`HAVE_IBV_DEVX_OBJ`，因此中间的几个原本在`infiniband/mlx5dv.h`中以及定义过的结构体被重新定义了，导致了开头所述的`error: redefinition of ...`的问题。

  - 将`drivers/meson.build`中的`'-Wl,-z,relro,-z,now,-z,noexecstack'`改为作为链接参数而非编译参数传入；clang不支持的`-pie`和`-Wtrampolines`参数则改为当编译器支持时才添加后可以修复上述问题



### 报错信息二

- 上述问题修复后构建软件包会出现报错：

  ```
  [2284/2284] /usr/bin/make -j4 -C /lib/modules/6.6.0-15.0.0.13.mg2403.x86_64/build M=/home/abuild/rpmbuild/BUILD/dpdk-23.11/openEuler-linux-build/kernel/linux/igb_uio src=/home/abuild/rpmbuild/BUILD/dpdk-23.11/kernel/linux/igb_uio modules
  FAILED: kernel/linux/igb_uio/igb_uio.ko 
  /usr/bin/make -j4 -C /lib/modules/6.6.0-15.0.0.13.mg2403.x86_64/build M=/home/abuild/rpmbuild/BUILD/dpdk-23.11/openEuler-linux-build/kernel/linux/igb_uio src=/home/abuild/rpmbuild/BUILD/dpdk-23.11/kernel/linux/igb_uio modules
  make: Entering directory '/usr/src/kernels/6.6.0-15.0.0.13.mg2403.x86_64'
  warning: the compiler differs from the one used to build the kernel
    The kernel was built by: clang version 17.0.6 ( 17.0.6-8.mg2403)
    You are using:           gcc_old (GCC) 12.3.1 (openEuler 12.3.1-25.mg2403)
    CC [M]  /home/abuild/rpmbuild/BUILD/dpdk-23.11/openEuler-linux-build/kernel/linux/igb_uio/igb_uio.o
  cc1: error: unrecognized command-line option ‘-mretpoline-external-thunk’
  cc1: note: unrecognized command-line option ‘-Wno-initializer-overrides’ may have been intended to silence earlier diagnostics
  cc1: note: unrecognized command-line option ‘-Wno-tautological-constant-out-of-range-compare’ may have been intended to silence earlier diagnostics
  cc1: note: unrecognized command-line option ‘-Wno-gnu’ may have been intended to silence earlier diagnostics
  ```

#### 修复过程

- `igb_uio`由`0001-add-igb_uio.patch`引入，该报错的原因是`igb_uio`使用make构建时的命令并未指定编译器，默认使用了gcc而非clang，与kernel构建时使用的编译器clang不同。源代码中仅仅在`kernel/linux/meson.build`中指定了仅当交叉编译时会显式指定CC以及LD，而全部使用clang的情况并不会判定为交叉编译。

- 修复方式：向`kernel/linux/igb_uio/meson.build`中执行的`make`命令添加指定CC参数，当kernel使用clang时添加`CC=clang`后上述问题消失，clang构建成功。 添加的获取kernel使用的编译器是否为clang的代码比较粗糙。原本有一个很好的方案，即直接使用`CC=cc.get_id()`，但这样在OBS验证工程中并没有全部通过，在验证工程Mega:24.09有遇到过与上述报错恰恰相反的报错，即kernel使用gcc构建而其他都使用clang构建（也就是meson检测到的C编译器为clang）的情况，原因不太清楚。



### 报错信息三

在线上OBS验证工程Mega-LLVM:24.03中提供的standard_aarch64环境中，会出现如下失败日志：

```
...
[   78s] Compiler for C supports arguments -Wcast-qual: NO
[   78s] Compiler for C supports arguments -Wdeprecated: NO
[   78s] Compiler for C supports arguments -Wformat: NO
[   78s] Compiler for C supports arguments -Wformat-nonliteral: NO
[   78s] Compiler for C supports arguments -Wformat-security: NO
[   78s] Compiler for C supports arguments -Wmissing-declarations: NO
[   78s] Compiler for C supports arguments -Wmissing-prototypes: NO
[   78s] Compiler for C supports arguments -Wnested-externs: NO
[   78s] Compiler for C supports arguments -Wold-style-definition: NO
[   78s] Compiler for C supports arguments -Wpointer-arith: NO
[   78s] Compiler for C supports arguments -Wsign-compare: NO
[   78s] Compiler for C supports arguments -Wstrict-prototypes: NO
[   78s] Compiler for C supports arguments -Wundef: NO
[   78s] Compiler for C supports arguments -Wwrite-strings: NO
[   78s] Compiler for C supports arguments -Wno-address-of-packed-member: NO
[   78s] Compiler for C supports arguments -Wno-packed-not-aligned: NO
[   78s] Compiler for C supports arguments -Wno-missing-field-initializers: NO
[   78s] Message: Arm implementer: Generic armv8
[   78s] Message: Arm part number: generic
[   78s] Compiler for C supports arguments -march=armv8-a: NO
[   78s]
[   78s] config/arm/meson.build:721:12: ERROR: Problem encountered: No suitable armv8 march version found.
[   78s]
[   78s] A full log can be found at /home/abuild/rpmbuild/BUILD/dpdk-23.11/openEuler-linux-build/meson-logs/meson-log.txt
[   78s] error: Bad exit status from /var/tmp/rpm-tmp.GLGYUR (%build)
[   78s]
[   78s] RPM build errors:
[   78s]     Bad exit status from /var/tmp/rpm-tmp.GLGYUR (%build)
[   78s]
[   78s]  failed "build dpdk.spec" at Thu Sep 19 05:59:41 UTC 2024.
```

#### 修复过程

- 分析过程：发现OBS里的Mega:24.09的aarch64不会出现该报错。经过对比构建日志发现二者环境中的CFLAGS不同，进而尝试之下发现添加`-fgcc-compatible`可以解决该问题。

- 原因分析：因为meson函数cc.has_argument函数会产生-wunused-commond-line报错，加-fgcc-compitable可以绕过。

- 修复方式：在spec中向`CFLAGS`中添加`-fgcc-compatible`：

  ```
  - CFLAGS="$(echo %{optflags} -fcommon)" \
  + CFLAGS="$(echo %{optflags} -fcommon %[ "%{toolchain}" == "clang" ? "-fgcc-compatible" : "" ])" \
  ```

  （实际上由于版本升级中已经解决了该问题，故未进行修改当作已修复）
