# ANNC 使用手册

## 1 ANNC 介绍

ANNC（Accelerated Neural Network Compiler）是专注于加速神经网络计算的编译器，旨在通过计算图优化、高性能融合算子生成与对接技术、以及高效的代码生成与优化能力，提升推荐和大模型的推理性能，支持主流开源推理框架接入。

## 2 ANNC 的安装构建

### 2.1 直接安装ANNC（通过eur获取）

```bash
wget https://eur.openeuler.openatom.cn/results/lesleyzheng1103/ANNC/openeuler-22.03_LTS_SP4-aarch64/00110327-ANNC/ANNC-0.0.2-3.aarch64.rpm

#  安装到 / 目录
rpm -ivh ANNC-0.0.2-3.aarch64.rpm
```

### 2.2 RPM包构建安装流程（推荐）

1. 使用 root 权限，安装 rpmbuild、rpmdevtools，具体命令如下：

   ```bash
   # 安装 rpmbuild
   yum install dnf-plugins-core rpm-build
   # 安装 rpmdevtools
   yum install rpmdevtools
   ```

2. 在主目录`/root`下生成 rpmbuild 文件夹：

   ```bash
   rpmdev-setuptree
   # 检查自动生成的目录结构
   ls ~/rpmbuild/
   BUILD  BUILDROOT  RPMS  SOURCES  SPECS  SRPMS
   ```

3. 使用`git clone -b master https://gitee.com/src-openeuler/ANNC.git`，从目标仓库的 `master` 分支拉取代码，并把目标文件放入 rpmbuild 的相应文件夹下：

   ``` shell
   cp ANNC/*.tar.gz* ~/rpmbuild/SOURCES
   cp ANNC/*.patch ~/rpmbuild/SOURCES/
   cp ANNC/ANNC.spec ~/rpmbuild/SPECS/
   ```

4. 用户可通过以下步骤生成 `ANNC` 的 RPM 包：

   ```bash
   # 安装 ANNC 所需依赖
   yum-builddep ~/rpmbuild/SPECS/ANNC.spec
   # 构建 ANNC 依赖包
   # 若出现 check-rpaths 相关报错，则需要在 rpmbuild 前添加 QA_RPATHS=0x0002，例如
   # QA_RPATHS=0x0002 rpmbuild -ba ~/rpmbuild/SPECS/ANNC.spec
   rpmbuild -ba ~/rpmbuild/SPECS/ANNC.spec
   # 安装 RPM 包
   cd ~/rpmbuild/RPMS/<arch>
   rpm -ivh ANNC-<version>-<release>.<arch>.rpm
   ```

   注意事项：若系统因存有旧版本的 RPM 安装包而导致文件冲突，可以通过以下方式解决：

   ```bash
   # 解决方案一：强制安装新版本
   rpm -ivh ANNC-<version>-<release>.<arch>.rpm --force
   # 解决方案二：更新安装包
   rpm -Uvh ANNC-<version>-<release>.<arch>.rpm
   ```

### 2.3 源码构建安装流程

ANNC 的源码地址：<https://gitee.com/openeuler/ANNC>。

保证以下依赖包已安装：

```shell
yum install -y gcc gcc-c++ bzip2 python3-devel python3-numpy python3-setuptools python3-wheel libstdc++-static java-11-openjdk java-11-openjdk-devel make
```

安装bazel，从该地址 <https://releases.bazel.build/6.5.0/release/bazel-6.5.0-dist.zip> 获取bazel-6.5.0包。

```bash
unzip bazel-6.5.0-dist.zip -d bazel-6.5.0
cd bazel-6.5.0
env EXTRA_BAZEL_ARGS="--tool_java_runtime_version=local_jdk" bash ./compile.sh

export PATH=/path/to/bazel-6.5.0/output:$PATH
bazel --version
```

准备XNNPACK。

