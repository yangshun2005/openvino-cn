# 配置预处理和后处理

预处理器在模型执行前运行，后处理器在模型执行后运行。二者共同定义运行器周边的输入和输出适配。

清单可同时声明这两个阶段。

```yaml
model:
  runner:
    type: action_chunking
    chunk_size: 50
  artifacts:
    openvino: model.xml
  preprocessors:
    - type: normalize
      artifact: stats.safetensors
  postprocessors:
    - type: denormalize
      artifact: stats.safetensors
```

同样的组件也可以用显式类路径声明。

```yaml
preprocessors:
  - class_path: physicalai.inference.preprocessors.StatsNormalizer
    init_args:
      artifact: stats.safetensors
```

流水线结构：

```text
observation
  -> preprocessors
  -> runner
  -> postprocessors
  -> action output
```

已注册的内置组件用 `type`。需要显式导入路径时用 `class_path`。