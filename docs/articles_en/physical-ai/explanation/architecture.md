# 架构

Physical AI 运行时由少量分层组成，导出包、推理、运行时编排和硬件 IO 之间边界清晰。

```text
exported package
    -> InferenceModel
    -> RobotRuntime
    -> Robot and cameras
```

## 组件

| 组件             | 职责                                                   |
| ---------------- | ------------------------------------------------------ |
| `Manifest`       | 描述导出的构建产物和推理流水线                         |
| `InferenceModel` | 加载构建产物并计算动作                                 |
| `ActionSource`   | 决定每个 tick 发送的动作（策略、遥操作或自定义）       |
| `RobotRuntime`   | 运行机器人控制循环                                     |
| `Execution`      | 决定推理在哪里运行                                     |
| `ActionQueue`    | 缓冲并合并动作块                                       |
| `Robot`          | 读取状态并发送命令                                     |
| `Camera`         | 读取图像帧                                             |

## 包边界

`physicalai` 是运行时包。导入或运行部署工作流时，它不应依赖训练相关的依赖。

训练包需要扩展运行时 CLI 时，可通过 CLI 入口点添加命令。

```text
physicalai
  run

training package
  fit
  validate
  test
  predict
  benchmark
  export
```

## 设计规则

- 配置对象保持为被动数据结构。
- 编排器负责执行工作流。
- `InferenceModel` 不负责机器人时序。
- `RobotRuntime` 不负责策略数学，也不决定发送什么动作——那是动作源的职责。
- Manifest 描述导出的包。
- 工作流配置描述启动前的期望执行方式。