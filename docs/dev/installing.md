# 安装

项目构建完成后，可将 OpenVINO™ Runtime 安装到自定义位置：

```
cmake --install <BUILDDIR> --prefix <INSTALLDIR>
```

## 构建并运行示例

1. 构建示例。

   构建 C++ 示例应用，运行以下命令：

   Linux 和 macOS：
   ```sh
   cd <INSTALLDIR>/samples/cpp
   ./build_samples.sh
   ```

   Windows 命令提示符：
   ```sh
   cd <INSTALLDIR>\samples\cpp
   build_samples_msvc.bat
   ```

   Windows PowerShell：
   ```sh
   & <path-to-build-samples-folder>/build_samples.ps1
   ```

2. 下载模型。

   可从
   [Hugging Face](https://huggingface.co/models?pipeline_tag=image-classification&sort=trending)
   下载一个图像分类模型来运行示例

4. 转换模型。

   Linux 和 macOS：
   ```sh
   ovc <path-to-your-model> --compress_to_fp16=True
   ```
   Windows：
   ```bat
   ovc <path-to-your-model> --compress_to_fp16=True
   ```

5. 在示例上运行推理。

   设置 OpenVINO 环境变量：

   Linux 和 macOS：
   ```sh
   source <INSTALLDIR>/setupvars.sh
   ```

   Windows 命令提示符：
   ```bat
   <INSTALLDIR>\setupvars.bat
   ```

   Windows PowerShell：
   ```bat
   . <path-to-setupvars-folder>/setupvars.ps1
   ```

   以下命令以 [`dog.bmp`](https://storage.openvinotoolkit.org/data/test_data/images/   224x224/dog.bmp) 文件作为输入图像、IR 格式的模型，在不同硬件设备上运行图像分类代码示例：

   Linux 和 macOS：

   ```sh
   cd ~/openvino_cpp_samples_build/<architecture>/Release
   ./classification_sample_async -i <path-to-input-image>/dog.bmp -m <path-to-your-model>/model.xml -d CPU
   ```
   其中 <architecture> 是 ``uname -m`` 的输出，例如 ``intel64``、``armhf`` 或 ``aarch64``。

   Windows：

   ```bat
   cd  %USERPROFILE%\Documents\Intel\OpenVINO\openvino_cpp_samples_build\<architecture>\Release
   .\classification_sample_async.exe -i <path-to-input-image>\dog.bmp -m <path-to-your-model>\model.xml -d CPU
   ```
   其中 <architecture> 为 ``intel64`` 或 ``aarch64``，取决于平台架构。

示例应用运行完成后，屏幕上会显示前 10 个类别的标签和置信度数据：

以下是使用 googlenet-v1 模型的结果。

```
Top 10 results:

Image dog.bmp

classid probability
------- -----------
156     0.6875963
215     0.0868125
218     0.0784114
212     0.0597296
217     0.0212105
219     0.0194193
247     0.0086272
157     0.0058511
216     0.0057589
154     0.0052615

```


## 将 OpenVINO Runtime 添加到项目

对于 CMake 项目，运行 CMake 工具时设置 `OpenVINO_DIR`：

```sh
cmake -DOpenVINO_DIR=<INSTALLDIR>/runtime/cmake .
```

然后可通过 [`find_package`] 查找 OpenVINO Runtime：

```cmake
find_package(OpenVINO REQUIRED)
add_executable(ov_app main.cpp)
target_link_libraries(ov_app PRIVATE openvino::runtime)

add_executable(ov_c_app main.c)
target_link_libraries(ov_c_app PRIVATE openvino::runtime::c)
```

## 另请参阅

 * [OpenVINO README](../../README.md)
 * [OpenVINO 开发者文档](index.md)
 * [OpenVINO 构建方法](build.md)