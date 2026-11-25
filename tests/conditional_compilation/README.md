# 条件编译测试

本文件夹包含条件编译（CC）测试框架代码和 CC 测试文件。

## 环境准备：
安装测试所需的 Python 模块：
```bash
pip3 install -r requirements.txt 
```

从 https://github.com/openvinotoolkit/open_model_zoo 克隆 Open Model Zoo

## 运行测试

```bash
pytest test_cc.py
```
**测试参数：**
- `sea_runtool` - `sea_runtool.py` 文件的路径。
- `collector_dir` - collector 文件父文件夹的路径。
- `artifacts` - 测试写入输出或读取输入的目录路径。
- `openvino_root_dir` - OpenVINO 仓库根目录的路径。
- `omz_repo` - Open Model Zoo 仓库根目录的路径。

**可选参数：**
- `test_conf` - 测试用例 .yml 配置的路径。
- `openvino_ref` - 已安装 OpenVINO 的根目录路径。若未指定该选项，CC 测试会先构建并安装插桩包到 `<artifacts>/ref_pkg` 文件夹，使用 `--openvino_root_dir` 选项指定的 OpenVINO 仓库。
    > 若 OpenVINO 插桩包已成功安装，后续可将 `--openvino_ref` 参数设为 `<artifacts>/ref_pkg` 以获得更好的性能。
- `omz_cache_dir` - Open Model Zoo 缓存目录的路径。

**用法示例：**
```bash
pytest test_cc.py --sea_runtool=./thirdparty/itt_collector/runtool/sea_runtool.py --collector_dir=./bin/intel64/Release --artifacts=../artifacts --openvino_root_dir=. --omz_repo=../_open_model_zoo
```