# python-dmidecode.1 #

## 1、问题现象 ##

出自：https://gitee.com/src-openeuler/python-dmidecode(openEuler-24.03-LTS)
```
[   75s] src/dmidecodemodule.c:916:28: error: incompatible function pointer types initializing 'PyCFunction' (aka 'struct _object *(*)(struct _object *, struct _object *)') with an expression of type 'PyObject *(PyObject *, PyObject *, PyObject *)' (aka 'struct _object *(struct _object *, struct _object *, struct _object *)') [-Wincompatible-function-pointer-types]
[   75s]   916 |         {(char *)"xmlapi", dmidecode_xmlapi, METH_VARARGS | METH_KEYWORDS,
[   75s]       |                            ^~~~~~~~~~~~~~~~
```
## 2、问题定位及根因分析 ##

即不兼容的函数指针类型错误：
在第src/dmidecodemodule.c的 916 行，尝试将一个函数指针类型为`PyObject *(PyObject *, PyObject *, PyObject *)`的表达式初始化为PyCFunction类型（即struct _object *(*)(struct _object *, struct _object *)）。这意味着函数的参数个数不匹配，一个接受三个参数，一个只接受两个参数。
```
static PyObject *dmidecode_xmlapi(PyObject *self, PyObject *args, PyObject *keywds)
```
```
{(char *)"xmlapi", dmidecode_xmlapi, METH_VARARGS | METH_KEYWORDS,
(char *) "Internal API for retrieving data as raw XML data"},
```

之所以在gcc里面不报错是因为incompatible function pointer types initializing这个类型的问题在gcc里是警告，而在clang里才会报错

## 3、修改建议 ##

首先我尝试进行封装，但是依旧报错，于是我再次尝试进行手动数据类型转换

即将
```
{(char *)"xmlapi", dmidecode_xmlapi, METH_VARARGS | METH_KEYWORDS,
(char *) "Internal API for retrieving data as raw XML data"},
```
改为
```
{"xmlapi", (PyCFunction)dmidecode_xmlapi, METH_VARARGS | METH_KEYWORDS, "Internal API for retrieving data as raw XML data"},
```
即可修复这个问题
## 4、相关PR ##
openEuler：https://gitee.com/src-openeuler/python-dmidecode/pulls/40
上游社区：https://github.com/nima/python-dmidecode/pull/58