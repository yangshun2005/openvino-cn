# Benchmark Python 工具

本页演示如何使用 Benchmark Python 工具估算受支持设备上的深度学习推理性能。

> **注意**：本页介绍 Benchmark 工具的 Python 实现用法。C++ 实现请参阅 [Benchmark C++ 工具](https://docs.openvino.ai/2026/get-started/learn-openvino/openvino-samples/benchmark-tool.html)页面。对将在 Python 应用中使用的模型进行基准测试，推荐使用 Python 版本；对将在 C++ 应用中使用的模型进行基准测试，推荐使用 C++ 版本。两个工具的命令接口和后端类似。

有关此示例工作原理的更多详细信息，请查看专门的[文章](https://docs.openvino.ai/2026/get-started/learn-openvino/openvino-samples/benchmark-tool.html)

## 要求

使用 [PyPI](https://docs.openvino.ai/2026/get-started/install-openvino/install-openvino-pip.html) 安装 OpenVINO Developer Tools 时，会自动安装 Python 版 benchmark_app。运行 ``benchmark_app`` 之前，请确保已激活 ``openvino_env`` 虚拟环境，并切换到模型所在目录。

基准测试应用支持 OpenVINO IR（``model.xml`` 和 ``model.bin``）和 ONNX（``model.onnx``）格式的模型。
如有必要，请务必[转换模型](https://docs.openvino.ai/2026/openvino-workflow/model-preparation/convert-model-to-ir.html)。