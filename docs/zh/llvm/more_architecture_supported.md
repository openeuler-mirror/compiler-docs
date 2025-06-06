# 新增架构支持

LLVM for openEuler 支持通过 `-mcpu=` 选项指定当前的 cpu 型号，使能该 cpu 所有默认特性。在使能对应的微架构亲和选项后，LLVM for openEuler 会依据对应架构的指令实现进行指令流水线调优，提升性能。硬件平台与该选项的配置项对应关系如下：

|硬件平台|配置项|
|-|-|
|鲲鹏920|tsv110|
|HiSilicon Hip09|hip09|
|HiSilicon Hip10c|hip10c|
|HiSilicon Hip11|hip11|
