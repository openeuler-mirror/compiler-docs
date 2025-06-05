# 内核反馈优化特性用户指南

## 简介

内核反馈优化（PGO kernel）特性为内核提供了反馈优化能力的支持，使用户可以为不同的应用程序构建针对性优化的内核，在单应用场景下提高目标应用的性能。同时，该特性一并在openEuler GCC内提供了相应的编译支持，以及在A-FOT中提供了自动优化的功能，使用户能够便捷地使能内核反馈优化特性。

## 安装与部署

### 软件要求

* 操作系统：openEuler 23.09

### 硬件要求

* aarch64架构
* x86_64架构

### 安装软件

安装内核源码、A-FOT和其他依赖软件包

```shell
yum install -y kernel-source A-FOT make gcc flex bison elfutils-libelf-devel diffutils openssl-devel dwarves
```

复制内核源码

```shell
cp -r /usr/src/linux-6.4.0-8.0.0.16.oe2309.aarch64 .
```

**注意：具体的版本号可能会有变化。**

## 使用方法

用户可以通过A-FOT工具使能内核反馈优化，一键得到优化内核。将opt_mode指定Auto_kernel_PGO则为PGO kernel模式。所有配置选项也可以通过命令行指定，例如./a-fot --pgo_phase 1，另外-s、-n选项只能在命令行指定。PGO kernel相关的选项说明如下表所示。

| 序号 | 选项名称（配置文件） | 选项说明                                                     | 默认值                   |
| ---- | -------------------- | ------------------------------------------------------------ | ------------------------ |
| 1    | config_file          | 配置文件路径；根据此文件内容读取用户的选项配置。             | ${afot_path}/a-fot.ini   |
| 2    | opt_mode             | 优化模式；工具将执行的优化模式，必须为AutoFDO、AutoPrefetch、AutoBOLT、Auto_kernel_PGO四者之一；分别代表自动反馈优化、自动预取、自动二进制优化和自动内核反馈优化。 | AutoPrefetch             |
| 3    | pgo_mode             | PGO模式；内核的反馈优化模式，GCOV或完整的PGO*，必须为arc或all；分别代表仅使用arc profile和使用arc+value profile。 | all                      |
| 4    | pgo_phase            | 内核反馈优化的执行阶段；工具根据阶段执行不同的操作，必须为1或2；1代表编译插桩内核的阶段，2代表收集数据、编译优化内核的阶段。 | 1                        |
| 5    | kernel_src           | 内核源码目录；指定则工具进入编译内核，否则工具自动下载源码。 | 无（可选）               |
| 6    | kernel_name          | 内核构建的本地名；工具将根据阶段添加"-pgoing"或"-pgoed"后缀。 | kernel                   |
| 7    | work_path            | 脚本工作目录；此目录用于存放日志文件、wrapper和profile。     | /opt（不能在/tmp目录下） |
| 8    | run_script           | 应用执行脚本路径；此脚本为目标应用的执行脚本，需要用户完成编写；工具将后台运行此脚本以执行目标应用。 | /root/run.sh             |
| 9    | gcc_path             | GCC路径；工具调用真正编译器GCC的路径。                       | /usr                     |
| 10   | -s                   | 安静模式；工具自动重启系统切换内核、执行第二阶段。           | 无                       |
| 11   | -n                   | 不要让工具来编译内核；适用于执行环境和内核编译环境分离的场景。 | 无                       |

配置完编译选项后，可以使用如下命令进行A-FOT自动化优化内核：

```shell
a-fot --config_file ./a-fot.ini -s
```

**注意：-s选项会让A-FOT工具自动重启机器切换内核，如果用户不希望自动进行这一项敏感操作，请去掉这一选项。但用户需要在重启后手动执行第二阶段（--pgo_phase 2）。**

**注意：所有路径名请使用绝对路径。**

**注意*：openEuler 23.09版本的内核暂不支持完整的PGO，请修改pgo_mode值为arc。**

## 兼容性说明

此节主要列出当前一些特殊场景下的兼容性问题。本项目持续迭代中，会尽快进行修复，也欢迎广大开发者加入。

* openEuler 23.09版本的内核暂不支持完整的PGO，目前只支持arc模式。
