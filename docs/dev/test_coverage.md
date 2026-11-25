# OpenVINO 覆盖率报告构建

覆盖率报告使用 Lcov 工具生成，基于 GCC 生成的性能分析数据。
生成的报告为 HTML 格式，位于 `<openvino_build>/coverage`。报告针对以下组件生成：

1. `inference` - 推理组件，属于 OpenVINO Runtime 库。
1. `core` - 核心组件，属于 OpenVINO Runtime 库。
1. `low_precision_transformations` - 低精度转换，属于 OpenVINO Runtime 库。
1. `openvino_transformations` - 通用转换，属于 OpenVINO Runtime 库。
1. `snippets` - OpenVINO snippets。
1. OpenVINO 开源插件：
    - `hetero_plugin` - 异构插件。
    - `auto_plugin` - Auto 插件。
    - `proxy_plugin` - Proxy 插件。
    - `auto_batch_plugin` - 自动批处理插件。
    - `template_plugin` - 模板插件。
    - `intel_gpu_plugin` - GPU 插件。
    - `intel_cpu_plugin` - CPU 插件。
    - `intel_npu_plugin` - Intel® NPU 插件。

## 构建时启用性能分析数据支持

要构建覆盖率报告，需在编译 OpenVINO 时添加 CMake 选项 `-DENABLE_COVERAGE=ON`：

```bash
$ cmake -DENABLE_COVERAGE=ON .
```

然后照常构建 OpenVINO。

## 生成覆盖率报告

要生成覆盖率报告，首先必须运行测试。运行的测试越多，覆盖率百分比越高。例如，对于 `openvino` 组件，必须运行 `InferenceEngineUnitTests`、`ieUnitTests`、`ieFuncTests` 以及插件测试。

```bash
$ ctest -V
```

OpenVINO 组件定义了若干通用分组，可用于运行单独组件的测试（使用 ctest 参数 `-L GROUP_NAME`）：

 - OV - OpenVINO 核心测试
 - IR_FE - IR 前端测试
 - ONNX_FE - ONNX 前端测试
 - PADDLE_FE - Paddle 前端测试
 - TF_FE - TensorFlow 前端测试
 - TFL_FE - TensorFlow Lite 前端测试
 - CPU - CPU 插件测试
 - GPU - GPU 插件测试
 - AUTO - Auto 插件测试
 - MULTI - 由 Auto 插件覆盖的多设备测试
 - AUTO_BATCH - 自动批处理插件测试
 - PROXY - Proxy 插件测试
 - NPUW - Intel® NPU 单元测试


执行足够数量的测试后，即可计算覆盖率数据。运行以下命令：

```bash
$ make ov_coverage
```

将生成以下报告目录树：

```bash
$ find coverage -maxdepth 2 -name index.html
coverage/auto_batch_plugin/index.html
coverage/auto_plugin/index.html
coverage/core/index.html
coverage/transformations/index.html
coverage/paddle_frontend/index.html
coverage/tf_frontend/index.html
coverage/openvino/index.html
coverage/onnx_frontend/index.html
coverage/ir_frontend/index.html
coverage/low_precision_transformations/index.html
coverage/proxy_plugin/index.html
coverage/template_plugin/index.html
coverage/inference/index.html
coverage/frontend_common/index.html
coverage/hetero_plugin/index.html
coverage/intel_cpu_plugin/index.html
coverage/intel_gpu_plugin/index.html
coverage/intel_npu_plugin/index.html
```

## 另请参阅
 * [OpenVINO™ README](../../README.md)
 * [开发者文档](../../docs/dev/index.md)