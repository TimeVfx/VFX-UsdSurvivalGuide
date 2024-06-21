# Standalone Utilities
Usd ships with a small set of commandline utilities. Below you find a short summary of the most important ones:

[ USD 附带一小组命令行实用程序. 下面是最重要的内容的简短摘要]

## TL;DR - Commandline Utilites In-A-Nutshell [概述]
~~~admonish tip
The following are a must have for anyone using Usd in production:

[ 以下是在生产中使用 USD 的所有人都必须具备的]
- [usdmanager](http://www.usdmanager.org/): Your go-to Usd ascii editing app

    [ [usdmanager](http://www.usdmanager.org/)：首选 USD ASCII 编辑应用程序]
- [usdview](http://www.usdmanager.org/): Your go-to standalone Usd 3d viewing/debugging app

    [ [usdview](http://www.usdmanager.org/)：首选独立 USD 3D 查看/调试应用程序]
    - [TheGrill/Grill](https://github.com/thegrill/grill): A Usd View plugin to view composition arcs

        [ [TheGrill/Grill](https://github.com/thegrill/grill)：USD View 的插件用于查看合成弧]
~~~

## Resources [资源]
- [Commandline Utilities](https://openusd.org/release/toolset.html)
- External Tools:
    - [usdmanager](http://www.usdmanager.org/): A Usd ascii editing app
    - [TheGrill/Grill](https://github.com/thegrill/grill): A Usd View plugin to view composition arcs

## Overview [概述]
The most notable ones are:

[ 最值得注意的是]

- [usdstitch](https://openusd.org/release/toolset.html#usdstitch): Combine multiple files into a single file. This combines the data, with the first file getting the highest strength and so forth. A typical use case is when you are exporting a layer per frame from a DCC and then need to combine it to a single file.

    [ usdstitch：将多个文件合并为一个文件. 这会将数据合并其中第一个文件具有最高的强度，依此类推. 一个典型的使用场景是当你从 DCC 软件按帧导出层时，然后需要将其合并为一个文件]
- [usdstitchclips](https://openusd.org/release/toolset.html#usdstitchclips): Create a file that links to multiple files for a certain time range (for example per frame). Unlike 'usdstitch' this does not combine the data, instead it creates a file (and few sidecar files to increase data lookup performance) that has a mapping what file to load per frame/a certain time range.

    [ usdstitchclips：创建一个链接到多个文件以覆盖特定时间范围（例如每帧）的文件. 与 “usdstitch” 不同它不会合并数据，而是创建一个文件（以及几个辅助文件以提高数据查找性能）该文件具有每帧/特定时间范围应加载哪个文件的映射]
- [usddiff](https://openusd.org/release/toolset.html#usddiff): Opens the diff editor of your choice with the difference between two .usd files. This is super useful to debug for example render .usd files with huge hierarchies, where a visual 3d diff is to expensive or where looking for a specific attribute would take to long, because you don't know where to start.

    [ usddiff：打开差异对比编辑器选择比较两个 .usd 文件之间的差异. 这对于调试具有巨大层级结构的渲染 .usd 文件特别有用，其中三维视觉差异过于昂贵，或者查找特定属性需要花费太多时间，因为你不知道从哪里开始]
- [usdGenSchema](https://openusd.org/release/api/_usd__page__generating_schemas.html): This commandline tool helps us generate our own schemas (Usd speak for classes) without having to code them ourselves in C++. Head over to our [schema](../plugins/schemas.md) section for a hands on example.

    [ usdGenSchema：这个命令行工具可以帮助我们生成自己的 schema（Usd中称为类），而无需我们自己用 C++ 编写. 请前往我们的 [schema](../plugins/schemas.md) 部分以获取动手示例]
- [usdrecord](https://openusd.org/release/toolset.html#usdrecord): A simple hydra to OpenGL proof of concept implementation. If you are interested in the the high level render API, this is good starting point to get a simple overview.

    [ usdrecord：一个简单的 Hydra 到 OpenGL 的概念验证实现. 如果你对高级渲染 API 感兴趣，这是一个很好的起点，可以简单地了解它]
- [usdview](https://openusd.org/release/toolset.html#usdview): A 3d viewer for Usd stages. If you are interested in the high level render API of Usd/Hydra, feel free to dive into the source code, as the Usd View exposes 99% of the functionality of the high level api. It is a great 'example' to learn and a good starting point if you want to integrate it into your apps with a handful of Qt widgets ready to use.

    [ usdview：一个用于 usd stage 的三维查看器. 如果你对 Usd/Hydra 的高级渲染 API 感兴趣，请随意深入研究源代码，因为 Usd View 暴露了高级 API 的99%的功能. 它是一个很好的“示例”供学习之用也是一个很好的起点，如果你想将其集成到你的应用程序中已经准备好了一些 Qt 小部件]

Most of these tools actually are small python scripts, so you can dive in and inspect them!

[ 大多数工具实际上都是小型 Python 脚本，因此您可以深入研究并检查它们！]

There are also a few very useful tools from external vendors:

[ 还有一些来自外部供应商的非常有用的工具]

- [usdmanager](http://www.usdmanager.org/): A Usd text editor from DreamWorksAnimation that allows you to interactively browse your Usd files. This is a must have of anyone using Usd!

    [ usdmanager：DreamWorksAnimation 的 usd 文本编辑器，允许您交互式地浏览 Usd 文件. 这是任何使用 Usd 的人必备的工具！]
- [TheGrill/Grill](https://github.com/thegrill/grill): A super useful Usd View plugin that allows you to visualize the layer stack/composition arcs. A must have when trying to debug complicated value resolution issues. For a visual overview check out this [link](https://grill.readthedocs.io/en/latest/views.html).

    [ TheGrill/Grill：一个超级有用的 Usd View 插件，它允许您可视化图层堆栈/合成弧线. 在尝试调试复杂的值解析问题时，这是必备的工具. 如需视觉概览，请查看此[链接](https://grill.readthedocs.io/en/latest/views.html)]

