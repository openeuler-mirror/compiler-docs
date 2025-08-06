# 动态反馈优化框架

## 介绍

D-FOT是动态反馈优化框架（**D**ynamic **F**eedback-oirected **O**ptimization **T**ool），目标是实现应用全自动、无感知使能反馈优化特性，全面提高反馈优化特性的易用性和性能表现。
本框架计划实现两类动态优化特性：启动时优化和运行时优化，当前已实现基于oeAware在线调优框架的启动时优化功能。
启动时优化：应用运行时自动执行采样和反馈优化，完成优化后自动接管下一次启动，不需要用户修改启动流程，应用重启后自动拉起优化版本。
运行时优化：应用运行时自动执行采样和反馈优化，不需要用户操作重启，整个过程无用户介入，仅少量中断后即可使能优化版本。

## 软件架构说明

本框架包含以下几个模块：

- 基于libkperf的采样数据处理
- 基于sysboost的启动接管和二进制优化特性使能
- 基于llvm-bolt的二进制反馈优化
- oeAware调优插件dfot_tuner_sysboost实现

## 依赖项

系统OS要求：当前支持openEuler-22.03-LTS-SP4、openEuler-25.03。

以下依赖组件均可通过yum安装。

| 组件 | 代码仓 | 说明 |
| --------------- | ---------------- | ----------- |
| oeAware-manager | [https://gitee.com/openeuler/oeAware-manager](https://gitee.com/openeuler/oeAware-manager) |  业务在线无感调优框架  |
| libkperf        | [https://gitee.com/openeuler/libkperf](https://gitee.com/openeuler/libkperf)               |  轻量级全内存化采集工具 |
| sysboost        | [https://gitee.com/openeuler/sysboost](https://gitee.com/openeuler/sysboost)               |  微架构优化工具     |
| llvm-bolt       | [https://gitee.com/src-openeuler/llvm-bolt](https://gitee.com/src-openeuler/llvm-bolt)                                                  |  二进制优化器      |

## 使用流程

### 被优化应用的必要条件

1. 被优化二进制需要带有重定位信息。使用自编译的软件：需要在编译时增加`-Wl,-q`链接选项，如MySQL：`cmake .. -DCMAKE_INSTALL_PREFIX=/usr/local/mysql -DBUILD_CONFIG=mysql_release -DWITH_BOOST=../boost -DCMAKE_C_LINK_FLAGS="-Wl,-q" -DCMAKE_CXX_LINK_FLAGS="-Wl,-q"`；使用oe软件包的场景：后续会提供对应应用的relocation包，直接安装即可。
2. 如何判断目标二进制是否带有重定位信息：`-Wl,-q`生效后，二进制会增加RELA段，可以通过`readelf -SW /path/to/bin`来判断，如MySQL加选项之前，仅有`.rela.dyn`和`.rela.plt`段，增加后会出现`.rela.text`、`.rela.eh_frame`等10+ RELA段；如果`-Wl,-q`未生效，则在手动perf采样并执行`perf2bolt`时，或者执行`llvm-bolt`优化时（无论是否通过sysboost机制）会有告警：`BOLT-WARNING: non-relocation mode for AArch64 is not fully supported`。

### D-FOT准备

yum安装D-FOT或通过以下命令源码构建（注意如果libkperf或oeAware-manager也是源码构建，执行cmake时需要额外指定lib和include路径）。

```shell
cd D-FOT
mkdir build && cd build
cmake ../ -DCMAKE_BUILD_TYPE=RelWithDebInfo -DCMAKE_SKIP_RPATH=TRUE
make -j`nproc`

cp build/libdfot.so /lib64/oeAware-plugin/
chmod 440 /lib64/oeAware-plugin/libdfot.so
mkdir -p /etc/dfot/
cp configs/dfot.ini /etc/dfot/
```

### 配置修改

修改`/etc/dfot/dfot/ini`，内容项参考如下描述：

公共配置：[general]

| 配置项                           | 值范围                                 | 是否可用 | 说明                                                                                     |
| ----------------------------- | ----------------------------------- | ---- | -------------------------------------------------------------------------------------- |
| LOG_LEVEL                     | `<FATAL, ERROR, WARN, INFO, DEBUG>` | 可用   | 优化服务日志级别，注意级别越低打印的日志越多                                                                 |
| COLLECTOR_SAMPLING_STRATEGY   | `<0>`                               | 不可用  | 采样策略<br>0表示插件enable后即持续低频采样<br>1表示enable时启动监控线程，只有负载达到阈值场景才采样<br>当前由oeaware控制采样流程，仅支持0 |
| COLLECTOR_HIGH_LOAD_THRESHOLD | `[0, cpus*100]`                     | 不可用  | 仅在采样策略1场景下生效，使用HIGH_LOAD_THRESHOLD作为触发采样的应用CPU使用率阈值，当前不支持                              |
| COLLECTOR_DATA_AGING_TIME     | 按实际需要确定                             | 可用   | 采样数据老化时间，当前数据与最老数据时间差值达到阈值时，丢弃累积采样数据，单位ms                                              |
| TUNER_TOOL                    | `["sysboost"]`                      | 不可用  | 二进制优化器，当前仅支持sysboost                                                                   |
| TUNER_CHECK_PERIOD            | `[10, max]`                         | 可用   | 优化插件检查时间间隔，每隔一段时间收集采样插件数据并决定是否进行优化，单位ms                                                |
| TUNER_PROFILE_DIR             | 按实际需要确定                             | 可用   | 采样数据存放位置，profile文件被命名为`[app_name]_[full_path_hash]_[threshold].profile`                |
| TUNER_OPTIMIZING_STRATEGY     | `[0, 1]`                            | 可用   | 优化策略，0表示只优化一次，1表示只要采样信息在刷新，可以持续多次优化                                                    |
| TUNER_OPTIMIZING_CONDITION    | `[0, 2]`                            | 不可用  | 触发优化的条件，0表示应用退出后即开始优化，1表示低负载时优化，2表示应用退出且低负载时优化，当前仅支持0                                  |

应用配置：[app]

| 配置项                           | 值范围            | 是否可用 | 说明                                                                                         |
| ----------------------------- | -------------- | ---- | ------------------------------------------------------------------------------------------ |
| FULL_PATH                     | 按实际需要确定        | 可用   | 应用二进制文件绝对路径                                                                                |
| DEFAULT_PROFILE               | 按实际需要确定        | 可用   | 应用的开箱profile文件，用于冷启动时使能二进制优化，没有则留空                                                         |
| COLLECTOR_DUMP_DATA_THRESHOLD | `[10000, max]` | 可用   | 采样数据达到该阈值行数时触发数据导出到profile，数值越大需要的采集的samples越多                                             |
| BOLT_DIR                      |    NA        | 不可用  | BOLT工具路径，留空则默认/usr/bin，内部会调用`${BOLT_DIR}/perf2bolt和${BOLT_DIR}/llvm-bolt`<br>当前由sysboost确定 |
| BOLT_OPTIONS                  | 按实际需要确定        | 可用   | BOLT优化选项，配置该项可以覆盖内置的默认选项，用于针对性的选项调优                                                        |
| UPDATE_DEBUG_INFO             | `[0, 1]`       | 可用   | 优化时是否同步更新调试信息，1表示更新，0表示不更新，注意更新调试信息会有额外耗时                                                  |

### 插件使用

参考[oeAware-manager](https://gitee.com/openeuler/oeAware-manager/tree/master/docs)。

## 约束限制

1. 支持容器部署场景使用，D-FOT等优化组件需部署在容器外。
2. 待优化的目标应用需要具备重定位信息。
3. 作为oeAware调优插件使用D-FOT时，当前对应的oeAware采集插件pmu_sampling_collector固定采样频率为100，可能会导致采样时间较久（作为参考：perf默认采样频率为4000）。

## 未来规划

- [ ] 虚拟化场景完善
- [ ] 运行时优化支持
- [ ] 进一步提升二进制优化效果
