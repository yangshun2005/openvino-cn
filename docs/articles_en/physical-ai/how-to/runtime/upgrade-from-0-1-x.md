# 从 0.1.x 升级

`PolicyRuntime` 和 `InferenceModel.load()` 已在 Runtime v0.2.0 中移除。请更新使用它们的代码，并按下方替换方案调整 YAML 配置。

| 0.1.x | 0.2.0 |
| --- | --- |
| `PolicyRuntime` | `RobotRuntime`，需显式指定 `PolicySource` |
| 策略模型和执行设置直接配置在 runtime 上 | 设置放在 `PolicySource` 上（YAML 中位于 `action_source.init_args` 下） |
| `InferenceModel.load(...)` | 本地导出用 `InferenceModel(export_dir=...)`，Hub 用 `InferenceModel.from_pretrained(...)` |
| 旧版 OpenVINO | OpenVINO >= 2026.4.0 和 OpenVINO Tokenizers >= 2026.4.0.0 |

旧的配置辅助函数 `FromConfig` 和 `instantiate_obj` 仍可用，但会发出弃用警告，并将在未来版本中移除。新代码请使用 `Config` 或 jsonargparse。

[runtime 配置示例](https://github.com/openvinotoolkit/physicalai/blob/v0.2.0/examples/runtime/runtime.yaml)是迁移 0.1.x 配置的良好起点。