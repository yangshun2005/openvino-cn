# 为 Linux 系统构建 OpenVINO™ Runtime

当前验证环境包括：
- GitHub Actions 中的 Ubuntu 22.04、24.04 和 26.04（64 位）

外部基础设施 Docker 仓库还包含 RHEL 8.10 和 9.6 构建镜像，但当前产品 CI 配置未引用它们。其他兼容发行版也可构建 OpenVINO，但不在当前 CI 矩阵覆盖范围内。

## 软件要求

- [CMake](https://cmake.org/download/) 3.26 或更高版本
- GCC 10 或更高版本，用于以 C++20 标准构建 OpenVINO Runtime
- Python 3.11 - 3.15，用于 OpenVINO Runtime Python API
- （可选）安装 Intel® Graphics Compute Runtime for OpenCL™ Driver 包，以在 Intel 集成 GPU 上启用推理。

## 构建方法

1. 克隆 OpenVINO 仓库并初始化子模块：
   ```sh
   git clone https://github.com/openvinotoolkit/openvino.git
   cd openvino
   git submodule update --init --recursive
   ```
   （可选）中国用户可通过 gitee 镜像克隆子模块
   ```sh
   chmod +x scripts/submodule_update_with_gitee.sh
   ./scripts/submodule_update_with_gitee.sh
   ```

2. 使用项目根目录下的 `install_build_dependencies.sh` 脚本安装构建依赖。
   ```sh
   sudo ./install_build_dependencies.sh
   ```

3. 创建构建目录：
   ```sh
     mkdir build && cd build
   ```

> **注意**：在 Linux 上从源码编译 OpenVINO 前，建议禁用 oneAPI 环境，否则可能导致构建失败。

4. OpenVINO Runtime 使用基于 CMake 的构建系统。在创建的 `build` 目录中运行 `cmake` 获取项目依赖并生成 Unix makefile，然后运行 `make` 构建项目：
   ```sh
     cmake -DCMAKE_BUILD_TYPE=Release ..
     cmake --build . --parallel
   ```
   该过程可能需要一些时间。如果系统资源有限，建议指定较少的并行任务数，以免系统过载。这有助于在构建期间保持系统响应性和稳定性。使用 `nproc` 查看可用处理单元数。例如，要使用 8 个并行任务，运行以下命令：
      ```sh
      cmake --build . --parallel 8
      ```

### 其他构建选项

可使用以下其他构建选项：

- 对于 IA32 操作系统，使用 [ia32.linux.toolchain.cmake](https://github.com/openvinotoolkit/openvino/blob/master/cmake/toolchains/ia32.linux.toolchain.cmake) CMake 工具链文件：

   ```sh
   cmake -DCMAKE_TOOLCHAIN_FILE=<openvino_repo>/cmake/toolchains/ia32.linux.toolchain.cmake ..
   ```

- OpenVINO 提供多个 CMake 选项，可用于构建自定义 OpenVINO runtime，从而加快编译速度。这些选项允许通过禁用/启用其他插件和前端来跳过其编译。可用选项列表见[此处](https://github.com/openvinotoolkit/openvino/blob/master/docs/dev/cmake_options_for_custom_compilation.md)

- 构建 OpenVINO Runtime Python API：
  1. 在上述 CMake 步骤（第 4 步）中启用 `-DENABLE_PYTHON=ON` 选项。要指定确切的 Python 版本，使用以下选项（需要 cmake 3.16 及更高版本）：
     ```
     -DPython3_EXECUTABLE=/usr/bin/python3.11
     ```
  2. 要构建 wheel 包（.whl），在上述 CMake 步骤（第 4 步）中启用 `-DENABLE_WHEEL=ON` 选项，并安装依赖：
     ```sh
     pip install -r <openvino source tree>/src/bindings/python/wheel/requirements-dev.txt
     ```
  3. 构建完成后，将新构建的 Python 库导出到用户环境变量：
     ```
     export PYTHONPATH=<openvino_repo>/bin/intel64/Release/python:<openvino_repo>/tools/ovc:$PYTHONPATH
     export LD_LIBRARY_PATH=<openvino_repo>/bin/intel64/Release:$LD_LIBRARY_PATH
     export PATH=<openvino_repo>/tools/ovc/openvino/tools/ovc:$PATH
     ```
     或使用 pip 安装 wheel：
     ```
     pip install <openvino_repo>/build/wheel/openvino-2022.2.0-000-cp37-cp37-manylinux_2_35_x86_64.whl
     ```

## 另请参阅

 * [OpenVINO README](../../README.md)
 * [OpenVINO 开发者文档](index.md)
 * [OpenVINO 入门](./get_started.md)
 * [如何构建 OpenVINO](build.md)