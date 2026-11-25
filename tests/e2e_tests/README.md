# 端到端测试用户文档
本文件夹包含在不同框架（PyTorch、TensorFlow 和 ONNX）的真实模型上运行 OpenVINO 端到端验证的代码。

本文档提供端到端验证运行的环境搭建、向验证中添加新模型以及启动验证的说明。

> 以下步骤假设当前工作目录为：
> `tests/e2e_tests`

### 环境准备：
   * 安装测试所需的 Python 模块：
       ```bash
       pip3 install -r requirements.txt 
       ```
     
### 将 TensorFlow Hub 仓库中的模型添加到端到端验证：
要为 TF Hub 仓库中的模型添加新测试，只需在 pipelines/production/tf_hub/precommit.yml 中添加一行。
该行应包含以逗号分隔的模型名称及其链接。
```
movenet/singlepose/lightning,https://www.kaggle.com/models/google/movenet/frameworks/tensorFlow2/variations/singlepose-lightning/versions/4
```

### 主入口点

有一个主测试入口点负责测试运行——test_base.py。该脚本执行以下操作：
1. 从源加载模型
2. 通过框架推理原始模型
3. 通过 OVC 转换模型转换原始模型
4. 通过 OpenVINO 推理转换后的模型
5. 提供框架与 OpenVINO 推理结果的逐元素比较结果

#### 启动测试

[test_base.py](https://github.com/openvinotoolkit/openvino/tree/master/tests/e2e_tests/test_base.py) 是运行端到端测试的主脚本。
运行 `pipelines/` 中的所有端到端测试：
```bash
pytest test_base.py
```
`test_base.py` 选项：

- `--modules=MODULES [MODULES ...]` - 测试路径。
- `-k TESTNAME [TESTNAME ...]`- 测试名称。
- `-s` - 逐步日志记录。
    
示例：
```bash
pytest test_base.py -s --modules=pipelines/production/tf_hub
```

> 有关 pytest 选项的完整信息，请运行 `pytest --help` 或参阅[文档](https://docs.pytest.org)