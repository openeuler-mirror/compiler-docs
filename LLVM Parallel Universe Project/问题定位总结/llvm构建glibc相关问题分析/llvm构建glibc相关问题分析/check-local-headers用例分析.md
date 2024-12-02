# check-local-headers用例分析

执行之后有个检查结果输出到/root/rpmbuild/BUILD/glibc-2.38/build-x86_64-openEuler-linux/check-local-headers.out

clang构建是的报错均为

```cpp
$(common-objpfx)support/xxx.oS: uses /usr/include/limits.h
/usr/include/limits.h: uses /usr/include/limits.h:
```

测试脚本

```cpp
AWK='gawk' scripts/check-local-headers.sh   "/usr/include" "/root/rpmbuild/BUILD/glibc-2.38/build-x86_64-openEuler-linux/" < /dev/null > /root/rpmbuild/BUILD/glibc-2.38/build-x86_64-openEuler-linux/check-local-headers.out; scripts/evaluate-test.sh check-local-headers $? false false > /root/rpmbuild/BUILD/glibc-2.38/build-x86_64-openEuler-linux/check-local-headers.test-result
```

实际执行的是这样一段

```cpp
exec gawk -v includedir="/usr/include" '
BEGIN {
  status = 0
  exclude = "^" includedir \
    "/(.*-.*-.*/|.*-.*/|)(asm[-/]|arch|linux/|selinux/|mach/|mach_debug/|device/|hurd/(((hurd|ioctl)_types|paths)\\.h|ioctls\\.defs|ihash\\.h|version\\.h)|gd|nss3/|nspr4?/|c\\+\\+/|sys/(capability|sdt(|-config))\\.h|libaudit\\.h)"
}
/^[^ ]/ && $1 ~ /.*:/ { obj = $1 }
{
  for (i = 1; i <= NF; ++i) {
    if ($i ~ ("^" includedir) && $i !~ exclude) {
      print "***", obj, "uses", $i
      status = 1
    }
  }
}
END { }' support/xwaitpid.oS.d
```

```cpp
exec gawk -v includedir="" '
BEGIN {
  status = 0
  exclude = "^" includedir \
    "/(.*-.*-.*/|.*-.*/|)(asm[-/]|arch|linux/|selinux/|mach/|mach_debug/|device/|hurd/(((hurd|ioctl)_types|paths)\\.h|ioctls\\.defs|ihash\\.h|version\\.h)|gd|nss3/|nspr4?/|c\\+\\+/|sys/(capability|sdt(|-config))\\.h|libaudit\\.h)"
}
/^[^ ]/ && $1 ~ /.*:/ { obj = $1 }
{
  for (i = 1; i <= NF; ++i) {
    if ($i ~ ("^" includedir) && $i !~ exclude) {
      print "***", obj, "uses", $i
      status = 1
    }
  }
}
END { }' support/xwaitpid.oS.d
```

分析差异原因依旧是llvm的limits.h头文件引用系统头文件导致