来源：https://gitee.com/src-openeuler/lvm2 (openEuler-24.03-LTS)

# 相关PR

openEuler：[fix 2 undeclared functions,and changelog format,support clang build · Pull Request !183 · src-openEuler/lvm2 - Gitee.com](https://gitee.com/src-openeuler/lvm2/pulls/183)

上游：不涉及

# 问题一：call to undeclared function问题

## 1、问题现象

```bash
[  104s] libdm-common.c:137:3: error: call to undeclared function 'vsyslog'; ISO C99 and later do not support implicit function declarations [-Wimplicit-function-declaration]
[  104s]   137 |                 vsyslog(level, f, copyap);
[  104s]       |                 ^
[  122s] device/dev-cache.c:1021:7: error: call to undeclared function 'gettimeofday'; ISO C99 and later do not support implicit function declarations [-Wimplicit-function-declaration]
[  122s]  1021 |         if (!gettimeofday(&end, NULL)) {
[  122s]       |              ^
[  122s] device/dev-cache.c:1041:8: error: call to undeclared function 'gettimeofday'; ISO C99 and later do not support implicit function declarations [-Wimplicit-function-declaration]
[  122s]  1041 |         ret = gettimeofday(&start,NULL);
[  122s]       |               ^
[  122s] 2 errors generated.
```

## 2、问题定位

[0010-bugfix-lvm2-fix-the-reuse-of-va_list.patch](https://gitee.com/src-openeuler/lvm2/blob/fb6a5827c6c2fbfb26e6e531fc29da497745de7d/0010-bugfix-lvm2-fix-the-reuse-of-va_list.patch)在libdm-common.c中调用vsyslog函数，但是没有包含其头文件syslog.h

[0012-lvm-code-reduce-cyclomatic-complexity.patch](https://gitee.com/src-openeuler/lvm2/blob/fb6a5827c6c2fbfb26e6e531fc29da497745de7d/0012-lvm-code-reduce-cyclomatic-complexity.patch)在dev-cache.c中调用gettimeofday函数，但是没有包含其头文件sys/time.h

## 3、修改建议

在libdm-common.c中#include <syslog.h>

在dev-cache.c中#include <sys/time.h>

# 问题二：%changelog entries must start with \*问题

## 1、问题现象

`[   90s] error: %changelog not in descending chronological order`

## 2、问题定位

### 2.1、问题定位分析

```c
* Wed Nov 22 2023 wangxiaomeng <wangxiaomeng@kylinos.cn> - 8:2.03.22-3
- lvmlockd: add support for loongarch64

* Tue Dec 19 2023 wangzhiqiang <wangzhiqiang95@huawei.com> - 8:2.03.22-2
- dm-event: release buffer on dm_event_get_version

* Fri Nov 03 2023 liweigang <weigangli99@gmail.com> - 8:2.03.22-1
- update to version 2.03.22
```

问题出在[lvmlockd: add support for loongarch64 · dee4806 · 云沧/lvm2 - Gitee.com](https://gitee.com/yuncang123/lvm2/commit/dee4806fb29598db7cbd4d9abe56db348cc79baa)这个commit在`Tue Jan 9 2024`提交，而changelog里写的是`Wed Nov 22 2023`，导致其提交比上一个晚，但是changelog里写的日期比上一个早

### 2.2、问题总结及根因确认

changelog的日期应该按从上到下时间降序，本问题由于changelog中写的日期与实际提交日期大大不符导致

## 3、修改建议

将`Wed Nov 22 2023 wangxiaomeng <wangxiaomeng@kylinos.cn> - 8:2.03.22-3`的日期修改成commit对应的日期，即将`Wed Nov 22 2023`改为`Tue Jan 9 2024`