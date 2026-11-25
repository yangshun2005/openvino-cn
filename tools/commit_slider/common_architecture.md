## 使用模板的 CS 工作流

通过 CS 解决问题有两种基本方式：纯配置和 `Template`。前者使用完整的底层功能。`Template` 可以创建自定义 API 和输出表示。

CS 通过配置使用 `Template`：
```
{
    'template' : {
        'name' : 'some_template',
        'parameter_1' : '...',
        ...
        'parameter_N' : '...'
    }
}
```
或通过 CLI：

`python3 commit_slider -t some_template <template parameters>`

```mermaid
sequenceDiagram
    participant User
    participant Template
    participant Config Manager
    participant Mode
    participant Traversal
    participant Commit Path
    User->>Template: 运行 commit_slider.py
    Note right of User: 通过配置或 CLI 传入参数
    Template->>Config Manager: 校验参数并生成对应配置
    Note right of Template: 确定 Mode 和 Traversal
    Config Manager->>Mode: 准备配置（规范化）
    Note right of Config Manager: 将路径转为绝对路径，检查配置一致性
    loop MainLoop
        Traversal->>Mode: 滑动到另一个提交
        Mode->>Traversal: 解析当前提交
        Traversal->>Commit Path: 用当前提交填充
    end
    Note right of Commit Path: Mode 追加额外信息，<br>对应提交状态和目标任务
    Commit Path->>Mode: 包含每个经过提交详情的提交路径
    Template-->>Mode: 表示函数
    Mode->>User: 格式化输出
```

## 不使用模板的 CS 工作流（纯配置）
```mermaid
sequenceDiagram
participant User
participant Config Manager
participant Mode
participant Traversal
participant Commit Path
User->>Config Manager: 带配置运行 commit_slider.py
Note right of User: 校验并准备配置，<br>Mode 和 Traversal 由用户直接确定
Config Manager->>Mode: 准备配置（规范化）
Note right of Config Manager: 将路径转为绝对路径，检查配置一致性
loop MainLoop
Traversal->>Mode: 滑动到另一个提交
Mode->>Traversal: 解析当前提交
Traversal->>Commit Path: 用当前提交填充
end
Note right of Commit Path: Mode 追加额外信息，<br>对应提交状态和目标任务
Commit Path->>Mode: 包含每个经过提交详情的提交路径
Mode->>User: 格式化输出
```