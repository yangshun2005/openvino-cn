# 为 Raspbian Stretch OS 构建 OpenVINO™ Runtime

> **注意**：自 2023.0 版本起，可在 ARM 平台上编译 [OpenVINO Intel CPU 插件](https://github.com/openvinotoolkit/openvino/tree/master/src/plugins/intel_cpu)。

## 硬件要求
* 搭载 Raspbian Stretch OS 或 Raspberry Pi OS（32 位或 64 位）的 Raspberry Pi。

  > **注意**：尽管 Raspberry Pi 的 CPU 是 ARMv8，32 位 OS 会识别为 ARMv7 指令集。默认的 `gcc` 编译器会应用 ARMv6 架构标志，以兼容较低版本的开发板。运行 `gcc -Q --help=target` 命令并参阅 `-march=` 选项的说明可了解更多信息。

## 编译
你可以为 Raspberry Pi 进行 OpenVINO Runtime 的本地编译，这是最直接的方式。但在 Raspberry Pi 3 上完成编译至少需要一小时。

1. 安装依赖：
  ```bash
  sudo apt-get update
  sudo apt-get install -y git cmake scons build-essential
  ```
2. 克隆仓库：
```
git clone --recurse-submodules --single-branch --branch=master https://github.com/openvinotoolkit/openvino.git 
```
3. 进入克隆的 `openvino` 仓库：

  ```bash
  cd openvino/
  ```
4. 创建构建目录：

  ```bash
  mkdir build && cd build/
  ```
5. 构建 OpenVINO Runtime：
  ```bash
  cmake -DCMAKE_BUILD_TYPE=Release \
        -DARM_COMPUTE_SCONS_JOBS=$(nproc --all) \
  .. && cmake --build . --parallel 
  ```

> **注意**：构建命令可能因内存不足而失败。可通过增大交换空间解决：
1. 停用当前交换空间：
```bash
sudo dphys-swapfile swapoff
```
2. 在 `/etc/dphys-swapfile` 中设置 `CONF_SWAPSIZE=8192` 以修改交换空间大小。
3. 重新创建交换文件：
```bash
sudo dphys-swapfile setup
```
3. 启用交换空间：
```bash
sudo dphys-swapfile swapon
```

## 为 Raspberry Pi ARM64 交叉编译

交叉编译使用一个主机专用的 CMake 工具链文件：

* `cmake/arm64.toolchain.cmake`：从 Linux 主机交叉编译到 Raspberry Pi AArch64 Linux；
* `cmake/rpi-aarch64-linux-from-macos.toolchain.cmake`：从 macOS 主机交叉编译到 Raspberry Pi AArch64 Linux；
* `cmake/rpi-aarch64-linux-from-windows.toolchain.cmake`：从 Windows 主机交叉编译到 Raspberry Pi AArch64 Linux。

macOS 和 Windows 工具链会在构建目录中生成编译器、binutils 和 Arm Compute Library 的 SCons 包装脚本。包装脚本模板和辅助源文件存放在 `cmake/rpi_cross/` 中。

为下载的主机工具和构建产物创建工作目录：

```bash
mkdir openvino-rpi-cross
export OV_RPI_HOME=${PWD}/openvino-rpi-cross
```

在 Windows PowerShell 中：

```powershell
$env:OV_RPI_HOME = "$PWD\openvino-rpi-cross"
New-Item -ItemType Directory -Force -Path $env:OV_RPI_HOME
```

### 准备目标 sysroot

macOS 和 Windows 工具链需要 Raspberry Pi ARM64 Linux sysroot。sysroot 必须与将要运行的 Raspberry Pi 用户空间匹配，尤其是 glibc 和 C++ 运行时版本。请使用可复现的软件包仓库、OS 镜像或已安装交叉工具链自带的 sysroot；不要从运行中的 Raspberry Pi 复制文件。

以下示例使用所选交叉工具链自带的 sysroot。如果使用来自其他来源、与发行版匹配的 sysroot，请通过 `-DOV_RPI_SYSROOT=<path>` 传入。

### Linux 主机

安装主机工具：

```bash
sudo apt-get update
sudo apt-get install -y git cmake ninja-build scons python3 python3-pip \
    gcc-aarch64-linux-gnu g++-aarch64-linux-gnu pkg-config xz-utils zstd
```

配置并构建：

```bash
cmake -S . -B build-rpi-aarch64 -G Ninja \
    -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_TOOLCHAIN_FILE=$PWD/cmake/arm64.toolchain.cmake \
    -DCMAKE_INSTALL_PREFIX=$PWD/install-rpi-aarch64

cmake --build build-rpi-aarch64 --parallel
cmake --install build-rpi-aarch64
```

Linux 工具链文件使用标准的 `aarch64-linux-gnu-*` 工具名，并有意保持精简。如果你的 Linux 交叉工具链需要显式的目标 sysroot，请传入与你的发行版工具链匹配的标准 CMake sysroot 设置。

### macOS 主机

安装主机工具并下载 Linux AArch64 GNU 交叉工具链：

```bash
brew install cmake ninja scons python xz zstd
brew install messense/macos-cross-toolchains/aarch64-unknown-linux-gnu
```

为 CMake 设置工具链 sysroot：

```bash
export OV_RPI_SYSROOT="$(aarch64-linux-gnu-gcc -print-sysroot)"
export OV_RPI_TOOLCHAIN_PREFIX=aarch64-linux-gnu
```

使用 macOS 主机的 Raspberry Pi ARM64 Linux 工具链配置并构建：

```bash
cmake -S . -B build-rpi-aarch64-macos -G Ninja \
    -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_TOOLCHAIN_FILE=$PWD/cmake/rpi-aarch64-linux-from-macos.toolchain.cmake \
    -DOV_RPI_TOOLCHAIN_PREFIX=$OV_RPI_TOOLCHAIN_PREFIX \
    -DOV_RPI_SYSROOT=$OV_RPI_SYSROOT \
    -DCMAKE_INSTALL_PREFIX=$PWD/install-rpi-aarch64-macos

cmake --build build-rpi-aarch64-macos --parallel
cmake --install build-rpi-aarch64-macos
```

macOS 工具链文件会在构建目录中生成主机包装脚本，并将 CMake、binutils 和 Arm Compute Library 指向目标 sysroot。

### Windows 主机

从 PowerShell 下载并解压 CMake 和 Ninja：

```powershell
$env:OV_RPI_TOOLS = "$env:OV_RPI_HOME\tools"
New-Item -ItemType Directory -Force -Path $env:OV_RPI_TOOLS

$CMakeVersion = "3.30.5"
$CMakeUrl = "https://github.com/Kitware/CMake/releases/download/v$CMakeVersion/cmake-$CMakeVersion-windows-x86_64.zip"
$CMakeZip = "$env:OV_RPI_HOME\cmake-$CMakeVersion-windows-x86_64.zip"
Invoke-WebRequest -Uri $CMakeUrl -OutFile $CMakeZip
Expand-Archive -Path $CMakeZip -DestinationPath $env:OV_RPI_TOOLS -Force
Move-Item -Force "$env:OV_RPI_TOOLS\cmake-$CMakeVersion-windows-x86_64" "$env:OV_RPI_TOOLS\cmake"

$NinjaVersion = "1.12.1"
$NinjaUrl = "https://github.com/ninja-build/ninja/releases/download/v$NinjaVersion/ninja-win.zip"
$NinjaZip = "$env:OV_RPI_HOME\ninja-win.zip"
New-Item -ItemType Directory -Force -Path "$env:OV_RPI_TOOLS\ninja"
Invoke-WebRequest -Uri $NinjaUrl -OutFile $NinjaZip
Expand-Archive -Path $NinjaZip -DestinationPath "$env:OV_RPI_TOOLS\ninja" -Force
```

安装 Python 和 SCons：

```powershell
$PythonVersion = "3.12"
winget install --id "Python.Python.$PythonVersion" -e
py "-$PythonVersion" -m pip install --user scons
$PythonScripts = py "-$PythonVersion" -c "import site; print(site.getuserbase() + r'\Scripts')"
```

下载并解压面向 AArch64 GNU/Linux 的 Windows 版 Arm GNU 工具链：

```powershell
$ToolchainUrl = "https://developer.arm.com/-/media/Files/downloads/gnu/14.3.rel1/binrel/arm-gnu-toolchain-14.3.rel1-mingw-w64-i686-aarch64-none-linux-gnu.zip"
$ToolchainZip = "$env:OV_RPI_HOME\arm-gnu-toolchain-aarch64-none-linux-gnu.zip"
Invoke-WebRequest -Uri $ToolchainUrl -OutFile $ToolchainZip
Expand-Archive -Path $ToolchainZip -DestinationPath $env:OV_RPI_TOOLS -Force
$env:OV_RPI_TOOLCHAIN_ROOT = "$env:OV_RPI_TOOLS\arm-gnu-toolchain-14.3.rel1-mingw-w64-i686-aarch64-none-linux-gnu"
```

Windows 版 Arm 包使用 `aarch64-none-linux-gnu` 前缀，并在 `$env:OV_RPI_TOOLCHAIN_ROOT\aarch64-none-linux-gnu\libc` 下提供目标 sysroot。

使用 Windows 主机的 Raspberry Pi ARM64 Linux 工具链配置并构建：

```powershell
$env:OV_RPI_REPO = (Get-Location).Path
$env:Path = "$env:OV_RPI_TOOLCHAIN_ROOT\bin;$env:OV_RPI_TOOLS\ninja;$env:OV_RPI_TOOLS\cmake\bin;$PythonScripts;$env:Path"
$env:OV_RPI_SYSROOT = "$env:OV_RPI_TOOLCHAIN_ROOT\aarch64-none-linux-gnu\libc"
$env:OV_RPI_TOOLCHAIN_PREFIX = "aarch64-none-linux-gnu"

& "$env:OV_RPI_TOOLS\cmake\bin\cmake.exe" -S $env:OV_RPI_REPO -B "$env:OV_RPI_REPO\build-rpi-aarch64-windows" -G Ninja `
    "-DCMAKE_MAKE_PROGRAM=$env:OV_RPI_TOOLS\ninja\ninja.exe" `
    -DCMAKE_BUILD_TYPE=Release `
    "-DCMAKE_TOOLCHAIN_FILE=$env:OV_RPI_REPO\cmake\rpi-aarch64-linux-from-windows.toolchain.cmake" `
    "-DOV_RPI_TOOLCHAIN_PREFIX=$env:OV_RPI_TOOLCHAIN_PREFIX" `
    "-DOV_RPI_SYSROOT=$env:OV_RPI_SYSROOT" `
    "-DCMAKE_INSTALL_PREFIX=$env:OV_RPI_REPO\install-rpi-aarch64-windows"

& "$env:OV_RPI_TOOLS\cmake\bin\cmake.exe" --build "$env:OV_RPI_REPO\build-rpi-aarch64-windows" --parallel
& "$env:OV_RPI_TOOLS\cmake\bin\cmake.exe" --install "$env:OV_RPI_REPO\build-rpi-aarch64-windows"
```

Windows 工具链文件会在构建目录中生成主机包装脚本、OpenVINO 的 CMake 探测所需的本地 `grep.exe`，以及 Arm Compute Library 的 SCons 包装脚本。除非已知 libc 和运行时版本兼容，否则不要将 Arm GNU 工具链的 C++ 运行时头文件与其他发行版的 sysroot 混用。

### 构建 CPU 基准测试运行时目标

若只需构建在 Raspberry Pi CPU 上运行 `benchmark_app` 所需的运行时组件，请显式构建 CPU 插件、IR 前端和 `benchmark_app`：

```bash
cmake --build build-rpi-aarch64 --target \
    openvino_intel_cpu_plugin openvino_ir_frontend benchmark_app \
    --parallel
```

对于 macOS 和 Windows 构建，将 `build-rpi-aarch64` 替换为 `build-rpi-aarch64-macos` 或 `build-rpi-aarch64-windows`。

### 在 Raspberry Pi 上验证并运行

任何交叉构建完成后，验证生成的二进制文件是 Linux AArch64 ELF 文件：

```bash
${OV_RPI_TOOLCHAIN_PREFIX:-aarch64-linux-gnu}-readelf -h \
    bin/aarch64/Release/benchmark_app
```

在 Windows 主机上，使用 Arm GNU 工具链的 `readelf.exe`：

```powershell
& "$env:OV_RPI_TOOLCHAIN_ROOT\bin\aarch64-none-linux-gnu-readelf.exe" -h `
    .\bin\aarch64\Release\benchmark_app
```

输出必须包含 `Class: ELF64` 和 `Machine: AArch64`。

如需快速进行 CPU 冒烟测试，将运行时输出目录复制到 Raspberry Pi，并在 `LD_LIBRARY_PATH` 中指定复制的库来运行 `benchmark_app`：

```bash
rsync -a bin/aarch64/Release/ rpi@raspberrypi:~/openvino-rpi-cross/bin/

ssh rpi@raspberrypi \
    'cd ~/openvino-rpi-cross/bin && \
     LD_LIBRARY_PATH=$PWD ./benchmark_app \
       -m ~/models/model.xml \
       -d CPU \
       -api sync \
       -niter 10 \
       -nireq 1 \
       -nstreams 1 \
       -hint none'
```

使用 `cmake --install` 安装构建产物是保留预期运行时布局的首选方式。

## 其他构建选项

- 要构建 Python API，请使用 `apt-get` 安装 `libpython3-dev:armhf` 和 `python3-pip` 包；并添加以下选项：
   ```sh
   -DENABLE_PYTHON=ON \
   -DPython3_EXECUTABLE=/usr/bin/python3.8
   ```

## 另请参阅

 * [OpenVINO README](../../README.md)
 * [OpenVINO 开发者文档](index.md)
 * [OpenVINO 入门](./get_started.md)
 * [如何构建 OpenVINO](build.md)