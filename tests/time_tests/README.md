# 时间测试

本测试套件包含流水线（即可执行文件）。流水线测量其执行时间，包括总时间和部分时间。Python 运行器调用这些流水线并计算平均执行时间。

## 前提条件

要构建时间测试，需已安装 OpenVINO™ 或从源码构建。

## 测量时间

要构建并运行测试，打开终端，设置 OpenVINO™ 环境并运行以下命令：

1. 构建测试：
``` bash
mkdir build && cd build
cmake .. && cmake --build . --target time_tests -j8
```

2. 安装测试：
``` bash
сmake --install <build_dir> --prefix <install_path>
```

3. 运行测试：
``` bash
./scripts/run_timetest.py ../../bin/intel64/Release/timetest_infer -m model.xml -d CPU
```

4. 使用 `pytest` 运行多个配置：
``` bash
pytest ./test_runner/test_timetest.py --exe ../../bin/intel64/Release/timetest_infer

# 用于 parse_stat 测试：
pytest ./scripts/run_timetest.py
```