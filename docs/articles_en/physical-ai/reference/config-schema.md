# 配置 Schema 参考

## Config

`physicalai.config.Config` 使用与 jsonargparse 兼容的
`class_path` + `init_args` 结构。

```yaml
class_path: package.module.ClassName
init_args:
  key: value
```

| 字段         | 必填 | 类型   | 描述                           |
| ------------ | ---- | ------ | ------------------------------ |
| `class_path` | 是   | string | 非空导入路径，解析为一个类     |
| `init_args`  | 否   | object | 字符串键的构造函数参数         |

省略 `init_args` 表示空映射。不接受其他顶层键。

值可以是 `null`、布尔值、整数、有限浮点数、字符串、列表或字符串键映射。嵌套组件使用另一个 `Config`。
任何包含 `class_path` 的映射都保留为嵌套组件，且只能包含 `class_path` 和可选的 `init_args`。通过组件、列表和映射的嵌套最多 10 层。

导出时，`Path` 转换为不变的字符串形式，`Enum` 转换为其 JSON 安全值，元组转换为列表。`instantiate()` 接受 JSON 表示，而非原始 Python 对象。

## 运行时文档

CLI 运行时文档将构造函数参数放在 `runtime:` 下，方法参数可放在 `run:` 下。

```yaml
runtime:
  robot:
    class_path: physicalai.robot.SO101
    init_args:
      port: /dev/ttyACM0
      calibration: ./calibration.json
  action_source:
    class_path: physicalai.runtime.TeleopSource
    init_args:
      leader:
        class_path: physicalai.robot.SharedRobot
        init_args:
          name: leader-arm
  fps: 30
run:
  duration_s: 60
```

`physicalai run --config` 和 `RobotRuntime.from_config()` 也接受裸导出的 `RobotRuntime` Config，其顶层 `class_path` 为 `physicalai.runtime.RobotRuntime`。

## Manifest ComponentSpec

推理 manifest 保留其独立的 `ComponentSpec` 兼容模型。它支持注册表 `type` 别名、构建产物解析以及现有的额外字段行为。它不是严格的捕获 `Config` Schema；manifest 统一是单独的设计决策。

## 安全

`class_path` 是可执行的本地配置。`instantiate()` 仅用于可信应用和用户编写的配置，绝不用于从对等方接收的元数据或控制消息。完整的部署时信任假设见[安全模型](../getting-started/security.md)。