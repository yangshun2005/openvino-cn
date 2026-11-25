# 加载导出的策略

先通过自动检测加载导出的包。

```python
from physicalai.inference import InferenceModel

model = InferenceModel("./exports/act_policy")
```

然后计算一个动作。

```python
model.reset()
action = model.select_action(observation)
```

通过仓库 ID 直接从 Hugging Face Hub 加载。

```python
model = InferenceModel.from_pretrained("OpenVINO/act-fp16-ov")
```

固定修订版本（分支、标签或提交 SHA），以保证加载可复现。

```python
model = InferenceModel.from_pretrained(
    "OpenVINO/act-fp16-ov",
    revision="main",
)
```

如有必要，显式指定后端。

```python
model = InferenceModel(
    "./exports/act_policy",
    backend="openvino",
    device="CPU",
)
```

当由运行时负责排队和时序时，使用分块预测。

```python
chunk = model.predict_action_chunk(observation)
```

不要围绕 `select_action()` 构建机器人循环时序。当策略驱动硬件时，使用 `RobotRuntime` 并搭配 `PolicySource` 动作源。