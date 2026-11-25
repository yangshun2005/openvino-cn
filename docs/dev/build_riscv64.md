# 为 RISCV64 系统交叉编译 OpenVINO™ Runtime

本指南介绍如何为 64 位 RISC-V 设备构建 OpenVINO Runtime。由于资源有限，目前使用交叉编译来构建面向 RISC-V 开发板的 OpenVINO。

交叉编译已在以下主机上测试：
- Ubuntu 24.04（64 位），x64

软件已在以下设备上验证：
- [Lichee Pi 4A](https://wiki.sipeed.com/hardware/en/lichee/th1520/lp4a.html)，RVV 0.7.1
- [Banana Pi BPI-F3](https://www.banana-pi.org/en/banana-pi-sbcs/175.html)，RVV 1.0
- [Orange Pi RV2](http://www.orangepi.org/)，RVV 1.0


## 软件要求

- [CMake](https://cmake.org/download/) 3.26 或更高版本
- GCC 10 或更高版本（用于非 RVV）/
  [riscv-gnu-toolchain](https://github.com/riscv-collab/riscv-gnu-toolchain.git)（用于 RVV）
- Python 3.12，用于下文所述的可选 OpenVINO Runtime Python API 交叉构建

## 如何构建
目前有两种方式为 64 位 RISC-V 平台构建 OpenVINO Runtime：

1. 使用 [`riscv-gnu-toolchain`](https://github.com/riscv-collab/riscv-gnu-toolchain.git) 构建带 RVV intrinsics 的版本。该 GNU 编译器工具链支持 RVV 0.7.1 和已批准的 RVV 1.0。向量 intrinsics 使用公共前缀 `__riscv_`。
2. 使用已安装的 Linux 包构建不带优化原语的版本。这些包中的编译器不支持 RVV intrinsics。

> **注意**：目前 OpenVINO 中的 CPU 插件在 RVV 1.0 设备上支持有限操作范围的[即时（JIT）代码生成](https://github.com/openvinotoolkit/openvino/blob/master/src/plugins/intel_cpu/src/emitters/README.md)。
  上述三种为 64 位 RISC-V 构建 OpenVINO Runtime 的方式均支持 JIT 代码生成。

### 步骤

0. 前置条件：
- 使用 `riscv-gnu-toolchain` 构建带 RVV intrinsics 的目标：
   ```sh
   git clone https://github.com/riscv-collab/riscv-gnu-toolchain.git
   cd riscv-gnu-toolchain
   ./configure --prefix=/opt/riscv
   make linux build-qemu -j$(nproc)
   ```
   > **注意**：`build-qemu` 目标为可选项，用于构建 `qemu` 模拟器。但建议构建 `qemu` 模拟器，因为在主机上验证软件比在设备上方便得多。更多信息见[此处](https://github.com/riscv-collab/riscv-gnu-toolchain)。
- 使用已安装的 Linux 包构建不带优化原语的目标：
   ```sh
   apt-get update
   apt-get install -y  gcc-riscv64-linux-gnu g++-riscv64-linux-gnu binutils-riscv64-linux-gnu
   ```

1. 克隆 OpenVINO 仓库并初始化子模块：
   ```sh
   git clone --recursive https://github.com/openvinotoolkit/openvino.git
   cd openvino
   ```

2. 使用项目根目录下的 `install_build_dependencies.sh` 脚本安装构建依赖。
   ```sh
   sudo ./install_build_dependencies.sh
   ```

3. 创建构建目录：
   ```sh
   mkdir build && cd build
   ```

4. 要为 RISC-V 设备交叉编译 OpenVINO Runtime，请使用指定的 `CMAKE_TOOLCHAIN_FILE` 和 `RISCV_TOOLCHAIN_ROOT` 运行 `cmake`（后者仅在使用 GNU 工具链构建时需要）。
- 使用 `riscv-gnu-toolchain` 构建带 RVV intrinsics 的目标：
   ```sh
   cmake .. \
     -DCMAKE_BUILD_TYPE=Release \
     -DCMAKE_INSTALL_PREFIX=<openvino_install_path> \
     -DCMAKE_TOOLCHAIN_FILE=../cmake/toolchains/riscv64-gnu.toolchain.cmake \
     -DRISCV_TOOLCHAIN_ROOT=/opt/riscv
   ```
   > **注意**：构建 `riscv-gnu-toolchain` 是因为交叉编译所需的必要文件位于 `/opt/riscv/sysroot` 下。Clang 或 GCC 的最新稳定版本均支持将源代码编译为 RISC-V 指令，因此可以通过指定 `-DCMAKE_C_COMPILER` 和 `CMAKE_CXX_COMPILER` 选择你偏好的编译器。但请记得添加 `-DCMAKE_SYSROOT=/opt/riscv/sysroot`，否则交叉编译时许多基础头文件和库将无法找到。
- 使用已安装的 Linux 包构建不带优化原语的目标：
   ```sh
   cmake .. \
     -DCMAKE_BUILD_TYPE=Release \
     -DCMAKE_INSTALL_PREFIX=<openvino_install_path> \
     -DCMAKE_TOOLCHAIN_FILE=../cmake/toolchains/riscv64.linux.toolchain.cmake
   ```

   然后运行 `make` 构建项目：
   ```sh
   make install -j$(nproc)
   ```

### （可选）构建 OpenVINO Runtime Python API
要启用 Python 交叉编译，主机上需有 `libpython3-dev:riscv64` 库。

使用 `apt` 或 `apt-get` 工具安装包时，包从 apt 软件仓库下载。在 Ubuntu 上，apt 软件仓库定义在 `/etc/apt/sources.list` 文件或 `/etc/apt/sources.list.d/` 目录下的单独文件中。主机在这些文件中包含主机特定的仓库（例如 x86-x64）。

1. 添加 riscv64 仓库以下载 riscv64 专用包：
    ```sh
    echo deb [arch=riscv64] http://ports.ubuntu.com/ubuntu-ports/ noble main >> riscv64-sources.list
    echo deb [arch=riscv64] http://ports.ubuntu.com/ubuntu-ports/ noble universe >> riscv64-sources.list
    echo deb [arch=riscv64] http://ports.ubuntu.com/ubuntu-ports/ noble-updates main >> riscv64-sources.list
    echo deb [arch=riscv64] http://ports.ubuntu.com/ubuntu-ports/ noble-security main >> riscv64-sources.list
    mv riscv64-sources.list /etc/apt/sources.list.d/
    dpkg --add-architecture riscv64
    apt-get update -o Dir::Etc::sourcelist=/etc/apt/sources.list.d/riscv64-sources.list
    ```

2. 使用 `apt-get` 安装 `libpython3-dev:riscv64`：
    ```sh
    apt-get install -y --no-install-recommends libpython3-dev:riscv64
    ```
   创建符号链接，使 Python 能在 `/usr/include/python3.12/` 中找到 `riscv64-linux-gnu/python3.12/pyconfig.h`（该头文件初始位于 `/usr/include/riscv64-linux-gnu/`）
    ```sh
    ln -s /usr/include/riscv64-linux-gnu/ /usr/include/python3.12/
    ```

3. 在构建 OpenVINO 时，向 cmake 命令添加 `-DENABLE_PYTHON=ON -DENABLE_WHEEL=ON`。

> **注意**：当前 GitHub Actions 的 RISC-V 工作流未执行此可选的 Python 3.12 交叉构建，该工作流使用 `ENABLE_PYTHON=OFF` 构建 C++ Runtime。使用这些手动验证的说明时，目标设备必须提供 Python 3.12。

### RISC-V 模拟软件
要在没有硬件的情况下测试应用，可以使用模拟软件。使用 riscv64 模拟启动可执行文件的命令行示例：
```sh
<riscv_toolchain_root>/bin/qemu-riscv64 -cpu=<target_cpu> <executable_file_path>
```
`<riscv_toolchain_root>` 与构建时使用的 `RISCV_TOOLCHAIN_ROOT` 一致（例如 `/opt/riscv`），或使用系统 `PATH` 中的 `qemu-riscv64`。

例如，模拟 RVV 0.7.1：
```sh
<riscv_toolchain_root>/bin/qemu-riscv64 -cpu rv64,v=true,vext_spec=v0.7.1 <executable_file_path>
```

或模拟 RVV 1.0：
```sh
<riscv_toolchain_root>/bin/qemu-riscv64 -cpu rv64,v=true,vext_spec=v1.0 <executable_file_path>
```

## 另请参阅

 * [OpenVINO README](../../README.md)
 * [OpenVINO 开发者文档](index.md)
 * [OpenVINO 入门](./get_started.md)
 * [如何构建 OpenVINO](build.md)