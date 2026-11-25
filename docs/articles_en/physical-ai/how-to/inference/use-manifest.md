# 使用 Manifest

导出的策略包包含一个 `manifest.json` 文件。

```text
exports/act_policy/
├── manifest.json
├── model.xml
└── stats.safetensors
```

多数情况下，通过 `InferenceModel` 加载策略包。

```python
model = InferenceModel("./exports/act_policy")
```

如需直接查看元数据，加载 manifest 本身。

```python
from physicalai.inference.manifest import Manifest

manifest = Manifest.load("./exports/act_policy/manifest.json")
print(manifest.model.runner)
print(manifest.model.artifacts)
```

## 声明推理回调

回调使用与预处理器、后处理器相同的 `ComponentSpec` 语法和声明顺序。内置回调可用注册的短名称加扁平构造参数，也可用完全限定的类路径加 `init_args`：

```yaml
model:
  callbacks:
    - type: rldx1_vtc
      video_length: 4
      video_stride: 2
    - class_path: physicalai.inference.callbacks.LatencyMonitor
      init_args:
        window_size: 100
```

上面的 RLDX-1 回调在预处理前创建时间图像窗口，无需在 Python 侧接线回调。Manifest 回调接收 `on_load`、`on_predict_start`、`on_predict_end` 和 `on_reset` 生命周期事件。

构造 `InferenceModel` 时，显式提供的 `callbacks` 列表优先于 manifest，显式提供的空列表也是如此。无效、未知或不兼容的回调规格会在模型加载时失败。

用 manifest 描述导出的构建产物。用工作流配置在执行前编写训练、推理或运行时工作流。