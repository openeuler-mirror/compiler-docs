# 新增架构支持

LLVM for openEuler 支持通过 `-mcpu=<CPU-name>` 选项指定当前的 CPU 型号，使能该 CPU 所有默认支持的硬件特性。在使能对应的微架构亲和选项后，LLVM for openEuler 会依据对应微架构的指令特征进行指令流水线调优，提升性能。

鲲鹏硬件平台与该选项的配置项对应关系如下：

|硬件平台|配置项|
|-|-|
|鲲鹏920|tsv110|
|HiSilicon HIP09|hip09|
|HiSilicon HIP10c|hip10c|
|HiSilicon HIP11|hip11|
|鲲鹏950|hip12|
