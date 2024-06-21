# Prim Cache Population (PCP) - Composition Cache
The **Prim Cache Population** module is the backend of what makes USD composition work. You could call it the `composition core`, that builds the prim index.
The prim index is like an ordered list per prim, that defines where values can be loaded from. For example when we call `Usd.Prim.GetPrimStack`, the `pcp` gives us a list of value sources as an ordered list, with the "winning" value source as the first entry.

[ Prim Cache Population 模块是 USD 合成工作的后端，你可以称它为 composition core 它负责构建 prim index . prim index 就像每个 prim 的有序列表，定义了可以从哪里加载值. 例如当我们调用 Usd.Prim.GetPrimStack 时，pcp 会给我们一个值源的有序列表，其中“获胜”的值源作为第一个条目]

When USD opens a stage, it builds the prim index, so that it knows for each prim/property where to pull data from.
This is then cached, and super fast to access.

[ 当 USD 打开一个 stage 时，它会构建 prim index ，以便知道每个 prim/property 的数据应该从哪里提取. 然后这些数据会被缓存起来，以便快速访问]

When clients (like hydra delegates or C++/Python attribute queries) request data, only then the data is loaded.
This way hierarchies can load blazingly fast, without actually loading the heavy attribute data.

[ 当客户端（如 Hydra 委托或 C++/Python 属性查询）请求数据时才会加载数据. 这样层次结构可以极快地加载，而无需实际加载繁重的属性数据]

~~~admonish tip
To summarize: Composition (the process of calculating the value sources) is cached, value resolution is not, to allow random access data loading.

[ 总结一下：组合（计算值源的过程）被缓存，值解析不被缓存，以允许随机访问数据加载]
~~~

