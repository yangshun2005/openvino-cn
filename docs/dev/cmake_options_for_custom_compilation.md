# 自定义编译的 CMake 选项

本文档说明可用于基于开源版本构建自定义 OpenVINO 运行时的 CMake 选项及其默认值。如需了解从开源仓库创建自定义运行时的全部依赖关系，请参阅 [OpenVINO 运行时简介]。

## 目录：

* [禁用/启用插件构建及其他组件](#禁用启用插件构建及其他组件)
* [影响二进制大小的选项](#影响二进制大小的选项)
* [使用自定义 OpenCV 构建](#使用自定义-opencv-构建)
* [使用自定义 TBB 构建](#使用自定义-tbb-构建)
* [测试功能](#测试功能)
* [其他选项](#其他选项)
- [其他资源](#其他资源)

## 禁用/启用插件构建及其他组件

* 推理插件：
    * `ENABLE_INTEL_CPU` 启用 CPU 插件编译：
        * x86 平台默认为 `ON`；其他平台为 `OFF`。
    * `ENABLE_INTEL_GPU` 启用 Intel GPU 插件编译：
        * x86 平台默认为 `ON`；其他平台不可用。
    * `ENABLE_INTEL_NPU` 启用 Intel NPU 插件编译：
        * Windows 和 Linux x86 平台默认为 `ON`；其他平台不可用。
    * `ENABLE_HETERO` 启用 HETERO 插件构建：
        * 默认为 `ON`。
    * `ENABLE_MULTI` 启用 MULTI 插件构建：
        * 默认为 `ON`。
    * `ENABLE_AUTO` 启用 AUTO 插件构建：
        * 默认为 `ON`。
    * `ENABLE_TEMPLATE` 启用 TEMPLATE 插件构建：
        * 默认为 `ON`。
    * `ENABLE_AUTO_BATCH` 启用 Auto Batch 插件构建：
        * 默认为 `ON`。
    * `ENABLE_PROXY` 启用 Proxy 插件编译：
        * 默认为 `ON`。
* 用于处理各框架模型的前端：
    * `ENABLE_OV_ONNX_FRONTEND` 启用 OpenVINO 运行时的 [ONNX] 前端插件：
        * 默认为 `ON`。
    * `ENABLE_OV_PADDLE_FRONTEND` 启用 OpenVINO 运行时的 [PDPD] 前端插件：
        * 默认为 `ON`。
    * `ENABLE_OV_TF_FRONTEND` 启用 OpenVINO 运行时的 [TensorFlow] 前端插件：
        * 默认为 `ON`。
    * `ENABLE_OV_TF_LITE_FRONTEND` 启用 OpenVINO 运行时的 [TensorFlow Lite] 前端插件：
        * 默认为 `ON`。
    * `ENABLE_OV_PYTORCH_FRONTEND` 启用 OpenVINO 运行时的 [PyTorch] 前端插件：
        * 默认为 `ON`。
    * `ENABLE_OV_JAX_FRONTEND` 启用 OpenVINO 运行时的 [JAX] 前端插件：
        * 默认为 `ON`。
    * `ENABLE_OV_IR_FRONTEND` 启用 OpenVINO 运行时的 OpenVINO 中间表示前端插件：
        * 默认为 `ON`。
* `OPENVINO_EXTRA_MODULES` 指定添加额外 OpenVINO 模块到构建的路径。
    * 参见 [OpenVINO Contrib] 以添加额外模块。
* `ENABLE_SAMPLES` 启用 OpenVINO 运行时示例构建：
    * 默认为 `ON`。
* `ENABLE_PYTHON` 启用 [Python] API 构建：
    * 若满足 Python 要求（由 CMake 自动发现），则为 `ON`。
* `ENABLE_WHEEL` 启用 [Python] OpenVINO 运行时和开发 wheel 构建：
    * 若满足要求（由 CMake 自动发现），则为 `ON`。
* `ENABLE_TESTS` 启用测试编译：
    * 默认为 `OFF`。
* `ENABLE_DOCS` 启用 OpenVINO 文档构建：
    * 在 Debian（Ubuntu）操作系统上为 `OFF`
    * 其他情况下为 `OFF`。
* `ENABLE_SYSTEM_PUGIXML` 若系统上可用，则使用系统版本的 [pugixml] 构建。
    * 默认为 `ON`。
    * 默认使用 [OpenVINO 第三方 pugixml]。
* `ENABLE_SYSTEM_PROTOBUF` 使用系统上安装的 [protobuf]（供 ONNX、PaddlePaddle 和 TensorFlow 前端使用）：
    * 默认为 `OFF`。
* `ENABLE_SYSTEM_FLATBUFFERS` 使用系统上安装的 [FlatBuffers]（供 TensorFlow Lite 前端使用）：
    * 默认为 `ON`。
* `ENABLE_SYSTEM_TBB` 使用系统上安装的 TBB：
    * 在 Debian（Ubuntu）操作系统上为 `ON`。
    * 其他情况下为 `OFF`。
* `ENABLE_SYSTEM_OPENCL` 使用系统上安装的 OpenCL：
    * 默认为 `ON`。

## 影响二进制大小的选项

* `ENABLE_LTO` 布尔选项，用于启用[链接时优化]：
    * 默认为 `OFF`，因为链接二进制文件耗时更长。
    * OpenVINO 发布版本二进制文件启用 `ON`。
    * 仅适用于 Unix* 编译器，如 GCC 或 CLANG。
    * 与构建 OpenVINO 所用的其他优化选项配合，可使二进制大小减少 30%。
* `THREADING` 指定 OpenVINO 线程接口：
    * `TBB` 为默认选项，启用 [Intel TBB] 和 `tbb::static_partitioner` 构建。
    * `TBB_AUTO` 启用 [Intel TBB] 构建。
    * `OMP` 启用 Intel OpenMP 构建。
    * `SEQ` 禁用线程优化。可在缺少 TBB 二进制文件时使用。
    * **注意：** 由于 TBB 是模板库，`tbb::parallel_for` 的多次实例化会增加二进制大小。
* `ENABLE_TBBBIND_2_5` 启用预构建静态 TBBBind 2.5：
    * 默认为 `ON`，因为 OpenVINO 运行时开箱即用应具备通用性。

> **注意：** 当 OpenVINO **推理**目标 CPU 具备以下特性时需要 TBBBind 2.5：
> * NUMA 支持（非统一内存架构），例如用于检测 NUMA 节点数量
> * 混合架构，用于区分性能核/能效核并以最优方式调度任务。

> **注意：** 若使用 [oneTBB] 支持构建 OpenVINO 运行时，且 TBBBind 2.5 由 TBB 在运行时自动加载，则将 `ENABLE_TBBBIND_2_5` 设为 `OFF`
> * 确保 libtbbbind.so 与 libtbb.so 位于同一文件夹。例如，Ubuntu 22.04 上的 oneTBB 包缺少 libtbbbind。https://bugs.launchpad.net/ubuntu/+source/onetbb/+bug/2006898
> * oneTBB 依赖更高版本的 hwloc 来正确识别混合 CPU 核心类型，某些机器上需要更高版本的 hwloc 才能正常工作。检查 `hwloc-info --version` 是否返回 hwloc 版本 >= 2.7.0，Ubuntu 20.04 为 hwloc 2.1.0

* `ENABLE_SSE42` 启用 SSE4.2 优化：
    * x86 平台默认为 `ON`；其他平台不可用。
    * 仅影响 OpenVINO 运行时公共部分和预处理插件，**不影响 oneDNN 库**
* `ENABLE_AVX2` 启用 AVX2 优化：
    * x86 平台默认为 `ON`，其他平台不可用。
    * 仅影响 OpenVINO 运行时公共部分和预处理插件，**不影响 oneDNN 库**
* `ENABLE_AVX512F` 启用 AVX512 优化：
    * x86 平台默认为 `ON`，其他平台不可用。
    * 仅影响 OpenVINO 运行时公共部分和预处理插件，**不影响 oneDNN 库**
* `ENABLE_PROFILING_ITT` 启用 [Intel ITT 和 VTune] 性能分析。
    * 默认为 `OFF`，因为会增加二进制大小。
* `SELECTIVE_BUILD` 启用[[条件编译|ConditionalCompilation]]功能。
    * 默认为 `OFF`。
* `ENABLE_MLAS_FOR_CPU` 为 CPU 插件启用 MLAS 库
    * x86_64 和 AARCH64 平台默认为 `ON`
    * 仅影响 OpenVINO CPU 插件

## 使用自定义 OpenCV 构建

部分 OpenVINO 示例可从 OpenCV 中获益，例如可读取更多图像格式作为输入。若机器上已安装 OpenCV，可通过 CMake 选项传入：

```sh
cmake -DOpenCV_DIR=<path to OpenCVConfig.cmake> ...
```

## 使用自定义 TBB 构建

当 OpenVINO CMake 脚本以启用 TBB 的方式运行（`-DTHREADING=TBB`，默认值，或 `-DTHREADING=TBB_AUTO`）时，CMake OpenVINO 脚本会自动下载与系统默认编译器 ABI 兼容的预构建 TBB 版本。若使用非默认编译器或希望使用自定义 TBB，可使用：

```sh
export TBBROOT=<path to TBB install root>
cmake ...
```
此时 OpenVINO CMake 脚本会考虑 `TBBROOT` 环境变量，并使用提供的 TBB。

**注意：** 若从源文件构建 TBB，请先安装 TBB，然后用 `TBBROOT` 指向安装根目录。
**注意：** 参见[影响二进制大小的选项](#影响二进制大小的选项)中的 oneTBB 注意事项

## 测试功能

* `ENABLE_SANITIZER` 使用 clang [地址消毒器]支持构建：
    * 默认为 `OFF`。
* `ENABLE_THREAD_SANITIZER` 使用 clang [线程消毒器]支持构建：
    * 默认为 `OFF`。
* `ENABLE_COVERAGE` 添加启用覆盖率的选项。参见专门指南[[如何测量测试覆盖率|InferenceEngineTestsCoverage]]：
    * 默认为 `OFF`。
* `ENABLE_FUZZING` 启用模糊测试的代码插桩：
    * 默认为 `OFF`。

## 其他选项

* `ENABLE_CLANG_FORMAT` 启用 [Clang format] 代码风格检查：
    * 默认为 `ON`。
* `ENABLE_FASTER_BUILD` 使用 CMake 启用[预编译头]和[统一构建]：
    * 默认为 `OFF`。
* `ENABLE_INTEGRITYCHECK` 使用 [/INTEGRITYCHECK] 标志构建 DLL：
    * 默认为 `OFF`。
    * 仅适用于 MSVC 编译器。
* `ENABLE_STATIC_LIBS_PDB` 当使用 `-DCMAKE_MSVC_DEBUG_INFORMATION_FORMAT=Embedded`（`/Z7`）时，将开发者包静态库的调试信息保留在单独的 PDB 文件（`/Zi`）中：
    * 默认为 `OFF`。
    * 仅适用于 MSVC 编译器。
    * 编译器缓存（如 ccache）需要嵌入式调试信息（`/Z7`），因为 ccache 无法缓存写入共享 PDB 文件（`/Zi`）的编译。使用 `/Z7` 时，每个目标文件携带自己的调试信息副本，因此归档这些目标文件的静态库会显著增大，而 DLL 和可执行文件不受影响，因为链接器会将调试信息移入其 PDB 文件。启用此选项后，安装到开发者包的静态库（以及归档到其中的目标库）使用 `/Zi` 构建以保持大小，而所有其他目标（包括 DLL）保持 `/Z7` 并保持可缓存。
* `ENABLE_QSPECTRE` 使用 [/Qspectre] 标志构建：
    * 默认为 `OFF`。
    * 仅适用于 MSVC 编译器。

## 其他资源

 * [OpenVINO README](../../README.md)
 * [OpenVINO 开发者文档](index.md)
 * [如何构建 OpenVINO](build.md)

[链接时优化]:https://llvm.org/docs/LinkTimeOptimization.html
[线程消毒器]:https://clang.llvm.org/docs/ThreadSanitizer.html
[地址消毒器]:https://clang.llvm.org/docs/AddressSanitizer.html
[Intel ITT 和 VTune]:https://software.intel.com/content/www/us/en/develop/documentation/vtune-help/top/api-support/instrumentation-and-tracing-technology-apis.html
[预编译头]:https://cmake.org/cmake/help/git-stage/command/target_precompile_headers.html
[统一构建]:https://cmake.org/cmake/help/latest/prop_tgt/UNITY_BUILD.html
[/INTEGRITYCHECK]:https://docs.microsoft.com/en-us/cpp/build/reference/integritycheck-require-signature-check?view=msvc-160
[/Qspectre]:https://learn.microsoft.com/en-us/cpp/build/reference/qspectre?view=msvc-170
[Intel TBB]:https://software.intel.com/content/www/us/en/develop/tools/threading-building-blocks.html
[Python]:https://www.python.org/
[Java]:https://www.java.com/ru/
[Clang format]:http://clang.llvm.org/docs/ClangFormat.html
[OpenVINO Contrib]:https://github.com/openvinotoolkit/openvino_contrib
[OpenVINO 第三方 pugixml]:https://github.com/openvinotoolkit/openvino/tree/master/thirdparty/pugixml
[pugixml]:https://pugixml.org/
[ONNX]:https://onnx.ai/
[protobuf]:https://github.com/protocolbuffers/protobuf
[OpenVINO 运行时简介]:https://docs.openvino.ai/2026/openvino-workflow/running-inference.html
[PDPD]:https://github.com/PaddlePaddle/Paddle
[TensorFlow]:https://www.tensorflow.org/
[TensorFlow Lite]:https://www.tensorflow.org/lite
[PyTorch]:https://pytorch.org/
[FlatBuffers]:https://google.github.io/flatbuffers/
[oneTBB]:https://github.com/oneapi-src/oneTBB
[JAX]:https://github.com/google/jax