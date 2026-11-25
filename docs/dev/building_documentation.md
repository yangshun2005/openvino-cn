# 如何构建文档

以下步骤已在 Windows 和 Ubuntu 操作系统上验证。

## 目录：

* [安装依赖](#installing-dependencies)
* [构建文档](#building-documentation)

## 安装依赖

除常规构建依赖外，还必须安装 `doxygen` 和 `latex`：

### Windows

* [miktex](https://miktex.org/)
* [doxygen](https://doxygen.nl/download.html)（版本 >= 1.8.20）
* [Graphviz](https://graphviz.org/download/)

### Ubuntu 系统

* Latex 和 graphviz：

```sh
apt-get install texlive-full graphviz
```

* goxygen 版本 >= 1.8.20：

```
$ git clone https://github.com/doxygen/doxygen.git
$ cd doxygen
$ git checkout Release_1_8_20
$ mkdir build
$ cd build
$ cmake ..
$ cmake --build . -j8
$ sudo make install
```

## 构建文档

应使用 `-DENABLE_DOCS=ON` 运行 `cmake`。在 Ubuntu 系统上会自动找到所有依赖，而在 Windows 上仍需指定已安装依赖的路径：

```sh
cmake -DLATEX_COMPILER="C:/Program Files/MiKTeX/miktex/bin/x64/latex.exe" \
      -DDOXYGEN_DOT_EXECUTABLE="C:/Program Files (x86)/Graphviz2.38/bin/dot.exe" \
      -DDOXYGEN_EXECUTABLE="C:/Program Files/doxygen/bin/doxygen.exe" \
      -DENABLE_DOCS=ON \
```

找到依赖后，必须使用 CMake 生成项目。需构建目标 `openvino_docs` 以生成 doxygen 文档，生成的文件位于 `<binary dir>/docs/html/index.html`

## 另请参阅

 * [OpenVINO README](../../README.md)
 * [OpenVINO 开发者文档](index.md)
 * [如何构建 OpenVINO](build.md)