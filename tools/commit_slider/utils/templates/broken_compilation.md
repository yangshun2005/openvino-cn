# 编译中断

`broken_compilation` — 用于查找首个编译中断的提交（即编译阶段失败的提交）的模板。

- 使用 Cmake 方式构建（不使用 wheel）
- 查找编译（而非链接）错误

---

## 如何运行
示例 1（CLI）
```
python3 commit_slider.py -t broken_compilation -c <start_commit>..<end_commit> -gitPath <pathToGitRepository> -buildPath <pathToBuildDirectory>
```
示例 2（配置，默认路径）
```
{ "template" : {
    "name" : "broken_compilation",
    "c" : "<start_commit>..<end_commit>"
}
```
### 独立变量：
```
- verbosity: -v {true|false}
```

### 预期输出：
```
*****************<Commit slider output>*******************
* Commit with broken compilation found:                  *
* Break commit: <comit_hash>, state: CommitState.BREAK   *
**********************************************************
```