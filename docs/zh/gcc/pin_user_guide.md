# 安装与部署

## 软件要求

* 操作系统：openEuler 23.03及后续版本

## 硬件要求

* x86_64架构
* ARM架构

## 环境准备

* 安装openEuler系统，安装方法参考 《[安装指南](https://docs.openeuler.openatom.cn/zh/docs/25.03/server/installation_upgrade/installation/installation-preparations.html)》。

### 安装依赖软件

#### 安装插件框架GCC客户端依赖软件

```shell
yum install -y grpc
yum install -y grpc-devel
yum install -y grpc-plugins
yum install -y protobuf-devel
yum install -y jsoncpp
yum install -y jsoncpp-devel
yum install -y gcc-plugin-devel
yum install -y llvm-mlir
yum install -y llvm-mlir-devel
yum install -y llvm-devel
```

#### 安装插件框架服务端依赖软件

```shell
yum install -y grpc
yum install -y grpc-devel
yum install -y grpc-plugins
yum install -y protobuf-devel
yum install -y jsoncpp
yum install -y jsoncpp-devel
yum install -y llvm-mlir
yum install -y llvm-mlir-devel
yum install -y llvm-devel
```

## 安装Pin

### rpm构建

#### 构建插件框架GCC客户端

```shell
git clone https://gitee.com/src-openeuler/pin-gcc-client.git
cd pin-gcc-client
mkdir -p ~/rpmbuild/SOURCES
cp *.path pin-gcc-client.tar.gz ~/rpmbuild/SOURCES
rpmbuild -ba pin-gcc-client.spec
cd  ~/rpmbuild/RPMS
rpm -ivh pin-gcc-client.rpm
```

#### 构建插件框架服务端

```shell
git clone https://gitee.com/src-openeuler/pin-server.git
cd pin-server
mkdir -p ~/rpmbuild/SOURCES
cp *.path pin-server.tar.gz ~/rpmbuild/SOURCES
rpmbuild -ba pin-server.spec
cd  ~/rpmbuild/RPMS
rpm -ivh pin-server.rpm
```

### 编译构建

#### 构建插件框架GCC客户端

```shell
git clone https://gitee.com/openeuler/pin-gcc-client.git
cd pin-gcc-client
mkdir build
cd build
cmake ../ -DMLIR_DIR=${MLIR_PATH} -DLLVM_DIR=${LLVM_PATH}
make
```

#### 构建插件框架服务端

```shell
git clone https://gitee.com/openeuler/pin-server.git
cd pin-server
mkdir build
cd build
cmake ../ -DMLIR_DIR=${MLIR_PATH} -DLLVM_DIR=${LLVM_PATH}
make
```

# 使用方法

用户可以通过`-fplugin`和`-fplugin-arg-libpin_xxx`使能插件工具。
命令如下：

```shell
$(TARGET): $(OBJS)
    $(CXX) -fplugin=${CLIENT_PATH}/build/libpin_gcc_client.so \
    -fplugin-arg-libpin_gcc_client-server_path=${SERVER_PATH}/build/pin_server \
    -fplugin-arg-libpin_gcc_client-log_level="1" \
    -fplugin-arg-libpin_gcc_client-arg1="xxx"
```

为了方便用户使用，可以通过`${INSTALL_PATH}/bin/pin-gcc-client.json`文件，进行插件配置。配置选项如下：

`path` : 配置插件框架服务端可执行文件路径

`sha256file` : 配置插件工具的校验文件`xxx.sha256`路径

`timeout` : 配置跨进程通信超时时间，单位`ms`

编译选项：

`-fplugin`：指定插件客户端.so所在路径

`-fplugin-arg-libpin_gcc_client-server_path`：指定插件服务端可执行程序所在路径

`-fplugin-arg-libpin_gcc_client-log_level`：指定日志系统默认记录等级，取值`0~3`。默认为`1`

`-fplugin-arg-libpin_gcc_client-argN`：用户可以根据插件工具要求，指定其他参数。argN代指插件工具要求的参数字段。

# 兼容性说明

此节主要列出当前一些特殊场景下的兼容性问题。本项目持续迭代中，会尽快进行修复，也欢迎广大开发者加入。

* 插件框架在`-flto`阶段使能时，不支持使用`make -j`多进程编译。建议改用`make -j1`进行编译。
