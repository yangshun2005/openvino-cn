这些测试在预生成的 IR 上运行 OV 示例。首次运行时，测试数据会下载到 `pytest` 生成的缓存目录中。

<INSTALL_DIR> - OpenVINO 安装目录

你不仅可以从 <INSTALL_DIR> 运行测试，但此时需要记得调整环境变量，例如 WORKSPACE（用于 `Test_model_creation_sample`）和 SHARE。

安装冒烟测试：
``` bash
cd <working directory>/tests/samples_tests/smoke_tests
mkdir build && cd build
cmake ../..
cmake -DCOMPONENT=tests -DCMAKE_INSTALL_PREFIX=<INSTALL_DIR> -P cmake_install.cmake
```
1. 从安装目录运行测试需要先安装一些依赖：
   ``` bash
    pip3 install -r  <INSTALL_DIR>\tests\smoke_tests\requirements.txt
   ```
2. 调用 setupvars 脚本，然后设置环境变量：
a. 必需：
    - IE_APP_PATH：C++ 和 C 示例的公共路径，例如 '<INSTALL_DIR>/samples_bin'
    - IE_APP_PYTHON_PATH：Python OV 示例的路径，例如 '<INSTALL_DIR>/samples/python/'
    - 安装 openvino-*.whl 以提供 Python benchmark_app 可执行文件
b. 可选：
    - TEST_DEVICE=CPU;MULTI:CPU;AUTO，默认为 CPU;MULTI:CPU;AUTO
4. 通过 pytest 运行所有测试：
   ``` bash
    python -m pytest
   ```
5. 只运行一个示例（例如 classification_sample_async）：
   ``` bash
    python -m pytest test_classification_sample_async.py
   ```
6. 要运行性能测试，添加 pytest 键："performance n"，其中 n 是性能迭代次数。
   测试会在示例输出中查找 'fps'，如果存在，
   则重新运行该示例，并添加键 'niter n'，其值为性能运行次数（即你通过 '--performance n' 传给 pytest 的值）。
   若不想添加 'niter' 键，请执行 pytest "--performance 0"。

这些测试使用预生成的 IR，目前位于上述共享文件夹中。数据（图像、视频等）也位于该共享文件夹中。