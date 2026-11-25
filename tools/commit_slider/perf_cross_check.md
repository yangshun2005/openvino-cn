# 案例 #1. 性能退化交叉验证

（[基本概念。](README.md) [关于模板的更多内容。](common_architecture.md)）

#### 主题：模式与模板定制、API 设计

对于由提交哈希定义的 2 个给定版本和 2 个模型，我们需要判断两个变更（ov 或模型）中哪一个导致了退化。换句话说，我们需要区分 2 种可能的情况：

模型导致：
|'坏'模型 | model_1.xml| model_2.xml|
| --------     | -------    | -------|
| ov_version_1 | 100 %（基准）|50%     |
| ov_version_2 | 98 %       |51%     |

以及 ov 导致

|'坏' ov 版本| model_1.xml| model_2.xml|
| --------     | -------    | -------|
| ov_version_1 | 100 %（基准）|98%     |
| ov_version_2 | 48% %       |51%     |

在接下来的步骤中，我们将使用逐步更高级的方法来解决给定问题。

## 版本 #1. 复用现有类

### 遍历
我们需要完整遍历提交集（本例中为一对提交），不受特定提交结果的影响。

这对应于现有的 `brute_force` 遍历。

### 模式

在每个提交上我们运行若干命令。

对于第一个版本，我们还复用现有的 `Nop`（无操作）模式，配合简单的 bash 命令列表。

### 配置
最简单的配置，使用默认设置（分别为 openvino/、build/ 和 bin/intel64/Release/ 路径）。
```
We use explicitList instead of commit interval.
{
    "appCmd" : [
        "./benchmark_app -m model_1_path.xml",
        "./benchmark_app -m model_2_path.xml"
        ],
    "runConfig" : {
        "commitList" : {
            "explicitList" : [ "<start_commit>", "<end_commit>" ]
        },
    "mode" : "nop",
    "traversal" : "bruteForce"
}
```
或指定自定义路径：
```
{
    "appCmd" : [
        "echo first model",
        "./benchmark_app -m model_1_path.xml",
        "echo second model",
        "./benchmark_app -m model_2_path.xml"
        ],
    "appPath": "<path_to_benchmark_app>",
    "gitPath" : "<path_to_git_repository>",
    "buildPath" : "<path_to_build_dir>",
    "runConfig" : {
        "commitList" : {
            "explicitList" : [ "<start_commit>", "<end_commit>" ]
    },
    "mode" : "nop",
    "traversal" : "bruteForce"
}
```
### 启动与输出示例
```
python3 commit_slider.py -cfg cfg.json

current commit: <start_commit_hash>
first model
<...> Throughput: 500 FPS
second model
<...> Throughput: 500 FPS

current commit: <end_commit_hash>
first model
<...> Throughput: 1000 FPS
second model
<...> Throughput: 1000 FPS

```
它能应对给定问题，但显得笨拙且冗长。我们还需要自己解读结果。

## 版本 #2：专用模式

