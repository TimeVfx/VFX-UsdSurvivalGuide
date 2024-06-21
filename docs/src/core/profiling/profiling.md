# Performance Profiling [性能分析]
For low level profiling Usd ships with the `trace` profiling module.

[ 对于低级分析，USD 附带 trace 分析模块]

This is also what a few DCCs (like Houdini) expose to profile Usd writing/rendering.

[这也是一些 DCC（如 Houdini）向 USD 写入/渲染配置文件公开的内容]

![](./GoogleChromeTraceProfiling.jpg#center)

## TL;DR - Profiling In-A-Nutshell [概述]
- The trace module offers easy to attach Python decorators (`@Trace.TraceMethod/TraceFunction`) that you can wrap your functions with to expose them to the profiler.

    [ Trace 模块提供了易于附加的 Python 装饰器 ( @Trace.TraceMethod/TraceFunction )，您可以用它来包装函数以将它们公开给探查器]
- You can dump the profiling result to .txt or the GoogleChrome tracing format you can open under [`chrome://tracing`](chrome://tracing). Even if you don't attach custom traces, you'll get extensive profiling stats of the underlying Usd API execution.

    [ 您可以将分析结果转储为 .txt 或可在 chrome://tracing 下打开的 GoogleChrome 跟踪格式. 即使您不附加自定义跟踪，您也将获得底层 Usd API 执行的广泛分析统计数据]

## What should I use it for?

[ 我应该用它做什么？]
~~~admonish tip
If you want to benchmark you Usd stage operations, the profiling module offers a fast and easy way to visualize performance.

[ 如果您想对 USD stage 的操作进行基准测试，分析模块提供了一种快速、简单的方法来可视化性能]
~~~

## Resources [资源]
- [Trace Overview](https://openusd.org/dev/api/trace_page_front.html)
- [Trace Details](https://openusd.org/dev/api/trace_page_detail.html)

## Overview [概述]
The trace module is made up of two parts:

[ 跟踪模块由两部分组成]
- `TraceCollector`: A singleton thread-safe recorder of (globale) events.

    [ TraceCollector ：（全局）事件的单例线程安全记录器]
- `TraceReporter`: Turn event data to meaning full views.

    [ TraceReporter ：将事件数据转为有意义的完整视图]

Via the C++ API, you can customize the behavior further, for Python 'only' the global collector is exposed.

[ 通过 C++ API，您可以进一步自定义行为，对于 Python，“仅”公开全局收集器]

### Marking what to trace [标记要追踪的内容]
First you mark what to trace. You can also mark nothing, you'll still have access to all the default profiling:

[ 首先标记要跟踪的内容. 您也可以不标记任何内容，您仍然可以访问所有默认分析]
~~~admonish info title=""
```python
{{#include ../../../../code/core/elements.py:profilingTraceAttach}}
```
~~~

### Trace collector & reporter [踪迹收集者和报告者]
Then you enable the collector during the runtime of what you want to trace and write the result to the disk.

[ 然后，您可以在要跟踪的运行时启用收集器，并将结果写入磁盘]
~~~admonish info title=""
```python
{{#include ../../../../code/core/elements.py:profilingTraceCollect}}
```
~~~

Here is an example (from the Usd docs) of a report to a .txt file. If you have ever rendered with Houdini this will be similar to when you increase the log levels.

[ 以下是 .txt 文件报告的示例（来自 Usd 文档）.如果您曾经使用 Houdini 进行渲染，这将类似于您增加日志级别时的情况]
~~~admonish info title=""
```text
Tree view  ==============
   inclusive    exclusive        
  358.500 ms                    1 samples    Main Thread
    0.701 ms     0.701 ms       8 samples    | SdfPath::_InitWithString
    0.003 ms     0.003 ms       2 samples    | {anonymous}::VtDictionaryToPython::convert
  275.580 ms   275.580 ms       3 samples    | PlugPlugin::_Load
    0.014 ms     0.014 ms       3 samples    | UcGetCurrentUnit
    1.470 ms     0.002 ms       1 samples    | UcIsKnownUnit
    1.467 ms     0.026 ms       1 samples    |   Uc::_InitUnitData [initialization]
    1.442 ms     1.442 ms       1 samples    |   | Uc_Engine::GetValue
    0.750 ms     0.000 ms       1 samples    | UcGetValue
    0.750 ms     0.750 ms       1 samples    |   Uc_Engine::GetValue
    9.141 ms     0.053 ms       1 samples    | PrCreatePathResolverForUnit
    0.002 ms     0.002 ms       6 samples    |   UcIsKnownUnit
```
~~~

Here is an example of a report to a Google Chrome trace .json file opened at [`chrome://tracing`](chrome://tracing) in Google Chrome with a custom python trace marked scope.

[ 以下是在 Google Chrome 中打开 chrome://tracing 的 Google Chrome 跟踪 .json 文件的报告示例，其中包含自定义 python 跟踪标记范围]

![](./GoogleChromePythonScopeTraceProfiling.jpg#center)


### Measuring time deltas [测量时间增量]
Usd ships with a simpel stop watch class that offers high precision time deltas.

[ USD 附带了一个简单的秒表类，可提供高精度时间增量]
~~~admonish info title=""
```python
{{#include ../../../../code/core/elements.py:profilingStopWatch}}
```
~~~

### Stage Stats [Stage 统计]
You can also gather stage stats, this is mainly used to expose statistics for UIs.

[ 您还可以收集 stage 统计信息，这主要用于公开 UI 的统计信息]
~~~admonish info title=""
```python
from pxr import UsdUtils
print(UsdUtils.ComputeUsdStageStats(stage))
# Returns (On stage with a single cube):
{
 'assetCount': 0, 
 'instancedModelCount': 0,
 'modelCount': 0,
 'primary': {'primCounts': {'activePrimCount': 2,
                            'inactivePrimCount': 0,
                            'instanceCount': 0, 
                            'pureOverCount': 0,
                            'totalPrimCount': 2},
 'primCountsByType': {'Cube': 1}}, 
 'prototypeCount': 0,
 'totalInstanceCount': 0,
 'totalPrimCount': 2,
 'usedLayerCount': 10
}
```
~~~