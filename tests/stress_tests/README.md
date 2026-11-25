# 压力测试套件

本测试套件包含在压力条件下评估各种 OpenVINO 用例行为的测试：

- MemCheckTests 测量用例所需内存，内存使用超过预设水平时失败。

- StressMemLeaksTests 确保用例持续执行时内存水平不增长。

- StressUnitTests 在多线程和多进程中并行执行各种 OpenVINO 用例。

每个测试引用 `<test dir>\.automation` 文件夹中的配置文件。

## 入门

压力测试基于 googletest 框架。可用 `--gtest_filter` 过滤测试，用 `--gtest_list_tests` 查看可用测试。

测量内存的测试有临时限制——需逐个执行，以免污染内存统计。可用 [gtest-parallel][gtest-parallel] 大规模执行测试。

### 前置条件

- 用于构建测试的 Linux 操作系统。

- 用于执行测试的 [gtest-parallel][gtest-parallel]。

### 构建测试

构建测试需已安装 OpenVINO™ 或从源码构建。构建前，打开终端，设置 OpenVINO™ 环境，然后运行以下命令：
``` bash
source <OpenVINO_install_dir>/setupvars.sh
mkdir build && cd build
cmake .. && cmake --build . -j8
```

### 准备测试数据

压力测试可使用 [Open Model Zoo][open_model_zoo] 中的模型。使用时，用 `./scripts/get_testdata.py` 脚本下载模型并转换为 IR。脚本会更新测试配置文件，写入 OMZ 模型执行所需数据。

在 Intel 网络内，可使用 `vdp_tests` 文件共享中的模型缓存。文件共享的使用方法参见 [VDP 共享文件夹][VDP-shared-folders]。

### 运行测试

``` bash
gtest-parallel <openvino_bin>/StressMemLeaksTests
```

MemCheckTests 推荐使用：
``` bash
python ./scripts/run_memcheck.py --gtest_parallel <gtest_parallel_py_path> 
<openvino_bin>/MemCheckTests -- --test_conf=<test_conf_path> --refs_conf=<refs_conf_path>
``` 

MemCheckTests 日志可用于根据当前内存消耗收集参考值：

``` bash
mkdir -p MemCheckTests-logs && \
gtest-parallel -d ./MemCheckTests-logs ./MemCheckTests && \
grep -rh ./MemCheckTests-logs -e ".*<model " | sed -e "s/.*<model /<model /" | sort
```

[VDP-shared-folders]: https://wiki.ith.intel.com/display/DLSDK/VDP+shared+folders
[gtest-parallel]: https://github.com/google/gtest-parallel
[open_model_zoo]: https://github.com/openvinotoolkit/open_model_zoo