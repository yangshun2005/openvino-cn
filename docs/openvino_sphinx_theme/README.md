# openvino_sphinx_theme

## 安装与使用

1. 使用 `python` 安装 `openvino_sphinx_theme`：
```
cd openvino/docs/openvino_sphinx_theme
python -m pip install --user .
```

2. 更新 `conf.py` 中的 `html_theme` 变量：

```
html_theme = 'openvino_sphinx_theme'
```

## 配置

### 主题 Logo

要在导航栏左侧添加 Logo，使用 `html_logo` 变量设置 Logo 文件路径。

```
html_logo = <path to the logo file>
```

### 版本和语言选择器

要启用版本和语言选择器，在 `conf.py` 的 `html_context` 中添加以下配置：

```
html_context = {
    'current_version': 'latest',
    'current_language': 'en',
    'languages': (('English', '/en/latest/'), ('Chinese', '/cn/latest/')),
    'versions': (('latest', '/en/latest/'), ('2022.1', '/en/2022.1'))
}
```

你可以只添加版本选择器或语言选择器。
要添加版本选择器，必须同时定义 `current_version` 和 `versions` 属性。
要添加语言选择器，必须同时定义 `current_language` 和 `languages` 属性。


### 维护者

* OpenVINO 文档团队