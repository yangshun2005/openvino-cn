# 编写推理配置

当需要在导出的清单之外编写推理流水线时，使用推理配置。

```yaml
model:
  class_path: physicalai.inference.InferenceModel
  init_args:
    export_dir: ./exports/act_policy
    backend: openvino
    device: CPU
```

如果清单已包含所需的 runner、构建产物、处理器和硬件元数据，优先从清单加载。

```python
model = InferenceModel("./exports/act_policy")
action = model.select_action(observation)
```

工作流配置用于表达用户编写的意图，清单用于描述导出包的元数据。

要从 Python 导出模型提供的构造参数，调用 `Config.from_instance(model)`。用 `config.instantiate()` 重建这份可信的本地配方；构造过程会加载模型包，可能占用大量资源。