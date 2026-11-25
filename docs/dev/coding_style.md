# OpenVINO 编码规范

## 修复顺序

使一组变更符合规范时，按以下顺序处理各项检查：

1. **版权头** — 速度快，无需构建
2. **clang-tidy** — 需要干净的构建；先修复编译错误，再应用 tidy 自动修复
3. **命名风格（ncc）** — 需要 clang，但无需完整重新构建；在 tidy 之后修复，避免重复改动
4. **clang-format** — 纯格式调整；最后运行，以免 tidy 自动修复重新引入格式问题

## 版权头

所有源文件必须包含标准的 OpenVINO 版权头。

C++（`.cpp`、`.hpp`、`.h`）：
```cpp
// Copyright (C) 2018-<current_year> Intel Corporation
// SPDX-License-Identifier: Apache-2.0
//
```

Python（`.py`）：
```python
# Copyright (C) 2018-<current_year> Intel Corporation
# SPDX-License-Identifier: Apache-2.0
```

Python 文件中，版权头之前可以有 shebang 和/或编码声明。

可使用 `.github/scripts/check_copyright.py` 脚本自动检查和修复版权头。若脚本以非零状态退出，会写出 `copyright_fixes.diff`，可用 `patch -p1 < copyright_fixes.diff` 应用。

注意：自动生成的补丁偶尔会产生错误改动——某些情况下需要手动修复。

## 静态分析（clang-tidy）

使用 `clang-tidy-18` 进行静态分析并强制现代 C++ 模式。

**前提条件：** 必须安装 `clang-tidy-18`，且项目必须使用 clang 作为编译器编译。启用前，先确认现有构建目录已使用 clang：
```bash
grep "CMAKE_CXX_COMPILER" <build_dir>/CMakeCache.txt | grep -i clang
```

**如果已使用**，在该目录中重新运行 cmake 以追加风格标志（保留现有配置）：
```bash
cmake -DENABLE_CLANG_FORMAT=ON -DENABLE_CLANG_TIDY=ON <build_dir>
```

**如果未使用**（或不存在构建目录），设置 `<build_dir>=build-clang` 并配置：
```bash
cmake -B <build_dir> -DCMAKE_CXX_COMPILER=clang++-18 -DCMAKE_C_COMPILER=clang-18 \
      -DENABLE_CLANG_FORMAT=ON -DENABLE_CLANG_TIDY=ON \
      -DCMAKE_BUILD_TYPE=Release
```

构建目标以运行检查（编译期间会内联显示警告）：
```bash
cmake --build  <build_dir> --target <your_target> -- -j$(nproc)
```

要自动修复诊断，启用修复模式并重新构建：
```bash
cmake -DENABLE_CLANG_TIDY_FIX=ON <build_dir>
cmake --build  <build_dir> --target <your_target> -- -j$(nproc)
```

注意：
- 必须先解决编译错误，才能应用 clang-tidy 诊断。
- 自动修复偶尔会产生错误改动——某些情况下需要手动修复。
- 最好为 clang-tidy 使用专用构建目录（如 `build-clang/`）——clang-tidy 会重写编译数据库，可能干扰共享目录中的增量构建。
- 不要用 `// NOLINT` 抑制诊断。抑制会隐藏问题而不修复它，并可能掩盖未来代码中的真实问题。强烈建议仅在检查确认为误报且没有正确替代方案时抑制，并在行内记录原因。

## 命名风格（ncc）

OpenVINO 对公共 API 的命名风格有严格规则。所有类必须以大写字母开头，方法和函数应使用 `snake_case` 风格命名。
为检查命名风格，OpenVINO 集成了 `ncc` 工具。命名风格的详细信息见[配置文件](../../cmake/developer_package/ncc_naming_style/openvino.style)。
要启用此工具，本地需安装 `clang` 并启用 CMake 选项 `ENABLE_NCC_STYLE`。
之后可使用 `ncc_all` 目标检查命名风格。

## 编码风格（clang-format）

大多数 OpenVINO 组件使用 `clang-format-18` 进行代码风格检查。

**前提条件：** 必须安装 `clang-format-18`。无需特定编译器——clang-format 作为独立工具运行，与项目的编译方式无关。

代码风格基于 Google Code Style，并有一些差异。所有差异见配置文件：
https://github.com/openvinotoolkit/openvino/blob/69f709028a5f8da596d1d0df9a0101e517c35708/src/.clang-format#L1-L28

要在构建中启用 clang-format 检查，设置相应的 CMake 标志：
```bash
cmake -DENABLE_CLANG_FORMAT=ON <other flags> ..
```

检查与修复目标：
```bash
cmake --build  <build_dir> --target clang_format_check_all   # 报告违规
cmake --build  <build_dir> --target clang_format_fix_all     # 自动修复违规
```

## 另见
 * [OpenVINO™ README](../../README.md)
 * [开发者文档](../../docs/dev/index.md)