# 构建 OpenVINO 静态库

## 目录

- [简介](#简介)
- [系统要求](#系统要求)
- [在 CMake 阶段配置 OpenVINO 运行时](#在-cmake-阶段配置-openvino-运行时)
- [构建静态 OpenVINO 库](#构建静态-openvino-库)
- [链接静态 OpenVINO 运行时](#链接静态-openvino-运行时)
- [静态 OpenVINO 库 + 针对特定模型的条件编译](#静态-openvino-库--针对特定模型的条件编译)
- [使用静态 MSVC 运行时构建](#使用静态-msvc-运行时构建)
- [为特定设备构建静态 OpenVINO](#为特定设备构建静态-openvino)
- [验证静态构建](#验证静态构建)
- [限制](#限制)
- [另见](#另见)

## 简介

构建静态 OpenVINO 运行时库，配合条件编译可进一步减小二进制体积。原因是静态构建时，OpenVINO 运行时库并非所有接口符号都会导出给最终用户，链接器可以将其移除。参见[静态 OpenVINO 库 + 针对特定模型的条件编译](#静态-openvino-库--针对特定模型的条件编译)。

## 系统要求

* 构建静态 OpenVINO 库需使用 CMake 3.26 或更高版本。
* 支持的操作系统：
    * Windows x64
    * Linux x64
    * 其他操作系统可能可用，但未经明确测试

## 在 CMake 阶段配置 OpenVINO 运行时

OpenVINO 运行时的默认架构假定以下组件在运行时动态加载：
* （设备）推理后端（CPU、GPU、NPU、MULTI、HETERO 等）
* （模型）前端（IR、ONNX、PDPD、TF、JAX 等）

使用静态 OpenVINO 运行时，所有这些模块都必须链接进最终用户应用，**且模块/配置列表必须在 CMake 配置阶段确定**。为最小化二进制体积，可显式将不需要的组件设为 `OFF`。OpenVINO CMake 配置可参考 [CMake Options for Custom Compilation](cmake_options_for_custom_compilation.md)。

例如，仅启用 IR v11 读取和 CPU 推理能力：

```sh
cmake -DENABLE_INTEL_GPU=OFF \
      -DENABLE_INTEL_NPU=OFF \
      -DENABLE_TEMPLATE=OFF \
      -DENABLE_HETERO=OFF \
      -DENABLE_MULTI=OFF \
      -DENABLE_AUTO=OFF \
      -DENABLE_AUTO_BATCH=OFF \
      -DENABLE_OV_ONNX_FRONTEND=OFF \
      -DENABLE_OV_PADDLE_FRONTEND=OFF \
      -DENABLE_OV_TF_FRONTEND=OFF \
      -DENABLE_OV_TF_LITE_FRONTEND=OFF \
      -DENABLE_OV_JAX_FRONTEND=OFF \
      -DENABLE_OV_PYTORCH_FRONTEND=OFF \
      -DENABLE_OV_JAX_FRONTEND=OFF \
      -DENABLE_INTEL_CPU=ON \
      -DENABLE_OV_IR_FRONTEND=ON
```

> **注意**：位于外部仓库的推理后端也可用于静态构建。使用 `-DOPENVINO_EXTRA_MODULES=<path to external plugin root>` 启用。构建外部插件不得使用 `OpenVINODeveloperPackage.cmake`，只有 `OPENVINO_EXTRA_MODULES` 是可行方案。

> **注意**：也可传入 `ENABLE_LTO` CMake 选项启用链接时优化以减小二进制体积。但链接静态 OpenVINO 库的目标也需通过 `set_target_properties(<target_name> PROPERTIES INTERPROCEDURAL_OPTIMIZATION_RELEASE ON)` 启用该属性。

## 构建静态 OpenVINO 库

要以静态模式构建 OpenVINO 运行时，需指定额外的 CMake 选项：

```sh
cmake -DBUILD_SHARED_LIBS=OFF <all other CMake options> <openvino_sources root>
```

然后使用常规的 CMake 构建命令：

```sh
cmake --build . --config Release -j12
```

> **注意**：不要将构建限制为 `--target openvino` —— 下面的安装步骤还期望其他组件（如 `openvino_c`），若未构建会失败：
> ```
> file INSTALL cannot find ".../bin/intel64/Release/openvino_c.lib": No error.
> ```
> 若只需构建部分目标，请在配置阶段禁用不需要的组件（如 `-DENABLE_JS=OFF`），或使用按组件安装：`cmake --install . --component core`。

然后执行安装步骤：

```sh
cmake -DCMAKE_INSTALL_PREFIX=<install_root> -P cmake_install.cmake
```

OpenVINO 运行时位于 `<install_root>/runtime/lib`

> **注意**：构建产物默认输出到*源码*树，而非构建目录。若从同一检出构建多个静态配置（如按设备，或 `/MD` 与 `/MT`），请为每个配置传入单独的 `-DOUTPUT_ROOT=<build_dir>`，否则会互相覆盖生成的文件。

## 链接静态 OpenVINO 运行时

构建并安装静态 OpenVINO 运行时库后，可用以下两种方式之一将其加入项目：

### CMake 接口

照常使用 CMake 的 `find_package` 并链接 `openvino::runtime`：

```cmake
find_package(OpenVINO REQUIRED)
target_link_libraries(<application> PRIVATE openvino::runtime)
```

`openvino::runtime` 会传递性地将所有其他静态 OpenVINO 库加入链接命令。

### 直接将库传给链接器

若要直接配置项目，需将 `<install_root>/runtime/lib` 中的所有库传给链接器命令。

> **注意**：静态库的顺序必须正确（依赖库应位于被依赖库**之前**），建议使用以下编译器特定标志链接静态 OpenVINO 库：

Microsoft Visual Studio 编译器：
```sh
/WHOLEARCHIVE:<ov_library 0> /WHOLEARCHIVE:<ov_library 1> ...
```

类 GCC 编译器：
```sh
gcc main.cpp -Wl,--whole-archive <all libraries from <root>/runtime/lib> > -Wl,--no-whole-archive -o a.out
```

## 静态 OpenVINO 库 + 针对特定模型的条件编译

OpenVINO 运行时可为特定模型编译，详见[针对特定模型的条件编译](conditional_compilation.md)指南。条件编译可与静态 OpenVINO 库配合，构建二进制体积更小的最终用户应用。步骤如下（基于详细的[针对特定模型的条件编译](conditional_compilation.md)指南）：

* 照常构建 OpenVINO 运行时，使用 CMake 选项 `-DSELECTIVE_BUILD=COLLECT`。
* 在目标模型和目标平台上运行目标应用以收集跟踪。
* 使用 `-DSELECTIVE_BUILD=ON -DSELECTIVE_BUILD_STAT=/path/*.csv -DBUILD_SHARED_LIBS=OFF` 构建最终的静态 OpenVINO 运行时。

## 使用静态 MSVC 运行时构建

要使用静态 MSVC 运行时构建，请使用专用的 [OpenVINO 工具链](https://github.com/openvinotoolkit/openvino/blob/master/cmake/toolchains/mt.runtime.win32.toolchain.cmake)文件：

```sh
cmake -DCMAKE_TOOLCHAIN_FILE=<openvino source dir>/cmake/toolchains/mt.runtime.win32.toolchain.cmake <other options>
```

> **注意**：所有其他依赖的应用和库都必须使用相同的 `mt.runtime.win32.toolchain.cmake` 工具链构建，以保证 `MSVC_RUNTIME_LIBRARY` 目标属性值一致。

## 为特定设备构建静态 OpenVINO

上述通用说明适用于所有设备。以下补充 `-DBUILD_SHARED_LIBS=OFF` 的设备特定行为，这些行为仅凭 CMake 错误信息不易察觉。

### CPU

无需额外选项；CPU 的静态构建方式与上述通用运行时相同。

### GPU

GPU 使用默认的 `-DGPU_RT_TYPE=OCL` 运行时静态构建，无需额外标志。

> **注意**：`OV_GPU_WITH_SYCL`（启用 SYCL 的 GPU 运行时）与 `-DENABLE_LTO=ON` 不兼容，CMake 配置会失败并报 `FATAL_ERROR: Intel GPU plugin with SYCL is not supported with ENABLE_LTO=ON`。若为 GPU 启用 SYCL，请勿启用 `ENABLE_LTO`。

### NPU

无需额外选项。`ENABLE_INTEL_NPU_INTERNAL` 控制 `compile_tool` 等内部 NPU 工具，它跟随 `BUILD_SHARED_LIBS`，在静态构建中默认禁用，因此安装清单不期望这些工具（[cmake/features.cmake](../../cmake/features.cmake)）。若显式用 `-DENABLE_INTEL_NPU_INTERNAL=ON` 重新启用，请同时构建相应目标（或如上所述构建默认目标），否则安装步骤会因找不到从未构建的二进制文件而失败。

NPU 离线编译器是预构建的共享库，**在静态构建中不可用**：`ENABLE_INTEL_NPU_COMPILER` 默认为 `${BUILD_SHARED_LIBS}`，静态构建无法强制设为 `ON`。不可用时，`CompilerAdapterFactory` 回退到驱动编译器（`NPU_COMPILER_TYPE=DRIVER`），而非共享构建默认使用的插件编译器（`NPU_COMPILER_TYPE=PLUGIN`）——参见 [compiler_adapter_factory.cpp](../../src/plugins/intel_npu/src/compiler_adapter/src/compiler_adapter_factory.cpp)。

### 从同一检出构建多个设备配置

仅当你刻意按设备构建*独立*的静态包时适用（例如独立验证每个设备，或发布更小的单设备二进制），不适用于同时启用多个设备的单次构建。为每个配置指定各自的 `OUTPUT_ROOT` 和安装前缀（见上文注意）：

```sh
cmake -S . -B build_cpu -DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED_LIBS=OFF -DOUTPUT_ROOT=$PWD/build_cpu
cmake --build build_cpu --config Release -j12
cmake --install build_cpu --config Release --prefix install_cpu

cmake -S . -B build_npu -DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED_LIBS=OFF -DOUTPUT_ROOT=$PWD/build_npu
cmake --build build_npu --config Release -j12
cmake --install build_npu --config Release --prefix install_npu
```

## 验证静态构建

可使用现有的 OpenVINO 示例验证静态构建：

* `hello_query_device` 报告编译进二进制的设备和插件；用它确认启用的插件（如 CPU、GPU、NPU）存在且可加载。
* `hello_classification`（或 `benchmark_app`）加载模型并在指定设备上运行推理，验证静态链接的插件和前端在运行时也能正常工作，而不仅仅是链接成功。

```sh
cmake -S samples/cpp -B samples_build -DOpenVINO_DIR=<install_root>/runtime/cmake
cmake --build samples_build --config Release
samples_build/intel64/Release/hello_query_device
samples_build/intel64/Release/hello_classification <model.xml> <image> <DEVICE>
```

这些示例与任何使用方应用一样，按[链接静态 OpenVINO 运行时](#链接静态-openvino-运行时)所述步骤链接静态包。

## 限制

* 静态构建中的设备支持：

  | 设备 | 可静态构建 | 说明 |
  |---|---|---|
  | CPU | 是 | |
  | GPU | 是 | 见上文 [GPU](#gpu) 说明 |
  | NPU | 是 | 见上文 [NPU](#npu) 说明 |
  | MULTI、HETERO、AUTO、BATCH | 是 | |
  | IR、ONNX、PDPD、TF、TF Lite 前端 | 是 | |

* 静态构建支持仅指为 OpenVINO 运行时库构建静态库。第三方预构建依赖保持原有格式——特别是 `TBB` 仍为共享库：它不会作为静态 OpenVINO 构建的一部分重新构建，预构建包会被复制到 `<install_root>/runtime/3rdparty/tbb/bin`，因此应用运行时必须能访问它（Windows 上在 `PATH` 中，Linux 上在 `LD_LIBRARY_PATH` 中）。

  > **注意**：预构建的 TBB 包也可直接下载：
  > * Linux x64：`https://storage.openvinotoolkit.org/dependencies/thirdparty/linux/oneapi-tbb-2021.13.1-lin-release.tgz`
  > * Windows x64：`https://storage.openvinotoolkit.org/dependencies/thirdparty/windows/oneapi-tbb-2021.13.3-vs2022-win.zip`
  >
  > TBB 版本因操作系统而异，且在不同 OpenVINO 发布版本间变化；下载前请查看 [cmake/dependencies.cmake](../../cmake/dependencies.cmake) 确认与你的检出匹配的版本。

### 使用自定义 TBB 构建

要针对自己的 TBB 而非预构建包构建，请遵循 [oneTBB 安装说明](https://github.com/uxlfoundation/oneTBB/blob/master/INSTALL.md)，并在配置 OpenVINO 前将 `TBBROOT` 设为安装目录。

若将 oneTBB 构建为静态库，请先阅读 [Static Linking of oneTBB](https://uxlfoundation.github.io/oneTBB/main/intro/static_linking.html)。这不是推荐的配置：仅当进程中恰好只有一份 oneTBB 副本时才安全，且会禁用需要运行时动态加载的功能（`tbbbind` 拓扑约束、`tbbmalloc_proxy`、TCM）。

> **注意**：你可能会注意到静态构建中 Windows x64 上 `ENABLE_TBBBIND_2_5` 选项为 `OFF`。这无影响——该选项仅对缺少动态 `tbbbind_2_5` 库的旧 TBB 包有意义。预构建的 oneTBB 自带该库，因此 NUMA 和混合核心检测仍可正常工作。

## 另见

 * [OpenVINO README](../../README.md)
 * [OpenVINO 开发者文档](index.md)
 * [如何构建 OpenVINO](build.md)