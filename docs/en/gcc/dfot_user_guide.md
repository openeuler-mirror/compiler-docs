# D-FOT User Guide

## Overview

**D**ynamic **F**eedback-directed **O**ptimization **T**ool (D-FOT) aims to enable automatic and seamless feedback-directed optimizations for applications, improving both usability and performance.
The framework is planned to support two types of dynamic optimizations: startup time optimization and runtime optimization. Currently, startup time optimization has been implemented based on the oeAware online tuning framework.
Startup time optimization: During application running, sampling and feedback-directed optimization are automatically performed. Once optimization is complete, the optimized version is automatically used in the next startup without requiring any user intervention. After a restart, the application is automatically launched with the optimized version.
Runtime optimization: During application running, sampling and feedback-directed optimization are automatically performed. No user intervention is required. The optimized version can be enabled after minimal interruptions.

## Software Architecture Description

The framework consists of the following modules:

- Sampling data processing based on libkperf
- sysboost-based startup handover and binary optimization
- Binary feedback optimization based on llvm-bolt
- Implementation of the oeAware tuning plugin dfot_tuner_sysboost

## Dependency Items

OS requirements: Currently, openEuler 22.03 LTS SP4 and openEuler 25.03 are supported.
The following dependent components can be installed using `yum`.

| Component| Code Repository| Description|
| --------------- | ---------------- | ----------- |
| oeAware-manager | [https://gitee.com/openeuler/oeAware-manager](https://gitee.com/openeuler/oeAware-manager) |  Online transparent service tuning framework |
| libkperf        | [https://gitee.com/openeuler/libkperf](https://gitee.com/openeuler/libkperf)               |  Lightweight in-memory collection tool|
| sysboost        | [https://gitee.com/openeuler/sysboost](https://gitee.com/openeuler/sysboost)               |  Microarchitecture tuning tool    |
| llvm-bolt       | [https://gitee.com/src-openeuler/llvm-bolt](https://gitee.com/src-openeuler/llvm-bolt)                                                  |  Binary optimizer     |

## Procedure

### Prerequisites for Applications to Be Optimized

1. The binary to be optimized must contain relocation information. For self-compiled software, the `-Wl,-q` linker option must be added during compilation. For example, for MySQL, add `cmake .. -DCMAKE_INSTALL_PREFIX=/usr/local/mysql -DBUILD_CONFIG=mysql_release -DWITH_BOOST=../boost -DCMAKE_C_LINK_FLAGS="-Wl,-q" -DCMAKE_CXX_LINK_FLAGS="-Wl,-q"`. For openEuler software packages, the relocation package of the corresponding application will be provided and can be directly installed.
2. Check whether the target binary contains relocation information: When the `-Wl,-q` linker option is effective, the binary will include RELA sections. You can check this using `readelf -SW /path/to/bin`. For example, in MySQL, before adding the option, only the `.rela.dyn` and `.rela.plt` sections exist. After option addition, more than 10 RELA sections appear, including `.rela.text` and `.rela.eh_frame`. If `-Wl,-q` is not effective, when manual perf sampling and `perf2bolt` are performed or `llvm-bolt` is used to perform optimization (with or without sysboost), the alarm `BOLT-WARNING: non-relocation mode for AArch64 is not fully supported` is reported.

### D-FOT Preparations

Install D-FOT using `yum` or build it from the source code using the following commands. (If libkperf or oeAware-manager is also built from the source code, you need to specify the library and include path when running `cmake`.)

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

### Configuration Modification

Modify `/etc/dfot/dfot/ini`. The configuration items can be set according to the following description.
Public configuration: [general]

| Configuration Item                          | Value Range                                | Available| Description                                                                                    |
| ----------------------------- | ----------------------------------- | ---- | -------------------------------------------------------------------------------------- |
| LOG_LEVEL                     | `<FATAL, ERROR, WARN, INFO, DEBUG>` | Yes  | Log level for the optimization service. Lower levels produce more log output.                                                                |
| COLLECTOR_SAMPLING_STRATEGY   | `<0>`                               | No | Sampling policy.<br>`0` indicates that low-frequency sampling is continuously performed after the plugin is enabled.<br>`1` indicates that the monitoring thread is started when the plugin is enabled. Sampling is performed only when the load reaches the threshold.<br>Currently, the sampling process is controlled by oeAware, and only `0` is supported.|
| COLLECTOR_HIGH_LOAD_THRESHOLD | `[0, cpus*100]`                     | No | Application CPU usage threshold (`HIGH_LOAD_THRESHOLD`) for triggering sampling. This parameter is available only when sampling policy is set to `1`. Currently, this parameter is not supported.                             |
| COLLECTOR_DATA_AGING_TIME     | Determined based on actual requirements                            | Yes  | Aging time of sampling data, in milliseconds. When the time difference between the current data and the earliest data reaches the threshold, the accumulated sampling data is discarded.                                             |
| TUNER_TOOL                    | `["sysboost"]`                      | No | Binary optimizer. Currently, only sysboost is supported.                                                                  |
| TUNER_CHECK_PERIOD            | `[10, max]`                         | Yes  | Interval for optimization plugin check, in milliseconds. Sampling plugin data is collected at a specified interval to determine whether to perform optimization.                                               |
| TUNER_PROFILE_DIR             | Determined based on actual requirements                            | Yes  | Location where the sampling data is stored. The profile file is named in format `[app_name]_[full_path_hash]_[threshold].profile`.               |
| TUNER_OPTIMIZING_STRATEGY     | `[0, 1]`                            | Yes  | Optimization policy. `0` indicates that the optimization is performed only once, and `1` indicates that the optimization can be performed for multiple times as long as the sampling information is updated.                                                   |
| TUNER_OPTIMIZING_CONDITION    | `[0, 2]`                            | No | Condition for triggering optimization. `0` indicates that the optimization starts immediately after the application exits. `1` indicates that the optimization is performed when the load is low. `2` indicates that the optimization is performed when the application exits and the load is low. Currently, only `0` is supported.|

Application configuration: [app]

| Configuration Item                          | Value Range           | Available| Description                                                                                        |
| ----------------------------- | -------------- | ---- | ------------------------------------------------------------------------------------------ |
| FULL_PATH                     | Determined based on actual requirements       | Yes  | Absolute path of the application binary file.                                                                               |
| DEFAULT_PROFILE               | Determined based on actual requirements       | Yes  | Default profile file of the application, which is used to enable binary optimization during cold start. If there is no such file, leave it empty.                                                        |
| COLLECTOR_DUMP_DATA_THRESHOLD | `[10000, max]` | Yes  | Row count threshold for exporting sampling data to the profile. A larger value indicates more samples to be collected before the export occurs.                                            |
| BOLT_DIR                      |    N/A        | No | BOLT tool path. If this parameter is left empty, the default value `/usr/bin` is used. `${BOLT_DIR}/perf2bolt` and `${BOLT_DIR}/llvm-bolt` are invoked internally.<br>This parameter is determined by sysboost.|
| BOLT_OPTIONS                  | Determined based on actual requirements       | Yes  | BOLT optimization option. This option can be configured to override the built-in default option for specific optimization.                                                       |
| UPDATE_DEBUG_INFO             | `[0, 1]`       | Yes  | Whether to update the debugging information synchronously during optimization. `1` indicates that the debugging information is updated, and `0` indicates that the debugging information is not updated. Note that updating the debugging information will take extra time.                                                 |

### Plugin Usage

See [oeAware-manager](https://gitee.com/openeuler/oeAware-manager/tree/master/docs).

## Constraints

1. This feature can be used in container-based deployment scenarios. Optimization components such as D-FOT must be deployed outside the container.
2. The target application to be optimized must include relocation information.
3. When D-FOT is used as the oeAware tuning plugin, the fixed sampling frequency of the oeAware collection plugin pmu_sampling_collector is 100. This may cause a long sampling time. (For reference, the default sampling frequency of perf is 4000.)

## Future Plans

- [ ] Virtualization scenario improvement
- [ ] Runtime optimization support
- [ ] Enhanced binary optimization