For a detailed explanation, checkout the [Value Resolution](https://openusd.org/release/glossary.html#usdglossary-valueresolution) docs page.

[ 有关详细说明，请查看[Value Resolution](https://openusd.org/release/glossary.html#usdglossary-valueresolution)]

# Table of Contents [目录]
1. [Prim Cache Population (PCP) In-A-Nutshell](#summary)
1. [What should I use it for?](#usage)
1. [Resources](#resources)
1. [Overview](#overview)
1. [Inspecting Composition](#pcpInspect)
    1. [Prim/Property Stack](#pcpPrimPropertyStack)
    1. [Prim Index](#pcpPrimPropertyIndex)
    1. [Prim Composition Query](#pcpPrimCompositionQuery)

## TL;DR - <Topic> In-A-Nutshell [概述]<a name="summary"></a>
- The **Prim Cache Population** module in USD computes and caches the composition (how different layers are combined) by building an index of value sources per prim called **prim index**.

    [ USD 中的 Prim Cache Population 模块通过为每个 prim 构建一个称为 prim index 的值源索引来计算和缓存组合（不同层的组合方式）]
- This process of calculating the value sources is cached, value resolution is not, to allow random access to data. This makes accessing hierarchies super fast, and allows attribute data to be streamed in only when needed. This is also possible due to USD's [binary crate format](https://openusd.org/release/glossary.html#crate-file-format), which allows sparse "read only what you need" access from USD files.

    [ 这个计算值源的过程是被缓存的，而值解析则不是，以允许随机访问数据. 这使得访问层次结构变得非常快，并且仅在需要时才允许属性数据流入. 这也是由于USD的二进制crate格式，允许从USD文件中进行稀疏的“只读所需”访问]
- The **Prim Cache Population (Pcp)** module is exposed via two ways:

    [ Prim Cache Population（Pcp）模块通过两种方式公开]
    - **High Level API**: Via the `Usd.PrimCompositionQuery` class.

        [ 高级 API：通过 Usd.PrimCompositionQuery 类]
    - **Low Level API**: Via the`Usd.Prim.GetPrimIndex`/`Usd.Prim.ComputeExpandedPrimIndex` methods.

        [ 低级 API：通过 Usd.Prim.GetPrimIndex / Usd.Prim.ComputeExpandedPrimIndex 方法]
- Notice how both ways are still accessed via the high level API, as the low level `Sdf` API is not aware of composition.

    [ 请注意这两种方式仍然通过高级 API 进行访问，因为低级 Sdf API 不知道组合]

## What should I use it for? <a name="usage"></a>

[ 我应该用它做什么？]

~~~admonish tip
We'll be using the prim cache population module for inspecting composition.
This is more of a deep dive topic, but you may at some point run into this in production.

[ 我们将使用 prim 缓存填充模块来检查组成. 这是一个更深入的主题，但您可能在生产中的某个时候遇到这个问题]

An example scenario might be, that when we want to author a new composition arc, we first need to check if there are existing strong arcs, than the arc we intend on authoring.
For example if a composition query detects a variant, we must also author at least a variant or a higher composition arc in order for our edits to come through.

[ 一个示例场景可能是，当我们想要创作新的合成弧时，我们首先需要检查是否存在现有的强弧，而不是我们打算创作的弧. 例如如果合成查询检测到变体，我们还必须至少创作一个变体或更高的合成弧，以便我们的编辑能够通过]
~~~

## Resources [资源]<a name="resources"></a>
- [PrimCache Population (Composition)](https://openusd.org/dev/api/pcp_page_front.html)
- [Prim Index](https://openusd.org/release/glossary.html#usdglossary-index)
- [Pcp.PrimIndex](https://openusd.org/release/api/class_pcp_prim_index.html)
- [Pcp.Cache](https://openusd.org/dev/api/class_pcp_cache.html)
- [Usd.CompositionArc](https://openusd.org/dev/api/class_usd_prim_composition_query_arc.html)
- [Usd.CompositionArcQuery](https://openusd.org/dev/api/class_usd_prim_composition_query.html)
- [Value Resolution](https://openusd.org/release/glossary.html#usdglossary-valueresolution)
- [USD binary crate file format](https://openusd.org/release/glossary.html#crate-file-format)

## Overview [概述]<a name="overview"></a>
This page currently focuses on the practical usage of the `Pcp` module, it doesn't aim to explain how the composition engine works under the hood. (As the author(s) of this guide also don't know the details 😉, if you know more in-depth knowledge, please feel free to share!)

[ 本章节当前重点关注 Pcp 模块的实际用法，其目的不是解释组合引擎在幕后如何工作.（由于本指南的作者也不知道详细内容😉，如果您知道更深入的知识，请随时分享！）]

There is a really cool plugin for the [UsdView](../elements/standalone_utilities.md) by [chrizzftd](https://github.com/chrizzFTD) called [The Grill](https://grill.readthedocs.io/en/latest/views.html), that renders out the dot graph representation interactively based on the selected prim.

[ [chrizzftd](https://github.com/chrizzFTD) 的 [UsdView](../elements/standalone_utilities.md) 有一个非常酷的插件，称为  [The Grill](https://grill.readthedocs.io/en/latest/views.html)，它根据所选的 prim 交互式地呈现点图表示]

In the examples below, we'll look at how to do this ourselves via Python.

[ 在下面的示例中，我们将了解如何通过 Python 自己完成此操作]

## Inspecting Composition <a name="pcpInspect"></a>
To query data about composition, we have to go through the high level Usd API first, as the `Sdf` low level API is not aware of composition related data.
The high level Usd API then queries into the low level Pcp (Prim cache population) API, which tracks all composition related data and builds a value source index called **prim index**.

[ 为了查询关于合成的数据，我们首先需要经过高级别的 Usd API，因为低级别的 Sdf API 并不了解与合成相关的数据. 然后, 高级别的 Usd API 查询低级别的 Pcp（Prim cache population）API，该 API 跟踪所有与组合相关的数据，并构建一个称为 prim index 的值源索引]

The prim stack in simple terms: A stack of layers per prim (and therefore also properties) that knows about all the value sources (layers) a value can come from. Once a value is requested, the highest layer in the stack wins and returns the value for attributes. For metadata and relationships the value resolution can consult multiple layers, depending on how it was authored (see [list editable ops](../composition/listeditableops.md) as an example for a multiple layer averaged value).

[ 简单来说，prim stack 就是每个 prim（因此也是属性）的层堆栈，它了解值可能来自的所有值源（层）. 当请求一个值时，堆栈中的最高层会胜出并为返回属性值. 对于metadata and relationships，值解析可以查阅多个层，具体取决于它们是如何编写的（请参阅 [list editable ops](../composition/listeditableops.md) 作为多层平均值的示例）]

### Prim/Property Stack <a name="pcpPrimPropertyStack"></a>
Let's first have a look at the prim and property stacks with a simple stage with a cubes that has written values in two different layers.
These return us all value sources for a prim or attribute.

[ 我们首先看一个简单的场景，该场景包含一个立方体，并且在两个不同的层中写入了值. 我们将查看这个场景的 prim 和 property 堆栈，它们会返回基元或属性的所有值源]

~~~admonish tip title=""
```python
{{#include ../../../../code/core/composition.py:pcpPrimPropertyStack}}
```
~~~

In Houdini/USD view we can also view these stacks in the UI.

[ 在Houdini/USD 视图中，我们还可以在 UI 中查看这些堆栈]

<video width="100%" height="100%" controls autoplay muted loop>
  <source src="./houdiniPrimPropertyStack.mp4" type="video/mp4" alt="Houdini Prim/Property Stack">
</video>


### Prim Index <a name="pcpPrimIndex"></a>
Next let's look at the prim index.

[ 接下来我们看一下 prim index]

~~~admonish tip title=""
```python
{{#include ../../../../code/core/composition.py:pcpPrimIndex}}
```
~~~

The prim index class can dump our prim index graph to the *dot* file format. The *dot* commandline tool ships with the most operating systems, we can then use it to visualize our graph as a .svg/.png file.

[ prim index 类可以将我们的 prim index 图转储为点文件格式. 大多数操作系统都附带了 dot 命令行工具，然后我们可以使用它将图形可视化为 .svg/.png 文件]

~~~admonish tip title="Result of: `print(prim_index.DumpToString())` | Click to view content" collapsible=true
```txt
Node 0:
    Parent node:              NONE
    Type:                     root
    DependencyType:           root
    Source path:              </bicycle>
    Source layer stack:       @anon:0x7f9eae9f2400:tmp.usda@,@anon:0x7f9eae9f1000:tmp-session.usda@
    Target path:              <NONE>
    Target layer stack:       NONE
    Map to parent:
        / -> /
    Map to root:
        / -> /
    Namespace depth:          0
    Depth below introduction: 0
    Permission:               Public
    Is restricted:            FALSE
    Is inert:                 FALSE
    Contribute specs:         TRUE
    Has specs:                TRUE
    Has symmetry:             FALSE
    Prim stack:
      </bicycle> anon:0x7f9eae9f2400:tmp.usda - @anon:0x7f9eae9f2400:tmp.usda@
Node 1:
    Parent node:              0
    Type:                     reference
    DependencyType:           non-virtual, purely-direct
    Source path:              </bicycle>
    Source layer stack:       @anon:0x7f9eae9f2b80:ReferenceExample@
    Target path:              </bicycle>
    Target layer stack:       @anon:0x7f9eae9f2400:tmp.usda@,@anon:0x7f9eae9f1000:tmp-session.usda@
    Map to parent:
        SdfLayerOffset(10, 1)
        /bicycle -> /bicycle
    Map to root:
        SdfLayerOffset(10, 1)
        /bicycle -> /bicycle
    Namespace depth:          1
    Depth below introduction: 0
    Permission:               Public
    Is restricted:            FALSE
    Is inert:                 FALSE
    Contribute specs:         TRUE
    Has specs:                TRUE
    Has symmetry:             FALSE
    Prim stack:
      </bicycle> anon:0x7f9eae9f2b80:ReferenceExample - @anon:0x7f9eae9f2b80:ReferenceExample@
```
~~~


~~~admonish tip title="Result of writing the graph to a dot .txt file | Click to view content" collapsible=true
```txt
{{#include pcpPrimIndex.txt}}
```
~~~

For example if we run it on a more advanced composition, in this case Houdini's pig asset:

[ 例如，如果我们在更高级的合成上运行它，在本例中为 Houdini pig asset]

~~~admonish tip title="Python print output for Houdini's pig asset | Click to view content" collapsible=true
```python
Pcp Node Ref
<pxr.Pcp.NodeRef object at 0x7f9ed3ad19e0> Pcp.ArcTypeRoot /pig /pig
<pxr.Pcp.NodeRef object at 0x7f9ed3ad17b0> Pcp.ArcTypeInherit /__class__/pig /pig
<pxr.Pcp.NodeRef object at 0x7f9ed3ad1cf0> Pcp.ArcTypeReference /pig /pig
<pxr.Pcp.NodeRef object at 0x7f9ed3ad1970> Pcp.ArcTypeInherit /__class__/pig /pig
<pxr.Pcp.NodeRef object at 0x7f9ed3ad1890> Pcp.ArcTypeVariant /pig{geo=medium} /pig{geo=medium}
<pxr.Pcp.NodeRef object at 0x7f9ed3ad1270> Pcp.ArcTypePayload /pig /pig
<pxr.Pcp.NodeRef object at 0x7f9ed3ad1660> Pcp.ArcTypeReference /pig /pig
<pxr.Pcp.NodeRef object at 0x7f9ed3ad1510> Pcp.ArcTypeVariant /pig{geo=medium} /pig{geo=medium}
<pxr.Pcp.NodeRef object at 0x7f9ed3ad13c0> Pcp.ArcTypeReference /ASSET_geo_variant_1/ASSET /pig
<pxr.Pcp.NodeRef object at 0x7f9ed3abbd60> Pcp.ArcTypeVariant /ASSET_geo_variant_1/ASSET{mtl=default} /pig{mtl=default}
<pxr.Pcp.NodeRef object at 0x7f9ed3abb6d0> Pcp.ArcTypeReference /ASSET_geo_variant_1/ASSET_mtl_default /pig
<pxr.Pcp.NodeRef object at 0x7f9ed3ad1a50> Pcp.ArcTypeReference /pig /pig
<pxr.Pcp.NodeRef object at 0x7f9ed3ad15f0> Pcp.ArcTypeReference /ASSET_geo_variant_2/ASSET /pig
<pxr.Pcp.NodeRef object at 0x7f9ed3abbe40> Pcp.ArcTypeVariant /ASSET_geo_variant_2/ASSET{geo=medium} /pig{geo=medium}
<pxr.Pcp.NodeRef object at 0x7f9ed3ad1ac0> Pcp.ArcTypeReference /ASSET_geo_variant_1/ASSET /pig
<pxr.Pcp.NodeRef object at 0x7f9ed3abbf90> Pcp.ArcTypeVariant /ASSET_geo_variant_1/ASSET{geo=medium} /pig{geo=medium}
<pxr.Pcp.NodeRef object at 0x7f9ed3abb430> Pcp.ArcTypeReference /ASSET_geo_variant_0/ASSET /pig
<pxr.Pcp.NodeRef object at 0x7f9ed3abb9e0> Pcp.ArcTypeVariant /ASSET_geo_variant_0/ASSET{geo=medium} /pig{geo=medium}
```
~~~

~~~admonish tip title="Result of writing the graph to a dot .txt file for Houdini's pig asset | Click to view content" collapsible=true
```txt
{{#include pcpPrimIndexPig.txt}}
```
~~~

![Alt text](pcpPrimIndexPig.png)

~~~admonish tip
We can also access the `Pcp.Cache` of the stage via: `pcp_cache = stage._GetPcpCache()`

[ 我们还可以通过以下方式访问舞台的 Pcp.Cache ： pcp_cache = stage._GetPcpCache()]
~~~

### Prim Composition Query <a name="pcpPrimCompositionQuery"></a>
Next let's look at prim composition queries. Instead of having to filter the prim index ourselves, we can use the `Usd.PrimCompositionQuery` to do it for us. More info in the [USD API docs](https://openusd.org/dev/api/class_usd_prim_composition_query.html).

[ 接下来让我们看看 prim 合成查询. 我们可以使用 Usd.PrimCompositionQuery 来帮我们过滤 prim index，而不必自己过滤 prim index.更多信息请参阅 [USD API docs](https://openusd.org/dev/api/class_usd_prim_composition_query.html)]

The query works by specifying a filter and then calling `GetCompositionArcs`.

[ 该查询通过指定过滤器然后调用 GetCompositionArcs 来工作]

USD provides these convenience filters, it returns a new `Usd.PrimCompositionQuery` instance with the filter applied:

[ USD 提供了这些便利的过滤器，它们会返回一个应用了过滤器的新 Usd.PrimCompositionQuery 实例]

- `Usd.PrimCompositionQuery.GetDirectInherits(prim)`: Returns all non ancestral inherit arcs
- `Usd.PrimCompositionQuery.GetDirectReferences(prim)`: Returns all non ancestral reference arcs
- `Usd.PrimCompositionQuery.GetDirectRootLayerArcs(prim)`: Returns arcs that were defined in the active layer stack.

These are the sub-filters that can be set. We can only set a single token value per filter:

[ 这些是可以设置的子过滤器.我们只能为每个过滤器设置一个标记值]

- **ArcTypeFilter**: Filter based on different arc(s).
    - `Usd.PrimCompositionQuery.ArcTypeFilter.All`
    - `Usd.PrimCompositionQuery.ArcTypeFilter.Inherit`
    - `Usd.PrimCompositionQuery.ArcTypeFilter.Variant`
    - `Usd.PrimCompositionQuery.ArcTypeFilter.NotVariant`
    - `Usd.PrimCompositionQuery.ArcTypeFilter.Reference`
    - `Usd.PrimCompositionQuery.ArcTypeFilter.Payload`
    - `Usd.PrimCompositionQuery.ArcTypeFilter.NotReferenceOrPayload`
    - `Usd.PrimCompositionQuery.ArcTypeFilter.ReferenceOrPayload`
    - `Usd.PrimCompositionQuery.ArcTypeFilter.InheritOrSpecialize` 
    - `Usd.PrimCompositionQuery.ArcTypeFilter.NotInheritOrSpecialize` 
    - `Usd.PrimCompositionQuery.ArcTypeFilter.Specialize`
- **DependencyTypeFilter**: Filter based on if the arc was introduced on a parent prim or on the prim itself.
    - `Usd.PrimCompositionQuery.DependencyTypeFilter.All`
    - `Usd.PrimCompositionQuery.DependencyTypeFilter.Direct`
    - `Usd.PrimCompositionQuery.DependencyTypeFilter.Ancestral`
- **ArcIntroducedFilter**: Filter based on where the arc was introduced.
    - `Usd.PrimCompositionQuery.ArcIntroducedFilter.All`
    - `Usd.PrimCompositionQuery.ArcIntroducedFilter.IntroducedInRootLayerStack`
    - `Usd.PrimCompositionQuery.ArcIntroducedFilter.IntroducedInRootLayerPrimSpec`
- **HasSpecsFilter**: Filter based if the arc has any specs (For example an inherit might not find any in the active layer stack)
    - `Usd.PrimCompositionQuery.HasSpecsFilter.All`
    - `Usd.PrimCompositionQuery.HasSpecsFilter.HasSpecs`
    - `Usd.PrimCompositionQuery.HasSpecsFilter.HasNoSpecs`

~~~admonish tip title=""
```python
{{#include ../../../../code/core/composition.py:pcpPrimCompositionQuery}}
```
~~~

The returned filtered `Usd.CompositionArc` objects, allow us to inspect various things about the arc. You can find more info in the [API docs](https://openusd.org/dev/api/class_usd_prim_composition_query_arc.html)

[ 返回的过滤后的 Usd.CompositionArc 对象使我们能够检查有关弧的各种内容. 您可以在[API docs](https://openusd.org/dev/api/class_usd_prim_composition_query_arc.html)中找到更多信息]