如果没有能解决我们任务的模式 [关于模式的详细信息](README.md# implementing-custom-traversal-rule)，或者我们需要更可读的输出以及直接处理每个提交的能力，我们可以实现自定义模式。我们还可以将特定参数直接传递给模式。
### 模式
```
# Mode supposes to be as much generic as possible
# and corresponds all similar problems,
# and not connected with the separate user case,
# but as the example we provide simplified version.
class  CrossCheckMode(Mode):
    def  __init__(self, cfg):
        super().__init__(cfg)

def  checkCfg(self, cfg):
    # preparation pattern for parsing and parameters
    self.firstAppCmd = cfg["runConfig"]["firstAppCmd"]
    self.secondAppCmd = cfg["runConfig"]["secondAppCmd"]
    self.outPattern = r'{spec}:\s*([0-9]*[.][0-9]*)\s*{measure}'.format(spec='Throughput', measure='FPS')
    super().checkCfg(cfg)

def  getPseudoMetric(self, commit, cfg):
    # main Mode method initially resolving two compared
    # commits (for performance goals it is numeric metric, as
    # soon as in some Modes 'metric' has non-numeric value,
    # e.g. filename we got pseudometric), for noncomparative
    # Modes it doesn't return any value, just run procedures
    # for handling single commit.
    commit = commit.replace('"', "")
    commitLogger = getCommitLogger(cfg, commit)
    self.commonLogger.info("New commit: {commit}".format(commit=commit))

    # building and compilation of given version
    handleCommit(commit, cfg)
    fullOutput = ""

    # passing each of the commandlines for execution
    simpleSubstitute(cfg, "actualAppCmd", "$.runConfig.firstAppCmd", "$.appCmd")

    # run first app
    checkOut = fetchAppOutput(cfg, commit)
    commitLogger.info(checkOut)
    foundThroughput = re.search(
        self.outPattern, checkOut, flags=re.MULTILINE
    ).group(1)

    # parsing of output and holding it with full commandline
    # and specific model for the final result
    self.firstThroughput = foundThroughput
    self.firstModel = cfg['appCmd']
    fullOutput = checkOut
    simpleSubstitute(cfg, "actualAppCmd", "$.runConfig.secondAppCmd", "$.appCmd")

    # run second app
    checkOut = fetchAppOutput(cfg, commit)
    commitLogger.info(checkOut)
    foundThroughput = re.search(
        self.outPattern, checkOut, flags=re.MULTILINE
    ).group(1)
    self.secondThroughput = foundThroughput
    self.secondModel = cfg['appCmd']

    # add commit data to commit path
    pc = Mode.CommitPath.PathCommit(
        commit,
        Mode.CommitPath.CommitState.DEFAULT
    )
    self.setOutputInfo(pc)
    self.commitPath.accept(self.traversal, pc)

def  printResult(self):
    # formatting of full output
    for  pathcommit  in  self.commitPath.getList():
        commitInfo = self.getCommitInfo(pathcommit)
        print(commitInfo)
        self.outLogger.info(commitInfo)

def  setOutputInfo(self, pathCommit):
    # populating of commit path
    pathCommit.firstThroughput = self.firstThroughput
    pathCommit.firstModel = self.firstModel
    pathCommit.secondThroughput = self.secondThroughput
    pathCommit.secondModel = self.secondModel

def  getCommitInfo(self, commit):
    return  "{hash}, throughput_1 = {t1}, throughput_2 = {t2}".format(
        hash=commit.cHash,
        t1=commit.firstThroughput,
        t2=commit.secondThroughput)
```
### 配置

假设我们通过两个参数 `cfg["runConfig"]["firstAppCmd"]` 和 `cfg["runConfig"]["secondAppCmd"]` 传入两条启动模型的命令行，即

```
cfg = { 
    "appCmd" : "{actualAppCmd}",
    "appPath": "<path_to_benchmark_app>",
    "gitPath" : "<path_to_git_repository>",
    "buildPath" : "<path_to_build_dir>",
        "runConfig" : {
            "commitList" : {
                "explicitList" : [ "{start}", "{end}" ]
            },
            "mode" : "crossCheck",
            "firstAppCmd" : "./benchmark_app -m model_1.xml --other_params",
            "secondAppCmd" : "./benchmark_app -m model_2.xml --other_params",
            "traversal" : "bruteForce"
        }
}
```
### 输出
```
python3 commit_slider.py -cfg cfg.json

<hash_1>, throughput_1 = 500.0, throughput_2 = 500.0
<hash_2>, throughput_1 = 1000.0, throughput_2 = 1000.0

```
我们摆脱了完整输出，学会了只提供必要信息，但仍然受限于冗长的配置风格 API，并且仍然提供原始输出信息。

## 版本 #3：专用模板
###  API 设计
我们希望为用户提供 2 种方式设置用于比较的命令行。```'appCmd' : ['./benchmark_app --parameters_1', './benchmark_app --parameters_2']``` 用于详细参数，以及简化的 ```'par_1' : 'parameter_1', 'par_2' : 'parameter_2'```，假设替换占位符 ```'appCmd' : 'prefix {actualPar} postfix'```。这种方法的示例可以是 ```'par_1' : 'model_1.xml', 'par_2' : 'model_2.xml' : 'parameter_2', 'appCmd' : './benchmark_app -m {actualPar} -i input.png'```
  
### 自定义模板
```
from  utils.templates.common  import  CommonTemplate

class  BenchmarkCrossCheckTemplate(CommonTemplate):

def  printResult(commitPath, outLogger, getCommitInfo):
    # formatting of output anf post analisys
    print("\n" + "*" * 17 "<Commit slider output>" + "*" * 17 "\nTable of throughputs:" + '\n\t\thr_1\thr_2')
    for  pathcommit  in  commitPath.getList():
        commitInfo = getCommitInfo(pathcommit)
        cHash = pathcommit.cHash
        cHash = cHash[:7]
        commMsg = "{}\t\t{}\t{}".format( cHash,
            pathcommit.firstThroughput,
            pathcommit.secondThroughput)
        print(commMsg)
        outLogger.info(commitInfo)
    cPath = commitPath.getList()
    thr00 = float(cPath[0].firstThroughput)
    thr01 = float(cPath[0].secondThroughput)
    thr10 = float(cPath[1].firstThroughput)
    
    print("{}\nSupposed rootcause is: {}".format(
        "*" * 58,
        'Model'  if  abs(thr00 - thr01) > abs(thr00 - thr10) else  'OV'
        ))
    print("*" * 58" + "\n")
    print("* m1 = {}".format(pathcommit.firstModel))
    print("** m2 = {}".format(pathcommit.secondModel))

def  generateFullConfig(cfg, customCfg):
    tmpJSON = cfg
    # handling input commits
    if  'c'  in  customCfg:
        curCfg = tmpJSON['runConfig']
        interval = customCfg['c'].split("..")
        start, end = interval[0], interval[1]
        curCfg['commitList'] = {'explicitList': [ start, end ] }
    tmpJSON['runConfig'] = curCfg
    # passing base parameters
    CommonTemplate.passParameters([
       'gitPath', 'buildPath', 'appPath'
        ], customCfg, tmpJSON)
    # resolving of API types
    if  isinstance(customCfg['appCmd'], list):
        curCfg = tmpJSON['runConfig']
        curCfg['par_1'] = customCfg['appCmd'][0]
        curCfg['par_2'] = customCfg['appCmd'][1]
        tmpJSON['runConfig'] = curCfg
    else:
        curCfg = tmpJSON['runConfig']
        curCfg['par_1'] = customCfg['appCmd'].format(model=customCfg['modelList'][0])
        curCfg['par_2'] = customCfg['appCmd'].format(model=customCfg['modelList'][1])   
        tmpJSON['runConfig'] = curCfg
    tmpJSON['appCmd'] = "{actualPar}"
```
---
### 配置与 CLI

我们根据用例设计了两个接口

API #1（完整命令行，因此我们可以分别定义参数）
```
"template" : {
    "name" : "bm_cc",
    "gitPath":"{gitPath}",
    "buildPath":"{buildPath}",
    "appPath":"{buildPath}",
    "c": "{start}..{end}",
    "appCmd" : ["{appCmd} -m first_model.xml", "{appCmd} -m second_model.xml"]
}
```
API #2（用于统一参数，仅模型变化）
```
# Here we use default paths.
"template" : {
    "name" : "bm_cc",
    "c": "{start}..{end}",
    "appCmd" : "./benchmark_app {actualPar} -hint throughput -i input.png",
    "par_1" : "first_model.xml",
    "par_2" : "second_model.xml",
}
```
CLI
```
python3 commit_slider -t bm_cc -c <start>..<end> -appCmd './benchmark_app {actualPar} -hint throughput -i input.png' -par_1 first_model.xml -par_2 second_model.xml
```
### 输出
```
python3 commit_slider.py -cfg cfg.json

*****************<Commit slider output>*******************

Table of throughputs:

                     m1      m2

<start_hash>         1000.0  500.0

<end_hash>           1000.0  500.0

**********************************************************

Supposed rootcause is: Model

**********************************************************



*  m1 = ./crossCheckSeparateTemplateBadModel -m good_model.xml

** m2 = ./crossCheckSeparateTemplateBadModel -m bad_model.xml

```

## 接下来我们可以做什么？

- 将遍历与性能交叉验证分离。

- 在 ov 导致退化的情况下，增加二分查找步骤以定位有问题的提交。