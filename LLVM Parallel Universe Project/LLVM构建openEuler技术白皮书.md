# LLVM构建openEuler技术白皮书
## 1、简介
## 2、LLVM编译器工具链介绍
[LLVM项目](https://llvm.org/)是一个[开源的编译器基础设施项目](https://github.com/llvm/llvm-project)，它提供了一套用于编译程序的工具链和库。近年来，LLVM项目越来越得到开发者的关注，社区非常活跃，商业公司纷纷基于LLVM项目推出商业编译器，OS社区也积极拥抱LLVM技术栈。[openEuler社区上的LLVM项目](https://gitee.com/openeuler/llvm-project)作为一个下游项目，致力于在开源LLVM基础上与openEuler协同创新，包括兼容性、性能和开发态安全编码特性，为openEuler上的编译器提供第二选择，并适配多种硬件平台，如鲲鹏、飞腾、龙芯等，充分释放多样性硬件算力。
### 2.1、LLVM架构描述
LLVM采用了模块化架构设计，将编译过程分为多个独立阶段，如前端、优化和后端。这种设计使得LLVM更加灵活和可扩展，有助于各阶段模块分别演进创新，而通过统一的IR表示又将不同的模块有机的结合起来。目前LLVM项目包含多个子项目，如clang、flang、llvm、mlir、lld等。LLVM 9.0版本之后采取Apache License。

<div align=center>

  <img align = "center" src="./images/LLVM架构.png" alt="LLVM架构" width="500" height="400" >

</div>
<p align="center"><font face="宋体">图1 LLVM架构：模块化解耦架构，统一IR表示，助力架构级创新 </font></p>

### 2.2、LLVM子项目简介
LLVM是一个伞形项目，包含模块化和可重复使用的编译器和工具链技术的集合。
LLVM的主要子项目有：
* LLVM Core：LLVM核心库提供了一个现代的独立于源码和目标的优化器，以及对许多流行CPU的代码生成支持。
* Clang：LLVM原生的C-family语言编译器。
* LLDB：构建在LLVM和Clang提供的库之上，以提供一个出色的原生调试器。
* libc++和libc++ ABI：提供一个符合标准且高性能的C++标准库实现。
* compiler-rt：编译器运行时库，包含一些低级别的代码生成的支持函数，也为动态测试工具提供运行时库。
* MLIR：一种构建可重用和可扩展编译器基础设施的新方法。
* OpenMP：提供了一个OpenMP运行时实现。
* polly：使用多面体模型实现了一套缓存位置优化以及自动并行和矢量化。
* libclc：OpenCL标准库的一个实现。
* klee：符号执行虚拟机的一个实现。
* LLD：LLVM原生链接器。
* BOLT：后链接优化器，通过优化应用程序的代码布局来达成优化效果。

### 2.3、基于Clang组装一个完整工具链
本章节参考[Assembling a Complete Toolchain](https://clang.llvm.org/docs/Toolchain.html)。
Clang只是C-family编程语言完整工具链中的一个组件。为了组装一个完整的工具链，需要额外的工具和运行时库。Clang被设计为与用于其目标平台的现有工具和库进行交互操作，并且LLVM项目为许多这些组件提供了替代方案。
下面描述类POSIX操作系统（如linux）上的Clang配置。
#### 2.3.1、工具
C-family编程语言的完整编译通常涉及以下工具：
* 预处理器(Preprocessor)
  * 执行 C 预处理器的操作：展开#include和#defines。
* 解析(Parsing)
  * 根据输入，解析和语义分析源语言，并构建一个源代码级中间表示(“AST”)。
* IR生成(IR generation)
  * 将源代码级中间表示转换为一个特定于优化器(optimizer)的中间表示(IR)，即LLVM IR。
* 编译器后端(Compiler backend)
  * 将中间表示转换为特定于目标的汇编代码。
* 汇编器(Assembler)
  * 将特定于目标的汇编代码转换为特定于目标的机器码目标文件。
* 链接器(Linker)
  * 将多个目标文件组合成一个映像（一个共享对象或一个可执行文件）。

Clang提供了除链接器之外的所有这些部分。当同一工具执行多个步骤时，通常将这些步骤合并在一起，以避免创建中间文件。
#### 2.3.2、Clang前端
Clang前端(clang -cc1)用于编译C-family语言。Clang前端的命令行接口被视为实现细节，故没有外部文档，并且可以在提示的情况下进行更改。
#### 2.3.3、汇编器
Clang既可以使用LLVM项目的集成汇编器，也可以使用外部特定于系统的汇编器（例如，GNU操作系统上的 GNU汇编器），作用是从汇编代码生成机器码。默认情况下，Clang在支持LLVM的所有目标机器上使用 LLVM的集成汇编器。如果想使用特定系统的汇编器，请使用`-fno-integrated-as`选项。
#### 2.3.4、链接器
Clang可以配置为使用几个不同的链接器其中一个：
* GNU ld
* GNU gold
* LLVM lld
* MSVC link.exe
[LLVM lld](https://lld.llvm.org/)原生支持链接时优化，使用gold时通过一个[链接器插件](https://llvm.org/docs/GoldPlugin.html)支持链接时优化。默认链接器在不同的目标机器上是不同的，可以通过`-fuse-ld=<linker name>`标志来切换。
#### 2.3.5、运行时库
C-family程序需要许多不同的运行时库提供不同的支持。Clang将隐式地链接每个运行时库的合适实现。
##### 2.3.5.1、编译器运行时
编译器运行时库提供编译器隐式调用的函数的定义，以支持底层硬件不支持的操作(例如，128位整数乘法)，或用在某些操作不适合用编译器内部实现的地方。
默认的运行时库是特定于目标架构的。对于GCC是主要编译器的目标，目前默认使用`libgcc_s`。在大多数其他目标上，默认情况下使用`compiler-rt`。
* compiler-rt (LLVM): [LLVM项目的编译器运行时库](https://compiler-rt.llvm.org/)提供了一组完整的运行时库函数。
* libgcc_s (GNU)：[GCC编译器的运行时库](https://gcc.gnu.org/onlinedocs/gccint/Libgcc.html)可以用来代替`compiler-rt`。但是，它缺少几个LLVM可能调用的函数，特别是在使用Clang的内置函数家族的`__builtin_*_overflow`时。

可以通过`rtlib=libgcc`或`--rtlib=libgcc`来切换编译器运行时库。
##### 2.3.5.2、原子库
如果您的程序使用了原子操作，编译器无法直接翻译到机器指令（因为没有合适的机器指令或不知道操作数如何适当对齐），将会生成对运行时库__atomic_*函数的调用。这些程序需要一个包含这些原子函数的运行时库。
* compiler-rt (LLVM)：LLVM项目的原子库的实现包含在`compiler-rt`中。
* libatomic (GNU)：libgcc_s不提供原子库的实现，事实上，GCC的[libatomic library](https://gcc.gnu.org/wiki/Atomic/GCCMM)在使用libgcc_s时被提供。
> 注意：
> 当Clang使用libgcc_s时，目前不会自动链接到`libatomic`。在使用非`compiler-rt`提供的原子操作时（如果您看到引用了__atomic_*函数的链接错误），可能需要手动添加`-latomic`来支持这种配置。
##### 2.3.5.3、Unwind库
Unwind库提供了一系列`_Unwind_*`函数，实现了`Itanium C++ ABI`([第I级](https://itanium-cxx-abi.github.io/cxx-abi/abi-eh.html#base-abi))的语言无关的堆栈Unwind操作部分，它是`C++ ABI`库的依赖项，有时是其他运行时的依赖项。
* libunwind (LLVM)：LLVM项目的Unwind库一种实现。
* libgcc_s (GNU)：libgcc_s有一个集成的Unwinder，不需要提供外部的Unwind库。
* libunwind (nongnu.org)：这是Unwind规范的另一个实现。请参阅[libunwind(nongnu.org)](https://www.nongnu.org/libunwind/)。
* libunwind (PathScale): 这是Unwind规范的另一个实现。请参阅[libunwind (pathscale)](https://github.com/pathscale/libunwind)。
##### 2.3.5.4、Sanitizer运行时库
Clang的sanitizers (-fsanitize=…)隐式地调用一个运行时库，以便维护程序执行的状态，并在检测到问题时发出诊断消息。
这些运行时的唯一支持实现由 LLVM 的compiler-rt提供。
#### 2.3.6、C标准库
Clang支持多种[C标准库实现](https://en.cppreference.com/w/c)。
#### 2.3.7、C++标准库
Clang支持使用LLVM项目的`libc++`或GCC的`libstdc++`C++标准库实现。
* libc++ (LLVM)：[libc++](https://libcxx.llvm.org/)是LLVM项目的C++标准库实现，旨在从C++ 11开始成为全面的C++标准实现。
* libstdc++ (GNU)：[libstdc++](https://gcc.gnu.org/onlinedocs/libstdc++/)是GCC 的C++标准库实现，Clang支持各种版本的`libstdc++`。

您可以通过`-stdlib=libc++`或`-stdlib=libstdc++`选项来切换C++标准库。

#### 2.3.8、C++ ABI库
C++ ABI库提供了Itanium C++ ABI库部分的实现，包括[Itanium c++ ABI文档中的支持功能](https://itanium-cxx-abi.github.io/cxx-abi/abi.html)和[异常处理支持的Level II](https://itanium-cxx-abi.github.io/cxx-abi/abi-eh.html#cxx-abi)。在编译C++代码时，Clang隐式地生成对这个库中函数和对象的引用。
> 注意
> 虽然同一个程序可能同时使用libstdc++和libc++（只要您不试图将C++标准库对象传递到边界之外），但是在一个程序中通常不可能有一个以上的C++ABI库。

* libc++ abi (LLVM)：[libc++abi](https://libcxxabi.llvm.org/)是LLVM项目对该规范的实现。
* libsupc++ (GNU)：libsupc++是GCC对该规范的实现。但是，只有在静态链接libstdc++时才使用这个库。libstdc++的动态库版本包含libsupc++的一个副本。
* libcxxrt (PathScale)：这是Itanium C++ ABI规范的另一个实现。

## 3、LLVM平行宇宙计划
23年3月份，软件所OERV团队和华为毕昇编译器团队在RISCV SIG和Compiler SIG联合在社区发起“LLVM平行宇宙计划”。该项目旨在通过社区化的方式推动LLVM工具链在openEuler社区的应用，包括构建openEuler系统。

### 3.1、计划描述
该计划已提交[oEEP](https://gitee.com/openeuler/TC/blob/master/oEEP/oEEP-0003%20LLVM%E5%B9%B3%E8%A1%8C%E5%AE%87%E5%AE%99%E8%AE%A1%E5%88%92--%E5%9F%BA%E4%BA%8ELLVM%E6%8A%80%E6%9C%AF%E6%A0%88%E6%9E%84%E5%BB%BAoE%E8%BD%AF%E4%BB%B6%E5%8C%85.md)，目前该oEEP状态处于`初始化`状态。

<div align=center>

  <img align = "center" src="./images/LLVM平行宇宙计划推进策略.png" alt="推进策略" width="600" height="280" >

</div>
<p align="center"><font face="宋体">图2 LLVM平行宇宙计划推进策略。</font></p>

通过社区协作的方式推进两个策略：
* **版本**：通过版本化构建，一方面厘清工作边界，另一方面可以让开发者、使用者对LLVM平行宇宙计划更加有信心。实际操作过程中可以通过“Preview版本”、“平行版本”、“正式版本”的方式循序渐进，形成一个“开发->发布->使用->问题反馈”的良性循环。
* **长效机制**：与Compass-CI结合，构建upstream测试，解决软件包版本升级后需要重复解决LLVM构建&运行问题，同时有效的卷入上游开发者贡献，达成LLVM原生构建、构建结果可视化的目的，增强开发者、使用者信心。

### 3.2、价值与意义
该计划致力于推动LLVM工具链在openEuler社区的应用，以获取如下收益：
* 助力openEuler开箱竞争力更优。
（1）协同openEuler使能更多高级优化，提升开箱性能、降低镜像体积。
（2）代码静态&动态检查，提升openEuler社区代码质量，提升可靠性和安全性。
* 助力openEuler繁荣生态。
（1）支持仓颉/Rust等多种语言对接，繁荣openEuler社区开发者数量。
（2）AI&异构支持，助力openEuler支持多样性算力生态。
（3）支持LLVM社区开发者无缝对接openEuler，促进openEuler竞争力和生态创新。

## 4、技术方案
### 4.1、openEuler软件包构建方式
openEuler软件包非常丰富，可以分为内核态软件包（南向）和用户态软件包（北向）。截止24.03 LTS版本，用户态软件包有3.6W+，而且仍在不断增加中。
当前（24.03 LTS之前），openEuler软件包默认的构建编译器是GNU工具链。

### 4.2、备选技术方案可行性分析
[Clang编译器](https://clang.llvm.org/)的一个设计需求是兼容GCC，所以理论上是可以替换GCC而构建不同的软件包。不过，Clang编译器作为一个后发者，存在一些软件包无法直接构建通过的情况，基于这个前提，软件包切换构建编译器有如下两个维度需要考虑：
* Clang构建软件包的范围。不同的版本上构建的软件包范围有哪些？
* Clang工具链的组件如何选择。参考2.3章节的描述组装一个完整的工具链。
#### 4.2.1、备选方案一
方案描述：openEuler**全部的软件包**基于Clang构建，Clang编译器工具链**全部选用LLVM原生的组件**。
方案可行性分析：openEuler存在大量软件包（3.6W+），同时存在个别软件包暂无法直接基于Clang构建（如glibc）。
方案结论：短期内不可行。

#### 4.2.2、备选方案二
方案描述：openEuler**部分软件包**基于Clang构建，Clang编译器工具链**全部选用LLVM原生的组件**。
方案可行性分析：基于2.3.8章节“注意”部分的描述，如果一个程序同时使用libstdc++和libc++时，必须非常小心地处理两个标准库的应用边界。更有甚者，一个程序中不允许同时存在两个C++ABI库。
方案结论：存在GCC和Clang混合使用的情况下，该方案不可行。

#### 4.2.3、备选方案三
方案描述：openEuler**部分软件包**基于Clang构建，Clang编译器工具链采用`Clang+llvm+lld+GNU运行时库`组合。
方案可行性分析：GCC和Clang混合构建，此方案也是业界目前已采用的方案，是Clang编译器的实际应用场景。
方案结论：方案总体是可行的，需要进一步进行白盒分析和黑盒验证。

#### 4.2.4、kernel态软件包
内核态的软件包相对用户态特殊，内核态软件包包含kernel及会构建出`.ko`的软件包。根据[Linux内核驱动接口](https://www.kernel.org/doc/html/latest/process/stable-api-nonsense.html)描述了两个概念：内核二进制接口和内核源码接口
对于内核源码接口，文章中进行了非常详尽的阐述，这里不赘述。
我们这里需要考虑的是内核二进制接口，因为不管是GCC还是Clang构建，我们首先要保证源码的一致性，所以源码接口应该是一致的。关于内核二进制接口，文章中有如下描述：
> Assuming that we had a stable kernel source interface for the kernel, a binary interface would naturally happen too, right? Wrong. Please consider the following facts about the Linux kernel:
>
> * Depending on the version of the C compiler you use, different kernel data structures will contain different alignment of structures, and possibly include different functions in different ways (putting functions inline or not.) The individual function organization isn’t that important, but the different data structure padding is very important.

即，当使用不同编译器的（甚至不同编译器版本）的情况，会出现二进制不兼容的情况，这是因为kernel中可能会用到C语言不可移植的特性（如变成数组）。值得欣慰的是，近些年[ClangBuiltLinux项目](https://clangbuiltlinux.github.io/)解决了这些问题。但使用不同的编译器版本构建kernel，仍可能会导致KABI不一致。
综上所述，较好的方案是用同一个编译器(版本)构建整个kernel态软件包。

### 4.3 建议技术方案
Kernel态使用LLVM构建，用户态软件包采用GCC和LLVM混编。
编译器组件采用Clang+llvm+llvm as+lld+GNU运行时库的方案。

<div align=center>

  <img align = "center" src="./images/技术方案.png" alt="推进策略" width="400" height="400" >

</div>
<p align="center"><font face="宋体">图3 LLVM构建openEuler软件包技术方案。</font></p>

## 5、关键技术挑战
## 6、应用场景
## 7、展望
