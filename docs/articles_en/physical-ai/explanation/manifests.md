# 清单

清单描述一个导出的策略包。它告诉运行时存在哪些构建产物，以及应如何重建推理流水线。

```text
export/
├── manifest.json
├── model.xml
└── stats.safetensors
```

## 示例

```yaml
format: policy_package
version: "1.0"

policy:
  name: pi05
  source:
    class_path: physicalai.policies.pi05.Pi05

model:
  artifacts:
    openvino: model.xml
  runner:
    type: action_chunking
    chunk_size: 50
  preprocessors:
    - type: normalize
      artifact: stats.safetensors

hardware:
  robots:
    - name: main
      type: SO101
  cameras:
    - name: wrist
      type: uvc
    - name: overhead
      type: realsense
```

## 清单与工作流配置

| 数据       | 含义                                       |
| -------- | ---------------------------------------- |
| 工作流配置    | 工作流配置描述运行或导出之前期望的工作流。                   |
| 清单       | 清单描述构建或导出之后具体的导出包。                      |

要加载包，使用导出的目录。

```python
model = InferenceModel("./export")
```

要直接查看元数据，加载清单本身。

```python
manifest = Manifest.load("./export/manifest.json")
```

两种 Schema 可以共用 `ComponentSpec`，但用途不同。