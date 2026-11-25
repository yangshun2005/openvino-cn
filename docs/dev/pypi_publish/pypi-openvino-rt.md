# OpenVINO™

<!--- The note below is intended for master branch only for pre-release purpose. Remove it for official releases. --->
> **注意**：此版本为预发布软件，尚未经过完整的发布验证或资格认证。预发布软件不提供支持，API/行为可能变更。请勿将其集成到任何生产软件/解决方案中，仅用于在等待最终发布版本期间的早期测试和集成。

Intel® Distribution of OpenVINO™ 工具套件是一个用于优化和部署 AI 推理的开源工具套件。它可用于开发基于深度学习任务的应用程序和解决方案，例如：模拟人类视觉、自动语音识别、自然语言处理、推荐系统、图像生成等。它提供高性能且丰富的部署选项，覆盖从边缘到云的场景。

选定模型后，你可以通过 OpenVINO™ 将其集成到应用中，并部署到各种设备上。OpenVINO™ Python 包包含一组库，便于将推理集成到你的产品中。

## 系统要求

开始安装前，请检查支持的操作系统和所需的 Python* 版本。支持的硬件完整列表见[系统要求页面](https://docs.openvino.ai/system_requirements)。

在 Windows* 上安装还需要 **C++ 库**。可通过[下载 Visual Studio Redistributable 文件（.exe）](https://aka.ms/vs/17/release/vc_redist.x64.exe)进行安装。

> **注意**：此包可能在其他 Linux 和 Windows 版本上运行，但仅系统要求中指定的版本经过完整验证。

## 安装 OpenVINO™

### 第 1 步：设置 Python 虚拟环境

使用虚拟环境以避免依赖冲突。创建虚拟环境请使用以下命令：

Windows：
```sh
python -m venv openvino_env
```

Linux 和 macOS：
```sh
python3 -m venv openvino_env
```

> **注意**：在 Linux 和 macOS 上，你可能需要[安装 pip](https://pip.pypa.io/en/stable/installation/)。

### 第 2 步：激活虚拟环境

Windows：
```sh
openvino_env\Scripts\activate
```

Linux 和 macOS：
```sh
source openvino_env/bin/activate
```

### 第 3 步：设置 PIP 并更新到最高版本

运行命令：
```sh
python -m pip install --upgrade pip
```

### 第 4 步：安装包

运行命令：
```sh
pip install openvino
```

### 第 5 步：验证包已安装

运行命令：
```sh
python -c "from openvino import Core; print(Core().available_devices)"
```

如果安装成功，你将看到可用设备列表。

## 包内容

<table>
  <tr>
    <th>组件</th>
    <th>内容</th>
    <th>描述</th>
  </tr>
  <tr>
    <td><a href="https://docs.openvino.ai/2026/openvino-workflow/running-inference.html">OpenVINO Runtime</a></td>
    <td>`openvino package`</td>
    <td>OpenVINO Runtime 是一组 C++ 库，提供 C 和 Python 绑定，提供通用 API 以在你选择的平台上交付推理解决方案。使用 OpenVINO Runtime API 可读取 PyTorch、TensorFlow、TensorFlow Lite、ONNX 和 PaddlePaddle 模型，并在首选设备上执行。OpenVINO Runtime 采用插件架构，包含以下插件：
        <a href="https://docs.openvino.ai/2026/openvino-workflow/running-inference/inference-devices-and-modes/cpu-device.html">CPU</a>、
        <a href="https://docs.openvino.ai/2026/openvino-workflow/running-inference/inference-devices-and-modes/gpu-device.html">GPU</a>、
        <a href="https://docs.openvino.ai/2026/openvino-workflow/running-inference/inference-devices-and-modes/automatic-batching.html">Auto Batch</a>、
        <a href="https://docs.openvino.ai/2026/openvino-workflow/running-inference/inference-devices-and-modes/auto-device-selection.html">Auto</a>、
        <a href="https://docs.openvino.ai/2026/openvino-workflow/running-inference/inference-devices-and-modes/hetero-execution.html">Hetero</a>，
    </td>
  </tr>
  <tr>
    <td><a href="https://docs.openvino.ai/2026/openvino-workflow/model-preparation.html#convert-a-model-in-cli-ovc">OpenVINO Model Converter (OVC)</a></td>
    <td>`ovc`</td>
    <td>OpenVINO Model Converter 将流行框架中训练的模型转换为 OpenVINO 组件可用的格式。</br>支持的框架包括 ONNX、TensorFlow、TensorFlow Lite 和 PaddlePaddle。
    </td>
  </tr>
  <tr>
    <td><a href="https://docs.openvino.ai/2026/get-started/learn-openvino/openvino-samples/benchmark-tool.html">Benchmark Tool</a></td>
    <td>`benchmark_app`</td>
    <td>Benchmark Application** 可用于在支持的设备上评估同步和异步模式下的深度学习推理性能。
    </td>
</table>



## 故障排除

一般故障排除请参阅 [OpenVINO 安装故障排除指南](https://docs.openvino.ai/2026/get-started/install-openvino/configurations/troubleshooting-install-config.html)。以下各节还对若干错误消息提供说明。

### 中国用户通过 PIP 安装时的错误

中国用户在 OpenVINO™ 安装期间通过 PIP 下载源时可能遇到错误。要解决此问题，请尝试以下方案：

* 在 Python ``pip`` 命令中使用 ``-i`` 参数添加下载源。例如：

   ``` sh
   pip install openvino -i https://mirrors.aliyun.com/pypi/simple/
   ```
   如果上述 URL 为 ``http`` 而非 ``https``，请使用 ``--trusted-host`` 参数。

### ERROR:root:Could not find OpenVINO Python API.

在 Windows 上，运行 OpenVINO 可能需要额外的库。要解决此问题，请安装 [C++ redistributable（.exe）](https://aka.ms/vs/17/release/vc_redist.x64.exe)。你也可以在[官方支持页面](https://docs.microsoft.com/en-us/cpp/windows/latest-supported-vc-redist)查看完整下载列表。

### ImportError: libpython3.10.so.1.0: cannot open shared object file: No such file or directory

要解决 Ubuntu* 上缺少外部依赖的问题，请执行以下命令：
```sh
sudo apt-get install libpython3.10
```

## 其他资源

- [Intel® Distribution of OpenVINO™ 工具套件](https://software.intel.com/en-us/openvino-toolkit)
- [OpenVINO™ 文档](https://docs.openvino.ai/)
- [OpenVINO™ Notebooks](https://github.com/openvinotoolkit/openvino_notebooks)
- [OpenVINO 安装选择工具](https://www.intel.com/content/www/us/en/developer/tools/openvino-toolkit/download.html)

Copyright © 2018-2026 Intel Corporation
> **法律声明**：你对本软件及任何所需依赖软件（“软件包”）的使用受软件包 [Apache 2.0 许可证](https://www.apache.org/licenses/LICENSE-2.0.html)条款和条件的约束，其中还可能包含软件包中或随附的第三方或开源软件的通知、免责声明或许可条款，你的使用即表示你接受所有此类条款。更多详情请参阅软件包中随附的“third-party-programs.txt”或其他类似名称的文本文件。

>Intel 致力于尊重人权并避免与人权侵犯行为同谋，这一政策体现在[英特尔全球人权原则](https://www.intel.com/content/www/us/en/policy/policy-human-rights.html)中。因此，访问本平台上的英特尔材料即表示你同意不会将该材料用于导致或促成违反国际公认人权的产品或应用中。

Intel、Intel 徽标及其他 Intel 标志是 Intel Corporation 或其子公司的商标。其他名称和品牌可能被主张为他人财产。