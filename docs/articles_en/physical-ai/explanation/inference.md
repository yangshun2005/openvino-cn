# 推理

`InferenceModel` 是导出策略的运行时 API。它加载导出的包，执行预处理和后处理，并产出动作。

```python
model = InferenceModel("./exports/act_policy")
action = model.select_action(observation)
```

## 流水线

```text
observation
  -> preprocessors
  -> runner
  -> postprocessors
  -> action
```

## API

| 方法                                | 用途                     |
| ----------------------------------- | ------------------------ |
| `select_action(observation)`        | 立即返回一个动作。       |
| `predict_action_chunk(observation)` | 返回一个动作块，供运行时排队。 |
| `reset()`                           | 清除状态，开始新回合。   |
| `close()`                           | 释放后端资源。           |

## 分块策略

产出动作块的策略仍支持 `select_action()`。调用方无需按 runner 类型分支处理。

```python
if cursor.empty():
    cursor.push_chunk(model.predict_action_chunk(obs))

return cursor.pop()
```

cursor 是模型层内部的便利工具。它不是运行时动作队列，也不应被当作动作队列使用。

## 运行时边界

脚本、测试、演示和评估循环使用 `select_action()`。

策略驱动机器人时，通过 `PolicySource`（`RobotRuntime` 中面向已训练策略的动作源）使用 `predict_action_chunk()`。

```text
RobotRuntime
  -> PolicySource.update(robot_state, camera_frames, step)
       -> Execution.maybe_request(obs)
            -> InferenceModel.predict_action_chunk(obs)
            -> ActionQueue.push_chunk(chunk)
       -> ActionQueue.pop()
  -> robot.send_action(action)
```

## 性能评估

`InferenceLatencyBenchmark` 在运行时循环之外测量 `InferenceModel` 的单块延迟。它用于比较后端、设备和导出配置，不是机器人循环的性能分析器。

```python
from physicalai.benchmark.performance.inference_benchmark import InferenceLatencyBenchmark
from physicalai.inference import InferenceModel

model = InferenceModel("./exports/act_policy", device="CPU")
metrics = InferenceLatencyBenchmark(max_iters=500, warmup_iters=5).run(model)
```

运行包含预热阶段和测量循环，测量循环受迭代上限和实际耗时预算双重约束，以先达到者为准。

```text
warmup_iters  -> measured loop -> metrics
                 (stop on max_iters OR max_duration OR inputs exhausted)
```

未提供 `inputs` 可迭代对象时，基准测试按 `model.input_features` 规格生成随机输入，因此无需录制数据集即可运行。传入自定义可迭代对象即可针对真实观测进行基准测试。

报告的指标（`num_iters`、`min_iter_time`、`max_iter_time`、`mean_iter_time`、`median_iter_time`、`std_iter_time`、`avg_warmup_iter_time`）为每次迭代的秒数，反映完整的预处理 → runner → 后处理流水线。