```bash
git clone https://gitee.com/openeuler/ANNC.git
export ANNC="your_path_to_ANNC"

cd $ANNC/annc/service/cpu/xla/libs
bash xnnpack.sh

cd $ANNC/annc/service/cpu/xla/libs/XNNPACK/build
cp libXNNPACK.so /usr/lib64
export XNNPACK_BASE="$ANNC/annc/service/cpu/xla/libs"
export XNNPACK_DIR="$XNNPACK_BASE/XNNPACK"

CPLUS_INCLUDE_PATH+="$ANNC/annc/service/cpu/xla/:"
CPLUS_INCLUDE_PATH+="$ANNC/annc/service/:"
CPLUS_INCLUDE_PATH+="$XNNPACK_DIR/:"
CPLUS_INCLUDE_PATH+="$XNNPACK_DIR/include/:"
CPLUS_INCLUDE_PATH+="$XNNPACK_DIR/src/:"
CPLUS_INCLUDE_PATH+="$XNNPACK_DIR/build/pthreadpool-source/include/:"
export CPLUS_INCLUDE_PATH
```

安装ANNC，从源码地址下载ANNC源码包。

```bash
cd $ANNC

bash build.sh

cp bazel-bin/annc/service/cpu/libannc.so /usr/lib64
mkdir -p /usr/include/annc
cp annc/service/cpu/kdnn_rewriter.h /usr/include/annc
cd python
python3 setup.py bdist_wheel
python3 -m pip install dist/*.whl
```

## 3 使用流程

>[!NOTE]注意
>ANNC使用者需提前部署好tf-serving，通过编译选项和代码补丁的方式接入ANNC编译优化扩展套件。

### 3.1 图融合结合手动大算子

下载基线模型。

```bash
git clone https://gitee.com/openeuler/sra_benchmark.git
```

从基线模型库中获取以下目标推荐模型 **DeepFM、DFFM、DLRM、W&D**。

通过以下命令行实现图融合。

```bash
# 安装依赖库

python3 -m pip install tensorflow==2.15.1

# 运行模型转换，以DeepFM模型为例

annc-opt -I /path/to/model_DeepFM/1730800001/1 -O deepfm_new/1 dnn_sparse linear_sparse
cp -r /path/to/model_DeepFM/1730800001/1/variables deepfm_new/1
```

完成上述命令后，输出目录`deepfm_new/1`下应生成新的模型文件`saved_model.pbtxt`，搜索`KPFusedSparseEmbedding`，确认图融合算子正确生成。
   
然后将ANNC提供的开源算子库注册到tf-serving：

```bash
# 进入tf-serving目录，创建自定义算子文件夹

cd /path/to/serving
mkdir tensorflow_serving/custom_ops

# 将ANNC算子拷贝到该目录下

cp /usr/include/annc/fused*.cc tensorflow_serving/custom_ops/
```

创建算子编译文件`tensorflow_serving/custom_ops/BUILD`，并在该文件中写入以下内容：

```ini
package(
   default_visibility = [
        "//visibility:public",
       ],
       licenses = ["notice"],
)

cc_library(
    name = 'recom_embedding_ops',
    srcs = [
      "fused_sparse_embedding.cc",
      "fused_linear_embedding_with_hash_bucket.cc",
      "fused_dnn_embedding_with_hash_bucket.cc"
     ],
     alwayslink = 1,
     deps = [
       "@org_tensorflow//tensorflow/core:framework",
     ]
)
```

```bash
# 打开 tensorflow_serving/model_servers/BUILD，搜索SUPPORTED_TENSORFLOW_OPS，添加以下内容注册我们的算子:

"//tensorflow_serving/custom_ops:recom_embedding_ops"
```

完成算子注册后，使用以下命令重新编译tf-serving，编译成功即表示算子成功注册：

```bash
bazel --output_user_root=./output build -c opt --distdir=./proxy \
   --define tflite_with_xnnpack=false \
   tensorflow_serving/model_servers:tensorflow_model_server
```

### 3.2 使能算子优化和图优化

在构建好的server的tensorflow的xla路径下，通过补丁脚本使能以下补丁：

```bash
export TF_PATH="$HOME/serving/output/XXX/external/org_tensorflow"
export XLA_PATH="$HOME/serving/output/XXX/external/org_tensorflow/third_party/xla"

# 通过方式一安装的ANNC：
cd /usr/include/annc/tfserver/xla

# 修改xla2.sh前两行为：
TF_PATCH_PATH="$ANNC" 
PATH_OF_PATCHES="$ANNC/xla"
export ANNC_PATH=/usr/include/annc
bash xla2.sh

# 通过方式二安装的ANNC：
cd $ANNC/install/tfserver/xla
export ANNC_PATH=$ANNC
bash xla2.sh

# 重新编译
bazel --output_user_root=./output build -c opt --distdir=./proxy \
   --define tflite_with_xnnpack=false \
   tensorflow_serving/model_servers:tensorflow_model_server
```

