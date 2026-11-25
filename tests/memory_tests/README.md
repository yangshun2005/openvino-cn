# 内存测试

本测试套件包含流水线（可执行文件）。
内存测试测量用例所需内存，当内存使用超过预设阈值时判定失败。

## 前置条件

构建内存测试需要已安装 OpenVINO™ 或从源码构建。

## 测量时间

构建并运行测试：打开终端，设置 OpenVINO™ 环境，然后执行以下命令：

1. 构建测试：
``` bash
mkdir build && cd build
cmake .. && make memory_tests
```

2. 安装测试：
``` bash
сmake install <build_dir> --prefix <install_path>
```

3. 运行测试：
``` bash
./scripts/run_memorytest.py <install_path>/tests/memtest_infer -m model.xml -d CPU
```

4. 使用 `pytest` 运行多组配置：
``` bash
pytest ./test_runner/test.py --exe <install_path>/tests/memorytest_infer
# 用于 parse_stat 测试：
pytest ./scripts/run_memorytest.py
```