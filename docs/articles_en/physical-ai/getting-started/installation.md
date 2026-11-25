# 安装

先安装运行时包。

```bash
pip install physicalai
```

仅在需要特定后端或机器人集成时，才安装硬件扩展。

```bash
pip install "physicalai[realsense]"
pip install "physicalai[so101]"
pip install "physicalai[robots]"
pip install "physicalai[capture]"  # 所有相机后端 + IPC 传输
```

本地开发时，安装仓库环境并运行测试套件。

```bash
uv sync
uv run pytest
```

## 包边界

`physicalai` 是运行时包。在未安装 Torch 或 Lightning 等训练依赖的部署主机上，它应保持可用。

训练命令可由单独的训练发行版或插件入口点提供。

与配套的 `physicalai-train` 发布版本一起安装时，共享的 `physicalai` 可执行文件还会暴露训练和基准测试子命令。

## Intel GPU 运行时（可选）

要在 Intel GPU 上运行 OpenVINO 推理（在推理配置中设置 `device: GPU` 或 `AUTO`），主机需要 OpenCL 加载器和 Intel 计算驱动。这是 Linux 上标准的 Intel GPU 主机配置，这些系统包无法通过 `pip` 安装：

```bash
# Debian / Ubuntu
sudo apt install ocl-icd-libopencl1 intel-opencl-icd

# 允许你的用户访问 GPU 设备（执行后需注销并重新登录）
sudo usermod -aG render $USER

# 验证 OpenCL 能识别到 GPU 设备
clinfo -l
```

缺少这些包时，加载 OpenVINO GPU 插件会失败，报错为 `libOpenCL.so.1: cannot open shared object file`（缺少加载器）或 `[GPU] Can't get PERFORMANCE_HINT property as no supported devices found`（缺少驱动或设备不可访问）。CPU 推理不受影响。

## 检查安装

```python
import physicalai
from physicalai.inference import InferenceModel
```

如果未安装硬件扩展，导入基础运行时仍应正常工作。