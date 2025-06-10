# Kernel FDO User Guide

## Overview

The feedback-directed optimization (FDO) of the kernel allows users to build optimized kernels for different applications to improve the application performance in single-application scenarios. In addition, FDO is integrated GCC for openEuler, and A-FOT provides automatic optimization, enabling users to easily enable FDO.

## Installation and Deployment

### Software Requirements

* OS: openEuler 23.09

### Hardware Requirements

* Architecture: AArch64 or x86_64

### Software Installation

#### Downloading the Kernel Source Code

Download the kernel source code, A-FOT, and dependency packages.

```shell
yum install -y kernel-source A-FOT make gcc flex bison elfutils-libelf-devel diffutils openssl-devel dwarves
```

Copy the kernel source code.

```shell
cp -r /usr/src/linux-6.4.0-8.0.0.16.oe2309.aarch64 .
```

**Note: Change the version number as required.**

## Usage

You can use A-FOT to enable kernel FDO and obtain the optimized kernel by specifying **opt_mode** as **Auto_kernel_PGO**. Other configuration items can be specified on the CLI, for example, `./a-fot --pgo_phase 1`. `-s` and `-n` options can be specified on CLI only. Options related to kernel FDO are as follows.

| No.| Option (Configuration File)| Description                                                    | Default Value                  |
| ---- | -------------------- | ------------------------------------------------------------ | ------------------------ |
| 1    | config_file          | Path of the configuration file. User configurations are read from this file.              | ${afot_path}/a-fot.ini   |
| 2    | opt_mode             | Optimization mode to be executed by the tool. The value can be **AutoFDO**, **AutoPrefetch**, **AutoBOLT**, or **Auto_kernel_PGO**.| AutoPrefetch             |
| 3    | pgo_mode             | Kernel FDO mode, which can be **arc** (GCOV, using only the arc profile) or **all** (full PGO, using the arc and value profiles).                | all                      |
| 4    | pgo_phase            | FDO execution phase. The value can be **1** (kernel instrumentation phase) or **2** (data collection and kernel optimization phase).          | 1                        |
| 5    | kernel_src           | Kernel source code directory. If this option is not specified, the tool automatically downloads the source code.  | None (optional)              |
| 6    | kernel_name          | File name of the kernel build. The tool will add the **-pgoing** or **-pgoed** suffix depending on the phase. | kernel                   |
| 7    | work_path            | Script working directory, which is used to store log files, wrappers, and profiles.      | **/opt** (**/tmp** cannot be used.)|
| 8    | run_script           | Application execution script. The user needs to write the script, which will be used by the tool to execute the target application.| /root/run.sh             |
| 9    | gcc_path             | GCC path.                        | /usr                     |
| 10   | -s                   | Silent mode. The tool automatically reboots and switches the kernel to execute the second phase.           | None                       |
| 11   | -n                   | The tool is not used to compile the kernel. This option applies to the scenario where the execution environment and kernel compilation environment are different. | None                       |

After configuring the compilation options, run the following command to use A-FOT to automatically optimize the kernel:

```shell
a-fot --config_file ./a-fot.ini -s
```

**Note: The `-s` option instructs A-FOT to automatically reboot into the compiled kernel. If you do not want the tool to automatically perform this sensitive operation, omit this option. However, you need to manually reboot and perform the second phase (`--pgo_phase 2`).**

**Note: All paths must be absolute paths.**

**Note: The kernel of openEuler 23.09 does not support full PGO. Change the value of pgo_mode to arc.**

## Compatibility

This section describes the compatibility issues in some special scenarios. This project is in continuous iteration and issues will be fixed as soon as possible. Developers are welcome to join this project.

* The kernel of openEuler 23.09 does not support full PGO. Change the value of pgo_mode to arc.
