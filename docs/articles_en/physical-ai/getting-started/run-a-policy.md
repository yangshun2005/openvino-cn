# 运行策略

使用 `RobotRuntime` 搭配 `PolicySource` 动作源，在真实硬件上运行训练好的策略。运行时负责控制循环——读取相机、发送动作——动作源负责构建观测并执行推理。

```python
from physicalai.inference import InferenceModel
from physicalai.runtime import RobotRuntime, PolicySource, SyncExecution
from physicalai.robot import SO101
from physicalai.capture import UVCCamera

model = InferenceModel("./exports/act_policy")
robot = SO101(port="/dev/ttyACM0")
cameras = {
    "wrist": UVCCamera(device="/dev/video0", width=640, height=480),
}

runtime = RobotRuntime(
    fps=30,
    robot=robot,
    action_source=PolicySource(model=model, execution=SyncExecution()),
    cameras=cameras,
)

with runtime:
    runtime.run(duration_s=60)
```

等效的 CLI 命令使用相同的运行时配置。

```bash
physicalai run --config runtime.yaml --run.duration_s=60
```

最小运行时配置如下。

```yaml
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

从高层看，运行时循环按以下顺序执行。

```text
读取机器人观测
读取相机帧
向动作源请求下一个动作
将动作发送给机器人
休眠至下一个周期
```