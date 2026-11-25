# 为 WebAssembly 构建 OpenVINO™ 运行时

本指南介绍如何使用 [Emscripten SDK](https://emscripten.org/) 为 WebAssembly 构建 OpenVINO。Emscripten SDK 可直接下载使用，但使用 [emscripten/emsdk](https://hub.docker.com/r/emscripten/emsdk) Docker 镜像更方便。

该方法已在 Linux、Windows 和 macOS 操作系统上验证。

## 软件要求

- [Docker Engine](https://docs.docker.com/engine/install/)

## 构建方法

1. 克隆 OpenVINO 仓库并初始化子模块：
```sh
git clone https://github.com/openvinotoolkit/openvino.git
cd openvino
git submodule update --init --recursive
```
2. 运行 Docker 镜像并挂载 OpenVINO 源码目录：
```sh
$ docker pull emscripten/emsdk
$ docker run -it --rm -v `pwd`:/openvino emscripten/emsdk bash
```
3. （CMake 配置）使用 emscripten 辅助命令运行 cmake 配置步骤：
```sh
$ mkdir build && cd build
$ emcmake cmake -DCMAKE_BUILD_TYPE=Release /openvino
```
4. （CMake 构建）构建 OpenVINO 项目：
```sh
$ emmake make -j$(nproc)
```
`openvino.wasm` 和 `openvino.js` 文件位于：
- 宿主机文件系统中的 `<openvino_source_dir>/bin/ia32/Release/`。
- Docker 环境中的 `/openvino/bin/ia32/Release`。
这些文件可用于浏览器应用。

## 另请参阅

 * [OpenVINO README](../../README.md)
 * [OpenVINO 开发者文档](index.md)
 * [OpenVINO 入门](./get_started.md)
 * [如何构建 OpenVINO](build.md)