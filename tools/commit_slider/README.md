# 提交滑动工具

用于按指定操作自动遍历提交集的工具。例如，按给定条件进行二分查找（检查应用输出、比较打印的 blob 等）。

## 前置条件

git >= *2.0*
cmake >= OpenVino 最低要求版本（[CMakeLists.txt](../CMakeLists.txt)）
python >= *3.6*
ccache >= *3.0*

## 准备（仅限 *Linux*）

 1. 安装 **CCache**：
`sudo apt install -y ccache`
`sudo /usr/sbin/update-ccache-symlinks`
`echo 'export PATH="/usr/lib/ccache:$PATH"' | tee -a ~/.bashrc`
`source ~/.bashrc && echo $PATH`
2. 通过 `which g++ gcc` 检查 **Ccache** 是否已安装
3. 运行 `sudo sh setup.sh`

## 设置自定义配置

*custom_cfg.json* 可覆盖通用 *util/cfg.json* 中的每个字段。以下是最必要的几项。

1. 定义 `makeCmd` —— 应用所需的构建命令。
2. 定义 `commandList`。如需更具体的构建方式，可调整 *commandList*。在 *Win OS* 下，建议用特定 make 命令覆盖 `commandList`，例如在 `{makeCmd}` 之后使用 `cmake --build . --config Release`。详见[自定义命令列表](#ccl)。
3. 若目标不在当前 **Openvino** 仓库内，替换 `gitPath, buildPath`。
4. 根据目标应用设置 `appCmd, appPath`（必填）
5. 设置 `runConfig`（必填）：
5.1. `getCommitListCmd` —— 返回提交列表的 *git* 命令（*若不想通过命令行参数设置提交区间*），或 `explicitList`（手动设置提交）。
示例：
```
"commitList" : {
    "getCommitListCmd" : "git log start_hash..end_hash --boundary --pretty=\"%h\""
}
```
```
"commitList" : {
    "explicitList" : ["hash_1", "hash_2", ... , "hash_N"]
}
```
5.2. `mode` = `{checkOutput|bmPerfMode|compareBlobs|<to_extend>}` —— 提交比较条件
5.3. `traversal` `{firstFailedVersion|firstFixedVersion|allBreaks|bruteForce|<to_extend>}` —— 遍历规则
5.4. `preprocess` —— 若构建提交前需要准备
5.5. 其他字段取决于 mode，例如 `checkOutput` 的 `stopPattern` 是应用失败输出的 *RegEx* 模式。
6. 通过 *envVars* 字段设置环境变量，格式：
`[{"name" : "key1", "val" : "val1"}, {"name" : "key2", "val" : "val2"}]`
7. 设置[预构建应用配置](#pba)。

## 运行提交滑动工具

`python3 commit_slider.py {-c commit1..commit2 | -cfg path_to_config_file}`
`-c` 会覆盖 *cfg.json* 中的 `getCommitListCmd`

## 输出

一般情况下，输出如下
```
    <build details>
    Break commit: "hash_1", <details>
    Break commit: "hash_2", <details>
    <...>
    Break commit: "hash_N", <details>
```
对每个 *x*，*hash_x* 表示导致中断的提交哈希，即前一个提交在当前 Mode 意义下是"好"的，而 *hash_x* 是"坏"的。`<details>` 由 Mode 决定。通用日志和每个提交的日志位于 *log/*。若启用 `printCSV` 标志，*log/* 中还会包含 *report.csv*。

## 示例

### 命令行
`python3 commit_slider.py`
`python3 commit_slider.py -c e29169d..e4cf8ae`
`python3 commit_slider.py -c e29169d..e4cf8ae -cfg my_cfg.json`


### 预定义配置
*utils/cfg_samples* 文件夹中有若干预定义配置，可复制到 *custom_cfg.json*。每个示例中 `<start_commit>..<end_commit>` 表示待分析的区间。所有示例均演示最简单的二分查找。

###### 性能任务
查找 *benchmark_app* 的性能退化。
*bm_perf.json*
`<model_path>` —— 用于基准测试的模型路径，可调整 `perfAppropriateDeviation` 使接受条件更严格或更宽松。
对于 ``benchmark_app`` 性能运行，commit_slider 从其自有的临时 ``-report_folder`` 中由 ``-report_type no_counters`` 生成的 ``benchmark_report.csv`` 读取指标，因此控制台延迟格式变化不影响指标收集。

###### Blob 比较
通过 blob 比较检查精度退化。
*blob.cmp.json*
`<model_path>` —— 模型路径，`<blob_dir>` —— blob 打印目录，`<out_blob_name_tail>` —— blob 名称模式（不含节点 id），例如 *Output_Reshape_2-0_in0*，`limit` 为逐行差异阈值，可修改或置零以进行按位比较，`OV_CPU_BLOB_DUMP_NODE_TYPE` 对应所需节点类型，也可使用其他 dump 参数。

###### 检查输出
查找 *benchmark_app* 的失败。
*custom_cfg.json*
`<model_path>` —— 模型路径，`<bm_error_message>` —— 典型错误消息或其片段，例如 *fail*。

###### 与 e2e 集成
通过 *e2e* 检查精度退化。`<e2e_path>` —— e2e 目录路径，`<e2e_args>` —— e2e 参数，`<ov_path>` —— *ov* 的绝对路径，`<e2e_error_message>` —— e2e 错误消息。


## 可能遇到的问题
若出现意外失败，可检查 */tmp/commit_slider/log/*
###### 构建命令集不足
若某些提交无法构建，可在自定义命令列表中扩展命令集。自定义命令列表示例如下：
```
"commandList" : [
    {"cmd" : "git rm --cached -r .", "path" : "{gitPath}", "tag" : "clean"},
    {"cmd" : "git reset --hard", "path" : "{gitPath}", "tag" : "clean"},
    {"cmd" : "git rm .gitattributes", "path" : "{gitPath}", "tag" : "clean"},
    {"cmd" : "git reset .", "path" : "{gitPath}", "tag" : "clean"},
    {"cmd" : "git checkout -- .", "path" : "{gitPath}", "tag" : "clean"},
    {"cmd" : "git rm --cached -r .", "path" : "{gitPath}", "tag" : "clean"},
    {"cmd" : "git reset --hard", "path" : "{gitPath}", "tag" : "clean"},
    {"cmd" : "git rm .gitattributes", "path" : "{gitPath}", "tag" : "clean"},
    {"cmd" : "git reset .", "path" : "{gitPath}", "tag" : "clean"},
    {"cmd" : "git checkout -- .", "path" : "{gitPath}"},
    {"cmd" : "git clean -fxd", "path" : "{gitPath}", "tag" : "clean"},
    {"cmd" : "mkdir -p build", "path" : "{gitPath}"},
    {"cmd" : "git checkout {commit}", "catchMsg" : "error", "path" : "{gitPath}"},
    {"cmd" : "git submodule init", "path" : "{gitPath}"},
    {"cmd" : "git submodule update --recursive", "path" : "{buildPath}"},
    {"cmd" : "{makeCmd}", "catchMsg" : "CMake Error", "path" : "{buildPath}"},
    {"cmd" : "make --jobs=4", "path" : "{buildPath}"},
    {"cmd" : "git checkout -- .", "path" : "{gitPath}"}
]
```
详见[自定义命令列表](#ccl)。

###### 应用因其他原因失败或输出未正确解析
有时目标 bug 被另一个意外 bug 掩盖。此时建议扩展错误模式，如 {err_msg_1|err_msg_2}，或单独运行以查找新问题。

## 实现自定义 mode
1. 重写 `checkIfBordersDiffer(i1, i2, list)`，定义给定提交在给定条件下是否不同。
2. 若通过 json map 的预定义缓存不满足当前任务，重写 `createCash(), getCommitIfCashed(commit), getCommitCash(commit, valueToCache)`。
3. `checkCfg()` —— 检查用户是否提供了所有必要参数。
4. `setOutputInfo(commit), getResult()` —— 设置并解释找到的提交的参数。


## 实现自定义遍历规则
要实现新的 `Traversal`，重写 `bypass(i1, i2, list, cfg, commitPath)` 方法，使用 Mode 的 `checkIfBordersDiffer(i1, i2, list)` 判断目标提交是否在给定区间内。

## 实现自定义预处理
1. 创建 *utils/preprocess/your_custom_pp.py* 文件。
2. 定义 `def your_custom_pp(cfg): <...>` 函数，实现子进程。
3. 在 `runConfig` 中添加 `"preprocess" : { "name" : your_custom_pp, <other parameters>`，并在构建命令列表中添加 `{"tag" : "preprocess"}`。

## <a name="pba"></a>使用预构建应用

'cachedPathConfig' 选项可加速搜索或解决环境/构建问题。有两种方案：'optional' 在提供的路径允许时调整遍历，否则构建提交。目的是提升性能并解决长区间重建问题（待实现）。'Mandatory' 仅使用提供的路径，遍历不受影响。缺失的提交标记为忽略（假定包含无关紧要的变更）。主要用于复杂环境。

1. 在配置中添加 `cachedPathConfig` 字段
2. 设置 `enable` 和 `scheme` 字段
3. 定义 `cashMap`。
4. `passCmdList` 标志为 true 时忽略 commandList（无需构建）
5. `changeAppPath` 标志表示缓存路径替换 `appPath`

示例：
```
"cachedPathConfig": {
    "enable" : true,
    "scheme" : "mandatory",
    "cashMap" : {
        "hash_1_": "app/path/hash_1_",
        "hash_2_": "app/path/hash_2_",
        ..............................
        "hash_N_": "app/path/hash_N_"
    }
}
```

## <a name="ccl"></a>自定义命令列表
构建命令的结构为
```
"commandList" : [
        {
        "cmd" : "git rm --cached -r .",
        "path" : "directory where command is to be runned",
        "tag" : "is used for managing of command flow (clean, preprocessing)",
        "catchMsg" : "RegEx, like ‘(.)*(error|failed|wrong executor)(.)*’"
        },
    <...>
    ]
```
*cmd* —— 要运行的命令，如 `git rm --cached -r .`，*path* —— 命令目录，通常为 git 根目录或构建目录，*tag* —— 用于检查命令是否应在特殊条件下执行，通常为 `preprocess` 或 `clean`，*catchMsg* —— 用于检查输出的字符串，因 python subprocess API 的异常处理不可靠而需要。