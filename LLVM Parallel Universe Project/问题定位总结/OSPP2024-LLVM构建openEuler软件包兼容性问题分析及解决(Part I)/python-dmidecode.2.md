# python-dmidecode.2 #

## 1、问题现象 ##

出自：https://gitee.com/src-openeuler/python-dmidecode(openEuler-24.03-LTS)
```
[   75s] src/dmidecodemodule.c:1027:72: error: incompatible function pointer types passing 'void (void *)' to parameter of type 'PyCapsule_Destructor' (aka 'void (*)(struct _object *)') [-Wincompatible-function-pointer-types]
[   75s]  1027 |         PyModule_AddObject(module, "options", PyCapsule_New(opt, NULL, destruct_options));
[   75s]       |                                                                        ^~~~~~~~~~~~~~~~
```
## 2、问题定位及根因分析 ##

即这个错误表明在将一个函数指针作为PyCapsule的析构函数传递时，函数指针类型不兼容。
具体来说，错误发生在src/dmidecodemodule.c文件的第 1027 行，尝试将一个类型为void (void *)的函数指针传递给期望类型为PyCapsule_Destructor（即void (*)(struct _object *)）的参数。
```
void destruct_options(void *ptr)
{
#ifdef IS_PY3K
        ptr = PyCapsule_GetPointer(ptr, NULL);
#endif
        options *opt = (options *) ptr;

```
## 3、修改建议 ##

将函数定义由`void destruct_options(void *ptr)`换为`void destruct_options(PyObject *capsule)` 

再将
```
ptr = PyCapsule_GetPointer(ptr, NULL);
options *opt = (options *) ptr;
```

改为
```
void *ptr = PyCapsule_GetPointer(capsule, NULL);
options *opt = (options *) ptr;
```
使该代码与原来的部分保持一致

即可解决问题
## 4、相关PR ##
openEuler：https://gitee.com/src-openeuler/python-dmidecode/pulls/40
上游社区：https://github.com/nima/python-dmidecode/pull/58