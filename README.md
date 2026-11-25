

> **[中文化] openvino**
>
> 此项目是 [openvinotoolkit/openvino](https://github.com/openvinotoolkit/openvino) 的中文翻译版本。
> - 原项目 Stars: 12579
> - 主语言: C++
> - 许可证: Apache-2.0
> - 翻译日期: 2026-11-25
> - 原始 README: [README_en.md](README_en.md)
>
> 说明：下方 README 正文为**原项目文档的中文翻译**，力求忠于原文；本页顶部的「中文快速上手」为**面向中文用户的补充导读**（非原文内容），目的是让你不用读完整篇 README 就能先把项目跑起来。
>
> 如有翻译不准确之处，欢迎提 Issue 或 PR。

---


## 🚀 中文快速上手（补充导读，非原文）

### 这是什么 / 解决什么问题

OpenVINO 是 Intel 开源的一套深度学习模型优化与部署工具包，主要用来提升模型在推理阶段的性能。它支持把 PyTorch、TensorFlow、ONNX、Keras、PaddlePaddle、JAX/Flax 等框架训练出的模型转换后部署，并且不依赖原始训练框架。它面向从边缘到云端的多种平台，可在 CPU（x86、ARM）、Intel 集成/独立 GPU 以及 Intel NPU 上做推理。

### 快速上手（最短路径）

1. 安装 OpenVINO（README 给出的快速安装命令）：
   ```sh
   pip install -U openvino
   ```
2. 验证安装是否成功，运行：
   ```python
   import openvino as ov
   print(ov.__version__)
   ```
3. 按 README 指引，跟着 [OpenVINO Quickstart example](https://docs.openvino.ai/2026/get-started.html) 走一遍部署第一个模型的流程。
4. 想跑现成示例，可参考 [OpenVINO Notebooks](https://github.com/openvinotoolkit/openvino_notebooks)（README 列了 LLM 聊天机器人、YOLOv11 优化、文生图、LLaVa 多模态、Whisper 语音识别等 notebook）。
5. 若要用生成式 AI 能力，README 指向 OpenVINO GenAI 的[安装说明](https://docs.openvino.ai/2026/get-started/install-openvino/install-openvino-genai.html)与[详细指南](https://docs.openvino.ai/2026/openvino-workflow-generative/inference-with-genai.html)。

### 环境要求

- **操作系统**：原文未在提供的 README 片段中逐条列出，仅给出系统要求文档链接（[system requirements](https://docs.openvino.ai/2026/about-openvino/release-notes-openvino/system-requirements.html)），具体以该文档为准。
- **硬件**：支持 CPU（x86、ARM）、Intel 集成与独立 GPU、Intel NPU 推理；具体支持设备见[supported devices](https://docs.openvino.ai/2026/documentation/compatibility-and-support/supported-devices.html)。
- **Python 版本**：`pyproject.toml` 中 `requires-python = ">=3.11"`，classifiers 列出 3.11–3.15。
- **Python 依赖**：`numpy>=1.16.6,<2.6.0`、`openvino-telemetry>=2023.2.1`。
- **从源码构建**：`CMakeLists.txt` 要求 `cmake_minimum_required(VERSION 3.26)`；构建依赖可用仓库根目录的 `install_build_dependencies.sh`（原文未说明其具体用法）。
- **国内网络提示**：README 未提及镜像源，若 pip 下载慢可自行更换国内 PyPI 镜像（此为通用建议，非原文内容）。

### 常见坑与注意事项

- **硬件相关**：GPU/NPU 推理依赖 Intel 对应硬件，非 Intel 设备上主要走 CPU 路径；具体支持列表原文指向外部文档，未在 README 中展开。
- **Python 版本门槛**：`pyproject.toml` 要求 Python ≥ 3.11，低于此版本无法按该配置安装。
- **生成式 AI 需额外组件**：LLM/GenAI 相关能力需按 OpenVINO GenAI 的独立安装说明操作，不是 `pip install openvino` 一步到位（README 将其单列章节）。
- **源码构建依赖较重**：涉及 CMake ≥ 3.26、conan/vcpkg 等构建文件，且需自行准备构建依赖，普通用户建议直接用 pip 安装。

---



---

<div align="center">
<img src="https://storage.openvinotoolkit.org/assets/branding/openvino-logo-transparent-white-outline.svg" width="400px" alt="OpenVINO">

<h3 align="center">
用于优化和部署深度学习模型的开源软件工具包。
</h3>

<p align="center">
 <a href="https://docs.openvino.ai/2026/index.html"><b>文档</b></a> • <a href="https://blog.openvino.ai"><b>博客</b></a> • <a href="https://docs.openvino.ai/2026/about-openvino/key-features.html"><b>核心特性</b></a> • <a href="https://docs.openvino.ai/2026/get-started/learn-openvino.html"><b>教程</b></a> • <a href="https://docs.openvino.ai/2026/about-openvino/openvino-ecosystem/openvino-integrations.html"><b>集成</b></a> • <a href="https://www.intel.com/content/www/us/en/developer/tools/openvino-toolkit/model-hub.html"><b>基准测试</b></a> • <a href="https://github.com/openvinotoolkit/openvino.genai"><b>生成式 AI</b></a>
</p>

[![PyPI Status](https://badge.fury.io/py/openvino.svg)](https://badge.fury.io/py/openvino)
[![NPM](https://img.shields.io/npm/v/openvino-node)](https://www.npmjs.com/package/openvino-node)
[![Anaconda Status](https://anaconda.org/conda-forge/openvino/badges/version.svg)](https://anaconda.org/conda-forge/openvino)
[![brew Status](https://img.shields.io/homebrew/v/openvino)](https://formulae.brew.sh/formula/openvino)

[![PyPI Downloads](https://static.pepy.tech/badge/openvino)](https://pepy.tech/project/openvino)
[![Anaconda Downloads](https://anaconda.org/conda-forge/libopenvino/badges/downloads.svg)](https://anaconda.org/conda-forge/openvino/files)
[![brew Downloads](https://img.shields.io/homebrew/installs/dy/openvino)](https://formulae.brew.sh/formula/openvino)
 </div>


- **推理优化**：提升计算机视觉、自动语音识别、生成式 AI、自然语言处理（含大语言模型和小语言模型）以及众多常见任务的深度学习性能。
- **灵活的模型支持**：可使用 PyTorch、TensorFlow、ONNX、Keras、PaddlePaddle 和 JAX/Flax 等主流框架训练的模型。通过 Optimum Intel 可直接集成 Hugging Face Hub 上用 transformers 和 diffusers 构建的模型。无需原始框架即可转换和部署模型。
- **广泛的平台兼容性**：降低资源需求，高效部署在从边缘到云端的各类平台。OpenVINO™ 支持在 CPU（x86、ARM）、GPU（Intel 集成与独立 GPU）和 AI 加速器（Intel NPU）上推理。
- **社区与生态**：加入活跃社区，共同提升各领域的深度学习性能。

快速参考请查看 [OpenVINO 速查表](https://docs.openvino.ai/2026/_static/download/OpenVINO_Quick_Start_Guide.pdf) 和 [核心特性](https://docs.openvino.ai/2026/about-openvino/key-features.html)。


## 安装

[获取适合你的 OpenVINO 发行版](https://docs.openvino.ai/2026/get-started/install-openvino.html)，或使用以下命令快速安装：

```sh
pip install -U openvino
```

### 验证安装

安装后，运行以下命令验证 OpenVINO 是否安装正确：

```python
import openvino as ov
print(ov.__version__)
```

详细信息系统要求请查看[系统要求](https://docs.openvino.ai/2026/about-openvino/release-notes-openvino/system-requirements.html)和[支持的设备](https://docs.openvino.ai/2026/documentation/compatibility-and-support/supported-devices.html)。

## 教程与示例

[OpenVINO 快速入门示例](https://docs.openvino.ai/2026/get-started.html)将带你了解部署第一个模型的基础知识。

通过 [OpenVINO Notebooks](https://github.com/openvinotoolkit/openvino_notebooks) 📚 学习如何优化和部署热门模型：
- [使用 OpenVINO 创建 LLM 驱动的聊天机器人](https://github.com/openvinotoolkit/openvino_notebooks/blob/latest/notebooks/llm-chatbot/llm-chatbot-generate-api.ipynb)
- [YOLOv11 优化](https://github.com/openvinotoolkit/openvino_notebooks/blob/latest/notebooks/yolov11-optimization/yolov11-object-detection.ipynb)
- [文本到图像生成](https://github.com/openvinotoolkit/openvino_notebooks/blob/latest/notebooks/text-to-image-genai/text-to-image-genai.ipynb)
- [基于 LLaVa 和 OpenVINO 的多模态助手](https://github.com/openvinotoolkit/openvino_notebooks/blob/latest/notebooks/llava-multimodal-chatbot/llava-multimodal-chatbot-genai.ipynb)
- [使用 Whisper 和 OpenVINO 进行自动语音识别](https://github.com/openvinotoolkit/openvino_notebooks/blob/latest/notebooks/whisper-asr-genai/whisper-asr-genai.ipynb)

在 [OpenVINO 示例（Python 与 C++）](https://docs.openvino.ai/2026/get-started/learn-openvino/openvino-samples.html)和 [Notebooks（Python）](https://docs.openvino.ai/2026/get-started/learn-openvino/interactive-tutorials-python.html)中发现更多示例。

以下是易于上手的代码示例，演示如何使用 OpenVINO 运行 PyTorch 和 TensorFlow 模型推理：

**PyTorch 模型**

```python
import openvino as ov
import torch
import torchvision

# load PyTorch model into memory
model = torch.hub.load("pytorch/vision", "shufflenet_v2_x1_0", weights="DEFAULT")

# convert the model into OpenVINO model
example = torch.randn(1, 3, 224, 224)
ov_model = ov.convert_model(model, example_input=(example,))

# compile the model for CPU device
core = ov.Core()
compiled_model = core.compile_model(ov_model, 'CPU')

# infer the model on random data
output = compiled_model({0: example.numpy()})
```

**TensorFlow 模型**

```python
import numpy as np
import openvino as ov
import tensorflow as tf

# load TensorFlow model into memory
model = tf.keras.applications.MobileNetV2(weights='imagenet')

# convert the model into OpenVINO model
ov_model = ov.convert_model(model)

# compile the model for CPU device
core = ov.Core()
compiled_model = core.compile_model(ov_model, 'CPU')

# infer the model on random data
data = np.random.rand(1, 224, 224, 3)
output = compiled_model({0: data})
```

OpenVINO 支持 CPU、GPU 和 NPU [设备](https://docs.openvino.ai/2026/openvino-workflow/running-inference/inference-devices-and-modes.html)，并支持来自 PyTorch、TensorFlow、ONNX、TensorFlow Lite、PaddlePaddle 和 JAX/Flax [框架](https://docs.openvino.ai/2026/openvino-workflow/model-preparation.html)的模型。它提供 C++、Python、C、NodeJS 的 [API](https://docs.openvino.ai/2026/api/api_reference.html)，并提供 GenAI API 用于优化模型流水线和性能。

## 使用 OpenVINO 的生成式 AI

从 OpenVINO GenAI [安装](https://docs.openvino.ai/2026/get-started/install-openvino/install-openvino-genai.html)开始，参阅[详细指南](https://docs.openvino.ai/2026/openvino-workflow-generative/inference-with-genai.html)探索使用 OpenVINO 的生成式 AI 能力。

在 [OpenVINO™ GenAI 仓库](https://github.com/openvinotoolkit/openvino.genai)中通过[示例](https://github.com/openvinotoolkit/openvino.genai/tree/master/samples)学习如何运行 LLM 和 GenAI。通过 Jupyter notebooks 了解 GenAI 的实际应用：[LLM 驱动的聊天机器人](https://github.com/openvinotoolkit/openvino_notebooks/tree/latest/notebooks/llm-chatbot)和 [LLM 指令遵循流水线](https://github.com/openvinotoolkit/openvino_notebooks/tree/latest/notebooks/llm-question-answering)。

## 文档

[用户文档](https://docs.openvino.ai/)包含 OpenVINO 的详细信息，引导你从安装到为 AI 应用优化和部署模型。

[开发者文档](./docs/dev/index.md)聚焦 OpenVINO 架构，描述[构建](./docs/dev/build.md)和[贡献](./CONTRIBUTING.md)流程。

## OpenVINO 生态

### OpenVINO 工具

-   [神经网络压缩框架（NNCF）](https://github.com/openvinotoolkit/nncf) - 高级模型优化技术，包括量化和稀疏化。
-   [GenAI 仓库](https://github.com/openvinotoolkit/openvino.genai)和 [OpenVINO Tokenizers](https://github.com/openvinotoolkit/openvino_tokenizers) - 用于开发和优化生成式 AI 应用的资源和工具。
-   [OpenVINO™ Model Server（OVMS）](https://github.com/openvinotoolkit/model_server) - 可扩展的高性能解决方案，用于为 Intel 架构优化的模型提供服务。
-   [Intel® Geti™](https://geti.intel.com/) - 面向计算机视觉用例的交互式视频和图像标注工具。

### 集成

-   [🤗Optimum Intel](https://github.com/huggingface/optimum-intel) - 在 Hugging Face API 中获取并使用基于 OpenVINO 的模型。
-   [Torch.compile](https://docs.openvino.ai/2026/openvino-workflow/torch-compile.html) - 通过将代码 JIT 编译为优化内核，在 Python 原生应用中使用 OpenVINO。
-   [ExecuTorch](https://github.com/pytorch/executorch/blob/main/backends/openvino/README.md) - 结合 OpenVINO 使用 ExecuTorch，高效优化和运行 AI 模型。
-   [使用 vLLM 进行 OpenVINO LLM 推理与服务](https://github.com/vllm-project/vllm-openvino) - 借助 OpenVINO 后端增强 vLLM 快速简便的模型服务能力。
-   [ONNX Runtime 的 OpenVINO Execution Provider](https://onnxruntime.ai/docs/execution-providers/OpenVINO-ExecutionProvider.html) - 在现有 ONNX Runtime 代码中将 OpenVINO 用作后端。
-   [LLMWare](https://github.com/llmware-ai/llmware/blob/main/solutions/openvino/README.md) - 使用 [100+ 个可直接推理的 OpenVINO 优化 SLM](https://huggingface.co/llmware/models?search=-ov)，无缝构建企业 AI 应用、智能体工作流、RAG 流水线、多模态用例等。
-   [LlamaIndex](https://docs.llamaindex.ai/en/stable/examples/llm/openvino/) - 使用 LlamaIndex 框架构建上下文增强的 GenAI 应用，并通过 OpenVINO 提升运行时性能。
-   [LangChain](https://python.langchain.com/docs/integrations/llms/openvino/) - 将 OpenVINO 与 LangChain 框架集成，提升 GenAI 应用的运行时性能。
-   [Keras 3](https://github.com/keras-team/keras) - Keras 3 是多后端深度学习框架。用户可通过 Keras API 将模型推理切换到 OpenVINO 后端。

查看 [Awesome OpenVINO](https://github.com/openvinotoolkit/awesome-openvino) 仓库，发现基于 OpenVINO 的社区 AI 项目合集！

## 性能

探索 [OpenVINO 性能基准测试](https://docs.openvino.ai/2026/about-openvino/performance-benchmarks.html)，发现最优硬件配置，并基于验证数据规划 AI 部署。

## 贡献与支持

详情请查看[贡献指南](./CONTRIBUTING.md)。
如果你想寻找贡献的切入点，请阅读[适合新手的 Issue 部分](./CONTRIBUTING.md#3-start-working-on-your-good-first-issue)。我们欢迎各种形式的贡献！

你可以在以下渠道提问和获取支持：

* [GitHub Issues](https://github.com/openvinotoolkit/openvino/issues)。
* [Intel DevHub Discord 服务器](https://discord.gg/7pVRxUwdWG)上的 OpenVINO 频道。
* Stack Overflow\* 上的 [`openvino`](https://stackoverflow.com/questions/tagged/openvino) 标签。


## 资源

* [发布说明](https://docs.openvino.ai/2026/about-openvino/release-notes-openvino.html)
* [OpenVINO 博客](https://blog.openvino.ai/)
* [Medium 上的 OpenVINO™ 工具包](https://medium.com/@openvino)


## 遥测

OpenVINO™ 收集软件性能和使用数据，用于改进 OpenVINO™ 工具。
这些数据由 OpenVINO™ 直接收集，或通过 Google Analytics 4 收集。
你可以随时运行以下命令退出：

``` bash
opt_in_out --opt_out
```

更多信息请参阅 [OpenVINO™ 遥测](https://docs.openvino.ai/2026/about-openvino/additional-resources/telemetry.html)。

## 许可证

OpenVINO™ 工具包基于 [Apache License Version 2.0](LICENSE) 许可。
向本项目贡献即表示你同意其中的许可和版权条款，并在这些条款下发布你的贡献。

---
\* 其他名称和品牌可能被主张为他人的财产。