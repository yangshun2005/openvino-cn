# AppVerifier 测试套件

本测试套件用于检测 AppVerifier 是否会报告内存泄漏。

## 入门

AppVerifier 测试基于 googletest 框架。可用 `--gtest_filter` 过滤测试，用 `--gtest_list_tests` 选项查看可用测试。

### 前置条件

- 用于构建测试的 Windows 操作系统。

### 构建测试

构建测试前，需已安装 OpenVINO™ 或从源码构建。构建前打开终端，设置 OpenVINO™ 环境，然后运行以下命令：
``` bash
<OpenVINO_install_dir>/setupvars.bat
mkdir build && cd build
cmake .. && cmake --build . --config Release -j8
```

### 运行测试

``` bash
.\test\Release\ov_appverifier_tests.exe
```

可直接用上述命令运行测试。但若要检测是否存在内存泄漏，需先将测试可执行文件添加到 AppVerifier（参见 https://learn.microsoft.com/en-us/windows-hardware/drivers/devtest/application-verifier-testing-applications），再运行上述命令。