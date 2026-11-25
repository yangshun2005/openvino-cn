# Runtime API 参考

## `RobotRuntime`

`RobotRuntime` 是在硬件上运行策略（或遥操作，或任意自定义控制逻辑）的主编排器。它始终接收一个显式、可插拔的 `action_source`。

```python
RobotRuntime(
    robot: Robot,
    action_source: ActionSource,
    fps: float,
    cameras: Mapping[str, Camera] | None = None,
    callbacks: Sequence[RuntimeCallback] = (),
)
```

最重要的方法如下所示。

```python
runtime.connect() -> None
runtime.disconnect() -> None
runtime.run(*, duration_s: float | None = None, stop_event: StopSignal | None = None) -> int
runtime.stop() -> None
runtime.last_run_reason -> RunReason | None
```

`RobotRuntime` 也支持上下文管理器用法，连接会自动清理。"步"是控制循环在 `fps` 下的一次迭代：读取一次观测，从 `action_source` 获取一个动作，并发送给机器人。`run()` 返回本次运行完成的步数——没有聚合统计对象。其他统计直接从调用方已持有的对象上读取，例如 `runtime.action_source.action_queue.total_pops` 或 `execution.inference_count`。每次 `run()` 都会将这些计数器从零开始，因此它们描述的是最近一次运行，而非累计值。

回调属于运行时。它们在连续的 `run()` 调用之间保持打开，并为每次调用分别接收 `start` 和 `shutdown` 生命周期事件。`disconnect()` 表示调用方已用完该运行时：它恰好关闭一次回调和硬件，且运行时无法重新连接。如果 `connect()` 失败，它会回滚已部分连接的硬件；直接再次调用 `connect()` 即可重试。

```python
with RobotRuntime(...) as runtime:
    steps = runtime.run(duration_s=60)
```

## 停止运行

`stop()` 是线程安全的，可以在运行前、运行中或运行之间调用。`run(stop_event=...)` 接受任何具有 `is_set()` 的对象，因此 `threading.Event` 和 `multiprocessing.Event` 都可用。两者都在每个 tick 开头轮询——先触发的那个结束运行，而正在执行的 tick 仍会完成并发送其动作。

```python
from physicalai.runtime import StopSignal   # Protocol: is_set() -> bool
```

在没有活动运行时调用 `stop()` 会被记住而非忽略：下一次 `run()` 会看到它，立即返回，并报告零步。运行结束后该请求会被清除，因此一个运行时可以服务连续会话。

再次运行会启动新会话，而非继续旧会话。上一次运行中仍在排队的动作会被丢弃，因为它们基于现已过时的观测计算。如果下一次运行开始时后台 worker 仍在模型内，`run()` 会等待片刻，若仍忙碌则抛出 `RuntimeError`，而不是让两次运行共享一个模型。

`run()` 返回步数。`last_run_reason` 报告结束原因，同一字符串会作为 `reason` 在 `shutdown` 生命周期事件中发出。

| `last_run_reason`  | 原因                                                         |
| ------------------ | ------------------------------------------------------------- |
| `stop_requested`   | 调用了 `runtime.stop()`，或 `stop_event.is_set()` 为 true |
| `duration_elapsed` | 达到 `duration_s`                                      |
| `interrupted`      | `KeyboardInterrupt`（被吞掉；`run()` 正常返回）     |
| `error`            | 任何其他异常，随后传播给调用方      |

`stop_event` 不在配置 Schema 和 CLI 中，因为实时同步对象没有可序列化形式；`physicalai run` 通过 `--run.duration_s` 或 Ctrl+C 停止。

## `ActionSource`

`ActionSource` 是开发者实现的协议，用于将自定义决策逻辑接入 `RobotRuntime`。三个必需方法，没有可选方法——没有能力协议，运行时中任何地方都没有 `isinstance` 检查。

```python
class ActionSource(Protocol):
    def connect(self, *, bus: _CallbackBus, session_id: str) -> None: ...
    def update(self, robot_state: RobotObservation, camera_frames: Mapping[str, Frame], step: int) -> np.ndarray: ...
    def disconnect(self) -> None: ...
```

`bus` 是内部、运行时作用域的回调总线，由 `RobotRuntime` 在每次 `run()` 时注入；动作源通常只将其转发给 `Execution` 策略（见下方 `PolicySource`），而非直接使用。

当前随包提供的动作源实现如下。

| 类          | 用途                                                                |
| -------------- | ---------------------------------------------------------------------- |
| `PolicySource` | 通过 `Execution` 策略 + `ActionQueue` 运行训练好的模型   |
| `TeleopSource` | 读取主臂并将其观测作为从臂动作转发 |

```python
PolicySource(
    model: InferenceModel,
    execution: Execution,
    action_queue: ActionQueue | None = None,
    *,
    task: str | None = None,
)

source.reset(*, reset_model: bool = True) -> None
source.warmup(observation: dict[str, Any]) -> None

TeleopSource(
    leader: Robot,
    *,
    to_action: Callable[[RobotObservation], np.ndarray] | None = None,
)
```

`reset()` 在不停止执行 worker 的情况下启动新的策略回合。它丢弃已排队、待处理和进行中的动作，清除上一个动作，并默认重置模型状态。如果推理已在运行，reset 会等待该模型调用完成后再重置模型状态，同时拒绝其过时结果。模型或执行重置错误会传播给调用方。仅当模型状态应跨回合边界延续时才传 `reset_model=False`。自定义执行策略必须实现 `Execution.reset()` 才能支持此操作。

`warmup()` 接受模型就绪输入，例如 `source.to_model_input(robot_state, camera_frames)`，并在下一个控制 tick 前填充队列。它是可选的：`update()` 会在 `connect()` 或 `reset()` 后惰性执行相同的预热。

## `Execution`

```python
class Execution:
    def start(self, model: InferenceModel, action_queue: ActionQueue) -> None: ...
    def maybe_request(self, observation: dict[str, Any]) -> None: ...
    def warmup(self, sample_observation: dict[str, Any]) -> None: ...
    def reset(self, *, reset_model: bool = True) -> None: ...
    def stop(self) -> None: ...
    @property
    def chunk_size(self) -> int: ...
```

当前随包提供的执行实现如下。

| 类            | 用途                               |
| ---------------- | ------------------------------------- |
| `SyncExecution`  | 在运行时线程中运行推理  |
| `AsyncExecution` | 在后台线程中运行推理 |

> **预览：** `RemoteExecution` 是计划中的 API，不属于当前包发布版本。

## `ActionQueue`

```python
queue.push_chunk(chunk)
action = queue.pop()
queue.clear()
```

动作队列负责运行时缓冲、合并、平滑以及处理迟到结果的策略。