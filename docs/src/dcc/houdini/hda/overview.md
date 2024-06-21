# Creating efficient LOPs HDAs [创建高效的 LOP HDA]
As with any other network in Houdini, we can also create HDAs in LOPs.

[ 与 Houdini 中的任何其他 network 一样，我们也可以在 LOP 中创建 HDA]

This page will focus on the most important performance related aspects of LOP HDAs, we will be referencing some of the points mentioned in the [performance optimizations](./performance/overview.md) section with a more practical view.

[ 本页将重点介绍 LOP HDA 最重要的性能相关方面，我们将以更实际的角度参考性能优化部分中提到的一些要点]

You can find all the .hip files of our shown examples in our [USD Survival Guide - GitHub Repo](https://github.com/LucaScheller/VFX-UsdSurvivalGuide/tree/main/files/dcc/houdini).

[ 您可以在 [USD Survival Guide - GitHub Repo](https://github.com/LucaScheller/VFX-UsdSurvivalGuide/tree/main/files/dcc/houdini) 中找到我们所示示例的所有 .hip 文件]

# Table of Contents [目录]
1. [Overview](#overview)
1. [HDA Template Setup](#hdaTemplate)
    1. [Order of operations](#hdaOrderOfOperations)
    1. [Dealing with time dependencies](#hdaTimeDependencies)
    1. [Layer Size/Count](#hdaLayerSizeCount)
1. [Composition](#hdaComposition)

## Overview [概述]<a name="overview"></a>
When building LOP HDAs with USD, the big question is:

[ 当使用 USD 构建 LOP HDA 时，最大的问题是]

~~~admonish question
What should we do in Python and would should we do with HDAs/nodes that Houdini ships with?

[ 我们应该在 Python 中做什么？我们应该使用 Houdini 附带的 HDAs/节点做什么？]
~~~

The answer depends on your knowledge level of USD and the pipeline resources you have.

[ 答案取决于您对 USD 的了解水平以及您拥有的流程资源]

If you want to go the "expert" route, this is the minimal working set of nodes you'll be using:

[ 如果您想走“专家”路线，这是您将使用的最小工作节点集]

![HDA Minimal Working Set Nodes](hdaMinimalNodeWorkingSet.jpg)

Technically we won't be needing more, because everything else can be done in Python. 
(There are also the standard control flow related nodes, we skipped those in the above image).
It is also faster to run operations in Python, because we can batch edits, where as with nodes, Houdini
has to evaluate the node tree.

[ 从技术上讲，我们不需要更多的功能，因为其他所有操作都可以使用 Python 来完成（还有一些标准的控制流相关节点，我们在上面的图片中跳过了这些节点）在Python中运行操作也更快，因为我们可以批量编辑，而使用节点时，Houdini需要评估节点树]

So does this mean we shouldn't use the Houdini nodes at all? Definitely not! Houdini's LOPs tool set offers a lot of useful nodes, especially when your are prototyping, we definitely recommend using these first. A common workflow for developers is therefore:

[ 那么这是否意味着我们根本不应该使用 Houdini 节点？当然不！ Houdini 的 LOPs 工具集提供了许多有用的节点，特别是当您进行原型设计时，我们绝对建议首先使用这些节点. 因此，开发人员的常见工作流程是]

~~~admonish tip title=""
Build everything with LOP HDAs first, then rebuild it, where possible, with Python LOPs when we notice performance hits.

[ 首先使用 LOP HDA 构建所有内容，然后在我们注意到性能下降时使用 Python LOP 重建它（如果可能）]
~~~

We'll always use the "Material Library" and "SOP Import" nodes as these pull data from non LOP networks.

[ 我们将经常使用 “Material Library” 和 “SOP Import” 节点提取非 LOP networks 的数据]

There are also a lot of UI related nodes, which are simply awesome. We won't be putting these in our HDAs, but they should be used by artists to complement our workflows.

[ 还有很多UI相关的节点，简直棒极了. 我们不会将这些放入 HDA 中，但艺术家应该使用它们来补充我们的工作流程]

![Houdini UI Custom Panel Nodes](houdiniUICustomPanelNodes.jpg)

## HDA Template Setup [HDA 模板设置]<a name="hdaTemplate"></a>
Let's take a look at how we can typically structure HDAs:

[ 让我们看一下通常如何构建 HDAs]

<video width="100%" height="100%" controls autoplay muted loop>
  <source src="./hdaTemplateStructure.mp4" type="video/mp4" alt="Houdini Hda Template Structure">
</video>

For parms we can make use of Houdini's internal loputils under the following path:

[ 对于 parms，我们可以在以下路径下使用 Houdini 的内部 loputils]
~~~admonish tip title=""
$HFS/houdini/python3.9libs/loputils.py
~~~
It is not an official API module, so use it with care, it may be subject to change.

[ 它不是官方 API 模块，因此请小心使用，它可能会发生变化]

You can simply import via `import loputils`. It is a good point of reference for UI related functions, for example action buttons on parms use it at lot.

[ 您可以简单地通过 import loputils 导入. 对于 UI 相关功能来说，这是一个很好的参考点，例如 parms 上的操作按钮就经常使用它]

Here you can find the [loputils.py - Sphinx Docs](https://ikrima.github.io/houdini_additional_python_docs/loputils.html) online.

[ 在这里您可以找到 [loputils.py - Sphinx Docs](https://ikrima.github.io/houdini_additional_python_docs/loputils.html)]

It gives us common lop related helpers, like selectors etc.

[ 它为我们提供了常见的 lop 相关助手，例如选择器等]

For the structure we recommend:

[ 对于结构我们推荐]
1. Create a new layer

    [ 创建一个新层]
2. Perform your edits (best-case only a single Python node) or merge in content from other inputs.

    [ 执行编辑（最好情况下只有单个 Python 节点）或合并来自其他输入的内容]
3. Create a new layer

    [ 创建一个新层]

Why should we spawn the new layers? See our [layer size and content](#hdaLayerSizeCount) section below for the answer.

[ 我们为什么要生成新层？请参阅 [layer size and content](#hdaLayerSizeCount) 以获得答案]

When merging other layers via the merge node we recommend first flattening your input layers and then using the "Separate Layers" mode. That way we also avoid the layer size problem and keep our network fast.

[ 通过合并节点来合并其他图层时，我们建议首先辗平输入图层，然后使用“单独图层”模式. 这样我们还可以避免层大小问题并保持网络快速]

### Order of Operations [操作顺序]<a name="hdaOrderOfOperations"></a>
The same as within SOPs, we have to pay attention to how we build our node network to have it perform well.

[ 与 SOP 中一样，我们必须注意如何构建节点网络才能使其良好运行]

Let's look at a wrong example and how to fix it:

[ 让我们看一个错误的示例以及如何修复它]

<video width="100%" height="100%" controls autoplay muted loop>
  <source src="./hdaOrderOfOperationSlow.mp4" type="video/mp4" alt="Houdini Order of Operations slow">
</video>

Here is a more optimized result:

[ 这是一个更优化的结果]

<video width="100%" height="100%" controls autoplay muted loop>
  <source src="./hdaOrderOfOperationOptimized.mp4" type="video/mp4" alt="Houdini Order of Operations fast">
</video>

The name of the game is isolating your time dependencies. Now the above is often different how a production setup might look, but the important part is that we try to isolate each individual component that can stand by itself to a separate node stream before combining it into the scene via a merge node.

[ 这个游戏的名字是“隔离时间依赖”. 现在，上述内容通常与生产环境的设置不同，但重要的是，我们尝试将每个可以独立存在的组件隔离到单独的节点流中，然后再通过合并节点将其组合到场景中]

In our HDAs we can often build a separate network and then merge it into the main node stream, that way not everything has to re-cook, only the merge, if there is an upstream time dependency.

[ 在我们的 HDA 中，我们通常可以构建一个单独的网络，然后将其合并到主节点流中，这样，如果存在上游时间依赖性，则不必重新烹饪所有内容，只需合并即可]

Now you may have noticed that in the first video we also merged a node stream  with itself.

[ 现在您可能已经注意到，在第一个视频中，我们还将节点流与其自身合并]

If Ghostbusters taught us one thing:

[ 如《捉鬼敢死队》教会了我们一件事]

~~~admonish danger title="Important | Merging node streams"
Don't cross the streams!*

[ 不要越过溪流！]

*(unless you use a layer break).

[ 除非您使用分层]
~~~

The solution is simple, add a layer break (and make sure that your have the "Strip Layers Above Layer Breaks" toggle turned on). We have these "diamond" shaped networks quite a lot, so make sure you always layer break them correctly. For artists it can be convenient to build a HDA that does this for them.

[ 解决方案很简单，添加一个图层中断（并确保您已打开 “Strip Layers Above Layer Breaks” ）.我们有很多这样的“钻石”形网络，因此请确保始终正确地对它们进行 layer break .对于艺术家来说，构建一个 HDA 来为他们执行此操作会很方便]

<video width="100%" height="100%" controls autoplay muted loop>
  <source src="./hdaOrderOfOperationsLayerBreak.mp4" type="video/mp4" alt="Houdini Merge Order">
</video>

~~~admonish danger title="Important | Merging a layer stack with itself"
USD is also very "forgiving", if we create content in a layer, that is already there from another layer (or layer stack via a reference or payload). The result is the same (due to composition), but the layer stack is "doubled". This is particularly risky and can lead to strange crashes, e.g. when we start to duplicate references on the same prim.

[ USD 也非常“宽容”，如果我们在一个层中创建内容，那么该内容已经来自另一个层（或通过 reference/payload 的层堆栈）.结果是相同的（对于合成来说），但层堆栈“加倍”. 这是特别危险的，可能会导致奇怪的崩溃，例如当我们开始在同一个 prim 上重复引用时]

Now if we do this in the same layer stack, everything is fine as our [list-editable ops](../../../core/composition/fundamentals.md#compositionFundamentalsListEditableOps) are merged in the active layer stack. So you'll only have the reference being loaded once, even though it occurs twice in the same layer stack.

[ 现在，如果我们在同一层堆栈中执行此操作，则一切都很好，因为我们的可编辑列表操作已合并到活动层堆栈中。因此，即使引用在同一层堆栈中出现两次，您也只会加载一次引用]

Now if we load a reference that is itself referenced, it is another story due to [encapsulation](../../../core/composition/fundamentals.md#compositionFundamentalsEncapsulation). We now have the layer twice, which gives us the same output visually/hierarchy wise, but our layers that are used by composition now has really doubled.

[ 现在，如果我们加载一个本身被引用的引用，由于 [封装](../../../core/composition/fundamentals.md#compositionFundamentalsEncapsulation)，这是另一个故事. 现在，我们有两个层，这在视觉/层级结构方面为我们得到了相同的输出，但是现在合成使用的图层确实增加了一倍]

Check out this video for comparison:

[ 看看这个视频进行比较]

<video width="100%" height="100%" controls autoplay muted loop>
  <source src="./hdaOrderOfOperationsLayerStackDuplication.mp4" type="video/mp4" alt="Houdini Layer Stack Duplication">
</video>

~~~


What also is important, is the merge order. Now in SOPs we can just merge in any order, and our result is still the same (except the point order).

[ 合并顺序也是同样重要. 现在在 SOP 中，我们可以按任何顺序合并，并且我们的结果仍然相同（除了点顺序）]

For LOPs this is different: The merge order is the sublayer order and therefore effects composition. As you can see in the video below, if we have an attribute that is the same in both layers (in this case the transform matrix), the order matters.

[ 对于 LOPs 这是不同的：合并顺序是子层顺序，因此会影响合成. 正如您在下面的视频中看到的，如果我们在两个层中都有相同的属性（在本例中为变换矩阵），则顺序很重要]

<video width="100%" height="100%" controls autoplay muted loop>
  <source src="./hdaOrderOfOperationsMergeOrder.mp4" type="video/mp4" alt="Houdini Merge Order">
</video>

### Dealing with time dependencies [处理时间依赖性]<a name="hdaTimeDependencies"></a>
As with SOPs, we should also follow the design principle of keeping everything non-time dependent where possible.

[ 与 SOP 一样，我们还应该遵循尽可能保持一切不依赖时间的设计原则]

When we have time dependencies, we should always isolate them, cache them and then merge them into our "main" node stream.

[ 当我们有时间依赖性时，我们应该始终隔离它们，缓存它们，然后将它们合并到我们的“主”节点流中]

~~~admonish important title="Pro Tip | Writing Time Samples Via Python"
When writing Python code, we can write time samples for the full range too. See our [animation section](../../../core/elements/animation.md) for more info. We recommend using the lower level API, as it is a lot faster when writing a large time sample count. A typical example would be to write the per image output file or texture sequences via Python, as this is highly performant.

[ 在编写Python代码时，我们也可以编写全范围的时间样本. 请参阅[动画](../../../core/elements/animation.md)部分了解更多信息. 我们建议使用较低级别的 API，因为在写入大量时间样本时它会快得多. 一个典型的例子是通过 Python 编写每个图像输出文件或纹理序列，因为这是高性能的]
~~~

The very cool thing with USD is that anything that comes from a cached file does not cause a Houdini time dependency, because the time samples are stored in the file/layer itself. This is very different to how SOPs works and can be confusing in the beginning.

[ USD 的一个非常酷的事情是，来自缓存文件的任何内容都不会导致 Houdini 时间依赖性，因为时间样本存储在文件/层本身中. 这与 SOP 的工作方式非常不同，一开始可能会令人困惑]

Essentially the goal with LOPs is to have no time dependency (at least when not loading live caches).

[ 本质上，LOP 的目标是没有时间依赖性（至少在不加载实时缓存时）]

Starting with H19.5 most LOP nodes can also whole frame range cache their edits. This does mean that a node can cook longer for very long frame ranges, but overall your network will not have a time dependency, which means when writing your node network to disk (for example for rendering), we only have to write a single frame and still have all the animation data. How cool is that!

[ 从 H19.5 开始，大多数 LOP 节点还可以在整个帧范围内缓存其编辑. 这确实意味着节点可以在很长的帧范围内烹饪更长的时间，但总的来说，您的网络不会有时间依赖性，这意味着将节点网络写入磁盘时（例如用于渲染），我们只需写入单个帧并且仍然拥有所有动画数据. 多么酷啊！]

<video width="100%" height="100%" controls autoplay muted loop>
  <source src="./hdaTimeDependencyPerNode.mp4" type="video/mp4" alt="Houdini Time Sample Per Node">
</video>

If a node doesn't have that option, we can almost always isolate that part of the network and pre cache it, that way we have the same effect but for a group of nodes.

[ 如果节点没有该选项，我们几乎总是可以隔离网络的该部分并预先缓存它，这样我们就可以达到相同的效果，但对于一组节点而言]

<video width="100%" height="100%" controls autoplay muted loop>
  <source src="./hdaTimeDependencyCache.mp4" type="video/mp4" alt="Houdini Time Sample Per Node">
</video>

~~~admonish danger title="Important | Previewing Xform/Deformation Motionblur"
If we want to preview xform/deformation motionblur that is not based on the `velocities`/`accelerations` attribute, then we have to pre-cache the time samples in interactive sessions. This is as simple as adding a LOPs cache node as shown above.

[ 如果我们想要预览不基于 velocities / accelerations 属性的 xform/deformation 运动模糊，那么我们必须在交互式会话中预先缓存时间样本. 这就像添加一个 LOP cache 节点一样简单，如上所示]
~~~

Also check out our [Tips & Tricks section](../faq/overview.md#timeSampleValueMightBeTimeVarying) to see how we can query if an attribute is time sampled or only has a default value. This is a bit different in Houdini than bare-bone USD, because we often only have a single time sample for in session generated data.

[ 另请查看我们的 [提示和技巧](../faq/overview.md#timeSampleValueMightBeTimeVarying)部分，了解如何查询属性是否经过时间采样或仅具有默认值. 这在 Houdini 中与简单的 USD 有点不同，因为我们通常只有会话中生成的数据的单个时间样本]

### Layer Size/Count <a name="hdaLayerSizeCount"></a>
As mentioned in the [overview](#overview), layer content size can become an issue.

[ 正如[概述](#overview)中提到的，图层内容大小可能成为一个问题]

We therefore recommend starting of with a new layer and ending with a new layer. That way our HDA starts of fast, can create any amount of data and not affect downstream nodes.

[ 因此，我们建议以新层开始并以新层结束. 这样我们的 HDA 启动速度很快，可以创建任意数量的数据并且不会影响下游节点]

For a full explanation see our [performance section](../performance/overview.md).

[ 如需完整说明，请参阅[性能](../performance/overview.md)部分]

## Composition [合成]<a name="hdaComposition"></a>
We strongly recommend reading up on our [composition section](../../../core/composition/overview.md) before getting started in LOPs.

[ 我们强烈建议您在开始 LOP 之前先阅读我们的[合成](../../../core/composition/overview.md)]

When setting up composition arcs in Houdini, we can either do it via nodes or code. We recommend first doing everything via nodes and then refactoring it to be code based, as it is faster. Our custom HDAs are usually the ones that bring in data, as this is the core task of every pipeline.

[ 在Houdini中设置合成弧时，我们可以通过节点或代码来完成. 我们建议首先通过节点完成所有操作，然后将其重构为代码，因为它更快. 我们的自定义 HDA 通常是引入数据的 HDA，因为这是每个流程的核心任务]

In our [Tips & Tricks section](../faq/overview.md), we have provided these common examples:

[ 在我们的[提示与技巧](../faq/overview.md)部分中，我们提供了以下常见示例]

- [Extracting payloads and references from an existing layer stack with anonymous layers](../faq/overview.md#compositionReferencePayloadLayerStack)

    [ 使用匿名层从现有层堆栈中提取payloads and references](../faq/overview.md#compositionReferencePayloadLayerStack)
- [Efficiently re-writing existing hierarchies as variants](../faq/overview.md#compositionArcVariantReauthor)

    [ 有效地将现有层次结构重写为变体](../faq/overview.md#compositionArcVariantReauthor)
- [Adding overrides via inherits](../faq/overview.md#compositionArcInherit)

    [ 通过继承添加覆盖](../faq/overview.md#compositionArcInherit)

These provide entry points of how you can post-process data do you needs, after you have SOP imported it.

[ 这些提供了在导入 SOP 后如何对所需数据进行后处理的入口点]