# 模糊测试套件

本测试套件包含用于 [libFuzzer](https://llvm.org/docs/LibFuzzer.html) 模糊测试引擎的[模糊测试](https://en.wikipedia.org/wiki/Fuzzing)。

## 入门

每个模糊测试都是一个可执行文件。它可以运行模糊测试来搜索新的失败，并将复现用例保存到文件中。之后可以用复现用例运行模糊测试，调试发现的失败。

## 前置条件

复现和调试失败没有特殊前置条件。

运行模糊测试需要 [LLVM](https://apt.llvm.org/) 组件：
- Clang 及相关工具
- libFuzzer
- lld（链接器）
- libc++

## 构建模糊测试

1. 构建 openvino

启用 `ENABLE_FUZZING` 和 `ENABLE_SANITIZER` 选项构建 openvino。建议使用 clang 编译器。

```bash
(\
mkdir -p build && cd build && \
CC=clang CXX=clang++ cmake .. -DENABLE_FUZZING=ON -DENABLE_SANITIZER=ON && \
cmake --build . \
)
```

2. 构建模糊测试

启用 `ENABLE_FUZZING` 和 `ENABLE_SANITIZER` 选项构建模糊测试。应使用与构建 openvino 相同的编译器。

```bash
(\
mkdir -p tests/fuzz/build && cd tests/fuzz/build && \
CC=clang CXX=clang++ cmake .. -DENABLE_FUZZING=ON -DENABLE_SANITIZER=ON -DOpenVINO_DIR=$(pwd)/../../../build && \
cmake --build . \
)
```

## 运行模糊测试

1. 准备模糊测试语料库

模糊测试引擎需要一组有效输入作为起点。这些文件称为模糊测试语料库。将模糊测试的有效输入放入目录中。

Intel 员工可按此处说明获取语料库
https://wiki.ith.intel.com/x/2N42bg。

2. 运行模糊测试

```bash
# LD_PRELOAD is required when OpenVINO build as shared library, the ASAN library has to be pre-loaded.
[LD_PRELOAD=path-to-asan-lib] ./read_network-fuzzer -max_total_time=600 ./read_network-corpus
```

建议添加以下有用的命令行选项：
- `-jobs=$(nproc)` 并行运行多个模糊测试任务。注意：需通过环境变量 `LLVM_PROFILE_FILE=deafult-%p.profraw` 配置代码覆盖率分析。
- `-rss_limit_mb=0` 忽略内存不足问题。

## 分析模糊测试质量

### 查看代码覆盖率

模糊测试执行后，运行以下命令生成覆盖率报告：

```
llvm-profdata merge -sparse *.profraw -o default.profdata && \
llvm-cov show ./read_network-fuzzer -object=lib/libopenvino.so -instr-profile=default.profdata -format=html -output-dir=read_network-coverage
```

## 复现发现的问题

模糊测试在发现第一个问题时停止，将问题详情打印到 stdout，并把复现该问题的数据保存为当前文件夹中的文件。要调试该问题，将复现用例作为命令行参数传给模糊测试

```bash
./read_network-fuzzer crash-409b5eeed46a8445b7f7b7a2ce5b60a9ad895e3b
```

建议（但非必须）使用为模糊测试构建的二进制文件调试问题。未启用 `ENABLE_FUZZING` 选项构建的二进制文件也可用于复现和调试问题。