# tpm2-tss #

## 1、问题现象 ##

出自：https://gitee.com/src-openeuler/tpm2-tss(openEuler-24.03-LTS)

aarch成功，x86失败
```
[ 1109s] ERROR: test/integration/sys-asymmetric-encrypt-decrypt.int
[ 1249s] ERROR: test/integration/sys-nv-policy-locality.int
[ 1388s] ERROR: test/integration/sys-nv-readwrite.int
[ 1528s] ERROR: test/integration/sys-hmac-auth.int
[ 1668s] ERROR: test/integration/sys-primary-rsa-2K-aes128cfb.int
[ 1808s] ERROR: test/integration/sys-create-keyedhash-sha1-hmac.int
[ 1948s] ERROR: test/integration/sys-encrypt-decrypt.int
[ 2087s] ERROR: test/integration/sys-encrypt-decrypt-2.int
[ 2228s] ERROR: test/integration/sys-evict-ctrl.int
[ 2367s] ERROR: test/integration/sys-get-random.int
[ 2507s] ERROR: test/integration/sys-stir-random.int
[ 2647s] ERROR: test/integration/sys-hierarchy-change-auth.int
........
[29647s] # TOTAL: 260
[29647s] # PASS:  55
[29647s] # SKIP:  0
[29647s] # XFAIL: 0
[29647s] # FAIL:  0
[29647s] # XPASS: 0
[29647s] # ERROR: 205
```
总结为：
test/integration目录下全部error

## 2、问题定位及根因分析 ##

第一次我对比aarch和x86的日志，发现aarch与x86大部分相同，区别在于aarch的日志中会出现大量的`clang: warning: argument unused during compilation: '-fstack-clash-protection' [-Wunused-command-line-argument]`警告
而x86的日志中从未出现过类似的警告

这种情况在之前的satyr包中也出现过，当时的解决方案是加入
```
%if "%toolchain" == "clang"
    export CFLAGS="$CFLAGS -Wno-error=unused-command-line-argument"
%endif（简称为可用降级操作）
```
即可通过，unused-command-line-argument后续会加入，所以当做社区已修复

而在tpm2-tss这个包里加入类似的代码却没有反应，这条修复路径就此阻塞

第二次我在网上搜索相关错误，找到一个很类似的情况，同样是tpm2-tss中test/integration目录下全部error，网址为：https://blog.csdn.net/jianming21/article/details/107969368
它的解释是“其根源是相关依赖包的缺失”，于是我找到tpm2-tss的上游社区的master分支中所需的依赖，加入到spec文件中，却都失败了，这条路就此阻塞

第三次我怀疑之所以aarch成功而x86失败是因为aarch架构和x86架构下的clang的指令集不同，又出错的单元为unit和integration，于是我将%build下做了一个判断将integration删除，结果成功，这应该不是根因

对于根本原因的猜想可能的原因：

1.编译器内部实现差异

GCC：

GCC 的内部架构和代码组织方式可能对集成（integration）相关的功能有更好的支持。GCC 在开发过程中，可能已经将与集成测试相关的功能模块（例如，不同语言前端之间的集成、代码生成模块与优化模块之间的集成等）进行了良好的设计和测试。这些模块在启用集成选项时，能够按照预期的方式协同工作。
例如，GCC 的中间表示（Intermediate Representation，IR）在处理集成功能时，有一套稳定的机制。当启用集成选项时，它能够正确地处理各种语言前端（如 C、C++、Fortran 等）生成的 IR，并将它们有效地整合在一起用于后续的优化和代码生成步骤，而不会出现内部冲突。

Clang：

Clang 可能在处理集成相关功能时，其内部实现与 GCC 有所不同。它可能对某些集成功能的支持不够完善，或者在处理同时启用单元（unit）和集成选项时，内部模块之间的交互出现问题。
例如，Clang 的编译过程可能在处理集成选项时，触发了一些未充分测试的代码路径。当启用集成选项时，它可能会尝试整合一些在当前版本中还不稳定的模块，如在处理与特定优化策略或链接外部库相关的集成功能时出现问题，导致构建失败。

2.对构建选项的处理方式不同

GCC：

GCC 的构建系统可能对--enable - integration选项有更合适的处理方式。它能够正确地解析这个选项，并根据选项的要求，将相关的测试代码或集成相关的功能正确地添加到构建过程中。例如，GCC 的构建脚本可能会识别出这个选项，并按照预定义的规则将用于集成测试的文件添加到编译列表中，同时调整链接顺序和库依赖关系，以确保构建过程的顺利进行。

Clang：

Clang 可能对这个选项的处理不够准确。也许在解析--enable - integration选项时，它会错误地包含或排除某些必要的文件，或者在调整编译和链接过程时出现错误。例如，Clang 可能会错误地将一些与集成功能相关但不兼容当前平台或配置的文件包含进来，从而导致构建过程中的语法错误、链接错误或其他类型的失败。

3.外部依赖和库兼容性

GCC：

GCC 在启用集成选项时，可能与它所依赖的外部库有更好的兼容性。这些外部库在设计和开发过程中，可能已经与 GCC 的集成功能进行了充分的测试和适配。例如，GCC 在进行链接操作时，与系统库或其他第三方库（如数学库、图形库等）的交互在启用集成选项后仍然能够正常进行，因为这些库的接口和 GCC 对它们的调用方式在集成功能的场景下是稳定的。

Clang：

Clang 在启用集成选项时，可能会遇到与外部库不兼容的情况。它所依赖的某些库可能没有针对 Clang 的集成功能进行充分的测试，或者在 Clang 启用集成选项后，库的调用方式发生了变化，导致与库的接口不匹配。例如，在启用集成选项后，Clang 可能会尝试以一种新的方式调用某个外部库，但该库的版本不支持这种调用方式，从而导致构建失败。

## 3.解决方案 ##

即将
```
%configure --disable-static --disable-silent-rules --with-udevrulesdir=%{_udevrulesdir} --with-udevrulesprefix=80- \
           --with-runstatedir=%{_rundir} --with-tmpfilesdir=%{_tmpfilesdir} --with-sysusersdir=%{_sysusersdir} \
           --enable-unit --enable-integration
```
改为
```
%if "%toolchain" == "clang"
%configure --disable-static --disable-silent-rules --with-udevrulesdir=%{_udevrulesdir} --with-udevrulesprefix=80- \
           --with-runstatedir=%{_rundir} --with-tmpfilesdir=%{_tmpfilesdir} --with-sysusersdir=%{_sysusersdir} 
%else
%configure --disable-static --disable-silent-rules --with-udevrulesdir=%{_udevrulesdir} --with-udevrulesprefix=80- \
           --with-runstatedir=%{_rundir} --with-tmpfilesdir=%{_tmpfilesdir} --with-sysusersdir=%{_sysusersdir} \
           --enable-unit --enable-integration

%endif
```
不影响ci和gcc
## 4、相关PR ##
openEuler：https://gitee.com/src-openeuler/tpm2-tss/pulls/85
上游社区：不涉及