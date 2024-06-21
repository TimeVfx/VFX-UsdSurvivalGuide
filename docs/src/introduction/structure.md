
# Structure [基础结构]
On this page will talk about how this guide is structured and what the best approach to reading it is.

[ 本页将介绍本指南的结构以及阅读本指南的最佳方法]
# Table of Contents [目录]
1. [Structure](#structure)
1. [Learning Path - 学习路径](#learningPath)
1. [How To Run Our Code Examples - 如何运行我们的示例代码](#code)


## Structure <a name="structure"></a>
Most of our sections follow this simple template:

[ 我们的大部分的板块都遵循这个简单的模板：]
- **Table of Contents**: Here we show the structure of the individual page, so we can jump to what we are interested in.

    [ 目录：这里我们显示单个页面的结构，可以跳转到感兴趣的内容阅读]
- **TL;DR (Too Long; Didn't Read) - In-A-Nutshell**: Here we give a short summary of the page, the most important stuff without all the details.

    [ TL;DR（不需要详细阅读）- 简单来说：这里我们给出了页面的简短摘要，重要内容的阐述，不包含详细的解释]
- **What should I use it for?**: Here we explain what relevance the page has in our day to day work with USD.

    [ 我应该用它做什么？：在这里我们解释在日常工作中如何去使用 USD ]
- **Resources**: Here we provide external supplementary reading material, often the USD API docs or USD glossary.

    [ 资源：这里我们提供补充阅读材料，通常是 USD API 文档或 USD 术语表]
- **Overview**: Here we cover the individual topic in broad strokes, so you get the idea of what it is about.

    [ 概述：在这里我们概括的对章节进行介绍了，以便您了解其内容]

This guide uses Houdini as its "backbone" for exploring concepts, as it is one of the easiest ways to get started with USD.

[ 本指南使用 Houdini 作为重心来展开，因为它是开始使用 USD 最简单的方法之一]

You can grab free-for-private use copy of Houdini via the [SideFX website](https://www.sidefx.com/). SideFX is the software development company behind the Houdini.

[ 您可以通过 [SideFX](https://www.sidefx.com/) 网站获取 Houdini 的个人免费版本, SideFX 是 Houdini 背后的软件开发公司]

Almost all demos we show are from within Houdini, although you can also save the output of all our code snippets to a .usd file and view it in [USD view](../core/elements/standalone_utilities.md) or [USD manager](http://www.usdmanager.org/) by calling `stage.Export("/file/path.usd")`/`layer.Export("/file/path.usd")`

[ 我们几乎所有演示都是在 Houdini 软件中进行的, 当然您也可以将所有的代码片段保存输出成 .usd 文件，然后通过 [USD view](../core/elements/standalone_utilities.md) 或者 [USD manager](http://www.usdmanager.org/) 的命令行 `stage.Export("/file/path.usd")`/`layer.Export("/file/path.usd")`来查看]

You can find all of our example files in our [Usd Survival Guide - GitHub Repository](https://github.com/LucaScheller/VFX-UsdSurvivalGuide/tree/main/files) as well in our supplementary [Usd Asset Resolver - GitHub Repository](https://lucascheller.github.io/VFX-UsdAssetResolver/). Among these files are Houdini .hip scenes, Python snippets and a bit of C++/CMake code.

[ 你可以在我们的GitHub仓库 [Usd Survival Guide](https://github.com/LucaScheller/VFX-UsdSurvivalGuide/tree/main/files) 和 [Usd Asset Resolver](https://lucascheller.github.io/VFX-UsdAssetResolver/) 中找到所有的示例文件. 包括不限于 Houdini 的 .hip 工程、Python 的代码片段和一些 C++/CMake 的代码]

We also indicate important to know tips with stylized blocks, these come in the form of:

[ 我们用标识块来表示重要的知识点，这些提示的样式如下：]

~~~admonish info title="Info | Useful information!"
Information blocks give you "good-to-know" information.

[ 信息块为您提供“须知”信息]
~~~

~~~admonish info title="Pro Tip | Here is an advanced tip!"
We often provide "Pro Tips" that give you pointers how to best approach advanced topics.

[ 我们经常提供“专业提示”，为您提供最佳方法以应对高级话题]
~~~

~~~admonish danger title="Danger | Here comes trouble!"
Danger blocks warn you about common pitfalls or short comings of USD and how to best workaround them.

[ 危险信息块会警告您有关USD的常见陷阱或不足，以及最佳的解决方法]
~~~

~~~admonish tip title="Collapsible Block | Click me to show my content!" collapsible=true
For longer code snippets, we often collapse the code block to maintain site readability.

[ 对于较长的代码片段，我们通常会折叠代码块以保持网站的可读性]

```python
print("Hello world!")
```
~~~

## Learning Path - 学习线路 <a name="learningPath"></a>
We recommend working through the guide from start to finish in chronological order. While we can do it in a random order, especially our [Basic Building Blocks of Usd](../core/elements/overview.md) and [Composition](../core/composition/overview.md) build on each other and should therefore be done in order.

[ 我们建议按顺序从头到尾阅读本指南, 虽然我们可以跳着章节阅读, 但特别是[USD基本架构](../core/elements/overview.md)和[组合](../core/composition/overview.md)部分是相互依赖的，因此建议按顺序阅读]

To give you are fair warning though, we do deep dive a bit in the beginning, so just make sure you get the gist of it and then come back later when you feel like you need a refresher or deep dive on a specific feature.

[ 提前打个招呼，我们会在开始时进行一些深入研究，所以请确保你已经掌握了一些基本的知识点，然后当你觉得需要回顾或深入了解某个特定功能的时候再回过头来阅读]

## How To Run Our Code Examples - 如何运行我们的示例代码 <a name="code"></a>
We also have code blocks, where if you hover other them, you can copy the content to you clipboard by pressing the copy icon on the right.
Most of our code examples are "containered", meaning they can run by themselves.

[ 我们的码块当您将鼠标悬停在上面的时候，您可以通过按右侧的复制图标将内容复制到剪贴板. 我们的大多数代码示例都是“完整”的，就是说它们可以独立运行]

This does come with a bit of the same boiler plate code per example. The big benefit though is that we can just copy and run them and don't have to initialize our environment.

[ 每个示例会带有一些相同的模板代码. 这样做的好处是我们可以复制然后运行，就不必做一些前期的准备工作了]

Most snippets create in memory stages or layers. If we want to use the snippets in a Houdini Python LOP, we have to replace the stage/layer access as follows:

[ 大多数的代码片段都是在内存中 stage 或 layer 中创建的. 如果我们想在 Houdini Python LOP 中使用，就必须转换 stage/layer 操作方式 如下所示：]

~~~admonish danger title="Danger | Houdini Python LOP Stage/Layer access"
In Houdini we can't call `hou.pwd().editableStage()` and `hou.pwd().editableLayer()` in the same Python LOP node.
Therefore, when running our high vs low level API examples, make sure you are using two different Python LOP nodes.

[ 在Houdini中，我们不能在同一个Python LOP节点中同时调用 hou.pwd().editableStage() 和 hou.pwd().editableLayer()  因此，在运行我们的高级与低级API示例时，请确保您使用的是两个不同的Python LOP节点]
~~~

~~~admonish tip title=""
```python
from pxr import Sdf, Usd
## Stages
# Native USD
stage = Usd.Stage.CreateInMemory()
# Houdini Python LOP
stage = hou.pwd().editableStage()
## Layers
# Native USD
layer = Sdf.Layer.CreateAnonymous()
# Houdini Python LOP
layer = hou.pwd().editableLayer()
```
~~~


