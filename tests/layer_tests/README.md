# 层测试

层测试主要验证 OpenVINO 对 PyTorch、TensorFlow、TensorFlow Lite 和 ONNX 框架算子的支持。
测试流水线包含以下步骤：
1. 使用原框架 API 创建含被测算子的模型
2. 使用 OpenVINO 的 `convert_model` 方法转换所创建的模型
3. 在随机输入数据上，分别用原框架和 OpenVINO 对原始模型和转换后模型进行推理
4. 检查 OpenVINO 与框架的推理结果是否相同，或在容差阈值内存在差异

## 环境搭建

搭建层测试运行环境，执行以下步骤：
1. 安装 OpenVINO wheel 包。若测试 OpenVINO 的改动，需构建本地 wheel 包用于安装。
构建方法见 [wiki 页面](https://github.com/openvinotoolkit/openvino/blob/master/docs/dev/build.md)。
    ```sh
    pip install openvino.whl
    ```
2. （可选）若测试使用 OpenVINO Tokenizers 的转换和算子扩展来支持算子，需安装 OpenVINO Tokenizers wheel 包。
若测试 OpenVINO Tokenizers 的改动，需构建本地 wheel 包用于安装。
构建方法见 [GitHub 页面](https://github.com/openvinotoolkit/openvino_tokenizers)
    ```sh
    pip install openvino_tokenizers.whl
    ```
3. 安装层测试运行所需的依赖。
    ```sh
    cd tests/layer_tests
    pip install -r requirements.txt
    ```

## 运行测试

设置环境变量 `TEST_DEVICE` 和 `TEST_PRECISION`，以选择 OpenVINO 推理所用的设备和推理精度。`TEST_DEVICE` 允许的值为 `CPU` 和 `GPU`。`TEST_PRECISION` 允许的值为 `FP32` 和 `FP16`。

示例：在 CPU 上以设备默认推理精度运行 `tf.raw_ops.Unique` 算子的 TensorFlow 层测试：
   ```sh
   cd tests/layer_tests
   export TEST_DEVICE="CPU"
   pytest tensorflow_tests/test_tf_Unique.py
   ```

示例：在 CPU 和 GPU 上以 `FP16` 和 `FP32` 推理精度运行 `torch.linalg.cross` 算子的 PyTorch 层测试：
   ```sh
   cd tests/layer_tests
   export TEST_DEVICE="CPU;GPU"
   export TEST_PRECISION="FP32;FP16"
   pytest pytorch_tests/test_cross.py
   ```