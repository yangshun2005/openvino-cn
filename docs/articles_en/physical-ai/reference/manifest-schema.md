# Manifest Schema 参考

磁盘上的 manifest 文件为 `manifest.json`。此处用 YAML 展示以便阅读。

## 顶层

```yaml
format: policy_package
version: "1.0"
policy: {}
model: {}
hardware: {}
```

字段：

| 字段       | 类型   | 描述                              |
| ---------- | ------ | --------------------------------- |
| `format`   | string | Manifest 格式，通常为 `policy_package` |
| `version`  | string | Manifest Schema 版本              |
| `policy`   | object | 策略标识与来源元数据              |
| `model`    | object | 构建产物与推理流水线              |
| `hardware` | object | 预期机器人与相机元数据            |

## Policy

```yaml
policy:
  name: pi05
  source:
    repo_id: physical-ai/example
    class_path: physicalai.policies.pi05.Pi05
```

## Model

```yaml
model:
  n_obs_steps: 1
  artifacts:
    openvino: model.xml
  runner:
    type: action_chunking
    chunk_size: 50
  preprocessors:
    - type: normalize
      artifact: stats.safetensors
  postprocessors:
    - type: denormalize
      artifact: stats.safetensors
  callbacks:
    - type: rldx1_vtc
      video_length: 4
      video_stride: 2
```

字段：

| 字段             | 类型                  | 描述                     |
| ---------------- | --------------------- | ------------------------ |
| `n_obs_steps`    | integer               | 观测历史长度             |
| `artifacts`      | mapping               | 逻辑产物名到文件路径的映射 |
| `runner`         | `ComponentSpec`       | 推理运行器               |
| `preprocessors`  | list[`ComponentSpec`] | 预处理流水线             |
| `postprocessors` | list[`ComponentSpec`] | 后处理流水线             |
| `callbacks`      | list[`ComponentSpec`] | 推理生命周期回调         |

## Hardware

```yaml
hardware:
  robots:
    - name: main
      type: SO101
      state:
        shape: [6]
        dtype: float32
        order: [shoulder, elbow, wrist, gripper, lift, rotate]
      action:
        shape: [6]
        dtype: float32
  cameras:
    - name: wrist
      shape: [3, 224, 224]
      dtype: uint8
```

机器人和相机规格仅为元数据。运行时配置仍负责选择具体的硬件类。