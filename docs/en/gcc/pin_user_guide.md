# Installation and Deployment

## Software

* OS: openEuler 23.03

## Hardware

* x86_64
* Arm

## Preparing the Environment

* Install the openEuler operating system. For details, see the [*openEuler Installation Guide*](https://docs.openeuler.openatom.cn/en/docs/24.03_LTS_SP2/server/installation_upgrade/installation/installation_on_servers.html).

### Install the dependency

#### Installing the Software on Which the PIN GCC Client Depends

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

#### Installing the Software on Which the PIN Server Depends

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

## Installing PIN

### rpmbuild

#### Building the PIN GCC Client

```shell
git clone https://gitee.com/src-openeuler/pin-gcc-client.git
cd pin-gcc-client
mkdir -p ~/rpmbuild/SOURCES
cp *.path pin-gcc-client.tar.gz ~/rpmbuild/SOURCES
rpmbuild -ba pin-gcc-client.spec
cd  ~/rpmbuild/RPMS
rpm -ivh pin-gcc-client.rpm
```

#### Building the PIN Server

```shell
git clone https://gitee.com/src-openeuler/pin-server.git
cd pin-server
mkdir -p ~/rpmbuild/SOURCES
cp *.path pin-server.tar.gz ~/rpmbuild/SOURCES
rpmbuild -ba pin-server.spec
cd  ~/rpmbuild/RPMS
rpm -ivh pin-server.rpm
```

### Build

#### Building the PIN GCC Client

```shell
git clone https://gitee.com/openeuler/pin-gcc-client.git
cd pin-gcc-client
mkdir build
cd build
cmake ../ -DMLIR_DIR=${MLIR_PATH} -DLLVM_DIR=${LLVM_PATH}
make
```

#### Building the PIN Server

```shell
git clone https://gitee.com/openeuler/pin-server.git
cd pin-server
mkdir build
cd build
cmake ../ -DMLIR_DIR=${MLIR_PATH} -DLLVM_DIR=${LLVM_PATH}
make
```

# Usage

You can use `-fplugin` and `-fplugin-arg-libpin_xxx` to enable the Plug-IN (PIN) tool.
Command:

```shell
$(TARGET): $(OBJS)
    $(CXX) -fplugin=${CLIENT_PATH}/build/libpin_gcc_client.so \
    -fplugin-arg-libpin_gcc_client-server_path=${SERVER_PATH}/build/pin_server \
    -fplugin-arg-libpin_gcc_client-log_level="1" \
    -fplugin-arg-libpin_gcc_client-arg1="xxx"
```

You can use the `${INSTALL_PATH}/bin/pin-gcc-client.json` file to configure PIN. The configuration options are as follows:

`path`: path of the executable file of the PIN server.

`sha256file`: path of the PIN verification file `xxx.sha256`.

`timeout`: timeout interval for cross-process communication, in milliseconds.

Compile options:

`-fplugin`: path of the .so file of the PIN client.

`-fplugin-arg-libpin_gcc_client-server_path`: path of the executable program of the PIN server.

`-fplugin-arg-libpin_gcc_client-log_level`: default log level. The value ranges from `0` to `3`. The default value is `1`.

`-fplugin-arg-libpin_gcc_client-argN`: other parameters that can be specified as required. `argN` indicates the argument required by PIN.

# Compatibility

This section describes the compatibility issues in some special scenarios. This project is in continuous iteration and will be fixed as soon as possible. Developers are welcome to join this project.

* When PIN is enabled in the `-flto` phase, multi-process compilation using `make -j` is not supported. You are advised to use `make -j1` for compilation.
