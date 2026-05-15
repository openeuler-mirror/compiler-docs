# ANNC User Manual

## 1 Introduction

Accelerated Neural Network Compiler (ANNC) speeds up neural network computing. It improves model inference for recommendation systems and foundation models by optimizing computation graphs, fusing and integrating high-performance operators, and generating efficient code. In addition, ANNC works with popular open-source inference frameworks.

## 2 Installing and Building ANNC

### 2.1 Direct Installation (via EUR)

```bash
wget https://eur.openeuler.openatom.cn/results/lesleyzheng1103/ANNC/openeuler-22.03_LTS_SP3-aarch64/00109829-ANNC/ANNC-0.0.2-1.aarch64.rpm

#  Install the package to the / directory
rpm -ivh ANNC-0.0.2-1.aarch64.rpm
```

### 2.2 Build and Installation Using RPM (Recommended)

1. Run as the root user to install rpmbuild and rpmdevtools. The commands are as follows:

   ```bash
   # Install rpmbuild
   yum install dnf-plugins-core rpm-build
   # Install rpmdevtools
   yum install rpmdevtools
   ```

2. Create the `rpmbuild` folder in the `/root` directory.

   ```bash
   rpmdev-setuptree
   # Check the automatically generated directory structure
   ls ~/rpmbuild/
   BUILD  BUILDROOT  RPMS  SOURCES  SPECS  SRPMS
   ```

3. Use `git clone -b master https://gitee.com/src-openeuler/ANNC.git` to pull code from the `master` branch of the target repository and place the target file in the corresponding folder of `rpmbuild`.

   ``` shell
   cp ANNC/*.tar.gz* ~/rpmbuild/SOURCES
   cp ANNC/*.patch ~/rpmbuild/SOURCES/
   cp ANNC/ANNC.spec ~/rpmbuild/SPECS/
   ```

4. Generate the RPM package of `ANNC` through the following steps:

   ```bash
   # Install ANNC dependencies
   yum-builddep ~/rpmbuild/SPECS/ANNC.spec
   # Build ANNC dependency packages
   # If check-rpaths errors are reported, add QA_RPATHS=0x0002 before rpmbuild as follows
   # QA_RPATHS=0x0002 rpmbuild -ba ~/rpmbuild/SPECS/ANNC.spec
   rpmbuild -ba ~/rpmbuild/SPECS/ANNC.spec
   # Install the RPM package
   cd ~/rpmbuild/RPMS/<arch>
   rpm -ivh ANNC-<version>-<release>.<arch>.rpm
   ```

   Note: If file conflicts arise from older RPMs already installed on your system, address them with the following methods:

   ```bash
   # Method 1: Install the new version forcibly
   rpm -ivh ANNC-<version>-<release>.<arch>.rpm --force
   # Method 2: Update the installation package
   rpm -Uvh ANNC-<version>-<release>.<arch>.rpm
   ```

### 2.3 Build and Installation Using Source Code

Obtain ANNC source code from <https://gitee.com/openeuler/ANNC>.

Check that the following dependencies have been installed:

```shell
yum install -y gcc gcc-c++ bzip2 python3-devel python3-numpy python3-setuptools python3-wheel libstdc++-static java-11-openjdk java-11-openjdk-devel make
```

Download `bazel-6.5.0` from <https://releases.bazel.build/6.5.0/release/bazel-6.5.0-dist.zip>, and install Bazel.

```bash
unzip bazel-6.5.0-dist.zip -d bazel-6.5.0
cd bazel-6.5.0
env EXTRA_BAZEL_ARGS="--tool_java_runtime_version=local_jdk" bash ./compile.sh

export PATH=/path/to/bazel-6.5.0/output:$PATH
bazel --version
```

Prepare XNNPACK.

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

Download the ANNC source package from the source code address, and install ANNC.

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

## 3 Usage Process

`Note:`

* You need to deploy TensorFlow serving (tf-serving) in advance and integrate it into the ANNC optimization extension kit through compilation options and code patches.

### 3.1 Graph Fusion with Hand-written Operators

Download a baseline model.

```bash
git clone https://gitee.com/openeuler/sra_benchmark.git
```

Obtain the following target recommendation models from the baseline model library: DeepFM, DFFM, DLRM, and W&D.

