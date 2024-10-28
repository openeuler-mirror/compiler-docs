# satyr #

## 1、问题现象 ##

出自：https://gitee.com/src-openeuler/satyr(openEuler-24.03-LTS)
```
FAIL: core_stacktrace
......
FAIL:  1
```
## 2、问题定位及根因分析 ##

该问题上游已经解决，obs平台上aarch是succeed
上游由
```
__attribute__((optimize((0))))
```

改为
```
 #if __clang__
__attribute__((optnone))  
 #else
__attribute__((optimize((0))))
 #endif
```

增加了判断当编译器为clang时的处理

x86是fail
报错为上述问题现象

在本地的x86中的clang运行该程序显示构建成功
推测为obs平台的测试代码有误
## 3、相关PR ##
openEuler：https://gitee.com/src-openeuler/satyr/pulls/39
上游社区：不涉及