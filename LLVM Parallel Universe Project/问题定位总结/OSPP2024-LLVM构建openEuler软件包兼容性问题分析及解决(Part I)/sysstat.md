# sysstat #

## 1、问题现象 ##

出自：https://gitee.com/src-openeuler/sysstat(openEuler-24.03-LTS)
```
[   72s] rd_stats.c:995:4: error: non-void function 'read_tty_driver_serial' should return a value [-Wreturn-type]
[   72s]   995 |                         return;
[   72s]       |                         ^
```
## 2、问题定位及根因分析 ##

上一个人提交的patch里，return后面没有返回值

https://gitee.com/src-openeuler/sysstat/pulls/68

```
+	if ((fp = fopen(TTYAMA, "r")) == NULL) {
+		if ((fp = fopen(SERIAL, "r")) == NULL) {
+			return;
+		}
+	}
```
## 3.解决方案 ##

在return后面加上返回值0即可解决问题
```
+	if ((fp = fopen(TTYAMA, "r")) == NULL) {
+		if ((fp = fopen(SERIAL, "r")) == NULL) {
+			return 0;
+		}
+	}
```
## 4、相关PR ##
openEuler：https://gitee.com/src-openeuler/sysstat/pulls/98
上游社区：不涉及