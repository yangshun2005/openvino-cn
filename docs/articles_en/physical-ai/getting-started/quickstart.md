# 快速开始

本教程演示最小可用的推理流程：加载导出的策略包并计算一个动作。

```python
from physicalai.inference import InferenceModel

model = InferenceModel("./exports/act_policy")
model.reset()

action = model.select_action(observation)
```

`observation` 输入是 NumPy 数组的字典。键名必须与导出策略期望的名称一致。

例如，一个 observation 可能如下所示。

```python
observation = {
    "state": joint_positions,
    "image.wrist": wrist_image,
    "image.front": front_image,
}
```

## 动作块策略

部分策略在内部生成动作块。即便如此，`select_action()` 每次调用仍只返回单个动作。

```python
for _ in range(100):
    action = model.select_action(observation)
    observation = env.step(action)
```

如果正在构建机器人控制循环，请通过 `RobotRuntime`（经由 `PolicySource` 动作源）使用动作块预测，而不是自行管理时序和缓冲。

```python
chunk = model.predict_action_chunk(observation)
```

## Manifest 路径

导出的包包含一个 manifest。

```text
exports/act_policy/
├── manifest.json
├── model.xml
└── stats.safetensors
```

manifest 记录了导出的构建产物、运行器配置、预处理与后处理流水线以及硬件元数据。