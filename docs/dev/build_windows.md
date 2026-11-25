# 为 Windows 系统构建 OpenVINO™ Runtime

OpenVINO 可在 Windows 上为不同架构编译：X64 或 ARM64。为 ARM64 架构构建时，需要一台运行 Windows on ARM 的机器，因为仅支持本机编译。详情参见[此页面](https://learn.arm.com/install-guides/llvm-woa/)。

支持的配置：
- Windows 10 x86 64 位或更高版本，使用 Visual Studio 2019 或更高版本为 X64 架构构建。
- Windows on ARM（简称 WoA），用于构建 ARM64 架构。OpenVINO 已在 [Windows DevKit 2023](https://developer.qualcomm.com/hardware/windows-on-snapdragon/windows-dev-kit-2023) 上验证。

当前验证包含使用 Visual Studio 2022 和 Visual Studio 2026 的 x86_64 构建。
Visual Studio 2019 仍为文档记载的最低版本，但当前 CI 矩阵未覆盖。

## 软件要求

- [CMake](https://cmake.org/download/) 3.26 或更高版本
- Microsoft Visual Studio 2019 或更高版本，16.8 或更新（MSVC 19.28，C++20 所需）
  > **注意**：适用于 WoA 的原生 Microsoft Visual Studio 自 3.11 版本起可用。
- Python 3.11 - 3.15，用于 OpenVINO Runtime Python API
- [Git for Windows*]
- （仅 Windows on ARM）[LLVM for Windows on ARM (WoA)](https://github.com/llvm/llvm-project/releases/download/llvmorg-15.0.6/LLVM-15.0.6-woa64.exe)
  > **注意**：安装后，确保 `clang-cl` 编译器可从 `PATH` 访问。

## 构建方法

> **注意**：默认情况下，构建会启用 OpenVINO Runtime GPU 插件，以便在您的英特尔® 处理器显卡上推理模型。这需要您在运行构建前下载并安装 [Intel® Graphics Driver for Windows](https://www.intel.com/content/www/us/en/download/19344/intel-graphics-windows-dch-drivers.html)。如果不使用 GPU 插件，请使用 `-DENABLE_INTEL_GPU=OFF` CMake 构建选项，并跳过英特尔® 显卡驱动的安装。

1. 克隆子模块：
    ```sh
    git clone https://github.com/openvinotoolkit/openvino.git
    cd openvino
    git submodule update --init --recursive
    ```

2. 创建构建目录：
    ```sh
    mkdir build && cd build
    ```
3. 在 `build` 目录中运行 `cmake`，获取项目依赖并生成 Visual Studio 解决方案：

    ```sh
    cmake -G "Visual Studio 17 2022" <path/to/openvino>
    ```

4. 在 Visual Studio 中构建生成的解决方案，或运行 `cmake --build . --config Release --verbose -j<number_of_jobs>` 从命令行构建。使用 `WMIC cpu get numberofLogicalProcessors` 查看可用处理器数量。注意此过程可能耗时较长。

5. 运行示例前，将构建所用的 Threading Building Blocks (TBB) 二进制文件路径添加到 `%PATH%` 环境变量。默认情况下，TBB 二进制文件由基于 CMake 的脚本下载到 `<path/to/openvino>/temp/<platform>/tbb/bin` 文件夹。

### 构建时支持 PDB 文件

如需生成 PDB 文件并调试构建或运行需要 PDB 文件的工具，必须正确设置 CMake 构建类型。
应使用以下 CMake 构建类型选项之一：
  * `-DCMAKE_BUILD_TYPE=RelWithDebInfo`：生成带发布信息的 PDB 文件，适合调试优化后的构建。
  * `-DCMAKE_BUILD_TYPE=Debug`：生成针对调试优化的 PDB 文件，提供全面的调试信息。
  * `-DCMAKE_BUILD_TYPE=Release -DENABLE_PDB_IN_RELEASE=ON`：以 Release 模式构建，但生成调试符号。

注意，PDB 符号文件同时支持动态和静态构建。

打包 PDB 文件时，必须运行 cmake 命令创建专用归档：

  ```sh
  cmake --install <build folder> --config <Release | Debug | RelWithDebInfo> --prefix <installation path> --component pdb
  ```

### 构建时支持 Python API

1. 在上述 CMake（第 3 步）选项中启用 `-DENABLE_PYTHON=ON`。如需指定确切的 Python 版本，使用以下选项（需要 cmake 3.16 及更高版本）：
    ```sh
    -DPython3_EXECUTABLE="C:\Program Files\Python11\python.exe"
    ```
2. 如需构建 wheel 包（.whl），在上述 CMake 步骤（第 3 步）中启用 `-DENABLE_WHEEL=ON` 选项，并安装依赖：
    ```sh
    pip install -r <openvino source tree>\src\bindings\python\wheel\requirements-dev.txt
    ```
3. 构建完成后，将新构建的 Python 库导出到用户环境变量：
    ```
    set PYTHONPATH=<openvino_repo>/bin/<arch>/Release/python;<openvino_repo>/tools/ovc;%PYTHONPATH%
    set OPENVINO_LIB_PATHS=<openvino_repo>/bin/<arch>/Release;<openvino_repo>/temp/<platform>/tbb/bin
    set PATH=<openvino_repo>/tools/ovc/openvino/tools/ovc:%PATH%
    ```
  或使用 pip 安装 wheel：
  ```
  pip install build/wheels/openvino-2026.2.1-9612-cp313-cp313-win_arm64.whl
  ```
  > **注意**：wheel 生成在 `<build>/wheels/` 目录下。请将文件名替换为与您的 Python 版本和目标架构匹配的构建产物。

### 使用 Ninja* 构建系统构建 OpenVINO

```sh
call "C:\Program Files (x86)\Microsoft Visual Studio\2019\Professional\VC\Auxiliary\Build\vcvars64.bat"
cmake -G Ninja -Wno-dev -DCMAKE_BUILD_TYPE=Release ..
cmake --build . --parallel
```

## 另请参阅

 * [OpenVINO README](../../README.md)
 * [OpenVINO 开发者文档](index.md)
 * [OpenVINO 入门](./get_started.md)
 * [如何构建 OpenVINO](build.md)