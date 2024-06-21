# Debugging & Performance Profiling [调试与性能分析]
Usd ships with extensive debugging and profiling tools. You can inspect the code execution a various levels, which allows you to really pinpoint where the performance issues are.

[ USD 附带了一套调试和性能分析工具. 你可以在不同层面上检查代码执行情况，这有助于你准确定位性能问题所在]

When starting out these to two interfaces are of interest:

[ 在开始时，以下两个接口特别值得关注]

- [Debug Symbols](./debug.md): Enable logging of various API sections to stdout. Especially useful for plugins like asset resolvers or to see how DCCs handle Usd integration.

    [ [Debug Symbols](./debug.md): 将各种 API 日志记录输送到标准输出流. 对于资产解析器等插件或了解 DCC 如何处理 USD 集成特别有用]
- [Performance Profiling](./profiling.md): Usd has a powerful performance profiler, which you can also view with Google Chrome's Trace Viewer.

    [ [Performance Profiling](./profiling.md)：USD 拥有强大的性能分析器，你还可以使用Google Chrome的Trace Viewer来查看分析结果]