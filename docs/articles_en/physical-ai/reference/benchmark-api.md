# 基准测试 API 参考

## `InferenceLatencyBenchmark`

```python
InferenceLatencyBenchmark(
    max_iters: int | None = 1000,
    warmup_iters: int = 1,
    max_duration: int | None = 60000,
)
```

测量 `InferenceModel` 的每块延迟。测量循环在达到任一上限时停止：`max_iters`、`max_duration`（毫秒）或输入耗尽。传入 `None` 可禁用某个上限。

### `run`

```python
metrics = benchmark.run(model, inputs=None)
```

`inputs` 是与 `model` 兼容的观测字典的可迭代对象。为 `None` 时，根据 `model.input_features` 规格生成随机输入；这要求导出的包声明了输入特征。

先运行 `warmup_iters` 次预热迭代，再运行测量循环，返回每次迭代秒数的字典：

| 键                     | 含义                                           |
| ---------------------- | ---------------------------------------------- |
| `avg_warmup_iter_time` | 预热期间每次迭代的平均时间。                   |
| `num_iters`            | 测量的迭代次数。                               |
| `min_iter_time`        | 最快的测量迭代。                               |
| `max_iter_time`        | 最慢的测量迭代。                               |
| `mean_iter_time`       | 测量迭代的平均值。                             |
| `median_iter_time`     | 测量迭代的中位数。                             |
| `std_iter_time`        | 总体标准差（n == 1 时为 0.0）。                |