Implement graph fusion using the command.

```bash
# Install dependencies

python3 -m pip install tensorflow==2.15.1

# Execute model conversion and the DeepFM model is used as an example

annc-opt -I /path/to/model_DeepFM/1730800001/1 -O deepfm_new/1 dnn_sparse linear_sparse
cp -r /path/to/model_DeepFM/1730800001/1/variables deepfm_new/1
```

A new model file `saved_model.pbtxt` is generated in the output directory `deepfm_new/1`. Search for `KPFusedSparseEmbedding` to ensure that the graph fusion operator is correctly generated.
   
Register the open-source operator library provided by ANNC with tf-serving.

```bash
# Go to the tf-serving directory and create a custom operator folder

cd /path/to/serving
mkdir tensorflow_serving/custom_ops

# Copy the ANNC operator to the directory

cp /usr/include/annc/fused*.cc tensorflow_serving/custom_ops/
```

Create the operator build file `tensorflow_serving/custom_ops/BUILD` and write the following content to the file:

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
# Open **tensorflow_serving/model_servers/BUILD**, search for **SUPPORTED_TENSORFLOW_OPS**, and add the following content to register the operator:

"//tensorflow_serving/custom_ops:recom_embedding_ops"
```

After the operator is registered, run the following command to rebuild tf-serving. After successful rebuilding, the operator is successfully registered.

```bash
bazel --output_user_root=./output build -c opt --distdir=./proxy \
   --define tflite_with_xnnpack=false \
   tensorflow_serving/model_servers:tensorflow_model_server
```

### 3.2 Enabling Operator Optimization and Graph Optimization

In the TensorFlow XLA path of the built server, use the patch script to enable the following patches:

```bash
export TF_PATH="$HOME/serving/output/XXX/external/org_tensorflow"
export XLA_PATH="$HOME/serving/output/XXX/external/org_tensorflow/third_party/xla"

# ANNC installed using method 1
cd /usr/include/annc/tfserver/xla

# Modify the first two lines of xla2.sh as follows
TF_PATCH_PATH="$ANNC" 
PATH_OF_PATCHES="$ANNC/xla"
export ANNC_PATH=/usr/include/annc
bash xla2.sh

# ANNC installed using method 2
cd $ANNC/install/tfserver/xla
export ANNC_PATH=$ANNC
bash xla2.sh

# Recompile
bazel --output_user_root=./output build -c opt --distdir=./proxy \
   --define tflite_with_xnnpack=false \
   tensorflow_serving/model_servers:tensorflow_model_server
```

### 3.3 Graph Optimization

Set environment variables and enable the optimization feature.

```bash
export 'TF_XLA_FLAGS=--tf_xla_auto_jit=2 --tf_xla_cpu_global_jit --tf_xla_min_cluster_size=16'
export OMP_NUM_THREADS=1
export PORT=7004 #Port number
ANNC_FLAGS="--graph-opt" ENABLE_BISHENG_GRAPH_OPT="" ./bazel-bin/tensorflow_serving/model_servers/tensorflow_model_server 
--port=$PORT --rest_api_port=7005 
--model_base_path=/path/to/model_Boss/ 
--model_name=deepfm 
--tensorflow_intra_op_parallelism=1 --tensorflow_inter_op_parallelism=-1 
--xla_cpu_compilation_enabled=true
```

### 3.4 Operator Optimization

Configure the environment variable `ANNC_FLAGS` to enable MatMul offloading and leverage OpenBLAS optimizations. Then start TF-Serving and specify the target model.

```bash
export 'TF_XLA_FLAGS=--tf_xla_auto_jit=2 --tf_xla_cpu_global_jit --tf_xla_min_cluster_size=16'
export OMP_NUM_THREADS=1
export PORT=7004 #Port number
ANNC_FLAGS="--gemm-opt"  XLA_FLAGS="--xla_cpu_enable_xnnpack=true" ./bazel-bin/tensorflow_serving/model_servers/tensorflow_model_server \
    --port=$PORT --rest_api_port=7005 \
    --model_base_path=/path/to/model_DeepFM/1730800001/ \
    --model_name=deepfm \
    --tensorflow_intra_op_parallelism=1 --tensorflow_inter_op_parallelism=-1 \
    --xla_cpu_compilation_enabled=true
```
