# rarian #

## 1、问题现象 ##

出自：https://gitee.com/src-openeuler/rarian(openEuler-24.03-LTS)
```
[   79s] rarian-example.c:80:1: error: type specifier missing, defaults to 'int'; ISO C99 and later do not support implicit int [-Wimplicit-int]
[   79s]    80 | info_for_each (RrnInfoEntry *entry, void *data)
[   79s]       | ^
[   79s]       | int
```
## 2、问题定位及根因分析 ##

即rarian-example.c 文件的第 80 行，定义函数时没有明确指定函数的返回类型，编译器默认认为是 int，但在 ISO C99 及更高版本中不支持这种隐式的 int 类型声明。
```
info_for_each (RrnInfoEntry *entry, void *data)
{
  if (entry->section)
    printf ("Info page: %s\n\tPath: %s#%s\n\tComment: %s\n",
	    entry->doc_name, entry->base_filename,
	    entry->section, entry->comment);
  else
    printf ("Info page: %s\n\tPath: %s\n\tComment: %s\n",
	    entry->doc_name, entry->base_filename,
	    entry->comment);
  return TRUE;
}

man_for_each (RrnManEntry *entry, void *data)
{
  printf ("Man page %s\n", entry->name);
  printf ("\tPath: %s\n", entry->path);
  return TRUE;
}
```
要解决这个问题，可以在函数定义处明确指定函数的返回类型。

## 3、修改建议 ##

只要在函数前加一个int即可
```
int
info_for_each (RrnInfoEntry *entry, void *data)
{
  if (entry->section)
    printf ("Info page: %s\n\tPath: %s#%s\n\tComment: %s\n",
	    entry->doc_name, entry->base_filename,
	    entry->section, entry->comment);
  else
    printf ("Info page: %s\n\tPath: %s\n\tComment: %s\n",
	    entry->doc_name, entry->base_filename,
	    entry->comment);
  return TRUE;
}

int
man_for_each (RrnManEntry *entry, void *data)
{
  printf ("Man page %s\n", entry->name);
  printf ("\tPath: %s\n", entry->path);
  return TRUE;
}
```
## 4、相关PR ##
openEuler：https://gitee.com/src-openeuler/rarian/pulls/14
上游社区：不涉及