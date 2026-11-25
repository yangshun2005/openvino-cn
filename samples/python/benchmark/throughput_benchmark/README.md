# 吞吐量基准测试 Python 示例

本示例演示如何在吞吐量模式下使用异步推理请求 API 估算模型性能。你可以配置基准测试的持续时间和最小迭代次数，也可以持续运行直到收到停止信号。

报告的结果可能与 [benchmark_app](https://docs.openvino.ai/2026/get-started/learn-openvino/openvino-samples/benchmark-tool.html) 报告的结果有偏差。例如计算机视觉任务的模型输入精度：benchmark_app 设为 uint8，而本示例使用默认模型精度，通常为 float32。

有关本示例工作原理的详细信息，请参阅专门的[文章](https://docs.openvino.ai/2026/get-started/learn-openvino/openvino-samples/sync-benchmark.html)

## 用法

```sh
python throughput_benchmark.py <path_to_model> [device_name] [--seconds-to-run SECONDS] [--niter NITER]
```

模型路径为必填项。设备默认为 `CPU`。

| 选项 | 描述 |
| --- | --- |
| `--seconds-to-run SECONDS` | 非负整数，单位为秒。默认为 `10`。设为 `0` 则持续运行直到收到停止信号。 |
| `--niter NITER` | 正整数，最小迭代次数。默认为 `10`。 |

对于有限时长，基准测试会运行到时长和最小迭代次数均满足为止。例如，至少运行 30 秒且完成 100 次迭代：

```sh
python throughput_benchmark.py model.xml CPU --seconds-to-run 30 --niter 100
```

持续运行：

```sh
python throughput_benchmark.py model.xml CPU --seconds-to-run 0
```

按 Ctrl+C（`SIGINT`）停止，在 Linux 或 macOS 上发送 `SIGTERM`，或在 Windows 上按 Ctrl+Break（`SIGBREAK`）。即使尚未达到 `--niter`，停止信号也会结束运行。

## 要求

| 选项                        | 值                                                                                                                 |
| -------------------------------| ----------------------------------------------------------------------------------------------------------------------- |
| 已验证的模型               | [yolo-v3-tf](https://github.com/openvinotoolkit/open_model_zoo/tree/master/models/public/yolo-v3-tf)                   |
|                                | [face-detection-0200](https://github.com/openvinotoolkit/open_model_zoo/tree/master/models/intel/face-detection-0200)  |
| 模型格式                   | OpenVINO™ 工具套件中间表示                                                                          |
|                                | (\*.xml + \*.bin)、ONNX (\*.onnx)                                                                                      |
| 支持的设备              | [全部](https://docs.openvino.ai/2026/documentation/compatibility-and-support/supported-devices.html)                    |
| 其他语言实现     | [C++](https://docs.openvino.ai/2026/get-started/learn-openvino/openvino-samples/sync-benchmark.html)                               |

应用程序使用以下 Python API：

| 功能                   | API                                             | 描述                                  |
| --------------------------| ------------------------------------------------|----------------------------------------------|
| OpenVINO API 版本      | [openvino.\_\_version__]                          | 获取 Openvino API 版本。                    |
| 基本推理流程          | [openvino.Core],                        | 执行推理的通用 API：编译模型、 |
|                           | [openvino.Core.compile_model]           | 配置输入张量。                     |
|                           | [openvino.InferRequest.get_tensor]      |                                              |
| 异步推理        | [openvino.AsyncInferQueue],             | 执行异步推理。                   |
|                           | [openvino.AsyncInferQueue.start_async], |                                              |
|                           | [openvino.AsyncInferQueue.wait_all],    |                                              |
|                           | [openvino.InferRequest.results]         |                                              |
| 模型操作          | [openvino.CompiledModel.inputs]         | 获取模型的输入。                       |
| 张量操作         | [openvino.Tensor.get_shape],            | 获取张量形状及其数据。             |
|                           | [openvino.Tensor.data]                  |                                              |