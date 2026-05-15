# New Architecture Support

LLVM for openEuler allows you to specify the current CPU model and enable all default features of the CPU using the `-mcpu=` option. After the microarchitecture affinity option is enabled, LLVM for openEuler optimizes the instruction pipeline based on the instruction implementation of the corresponding architecture to improve performance. The mapping between hardware platforms and configuration options is as follows:

|Hardware Platform|Configuration Option|
|-|-|
|Kunpeng 920|tsv110|
|HiSilicon Hip09|hip09|
|HiSilicon Hip10c|hip10c|
|HiSilicon Hip11|hip11|
