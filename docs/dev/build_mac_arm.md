# 为 macOS 系统构建 OpenVINO™ Runtime（Apple Silicon）

本指南介绍如何在 macOS 上为 Apple Silicon 构建 OpenVINO Runtime，以便后续进行推理。

在 Apple Silicon 上使用 OpenVINO 有两种方式：
- （原生）为 arm64 架构编译 OpenVINO。注意，构建步骤默认采用此方案。
- （Rosetta）编译 `x86_64` 架构的 Intel CPU 插件，并在 [Rosetta](https://support.apple.com/en-us/HT211861) 下运行。

该软件：
- 已在 macOS 15（原生 arm64）上验证
- 面向 macOS 11.0（部署目标）构建

## 软件要求

- [brew](https://brew.sh) 包管理器，用于安装额外依赖。请参考 [install brew](https://brew.sh) 指南完成安装。

- Python 及 Python 库的安装步骤因主机架构而异：
  - **arm64** Python 3.11 - 3.15，用于 OpenVINO Runtime Python API：
  ```sh
  % # let's have a look what python versions are available in brew
  % brew search python
  % # select preferred version of python based on available ones, e.g. 3.11
  % brew install python@3.11
  ```
  - **x86_64** 从 [Python releases](https://www.python.org/downloads/macos/) 下载页面选择 universal2 安装程序，并安装 `python-3.X.Y-macos11.pkg` 镜像。这样可获得 OpenVINO Python API 的通用 Python 库（构建 x86_64）。

- Xcode 13.0 或更高版本中的 Clang 编译器及其他命令行工具：
  ```sh
  % xcode-select --install
  ```
- [CMake](https://cmake.org/download/) 3.26 或更高版本及其他构建依赖：
  ```sh
  % brew install cmake scons fdupes git-lfs ninja
  ```
- （仅 arm64）产品和示例依赖：
  ```sh
  % brew install tbb pugixml flatbuffers snappy protobuf
  ```
- 构建 OpenVINO Runtime Python API 所需的额外 `pip` 依赖：
  ```sh
  % # update pip and setuptools to newer versions
  % python3 -m pip install -U pip
  % python3 -m pip install -r <openvino source tree>/src/bindings/python/requirements.txt
  ```
  将 OpenVINO Python API 构建为 wheel 包所需的额外安装依赖（在克隆 OpenVINO 仓库后）：
  ```sh
  % python3 -m pip install -r <openvino source tree>/src/bindings/python/wheel/requirements-dev.txt
  ```

## 构建方法

1. （获取源码）克隆仓库及子模块：
```sh
git clone https://github.com/openvinotoolkit/openvino.git
cd openvino
git submodule update --init
```
2. 创建构建目录：
```sh
mkdir build && cd build
```
3. （CMake 配置）OpenVINO 项目使用基于 CMake 的构建系统。在创建的 `build` 目录中运行 `cmake`，以获取项目依赖并生成构建规则：
```sh
cmake -G "Ninja Multi-Config" -DENABLE_SYSTEM_PUGIXML=ON -DENABLE_SYSTEM_SNAPPY=ON -DENABLE_SYSTEM_PROTOBUF=ON ..
```
> **注意**：默认情况下，OpenVINO CMake 脚本会尝试探测系统，并据此启用所有可能的功能。你可以查看 CMake 输出中的警告，了解哪些功能被关闭及对应原因，从而指导你额外安装什么以启用不可用的功能。此外，你可以修改 CMake 选项来启用/禁用某些功能、添加/移除编译标志、提供自定义版本的依赖（如 TBB、PugiXML、Protobuf）。更多信息请参阅 [CMake Options for Custom Compilation](./cmake_options_for_custom_compilation.md)。
4. （CMake 构建）构建 OpenVINO 项目：
```sh
cmake --build . --config Release --parallel $(sysctl -n hw.ncpu)
```
所有构建产物位于 `<openvino_source_dir>/bin/<arm64 | intel64>/Release/`，wheel 包位于 `<openvino_build_dir>/wheels`。

5. （可选安装）构建完成后，可将构建产物安装到指定位置：
```sh
cmake -DCMAKE_INSTALL_PREFIX=<installation location> -P cmake_install.cmake
```

### 构建 x86_64 二进制文件

自 OSX 11.x 和 Xcode 12.2 起，Apple 开发工具允许在 x86 主机上编译 arm64 代码，反之亦然。基于此，OpenVINO 可编译为 x86_64 二进制文件，然后在 Apple Silicon 主机上通过 [Rosetta](https://support.apple.com/en-us/HT211861) 运行。为此，必须先安装 Rosetta：

```sh
softwareupdate --install-rosetta
```

然后按上述步骤编译 OpenVINO，但在 CMake 配置阶段添加 `-DCMAKE_OSX_ARCHITECTURES=x86_64`。但是，**不要通过 CMake 选项显式启用任何系统库**（移除所有 `-DENABLE_SYSTEM_*` 选项），因为它们都是 `arm64` 架构，例如：
```sh
file /opt/homebrew/Cellar/tbb/2021.5.0_2/lib/libtbb.12.5.dylib
/opt/homebrew/Cellar/tbb/2021.5.0_2/lib/libtbb.12.5.dylib: Mach-O 64-bit dynamically linked shared library arm64
```

其他外部依赖（如 `pugixml`）同理。

> **注意**：使用 `universal2` Python 库构建时，wheel 包命名为 `openvino-2022.3.0-000-cp311-cp311-macosx_12_0_universal2.whl`，并带有正确的 `universal2` 标签，因此*有可能*在 Apple Silicon 和 Intel CPU 上同时使用。

## 另请参阅

 * [OpenVINO README](../../README.md)
 * [OpenVINO Developer Documentation](index.md)
 * [OpenVINO Get Started](./get_started.md)
 * [How to build OpenVINO](build.md)