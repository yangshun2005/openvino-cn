# 入门

```{toctree}
:hidden:
:maxdepth: 1

installation
quickstart
run-a-policy
security
```

本节包含帮助首次成功运行 Physical AI 工作流的简短教程。

## 推荐顺序

1. [安装](installation.md)
2. [快速开始](quickstart.md)
3. [运行策略](run-a-policy.md)
4. [安全模型](security.md)

## 最简路径

```bash
pip install physicalai
```

```python
from physicalai.inference import InferenceModel

model = InferenceModel("./exports/act_policy")
model.reset()
action = model.select_action(observation)
```

需要直接控制对象时使用 Python。需要可共享、可重复的可复现运行时，使用 YAML 和 CLI。