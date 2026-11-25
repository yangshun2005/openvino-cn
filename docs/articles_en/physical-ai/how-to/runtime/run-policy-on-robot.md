# 在机器人上运行策略

## Python API

```python
from physicalai.runtime import RobotRuntime, PolicySource, SyncExecution
from physicalai.inference import InferenceModel
from physicalai.robot import SO101
from physicalai.capture import UVCCamera

runtime = RobotRuntime(
    fps=30,
    robot=SO101(port="/dev/ttyACM0"),
    action_source=PolicySource(
        model=InferenceModel("./exports/act_policy"),
    ),
    cameras={
        "wrist": UVCCamera(device="/dev/video0", width=640, height=480),
    },
)

with runtime:
    runtime.run(duration_s=60)
```

## 不设固定时长地停止

省略 `duration_s`，运行会持续到有东西将其停止。传入一个 `threading.Event` 作为 `stop_event`，在工作线程上运行循环，并在需要结束时设置该事件。

```python
import threading

runtime = RobotRuntime(fps=30, robot=robot, action_source=source)
stop = threading.Event()

with runtime:
    worker = threading.Thread(target=runtime.run, kwargs={"stop_event": stop})
    worker.start()
    ...
    stop.set()          # 完成当前 tick 后返回
    worker.join()

print(runtime.last_run_reason)      # stop_requested
```

该事件只需提供 `is_set()`，因此当会话在子进程中运行时，`multiprocessing.Event` 同样适用。无论哪种方式，运行都通过正常的关闭路径结束：触发 `shutdown` 生命周期事件、刷新回调、断开动作源。回调资源会保持打开以供下一次 `run()` 使用，并在运行时上下文退出或调用 `disconnect()` 时关闭。

## 通过配置

编写运行时配置文件。

```yaml
# runtime.yaml
runtime:
  robot:
    class_path: physicalai.robot.SO101
    init_args:
      port: /dev/ttyACM0
      calibration: ./calibration.json
  action_source:
    class_path: physicalai.runtime.PolicySource
    init_args:
      model:
        class_path: physicalai.inference.InferenceModel
        init_args:
          export_dir: ./exports/act_policy
      execution:
        class_path: physicalai.runtime.SyncExecution
  cameras:
    wrist:
      class_path: physicalai.capture.UVCCamera
      init_args:
        device: /dev/video0
        width: 640
        height: 480
  fps: 30
```

从 CLI 运行。

```bash
physicalai run --config runtime.yaml --run.duration_s=60
```

## 组件职责

| 对象             | 负责                                                  |
| ---------------- | ----------------------------------------------------- |
| `InferenceModel` | 策略推理                                              |
| `PolicySource`   | 动作源，连接模型 + 执行 + 动作队列                    |
| `RobotRuntime`   | 机器人循环与计时                                      |
| `Execution`      | 推理运行的位置                                        |
| `Robot`          | 硬件 IO                                               |
| `Camera`         | 图像采集                                              |