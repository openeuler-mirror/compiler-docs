# elf/check-localplt

这个测试用于检测一些库函数是否使用plt寻址，llvm构建的glibc在以下函数上额外使用了plt

Extra PLT reference: [libc.so](http://libc.so/): abort
Extra PLT reference: [libc.so](http://libc.so/): stpcpy
Extra PLT reference: [libc.so](http://libc.so/): strcpy
Extra PLT reference: [libm.so](http://libm.so/): fabsf128

评估没有功能影响，先不分析