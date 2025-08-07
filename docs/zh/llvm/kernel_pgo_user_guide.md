# 内核反馈优化特性

## 简介

[PGO（Profile-Guided Optimization）](https://clang.llvm.org/docs/UsersManual.html#profile-guided-optimization)是一种编译器反馈优化技术，通过收集程序运行时的信息，指导编译器的优化决策。内核反馈优化（PGO kernel）特性为内核提供了反馈优化能力的支持，使用户可以为不同的应用程序构建针对性优化的内核，在单应用场景下提高目标应用的性能。根据业界经验，对于数据中心大型应用（如MySQL、Nginx、Redis等）通过PGO优化应用和内核有较好的效果。

## 环境准备

### 软件要求

* 操作系统：openEuler 24.03 LTS (SP1) 
* clang、llvm、lld：17.0.6

### 硬件要求

* AArch64架构
* x86_64架构

### 安装软件

安装内核源码、编译工具链及其他依赖包：

```bash
yum install -y kernel-source clang llvm lld flex bison rpm-build elfutils-libelf-devel dwarves openssl-devel rsync
```

复制内核源码：

```bash
cp -r /usr/src/linux-6.6.0-54.0.0.57.oe2403.aarch64 .
```

> [!WARNING]注意 \
> 具体版本号可能有变化。

## 使用方法

和一般的插桩式反馈优化类似，这里也先构建一个插桩版本，使用编译器插入的探针收集运行时信息并写入到文件，使用收集到的文件二次构建，指导编译器的优化决策。

### 插桩版本构建

```bash
make LLVM=1 LLVM_IAS=1 openeuler_defconfig
scripts/config -e PGO_CLANG
make LLVM=1 LLVM_IAS=1 binrpm-pkg -j$(getconf _NPROCESSORS_ONLN)
```

### 安装插桩版本内核并重启

```bash
rpm -ivh kernel-6.6.0-1.aarch64.rpm # 注意文件名可能不同
grub2-reboot 0                      # 指定下一次重启的启动项为刚安装的内核
reboot
```

### 收集profile信息

重置数据：

```bash
echo 1 > /sys/kernel/debug/pgo/reset
```

运行程序后，获取采集到的profile信息：

```bash
cp -a /sys/kernel/debug/pgo/vmlinux.profraw /tmp/vmlinux.profraw
llvm-profdata merge /tmp/vmlinux.profraw -o vmlinux.profdata
```

### 二次构建

```bash
make LLVM=1 LLVM_IAS=1 openeuler_defconfig
make LLVM=1 LLVM_IAS=1 KCFLAGS=-fprofile-use=/the/profile/data/path/vmlinux.profdata binrpm-pkg -j$(getconf _NPROCESSORS_ONLN)
```

### 安装优化版本内核并重启

```bash
rpm -ivh kernel-6.6.0-2.aarch64.rpm
grub2-reboot 0
reboot
```
