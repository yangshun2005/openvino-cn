# 编写运行时配置

运行时配置描述机器人控制工作流在开始执行前的状态。

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
        device: /dev/v4l/by-id/usb-Example_Wrist_Camera-video-index0
        width: 640
        height: 480
  fps: 30
```

通过 CLI 运行同一配置：

```bash
physicalai run --config runtime.yaml --run.duration_s=60
```

嵌套组件使用相同的 `class_path` 和 `init_args` 结构。

```yaml
class_path: module.ClassName
init_args:
  key: value
```

配置文件始终是被动数据。只有调用 `RobotRuntime.run()` 时，工作流才会启动。