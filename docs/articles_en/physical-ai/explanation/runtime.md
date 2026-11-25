# 运行时

`RobotRuntime` 在机器人硬件上运行控制循环。它负责硬件 I/O、回调生命周期和时序；而一个必需且可插拔的 `action_source` 负责决定每个 tick 发送什么动作。`PolicySource` 封装训练好的策略（模型 + 执行策略 + 动作队列）；`TeleopSource` 读取主臂；自定义逻辑可直接实现 `ActionSource` 协议。

```python
runtime = RobotRuntime(
    fps=30,
    robot=robot,
    action_source=PolicySource(model=model, execution=SyncExecution()),
)

with runtime:
    runtime.run(duration_s=60)
```

## 职责划分

| 组件             | 负责                                                             | 不负责                      |
| ---------------- | ---------------------------------------------------------------- | --------------------------- |
| `InferenceModel` | 模型加载、预处理、推理、后处理                                   | 机器人循环时序              |
| `Execution`      | 推理运行的位置和时机                                             | 机器人 I/O                  |
| `ActionQueue`    | 动作块和缓冲                                                     | 模型推理                    |
| `PolicySource`   | 将模型 + 执行 + 动作队列组合为每 tick 一个动作                   | 机器人 I/O、循环时序        |
| `RobotRuntime`   | 观测、调用动作源、发送动作、回调、时序                           | 策略计算、决策逻辑          |
| `Robot`          | 硬件连接、观测、动作                                             | 策略推理                    |

## 循环

运行时循环遵循以下模式：

```text
while running:
    if stop_requested() or duration_reached():
        break
    robot_state, camera_frames = read_observation()
    action = action_source.update(robot_state, camera_frames, step)
    action = on_action_ready(action)  # callback hook, may transform
    send_action_to_robot(action)
    on_action_sent(action)            # callback hook, notification only
    sleep_until_next_tick()
```

观测结构和合并策略可能随 API 稳定而调整。`action_source.update()` 左侧的一切——决定是否运行推理、从队列取动作、保持上一个动作——都是动作源的内部实现；`RobotRuntime` 本身每 tick 只看到一个动作。

## 停止

停止在 tick 之间生效，绝不在 tick 内部生效。已开始的 tick 会完成并发送其动作，因此机器人不会停留在命令执行到一半的状态。这意味着停止并非即时：它在当前 tick 完成后才落地，而恰好正在等待推理或慢速机器人读取的 tick 需要多久就等多久。

当请求停止的代码与运行时在同一程序中时，使用 `runtime.stop()`。不在同一程序中时，使用 `run(stop_event=...)`：传入任何具有 `is_set()` 方法的对象，一个进程即可停止另一个进程中运行的会话。运行时会同时检查两者，先触发者结束运行。

停止不等于关闭。停止结束控制循环，但机器人、摄像头和回调保持活跃，因此同一个运行时可以停止后再次运行。每次运行都会发出自己的 `start` 和 `shutdown` 生命周期事件。`last_run_reason` 和 `shutdown` 事件的 `reason` 字段说明运行结束的原因。

`disconnect()` 是最终操作。它关闭回调并断开硬件连接，且仅执行一次；再次连接需创建新的运行时。`connect()` 失败时已回滚部分硬件设置，因此直接再次调用 `connect()` 重试，无需调用 `disconnect()`。这样回调资源（如文件和后台线程）可在多次运行间复用，同时拥有一个明确的释放点。

## 执行模式

> **预览：** `RemoteExecution` 是计划中的 API。

| 模式               | 推理运行位置   | 用途                     |
| ------------------ | -------------- | ------------------------ |
| `SyncExecution()`  | 运行时线程     | 简单部署和调试           |
| `AsyncExecution()` | 工作线程       | 避免阻塞控制循环         |
| `RemoteExecution`  | 远程服务器     | 计划中的 API             |

## 产品工作流

HIL、录制、高亮和 DAgger 应通过回调组合，直到它们足以证明需要可复用的运行时原语。

```python
class HILCallback:
    def on_action_ready(self, *, action, step):
        if teleop.enabled:
            return teleop.read_action()
        return action
```