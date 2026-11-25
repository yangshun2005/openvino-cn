# 为 Android 系统构建 OpenVINO™ Runtime

本文介绍如何为 Android 操作系统构建 OpenVINO。

## 软件要求

- [CMake](https://cmake.org/download/) 3.26 或更高版本
- [SCons](https://scons.org/pages/download.html) 4.6.0 或更高版本
- [Android SDK Platform Tools](https://developer.android.com/tools/releases/platform-tools)（本指南已用 35.0.0 版本验证）
- [Android NDK](https://developer.android.com/ndk/downloads)（本指南已用 r28c 版本验证）

## 使用预构建的 x86-64 归档包

对于 Android x86-64 目标，可直接使用预构建的 OpenVINO 2026.4.0 Runtime 包，无需从源码构建：

```sh
mkdir openvino-android
export OPV_HOME_DIR=${PWD}/openvino-android
curl -L https://storage.openvinotoolkit.org/repositories/openvino/packages/2026.4/linux/openvino_toolkit_android_2026.4.0.22959.99c81491cc3_x86_64.tgz \
  --output $OPV_HOME_DIR/openvino_toolkit_android_2026.4.0.tgz
tar -xf $OPV_HOME_DIR/openvino_toolkit_android_2026.4.0.tgz -C $OPV_HOME_DIR
export OpenVINO_DIR=$OPV_HOME_DIR/openvino_toolkit_android_2026.4.0.22959.99c81491cc3_x86_64/runtime/cmake
```

该归档包提供 OpenVINO 头文件、x86-64 Runtime 库、CMake 配置及所需的 oneTBB 库。如需其他 Android 架构或自定义配置，请按下文从源码构建 OpenVINO。本指南后文的 ADB 示例演示的是 aarch64 源码构建，不直接适用于此 x86-64 归档包。

## 构建方法

### 创建工作目录
  ```sh
  mkdir openvino-android
  export OPV_HOME_DIR=${PWD}/openvino-android
  export HOST_OS_TYPE=$(uname -s | tr '[:upper:]' '[:lower:]') # linux/darwin
  ```

### 下载并解压 Android 包
* 下载并解压 [Android NDK](https://developer.android.com/ndk/downloads)
  ```sh
  wget https://dl.google.com/android/repository/android-ndk-r28c-${HOST_OS_TYPE}.zip --directory-prefix $OPV_HOME_DIR

  unzip $OPV_HOME_DIR/android-ndk-r28c-${HOST_OS_TYPE}.zip -d $OPV_HOME_DIR
  mv $OPV_HOME_DIR/android-ndk-r28c $OPV_HOME_DIR/android-ndk
  export ANDROID_NDK_PATH=$OPV_HOME_DIR/android-ndk
  ```
* 下载并解压 [Android SDK Platform Tools](https://developer.android.com/tools/releases/platform-tools)
  ```sh
  wget https://dl.google.com/android/repository/platform-tools-latest-${HOST_OS_TYPE}.zip --directory-prefix $OPV_HOME_DIR

  unzip $OPV_HOME_DIR/platform-tools-latest-${HOST_OS_TYPE}.zip -d $OPV_HOME_DIR
  export ANDROID_TOOLS_PATH=$OPV_HOME_DIR/platform-tools
  ```
_Windows 和 Mac 操作系统的下载与解压步骤类似。_

### 设置构建环境变量
  ```sh
  # If you have no android devices please set CURRENT_ANDROID_ABI according to your preferences e.g. export CURRENT_ANDROID_ABI=arm64-v8a
  export CURRENT_ANDROID_ABI=`$ANDROID_TOOLS_PATH/adb shell getprop ro.product.cpu.abi`
  export CURRENT_ANDROID_PLATFORM=35
  export CURRENT_ANDROID_STL=c++_shared
  export CURRENT_CMAKE_TOOLCHAIN_FILE=$ANDROID_NDK_PATH/build/cmake/android.toolchain.cmake
  ```
* `ANDROID_ABI` 指定目标架构：
    * `x86_64` 用于 x64 构建
    * `armeabi-v7a with NEON` 用于支持 NEON 的 ARM
    * `arm64-v8a` 用于 ARM 64 位
    * `riscv64` 用于 RISC-V 64 位（实验性）
* `ANDROID_PLATFORM` 指定 Android API 版本。
* `ANDROID_STL` 表示使用共享 C++ 运行时。

### 构建并安装 OneTBB™（不适用于 RISC-V 64 架构）
为借助 OneTBB 提升 OpenVINO™ 库的并行性能，需针对特定版本的 Android NDK 单独构建 OneTBB：
使用 `THREADING=TBB` 构建 Android OpenVINO 时，需要这个单独构建的 OneTBB。
调用 OpenVINO CMake 配置时必须传入 `-DTBB_DIR=$OPV_HOME_DIR/one-tbb-install/lib/cmake/TBB`。
若未提供 `TBB_DIR`，Android 配置会按设计失败。
  ```sh
  # Clone OneTBB™ repository 
  git clone --recursive https://github.com/oneapi-src/oneTBB $OPV_HOME_DIR/one-tbb
  # Create build and install directory 
  mkdir $OPV_HOME_DIR/one-tbb-build $OPV_HOME_DIR/one-tbb-install
  # Configure OneTBB™ CMake project 
  cmake -S $OPV_HOME_DIR/one-tbb \
        -B $OPV_HOME_DIR/one-tbb-build \
        -DCMAKE_BUILD_TYPE=Release \
        -DCMAKE_INSTALL_PREFIX=$OPV_HOME_DIR/one-tbb-install \
        -DCMAKE_TOOLCHAIN_FILE=$CURRENT_CMAKE_TOOLCHAIN_FILE \
        -DANDROID_ABI=$CURRENT_ANDROID_ABI \
        -DANDROID_PLATFORM=$CURRENT_ANDROID_PLATFORM \
        -DANDROID_STL=$CURRENT_ANDROID_STL \
        -DTBB_TEST=OFF \
        -DCMAKE_SHARED_LINKER_FLAGS="-Wl,--undefined-version"
  # Build OneTBB™ project 
  cmake --build $OPV_HOME_DIR/one-tbb-build --parallel
  # Install OneTBB™ project 
  cmake --install $OPV_HOME_DIR/one-tbb-build
  ```

Android OpenVINO 配置需要这个单独构建的 OneTBB 包。
配置 OpenVINO 时必须提供 `TBB_DIR`。

### 克隆 OpenVINO™ GenAI（可选）
  ```sh
  git clone --recursive https://github.com/openvinotoolkit/openvino.genai $OPV_HOME_DIR/openvino.genai
  ```

### 构建并安装 OpenVINO™
  ```sh
  # Clone OpenVINO™ repository 
  git clone --recursive https://github.com/openvinotoolkit/openvino $OPV_HOME_DIR/openvino
  # Create build and install directory 
  mkdir $OPV_HOME_DIR/openvino-build $OPV_HOME_DIR/openvino-install
  # Configure OpenVINO™ CMake project 
  cmake -S $OPV_HOME_DIR/openvino \
        -B $OPV_HOME_DIR/openvino-build \
        -DCMAKE_BUILD_TYPE=Release \
        -DCMAKE_INSTALL_PREFIX=$OPV_HOME_DIR/openvino-install \
        -DCMAKE_TOOLCHAIN_FILE=$CURRENT_CMAKE_TOOLCHAIN_FILE \
        -DANDROID_ABI=$CURRENT_ANDROID_ABI \
        -DANDROID_PLATFORM=$CURRENT_ANDROID_PLATFORM \
        -DANDROID_STL=$CURRENT_ANDROID_STL \
        -DTBB_DIR=$OPV_HOME_DIR/one-tbb-install/lib/cmake/TBB \
        -DOPENVINO_EXTRA_MODULES=$OPV_HOME_DIR/openvino.genai
  # Build OpenVINO™ project 
  cmake --build $OPV_HOME_DIR/openvino-build --parallel
  # Install OpenVINO™ project 
  cmake --install $OPV_HOME_DIR/openvino-build
  ```

### 下载示例模型
  我们将使用 `mobelinet-v3-tf` 示例模型
  ```sh
  mkdir $OPV_HOME_DIR/mobelinet-v3-tf
  wget https://storage.openvinotoolkit.org/repositories/openvino_notebooks/models/mobelinet-v3-tf/FP32/v3-small_224_1.0_float.xml -P $OPV_HOME_DIR/mobelinet-v3-tf/
  wget https://storage.openvinotoolkit.org/repositories/openvino_notebooks/models/mobelinet-v3-tf/FP32/v3-small_224_1.0_float.bin -P $OPV_HOME_DIR/mobelinet-v3-tf/
  ```

### 使用 ADB 工具运行示例模型并检查库
_此示例针对 aarch64 架构演示_
  ```sh
  # Copy OpenVINO™ libraries to android device
  $ANDROID_TOOLS_PATH/adb push --sync $OPV_HOME_DIR/openvino-install/runtime/lib/aarch64/* /data/local/tmp/
  # Copy OneTBB libraries to android device
  $ANDROID_TOOLS_PATH/adb push --sync $OPV_HOME_DIR/one-tbb-install/lib/* /data/local/tmp/
  # Copy shared STL library to android device
  $ANDROID_TOOLS_PATH/adb push --sync $ANDROID_NDK_PATH/toolchains/llvm/prebuilt/$HOST_OS_TYPE-x86_64/sysroot/usr/lib/aarch64-linux-android/libc++_shared.so /data/local/tmp/
  # Copy example model files
  $ANDROID_TOOLS_PATH/adb push --sync $OPV_HOME_DIR/mobelinet-v3-tf /data/local/tmp/
  # Copy OpenVINO™ benchmark_app tool to android device
  $ANDROID_TOOLS_PATH/adb push --sync $OPV_HOME_DIR/openvino/bin/aarch64/Release/benchmark_app /data/local/tmp/
  # Run OpenVINO™ benchmark_app tool
  $ANDROID_TOOLS_PATH/adb shell "LD_LIBRARY_PATH=/data/local/tmp ./data/local/tmp/benchmark_app -m /data/local/tmp/mobelinet-v3-tf/v3-small_224_1.0_float.xml -hint latency"
  ```

## 另请参阅

 * [OpenVINO README](../../README.md)
 * [OpenVINO 开发者文档](index.md)
 * [OpenVINO 入门](./get_started.md)
 * [如何构建 OpenVINO](build.md)