# Kernel PGO

## Overview

[Profile-guided optimization (PGO)](https://clang.llvm.org/docs/UsersManual.html#profile-guided-optimization) is a feedback-directed compiler optimization technology that collects program runtime information to guide the compiler through optimization decision-making. The kernel FDO allows users to build optimized kernels for different applications to improve the application performance in single-application scenarios. Based on industry experience, PGO can be used to optimize large-scale data center applications (such as MySQL, Nginx, and Redis) and kernels.

## Environment Preparations

### Software Requirements

* OS: openEuler 24.03 LTS (SP1)
* Clang, LLVM, LLD: 17.0.6

### Hardware Requirements

* AArch64 architecture
* x86_64 architecture

### Software Installation

Install the kernel source code, compilation toolchain, and other dependency packages.

```bash
yum install -y kernel-source clang llvm lld flex bison rpm-build elfutils-libelf-devel dwarves openssl-devel rsync
```

Copy the kernel source code.

```bash
cp -r /usr/src/linux-6.6.0-54.0.0.57.oe2403.aarch64 .
```

> ![WARNING](./public_sys_resources/icon_warning.gif)
> 
> The version number may vary.

## How to Use

Similar to common instrumentation-based FDO, an instrumentation version is built first. A probe inserted by the compiler is used to collect runtime information and write the information to a file. The collected file is then used for secondary build to guide the compiler through optimization decision-making.

### Building an Instrumentation Version

```bash
make LLVM=1 LLVM_IAS=1 openeuler_defconfig
scripts/config -e PGO_CLANG
make LLVM=1 LLVM_IAS=1 binrpm-pkg -j$(getconf _NPROCESSORS_ONLN)
```

### Installing and Restarting the Instrumentation Version Kernel

```bash
rpm -ivh kernel-6.6.0-1.aarch64.rpm # Note that the file name may vary.
grub2-reboot 0                      # Specify the newly installed kernel as the boot option for the next reboot.
reboot
```

### Collecting Profile Information

Reset data.

```bash
echo 1 > /sys/kernel/debug/pgo/reset
```

After running the program, obtain the collected profile information.

```bash
cp -a /sys/kernel/debug/pgo/vmlinux.profraw /tmp/vmlinux.profraw
llvm-profdata merge /tmp/vmlinux.profraw -o vmlinux.profdata
```

### Secondary Build

```bash
make LLVM=1 LLVM_IAS=1 openeuler_defconfig
make LLVM=1 LLVM_IAS=1 KCFLAGS=-fprofile-use=/the/profile/data/path/vmlinux.profdata binrpm-pkg -j$(getconf _NPROCESSORS_ONLN)
```

### Installing and Restarting the Optimized Version Kernel

```bash
rpm -ivh kernel-6.6.0-2.aarch64.rpm
grub2-reboot 0
reboot
```
