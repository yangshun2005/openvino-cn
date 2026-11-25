---
explore_github_url: https://github.com/openvinotoolkit/physicalai
---

# OpenVINO™ Physical AI

```{toctree}
:hidden:
:maxdepth: 2

physical-ai/getting-started/README
physical-ai/how-to/README
physical-ai/explanation/README
physical-ai/reference/README
```

OpenVINO™ Physical AI 提供运行时组件，用于处理导出的机器人策略。文档围绕主要部署任务及其背后的运行时概念组织。

## 快速链接

| 我想要               | 前往                                                   |
| ----------------------- | ------------------------------------------------------- |
| 安装包     | [安装](physical-ai/getting-started/installation.md)         |
| 运行首次推理     | [快速开始](physical-ai/getting-started/quickstart.md)             |
| 在机器人上运行策略 | [运行策略](physical-ai/getting-started/run-a-policy.md)         |
| 编写运行时 YAML      | [运行时配置](physical-ai/how-to/config/write-runtime-config.md) |
| 使用运行时 CLI     | [CLI 运行](physical-ai/how-to/cli/run.md)                            |
| 了解架构 | [架构](physical-ai/explanation/architecture.md)             |
| 查阅 Schema         | [配置 Schema](physical-ai/reference/config-schema.md)             |

## 文档结构

```text
docs/
├── getting-started/  # 教程
├── how-to/           # 任务指南
├── explanation/      # 概念与边界
└── reference/        # 精确命令、Schema、API
```

## 工作流

大多数部署工作流都遵循相同路径：从导出的包到运行中的机器人循环。

```text
导出的策略包
    -> InferenceModel
    -> RobotRuntime
    -> Robot
```

Python 示例：

```python
from physicalai.inference import InferenceModel
from physicalai.runtime import RobotRuntime, PolicySource, SyncExecution
from physicalai.robot import SO101
from physicalai.capture import UVCCamera

model = InferenceModel("./exports/act_policy")
robot = SO101(port="/dev/ttyACM0")
cameras = {"wrist": UVCCamera(device="/dev/video0", width=640, height=480)}

runtime = RobotRuntime(
    fps=30,
    robot=robot,
    action_source=PolicySource(model=model, execution=SyncExecution()),
    cameras=cameras,
)

with runtime:
    runtime.run(duration_s=60)
```

CLI 示例：

```bash
physicalai run --config runtime.yaml --run.duration_s=60
```