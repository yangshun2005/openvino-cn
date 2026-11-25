# 性能交叉检查

`bm_cc` — 用于检查性能下降是由 OV 版本还是模型导致的模板。

---

## 如何运行

示例 1
```
python3 commit_slider -t bm_cc -c <start>..<end> -appCmd './benchmark_app {actualPar} -hint throughput -i input.png' -par_1 first_model.xml -par_2 second_model.xml -gitPath <pathToGitRepository> -buildPath <pathToBuildDirectory>
```
示例 2（带配置）
```
python3 commit_slider -cfg custom_cfg.json
```
其中
```
cfg = { "template" : { 
    "name" : "bm_cc",
    "c" : "<start>..<end>",
    "appCmd" : [
        "./benchmark_app -m model_1 -hint throughput -i input_1.png",
        "./benchmark_app -m model_2 -hint throughput -i input_2.png"
    ]
}}
```
### 独立变量：

```
- verbosity: -v {true|false}
```
### 预期输出：

```
*****************<Commit slider output>*******************
Table of throughputs:
             m1               m2
<start_hash> <throughput_00>  <throughput_01>
<end_hash>   <throughput_10>  <throughput_11>
**********************************************************
Supposed rootcause is: <Model | OV>
**********************************************************

*  m1 = ./benchmark_app -m first_model.xml
** m2 = ./benchmark_app -m second_model.xml
```