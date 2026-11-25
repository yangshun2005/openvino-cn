# 添加运行时回调

当需要在运行时循环周围实现产品特定行为时，使用回调。

以下示例记录观测和动作。

```python
class RecordingCallback:
    def on_action_ready(self, *, action, step):
        recorder.write_policy_action(step, action)
        return action

    def on_action_sent(self, *, action, step):
        recorder.write_sent_action(step, action)
```

在构造运行时时附加回调。

```python
runtime = RobotRuntime(
    robot=robot,
    action_source=PolicySource(model=model, execution=execution),
    fps=30,
    callbacks=[RecordingCallback()],
)
```

在运行时断开连接之前，相同的回调实例会接收每次执行的 `run()`。使用 `start` 和 `shutdown` 生命周期事件来区分每次运行的状态。`RobotRuntime.disconnect()` 或上下文退出会恰好一次关闭回调资源，并永久释放该运行时。

一般规则：将工作流特定逻辑放在回调中，除非同一行为成为可复用的运行时原语。