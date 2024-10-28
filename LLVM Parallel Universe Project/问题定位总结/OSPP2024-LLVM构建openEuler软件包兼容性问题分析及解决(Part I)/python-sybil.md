# python-sybil #

## 1、问题现象 ##

出自：https://gitee.com/src-openeuler/python-sybil
(openEuler-24.03-LTS)、
```
[   75s] FAILED tests/test_functional.py::test_pytest - AssertionError: 
[   75s] ========================= 1 failed, 80 passed in 2.82s =========================
[   75s] error: Bad exit status from /var/tmp/rpm-tmp.vV5eGG (%check)
```
## 2、问题定位及根因分析 ##

问题为偶发性问题，十次构建未必失败一次，本地构建也成功，怀疑为obs的问题
## 3、相关PR ##
openEuler：https://gitee.com/src-openeuler/python-sybil/pulls/7
上游社区：不涉及