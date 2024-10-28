# trace-cmd #

## 1、问题现象 ##

出自：https://gitee.com/src-openeuler/trace-cmd(openEuler-24.03-LTS)
```
[  111s] trace-hooks.c:135:5: error: call to undeclared function 'warning'; ISO C99 and later do not support implicit function declarations [-Wimplicit-function-declaration]
[  111s]   135 |                                 warning("unknown flag %c\n", flags[i]);
[  111s]       |                                 ^
[  111s] trace-hooks.c:152:2: error: call to undeclared function 'warning'; ISO C99 and later do not support implicit function declarations [-Wimplicit-function-declaration]
[  111s]   152 |         warning("Invalid hook format '%s'", arg);
[  111s]       |         ^
[  111s] 2 errors generated.
```
即调用了未声明的函数warning

## 2、问题定位及根因分析 ##

即在trace-hooks.c文件中调用了未声明的函数warning
```
invalid_tok:
	warning("Invalid hook format '%s'", arg);
	return NULL;
}
```
尝试在这个文件里加入相关定义后，别的文件又出现了同样的问题
依次加上之后就解决了问题

## 3.解决方案 ##

在相关c文件里补上warning函数定义即可
`void warning(const char *fmt, ...); `
## 4、相关PR ##
openEuler：https://gitee.com/src-openeuler/trace-cmd/pulls/76
上游社区：https://github.com/rostedt/trace-cmd/pull/22