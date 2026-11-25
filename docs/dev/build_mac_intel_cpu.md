# 为 macOS 系统构建 OpenVINO™ 运行时（Intel CPU）

本指南介绍如何在搭载 Intel CPU 的 macOS 上构建 OpenVINO 运行时，以便后续在 Intel CPU 上进行推理。有两种方式：
- 在 Intel CPU 主机上使用本地编译。注意，[构建步骤](#how-to-build)展示的是此场景。
- 在 OSX Apple Silicon 上交叉编译。

本软件已：
- 在 macOS 15（原生 arm64）上验证

本指南描述的 x86_64 本地编译和交叉编译路径未纳入当前 CI 矩阵。

## 软件要求

- [brew](https://brew.sh) 包管理器，用于安装额外依赖。请参阅[安装 brew](https://brew.sh) 指南完成安装。
- Python 和 Python 库的安装步骤因主机架构而异：
  - **x86_64** 用于 OpenVINO 运行时 Python API 的 Python 3.11 - 3.15：
  ```sh
  % # let's have a look what python versions are available in brew
  % brew search python
  % # select preferred version of python based on available ones, e.g. 3.11
  % brew install python@3.11
  ```
  - **arm64** 从 [Python 发布页](https://www.python.org/downloads/macos/)下载页面选择 universal2 安装程序，并安装 `python-3.X.Y-macos11.pkg` 镜像。这样可获得 OpenVINO Python API 的通用 Python 库（构建 x86_64）。
- [CMake](https://cmake.org/download/) 3.26 或更高版本及其他开发工具：
  ```sh
  % brew install cmake scons fdupes git-lfs ninja
  ```
- Xcode 13.0 或更高版本中的 Clang 编译器及其他命令行工具：
  ```sh
  % xcode-select --install
  ```
- （可选；仅本地编译，x86_64）产品和示例依赖：
  ```sh
  % brew install tbb pugixml flatbuffers snappy protobuf
  ```
- 构建 OpenVINO 运行时 Python API 所需的额外 `pip` 依赖：
  ```sh
  % # update pip and setuptools to newer versions
  % python3 -m pip install -U pip
  % python3 -m pip install -r <openvino source tree>/src/bindings/python/requirements.txt
  ```
  构建 OpenVINO Python API 所需的额外安装依赖（在克隆 OpenVINO 仓库之后）：
  ```sh
  % python3 -m pip install -r <openvino source tree>/src/bindings/python/wheel/requirements-dev.txt
  ```

## 如何构建

1. 创建构建文件夹：
```sh
mkdir build && cd build
```
2. （CMake 配置）OpenVINO 项目使用基于 CMake 的构建系统。在创建的 `build` 目录中运行 `cmake`，以获取项目依赖并创建构建规则：
```sh
cmake -G "Ninja Multi-Config" -DENABLE_SYSTEM_PUGIXML=ON -DENABLE_SYSTEM_SNAPPY=ON -DENABLE_SYSTEM_PROTOBUF=ON ..
```
> **注意**：默认情况下，OpenVINO CMake 脚本会尝试探测系统，并据此启用所有可能的功能。你可以查看 CMake 输出中的警告，了解哪些功能被关闭及对应原因，从而指导需要额外安装什么来启用不可用的功能。此外，你可以修改 CMake 选项来启用/禁用某些功能、添加/移除编译标志、提供自定义版本的依赖（如 TBB、PugiXML、OpenCV、Protobuf）。更多信息请参阅[自定义编译的 CMake 选项](./cmake_options_for_custom_compilation.md)。
3. （CMake 构建）构建 OpenVINO 项目：
```sh
cmake --build . --config Release --parallel $(sysctl -n hw.ncpu)
```
所有构建的二进制文件位于 `<openvino_source_dir>/bin/intel64/Release/`，wheel 包位于 `<openvino_build_dir>/wheels`。

4. （可选安装）构建完成后，可将构建产物安装到指定位置：
```sh
cmake -DCMAKE_INSTALL_PREFIX=<installation location> -P cmake_install.cmake
```

### 交叉编译

自 OSX 11.x 和 Xcode 12.2 起，Apple 开发工具允许在 x86 主机上编译 arm64 代码，反之亦然。基于此，OpenVINO 甚至可以在 Apple Silicon 机器上编译，然后这些构建产物可在 Intel CPU 主机和 Apple Silicon 主机上运行（使用 [Rosetta](https://support.apple.com/en-us/HT211861)）。为此，在按上述步骤编译 OpenVINO 时，在 cmake 配置步骤中添加 `-DCMAKE_OSX_ARCHITECTURES=x86_64`。**不要通过 CMake 选项显式启用任何系统库**（移除所有 `-DENABLE_SYSTEM_*` 选项），因为它们具有 `arm64` 架构，例如：
```sh
file /opt/homebrew/Cellar/tbb/2021.5.0_2/lib/libtbb.12.5.dylib
/opt/homebrew/Cellar/tbb/2021.5.0_2/lib/libtbb.12.5.dylib: Mach-O 64-bit dynamically linked shared library arm64
```

如果看到如下错误：
```sh
ld: warning: ignoring file /opt/homebrew/lib/libopencv_imgproc.4.6.0.dylib, building for macOS-x86_64 but attempting to link with file built for macOS-arm64
Undefined symbols for architecture x86_64:
  "cv::Mat::Mat(cv::Size_<int>, int, void*, unsigned long)", referenced from:
      _image_resize in opencv_c_wrapper.cpp.o
      _image_save in opencv_c_wrapper.cpp.o
....
ld: symbol(s) not found for architecture x86_64
clang: error: linker command failed with exit code 1 (use -v to see invocation)
```
请在 cmake 中禁用其使用，或从系统中完全移除该库（如 `brew uninstall opencv`），因为它是纯 arm64 的，无法用于编译 x86_64 二进制文件。

或者，你必须自行显式查找/编译 x86_64（甚至 `universal2`）依赖，并传给 OpenVINO cmake 脚本。例如，使用额外选项 `-DCMAKE_OSX_ARCHITECTURES="x86_64;arm64"` 编译 oneTBB，安装后设置 `export TBBROOT=<universal oneTBB install root>`，供 OpenVINO 使用。

## 另请参阅

 * [OpenVINO README](../../README.md)
 * [OpenVINO 开发者文档](index.md)
 * [OpenVINO 入门](./get_started.md)
 * [如何构建 OpenVINO](build.md)