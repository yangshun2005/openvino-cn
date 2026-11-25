# 推理 API 参考

## `InferenceModel`

```python
InferenceModel(
    export_dir: str | Path,
    policy_name: str | None = None,
    backend: str = "auto",
    device: str = "auto",
    runner: InferenceRunner | None = None,
    preprocessors: list[Preprocessor] | None = None,
    postprocessors: list[Postprocessor] | None = None,
    callbacks: list[Callback] | None = None,
    **adapter_kwargs,
)
```

模型可直接从导出目录构建，也可从配置加载。

## 构造函数

```python
model = InferenceModel("./exports/act_policy")
```

### `from_pretrained`

```python
model = InferenceModel.from_pretrained(
    "OpenVINO/act-fp16-ov",
    revision="main",      # 分支、标签或提交 SHA（可选）
    cache_dir=None,        # 下载缓存目录（可选）
)
```

该方法从 Hugging Face Hub 下载策略包快照，并像本地导出一样加载。
额外的关键字参数会转发给构造函数（如 `backend`、`device`）。

> **注意：** `InferenceModel.from_config()` 是计划中的 API。

## 方法

### `select_action`

```python
action = model.select_action(observation)
```

该方法返回一个动作。

### `predict_action_chunk`

```python
chunk = model.predict_action_chunk(observation)
```

该方法返回一组动作，供运行时排队。

### `reset`

```python
model.reset()
```

该方法清除 runner 状态和动作游标。

### `close`

```python
model.close()
```

该方法释放后端资源。

## 观测

观测是 NumPy 数组的字典。

```python
observation = {
    "state": joint_positions,
    "image.wrist": wrist_image,
}
```

期望的键和形状来自导出的包及其预处理流水线。