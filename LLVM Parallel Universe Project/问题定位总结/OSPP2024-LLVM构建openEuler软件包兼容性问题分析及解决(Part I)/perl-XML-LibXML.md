# perl-XML-LibXML #

## 1、问题现象 ##

出自：https://gitee.com/src-openeuler/perl-XML-LibXML(openEuler-24.03-LTS)
```
68s] perl-libxml-mm.c:142:18:
incompatible fuction pointer types passing 'void (void*, void*, mmlchar *)’(aka'void (wvoid*, void*, unsigned char t)’) to parameter of type'mmashscamner'(aka
void (*)(void *, void *, const unsigned char *)’)[-Wincompatible-function pointer types]
68s]
142
xmlHashScan(r, PmmRegistryDumpllashScanner, NuLL);
```
## 2、问题定位及根因分析 ##

函数指针类型不兼容
问题出在源码中perl-libxml-mm.c文件中PmmRegistryDumpHashScanner函数定义（也就是 void (void *, void *, unsigned char *) ）与xmlHashScan 函数期望的函数指针类型 xmlHashScanner（也就是 void (*)(void *, void *, const unsigned char *) ）不匹配需要修改函数定义
`void PmmRegistryDumpHashScanner(void * payload, void * data, xmlChar * name)`
## 3、修改建议 ##

在perl-libxml-mm.c文件中PmmRegistryDumpHashScanner函数定义部分的第三个参数前加上const

即将
```
void PmmRegistryDumpHashScanner(void * payload, void * data, xmlChar * name)
```
改为
```
void PmmRegistryDumpHashScanner(void * payload, void * data,const xmlChar * name)
```
## 4、相关PR ##
openEuler：https://gitee.com/src-openeuler/perl-XML-LibXML/pulls/45
上游社区：https://github.com/shlomif/perl-XML-LibXML/pull/75