### 3.3 图优化

设置环境变量，开启优化特性。

```bash
export 'TF_XLA_FLAGS=--tf_xla_auto_jit=2 --tf_xla_cpu_global_jit --tf_xla_min_cluster_size=16'
export OMP_NUM_THREADS=1
export PORT=7004  # 端口号
ANNC_FLAGS="--graph-opt" ENABLE_BISHENG_GRAPH_OPT="" ./bazel-bin/tensorflow_serving/model_servers/tensorflow_model_server 
--port=$PORT --rest_api_port=7005 
--model_base_path=/path/to/model_Boss/ 
--model_name=deepfm 
--tensorflow_intra_op_parallelism=1 --tensorflow_inter_op_parallelism=-1 
--xla_cpu_compilation_enabled=true
```

### 3.4  算子优化

配置环境变量`ANNC_FLAGS`，开启MatMul下发和对接OpenBLAS优化选项，启动TF-Serving，指定目标模型。

```bash
export 'TF_XLA_FLAGS=--tf_xla_auto_jit=2 --tf_xla_cpu_global_jit --tf_xla_min_cluster_size=16'
export OMP_NUM_THREADS=1
export PORT=7004  # 端口号
ANNC_FLAGS="--gemm-opt"  XLA_FLAGS="--xla_cpu_enable_xnnpack=true" ./bazel-bin/tensorflow_serving/model_servers/tensorflow_model_server \
    --port=$PORT --rest_api_port=7005 \
    --model_base_path=/path/to/model_DeepFM/1730800001/ \
    --model_name=deepfm \
    --tensorflow_intra_op_parallelism=1 --tensorflow_inter_op_parallelism=-1 \
    --xla_cpu_compilation_enabled=true
```

`--gemm-opt` 选项启用了所有 GEMM 相关优化，包括常量折叠（layout-matmul）。关于常量折叠优化的详细原理和架构说明，请参见 [常量折叠优化](./constant_folding.md)。

### 3.5 Remapper(Tensorflow)图融合优化

该特性基于原生Tensorflow框架开发，在Remapper优化器中调用ANNC graph optimizer优化。

#### 步骤1：下载Tensorflow2.15版本

```bash
git clone https://gitee.com/mirrors/tensorflow.git -b v2.15.0
```

#### 步骤2：应用ANNC补丁和融合算子

```bash
export ANNC="your_path_to_ANNC"

cd tensorflow
patch -p1 < $ANNC/annc/tensorflow/tf_annc_optimizer.patch
cp -r $ANNC/annc/tensorflow/graph_optimizer ./tensorflow/core/grappler/optimizers/
cp $ANNC/annc/tensorflow/kernels/* ./tensorflow/core/kernels/
cp $ANNC/annc/tensorflow/ops/* ./tensorflow/core/ops/
cp $ANNC/annc/tensorflow/api_def/* ./tensorflow/core/api_def/python_api/
cp $ANNC/annc/tensorflow/api_def/* ./tensorflow/core/api_def/base_api/
```

#### 步骤3：编译tensorflow

```bash
bazel build --config=v2 --config=xla --config=noaws --distdir=./proxy //tensorflow:tensorflow_cc

cd ./bazel-bin/tensorflow

ln -s libtensorflow_framework.so.2.15.0 libtensorflow_framework.so.2
ln -s libtensorflow_framework.so.2 libtensorflow_framework.so
ln -s libtensorflow_cc.so.2.15.0 libtensorflow_cc.so.2
ln -s libtensorflow_cc.so.2 libtensorflow_cc.so

# 配置环境变量
export LD_LIBRARY_PATH=/path_to_tensorflow/bazel-bin/tensorflow
```

#### 步骤4：使能图融合优化

```bash
# 使用Tensorflwo推理时，开启优化选项即可：
ANNC_FUASED_ALL = 1
```
