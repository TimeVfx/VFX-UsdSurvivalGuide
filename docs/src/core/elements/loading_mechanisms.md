# Loading & Traversing Data [加载和遍历数据]

# Table of Contents [目录]
1. [Traversing & Loading Data In-A-Nutshell](#summary)
1. [What should I use it for?](#usage)
1. [Resources](#resources)
1. [Overview](#overview)
1. [Loading Mechanisms](#loadingMechanisms)
    1. [Layer Muting](#loadingMechanismsLayerMuting)
    1. [Prim Path Loading (USD speak: Prim Population Mask)](#loadingMechanismsLayerPrimPopulationMask)
    1. [Payload Loading](#loadingMechanismsLayerPayloadLoading)
    1. [GeomModelAPI->Draw Mode](#loadingMechanismsGeomModelAPIDrawMode)
1. [Traversing Data](#traverseData)
    1. [Traversing Stages](#traverseDataStage)
    1. [Traversing Layers](#traverseDataLayer)
    1. [Traverse Sample Data/Profiling](#traverseDataProfiling)

## TL;DR - Loading & Traversing Data In-A-Nutshell [概述]<a name="summary"></a>
#### Loading Mechanisms [加载机制]
When loading large scenes, we can selectively disabling loading via the following loading mechanisms:
There are three ways to influence the data load, from lowest to highest granularity .

[ 当加载大型场景时，我们可以通过以下加载机制选择性地禁用加载： 影响数据加载的方式有三种，从最低到最高]

- **Layer Muting**: This controls what layers are allowed to contribute to the composition result.

  [ Layer Muting：控制哪些层对合成结果做出贡献]
- **Prim Population Mask**: This controls what prim paths to consider for loading at all.

  [ Prim Population Mask: 控制加载哪些 prim paths]
- **Payload Loading**: This controls what prim paths, that have payloads, to load.

  [ Payload Loading: 控制加载哪些 payloads]
- **GeomModelAPI->Draw Mode**: This controls per prim how it should be drawn by delegates. It can be one of "Full Geometry"/"Origin Axes"/"Bounding Box"/"Texture Cards". It requires the kind to be set on the prim and all its ancestors. Therefore it is "limited" to (asset-) root prims and ancestors.

  [ GeomModelAPI->Draw Mode：这控制每个 prim 如何由委托绘制. 它可以是 "Full Geometry"/"Origin Axes"/"Bounding Box"/"Texture Cards" 之一. 它要求将类型设置在 prim 及其所有祖先上. 因此，它“仅限于”（资产）根 prim 和祖先]
- **Activation**: Control per prim whether load itself and its child hierarchy. This is more a an artist facing mechanism, as we end up writing the data to the stage, which we don't do with the other methods.

  [ Activation：控制每个 prim 是否加载其自身及其子层次结构. 这更多的是一个面向艺术家的机制，因为最终我们会将数据写入到 stage 上，这是其他方法不会做的]

#### Traversing/Iterating over our stage/layer [遍历/迭代 stage/layer]
To inspect our stage, we can iterate (traverse) over it:

[ 为了检查 stage，我们可以迭代（遍历）它]

When traversing, we try to pre-filter our prims as much as we can, via our prim metadata and USD core features(metadata), before inspecting their properties. This keeps our traversals fast even with hierarchies with millions of prims. We recommend first filtering based on metadata, as this is a lot faster than trying to access attributes and their values.

[ 在遍历过程中，我们会尝试通过 prim 元数据和 USD 核心功能（元数据）尽可能多地预先过滤 prims，然后再检查它们的属性. 这可以保持我们的遍历速度，即使面对包含数百万个 prims 的层级结构也能保持高效. 我们建议先基于元数据进行过滤，因为这比尝试访问属性和它们的值要快得多]

We also have a thing called [predicate](https://openusd.org/dev/api/prim_flags_8h.html#Usd_PrimFlags), which just defines what core metadata to consult for pre-filtering the result.

[ 我们还有一个叫做 [predicate](https://openusd.org/dev/api/prim_flags_8h.html#Usd_PrimFlags) 的东西，它定义了在预过滤结果时要参考哪些核心元数据]

Another important feature is stopping traversal into child hierarchies. This can be done by calling `ìterator.PruneChildren()

[ 另一个重要的功能是停止遍历子层级结构. 这可以通过调用 ìterator.PruneChildren() 来实现]

~~~admonish tip title="Stage/Prim Traversal"
```python
{{#include ../../../../code/core/elements.py:traverseDataStageTemplate}}
```
~~~

Layer traversal is a bit different. Instead of iterating, we provide a function, that gets called with each `Sdf.Path` representable object in the active layer. So we also see all properties, relationship targets and variants.

[ 层的遍历有些不同. 我们不是进行迭代，而是提供一个函数，该函数会在激活层的每个 Sdf.Path 对象上被调用. 因此，我们也可以看到所有的properties, relationship targets 和 variants]

~~~admonish tip title="Layer Traversal"
```python
{{#include ../../../../code/core/elements.py:traverseDataLayerTemplate}}
```
~~~

## What should I use it for? <a name="usage"></a>

[ 我应该用它做什么？]
~~~admonish tip
We'll be using loading mechanisms to optimize loading only what is relevant for the current task at hand.

[ 我们将使用加载机制来优化仅加载与当前任务相关的内容]
~~~

## Resources [资源]<a name="resources"></a>
- [Prim Cache Population (Pcp)](https://openusd.org/dev/api/pcp_page_front.html)
- [Stage Payload Loading](https://openusd.org/dev/api/class_usd_stage.html#Usd_workingSetManagement)
- [Pcp.Cache](https://openusd.org/dev/api/class_pcp_cache.html)
- [Usd.StagePopulationMask](https://openusd.org/dev/api/class_usd_stage_population_mask.html)
- [Usd.StageLoadRules](https://openusd.org/dev/api/class_usd_stage_load_rules.html)

## Loading Mechanisms [加载机制]<a name="loadingMechanisms"></a>
Let's look at load mechanisms that USD offers to make the loading of our hierarchies faster.

[ 让我们来看看USD提供的加载机制，以便更快地加载我们的层级结构]

Before we proceed, it is important to note, that USD is highly performant in loading hierarchies. When USD loads .usd/.usdc binary crate files, it sparsely loads the content: It can read in the hierarchy without loading in the attributes. This allows it to, instead of loading terabytes of data, to only read the important bits in the file and lazy load on demand the heavy data when requested by API queries or a hydra delegate.

[ 在我们继续之前，需要重点注意的是，USD在加载层次结构方面性能非常高. 当USD加载.usd/.usdc二进制文件时，它会稀疏地加载内容：它可以在不加载属性的情况下读取层级结构. 这允许它只读取文件中的重要部分，而不是加载数TB级别的数据，并在API查询或hydra代理请求时按需延迟加载大量数据]

When loading stages/layers per code only, we often therefore don't need to resort to using these mechanisms.

[ 当仅通过代码加载 stages/layers 时，我们通常不需要使用这些机制]

There are three ways to influence the data load, from lowest to highest granularity .

[ 有三种方法可以影响数据负载，从最低到最高]

- **Layer Muting**: This controls what layers are allowed to contribute to the composition result.
- [ Layer Muting：控制哪些层对合成结果做出贡献]
- **Prim Population Mask**: This controls what prim paths to consider for loading at all.
- [ Prim Population Mask: 控制加载哪些 prim paths]
- **Payload Loading**: This controls what prim paths, that have payloads, to load.
- [ Payload Loading: 控制加载哪些 payloads]
- **GeomModelAPI->Draw Mode**: This controls per prim how it should be drawn by delegates. It can be one of "Full Geometry"/"Origin Axes"/"Bounding Box"/"Texture Cards". It requires the kind to be set on the prim and all its ancestors. Therefore it is "limited" to (asset-) root prims and ancestors.
- [ GeomModelAPI->Draw Mode：这控制每个 prim 如何由委托绘制. 它可以是 "Full Geometry"/"Origin Axes"/"Bounding Box"/"Texture Cards" 之一. 它要求将类型设置在 prim 及其所有祖先上. 因此，它“仅限于”（资产）根 prim 和祖先]
- **Activation**: Control per prim whether load itself and its child hierarchy. This is more a an artist facing mechanism, as we end up writing the data to the stage, which we don't do with the other methods.
- [ Activation：控制每个 prim 是否加载其自身及其子层次结构. 这更多的是一个面向艺术家的机制，因为最终我们会将数据写入到 stage 上，这是其他方法不会做的]

Stages are the controller of how our [Prim Cache Population (PCP)](../composition/pcp.md) cache loads our composed layers. Technically the stage just exposes the PCP cache in a nice API, that forwards its requests to the its pcp cache `stage._GetPcpCache()`, similar how all `Usd` ops are wrappers around `Sdf` calls.

[ stage 是控制我们如何通过 [Prim Cache Population (PCP)](../composition/pcp.md) 缓存加载合成层的控制器. 从技术上讲，stage 只是将 PCP 缓存以友好的 API 形式公开，并将其请求转发给它的 pcp 缓存 stage._GetPcpCache()，这与所有 Usd 操作都是 Sdf 调用的包装器类似]

Houdini exposes all three in two different ways:

[ Houdini 以两种不同的操作方式提供了所有三种功能]

- **Configue Stage** LOP node: This is the same as setting it per code via the stage.
- [ Configue Stage : 这与通过 stage 在代码中设置相同]
- **Scene Graph Tree** panel: In Houdini, that stage that gets rendered, is actually not the stage of your node (at least what we gather from reverse engineering). Instead it is a duplicate, that has overrides in the session layer and loading mechanisms listed above.
- [ Scene Graph Tree : 在Houdini中，被渲染的 stage 实际上并不是节点（至少从逆向工程收集到的信息来看）的 stage. 相反，它是一个副本，它在会话层中有覆盖，并使用了上面列出的加载机制]

<video width="100%" height="100%" controls autoplay muted loop>
  <source src="houdiniLoadingMechanisms.mp4" type="video/mp4" alt="Houdini Configure Stage Node">
</video>

<video width="100%" height="100%" controls autoplay muted loop>
  <source src="houdiniSceneGraphTreePanel.mp4" type="video/mp4" alt="Houdini Scene Graph Tree Panel">
</video>

More Houdini specific information can be found in our [Houdini - Performance Optimizations](../../dcc/houdini/performance/overview.md#loadingMechanisms) section.

[ 更多 Houdini 特定信息可以阅读 [Houdini - Performance Optimizations](../../dcc/houdini/performance/overview.md#loadingMechanisms)]

### Layer Muting <a name="loadingMechanismsLayerMuting"></a>
We can "mute" (disable) layers either globally or per stage.

[ 我们可以在全局或每个 stage 上 "mute" (disable) layers]

Globally muting layers is done via the singleton, this mutes it on all stages that use the layer.

[ 全局 mute layers 是通过单例完成的，这会让所有 stage 上使用的该层都 mutes]

~~~admonish tip title=""
```python
from pxr import Sdf
layer = Sdf.Layer.FindOrOpen("/my/layer/identifier")
Sdf.Layer.AddToMutedLayers(layer.identifier)
Sdf.Layer.RemoveFromMutedLayers(layer.identifier)
```
~~~

Muting layers per stage is done via the `Usd.Stage` object, all function signatures work with the layer identifier string. If the layer is muted globally, the stage will not override the muting and it stays muted.

[ 在每个 stage 上 muting layers 层是通过 Usd.Stage 对象完成的，所有函数签名都与层标识符字符串一起使用. 如果层在全局范围内 mute，则stage 不会覆盖 mute 设置，它仍将保持 mute 状态]

~~~admonish tip title=""
```python
{{#include ../../../../code/core/elements.py:loadingMechanismsLayerMuting}}
```
~~~

~~~admonish tip title="Pro Tip | Layer Muting"
We use layer muting in production for two things:
- Artists can opt-in to load layers that are relevant to them. For example in a shot, a animator doesn't have to load the background set or fx layers.
- Pipeline-wise we have to ensure that artists add shot layers in a specific order (For example: lighting > fx > animation > layout >). Let's say a layout artist is working in a shot, we only want to display the layout and camera layers. All the other layers can (should) be muted, because A. performance, B. there might be edits in higher layers, that the layout artist is not interested in seeing yet. If we were to display them, some of these edits might block ours, because they are higher in the layer stack.
~~~

Here is an example of global layer muting:

[ 以下是全局 muting layer 示例]

<video width="100%" height="100%" controls autoplay muted loop>
  <source src="layerMutingGlobal.mp4" type="video/mp4" alt="Houdini Layer Muting Global">
</video>

We have to re-cook the node for it to take effect, due to how Houdini caches stages.

[ 由于 Houdini 缓存 stage 的方式，我们必须重新运行节点才能使其生效]

### Prim Path Loading Mask (USD speak: Prim Population Mask) <a name="loadingMechanismsLayerPrimPopulationMask"></a>

~~~admonish tip title="Pro Tip | Prim Population Mask"
Similar to prim activation, the prim population mask controls what prims (and their child prims) are even considered for being loaded into the stage. Unlike activation, the prim population mask does not get stored in a USD layer. It is therefore a pre-filtering mechanism, rather than an artist facing "what do I want to hide from my scene" mechanism.
~~~

One difference to activation is that not only the child hierarchy is stripped away for traversing, but also the prim itself, if it is not included in the mask.

[ 与激活的区别是，不仅子层级结构会被剥离以进行遍历，而且 prim 本身也会被剥离（如果它未包含在mask中）]

The population mask is managed via the `Usd.StagePopulationMask` class.

[ mask 通过 Usd.StagePopulationMask 类进行管理]


~~~admonish tip title=""
```python
{{#include ../../../../code/core/elements.py:loadingMechanismsLayerPrimPopulationMask}}
```
~~~

What's also really cool, is that we can populate the mask by relationships/attribute connections.

[ 同样非常酷的是，我们可以通过 relationships/attribute 连接来填充 mask]

~~~admonish tip title=""
```python
stage.ExpandPopulationMask(relationshipPredicate=lambda r: r.GetName() == 'material:binding',
                           attributePredicate=lambda a: False)
```
~~~

<video width="100%" height="100%" controls autoplay muted loop>
  <source src="houdiniPopulationMaskExpand.mp4" type="video/mp4" alt="Houdini Population Mask Expand">
</video>

### Payload Loading <a name="loadingMechanismsLayerPayloadLoading"></a>
~~~admonish tip title="Pro Tip | Payload Loading"
Payloads are USD's mechanism of disabling the load of heavy data and instead leaving us with a bounding box representation (or texture card representation, if you set it up). We can configure our stages to not load payloads at all or to only load payloads at specific prims.
~~~

What might be confusing here is the naming convention: USD refers to this as "loading", which sounds pretty generic. Whenever we are looking at stages and talking about loading, know that we are talking about payloads.

[ 这里可能令人困惑的是命名约定：USD将此称为“loading”，这听起来相当通用. 每当我们在 stage 上谈论加载时，要知道我们其实是在谈论payloads]

You can find more details in the [API docs](https://openusd.org/dev/api/class_usd_stage.html#Usd_workingSetManagement).

[ 您可以在 [API docs](https://openusd.org/dev/api/class_usd_stage.html#Usd_workingSetManagement) 文档中找到更多详细信息]

~~~admonish tip title=""
```python
{{#include ../../../../code/core/elements.py:loadingMechanismsLayerPayloadLoading}}
```
~~~

### GeomModelAPI->Draw Mode <a name="loadingMechanismsGeomModelAPIDrawMode"></a>
~~~admonish tip title="Pro Tip | Draw Mode"
The draw mode can be used to tell our Hydra render delegates to not render a prim and its child hierarchy. Instead it will only display a preview representation.

The preview representation can be one of:
- Full Geometry
- Origin Axes
- Bounding Box
- Texture Cards

Like visibility, the draw mode is inherited downwards to its child prims. We can also set a draw mode color, to better differentiate the non full geometry draw modes, this is not inherited though and must be set per prim.
~~~

~~~admonish danger title="Important | Draw Mode Requires Kind"
In order for the draw mode to work, the prim and all its ancestors, must have a [kind](../plugins/kind.md) defined. Therefore it is "limited" to (asset-)root prims and its ancestors.
See the [official docs](https://openusd.org/dev/api/class_usd_geom_model_a_p_i.html) for more info.
~~~

Here is how we can set it via Python, it is part of the `UsdGeomModelAPI`:

[ 以下是我们如何通过 Python 设置它，它是 UsdGeomModelAPI 的一部分]

~~~admonish tip title=""
```python
{{#include ../../../../code/core/elements.py:loadingMechanismsGeomModelAPIDrawMode}}
```
~~~

<video width="100%" height="100%" controls autoplay muted loop>
  <source src="houdiniLoadingMechanismsDrawMode.mp4" type="video/mp4" alt="Houdini Draw Mode">
</video>


## Traversing Data <a name="traverseData"></a>
When traversing (iterating) through our hierarchy, we commonly use these metadata and property entries on prims to pre-filter what we want to access:

[ 当遍历（迭代）层级结构时，我们通常在 prims 上使用这些元数据和属性条目来预先过滤我们想要访问的内容]

- .IsA Typed Schemas (Metadata)
- Type Name (Metadata)
- Specifier (Metadata)
- Activation (Metadata)
- Kind (Metadata)
- Purpose (Attribute)
- Visibility (Attribute)

~~~admonish tip title="Pro Tip | High Performance Traversals"
When traversing, using the above "filters" to narrow down your selection well help keep your traversals fast, even with hierarchies with millions of prims. We recommend first filtering based on metadata, as this is a lot faster than trying to access attributes and their values.

Another important feature is stopping traversal into child hierarchies. This can be done by calling `ìterator.PruneChildren()`:
```python
from pxr import Sdf, UsdShade
root_prim = stage.GetPseudoRoot()
# We have to cast it as an iterator to gain access to the .PruneChildren() method.
iterator = iter(Usd.PrimRange(root_prim))
for prim in iterator:
    if prim.IsA(UsdShade.Material):
        # Don't traverse into the shader network prims
        iterator.PruneChildren()
```
~~~

### Traversing Stages <a name="traverseDataStage"></a>
Traversing stages works via the `Usd.PrimRange` class. The `stage.Traverse`/`stage.TraverseAll`/`prim.GetFilteredChildren` methods all use this as the base class, so let's checkout how it works:

[ 遍历 stage 是通过 Usd.PrimRange 类进行 stage.Traverse / stage.TraverseAll / prim.GetFilteredChildren 方法都使用它作为基类，所以让我们看看它是如何工作的]

We have two traversal modes:

[ 我们有两种遍历方式]

- Default: Iterate over child prims
- [ Default : 遍历子 prims ]
- PreAndPostVisit: Iterate over the hierarchy and visit each prim twice, once when first encountering it, and then again when "exiting" the child hierarchy. See our [primvars query](../../production/caches/attribute.md#primvars) section for a hands-on example why this can be useful.
- [ PreAndPostVisit：遍历层级结构，每个基元都会访问两次，一次是在首次遇到它时，另一次是在“退出”子层级结构时. 请参阅 [primvars query](../../production/caches/attribute.md#primvars)，通过实际示例了解为什么这种模式有用]

We also have a thing called "predicate"([Predicate Overview](https://openusd.org/dev/api/prim_flags_8h.html#Usd_PrimFlags)), which just defines what core metadata to consult for pre-filtering the result:

[ 我们还有一个称为 "predicate"([Predicate Overview](https://openusd.org/dev/api/prim_flags_8h.html#Usd_PrimFlags))的功能, 它仅定义了在预过滤结果时要参考哪些核心元数据]

- Usd.PrimIsActive: Usd.Prim.IsActive() - If the "active" metadata is True
- [ Usd.PrimIsActive：Usd.Prim.IsActive() - 如果“active”元数据为 True]
- Usd.PrimIsLoaded: Usd.Prim.IsLoaded() - If the (ancestor) payload is loaded
- [ Usd.PrimIsLoaded：Usd.Prim.IsLoaded() - 如果（祖先）payload 被加载]
- Usd.PrimIsModel: Usd.Prim.IsModel() - If the kind is a sub kind of `Kind.Tokens.model`
- [ Usd.PrimIsModel：Usd.Prim.IsModel() - 如果 kind 是 Kind.Tokens.model 的子类]
- Usd.PrimIsGroup: Usd.Prim.IsGroup() - If the kind is `Kind.Tokens.group`
- [ Usd.PrimIsGroup：Usd.Prim.IsGroup() - 如果 kind 是 Kind.Tokens.group]
- Usd.PrimIsAbstract: Usd.Prim.IsAbstract() - If the prim specifier is `Sdf.SpecifierClass`
- [ Usd.PrimIsAbstract：Usd.Prim.IsAbstract() - 如果 prim 说明符是 Sdf.SpecifierClass]
- Usd.PrimIsDefined: Usd.Prim.IsDefined() - If the prim specifier is `Sdf.SpecifierDef`
- [ Usd.PrimIsDefined：Usd.Prim.IsDefined() - 如果 prim 说明符是 Sdf.SpecifierDef]
- Usd.PrimIsInstance: Usd.Prim.IsInstance() - If prim is an instance root (This is false for prims in instances)
- [ Usd.PrimIsInstance：Usd.Prim.IsInstance() - 如果 prim 是实例根（对于实例中的 prims 为 false）]

Presets:

[ 预设]

- Usd.PrimDefaultPredicate: `Usd.PrimIsActive & Usd.PrimIsDefined & Usd.PrimIsLoaded & ~Usd.PrimIsAbstract`
- [ Usd.PrimDefaultPredicate： Usd.PrimIsActive & Usd.PrimIsDefined & Usd.PrimIsLoaded & ~Usd.PrimIsAbstract]
- Usd.PrimAllPrimsPredicate: Shortcut for selecting all filters (basically ignoring the prefilter).
- [ Usd.PrimAllPrimsPredicate：选择所有过滤器（基本上忽略预过滤器）的快捷方式]

By default the Usd.PrimDefaultPredicate is used, if we don't specify one.

[ 如果我们不指定的话，默认情况下会使用 Usd.PrimDefaultPredicate]

~~~admonish tip title="Pro Tip | Usd Prim Range"
Here is the most common syntax you'll be using:
```python
{{#include ../../../../code/core/elements.py:traverseDataStageTemplate}}
```
~~~

The default traversal also doesn't go into [instanceable prims](../composition/livrps.md#compositionInstance).
To enable it we can either run `pxr.Usd.TraverseInstanceProxies(<existingPredicate>)` or `predicate.TraverseInstanceProxies(True)`

[ 默认的遍历模式不会进入 [instanceable prims](../composition/livrps.md#compositionInstance) 要启用此功能，我们可以运行 pxr.Usd.TraverseInstanceProxies() 或 predicate.TraverseInstanceProxies(True)]

Within instances we can get the prototype as follows, for more info see our [instanceable section](../composition/livrps.md#compositionInstance):

[ 在实例中，我们可以按如下方式获取 prototype 更多信息请参见 [instanceable section](../composition/livrps.md#compositionInstance)]

~~~admonish tip title=""
```python
{{#include ../../../../code/core/composition.py:compositionInstanceable}}
```
~~~

Let's look at some traversal examples:

[ 我们来看一些遍历的例子]

~~~admonish tip title="Stage/Prim Traversal | Click to expand" collapsible=true
```python
{{#include ../../../../code/core/elements.py:traverseDataStage}}
```
~~~

### Traversing Layers <a name="traverseDataLayer"></a>
Layer traversal is different, it only looks at the active layer and traverses everything that is representable via an `Sdf.Path` object.
This means, it ignores activation and it traverses into variants and relationship targets. This can be quite useful, when we need to rename something or check for data in the active layer.

[ 层的遍历是不同的，它只检查激活层，并遍历可以通过 Sdf.Path 对象表示的所有内容. 这意味着，它会忽略激活状态，并且会遍历变体（variants）和关系目标（relationship targets）. 我们需要重命名某些内容或检查激活层中的数据时，这可能会非常有用]

We cover it in detail with examples over in our [layer and stages](./layer.md#layerTravesal) section.

[ 我们在 [layer and stages](./layer.md#layerTravesal) 章节中通过示例详细介绍了它]

~~~admonish tip title="Pro Tip | Layer Traverse"
The traversal for layers works differently. Instead of an iterator, we have to provide a
"kernel" like function, that gets an `Sdf.Path` as an input.
Here is the most common syntax you'll be using:
```python
{{#include ../../../../code/core/elements.py:traverseDataLayerTemplate}}
```
~~~

### Traverse Sample Data/Profiling <a name="traverseDataProfiling"></a>
To test profiling, we can setup a example hierarchy. The below code spawns a nested prim hierarchy.
You can adjust the `create_hierarchy(layer, prim_path, <level>)`, be aware this is exponential, so a value of 10 and higher will already generate huge hierarchies.

[ 为了测试分析，我们设置一个层级结构示例. 下面的代码生成一个嵌套的 prim 层级结构. 您可以调整 create_hierarchy(layer, prim_path, <level>) ，请注意这是指数级的，因此 10 及更高的值已经会生成巨大的层级结构]

The output will be something like this:

[ 输出将是这样的]

![Houdini Traversal Profiling Hierarchy](traversalProfilingHierarchy.jpg)

~~~admonish tip title=""
```python
{{#include ../../../../code/core/elements.py:traverseSampleData}}
```
~~~

Here is how we can run profiling (this is kept very simple, check out our [profiling](../profiling/overview.md) section how to properly trace the stats) on the sample data:

[ 以下是我们如何对示例数据运行分析（这非常简单，请查看  [profiling](../profiling/overview.md) 部分如何正确跟踪统计数据）]

~~~admonish tip title=""
```python
{{#include ../../../../code/core/elements.py:traverseSampleDataProfiling}}
```
~~~

Here is a sample output, we recommend running each traversal multiple times and then averaging the results.
As we can see running attribute checks against attributes can be twice as expensive than checking metadata or the type name. (Surprisingly kind checks take a long time, even though it is also a metadata check)

[ 这是一个样本输出，我们建议每个遍历运行多次，然后取结果的平均值. 我们可以看到，对属性进行属性检查可能比检查元数据或类型名称昂贵两倍.（令人惊讶的是，kind 检查也花费了很长时间，尽管它也是一种元数据检查）]

~~~admonish tip title=""
```python
0.17678 Seconds | IsA(Boundable) | Match 38166
0.17222 Seconds | GetTypeName | Match 44294
0.42160 Seconds | Kind | Match 93298
0.38575 Seconds | IsLeaf AssetInfo  | Match 44294
0.27142 Seconds | IsLeaf Attribute Has | Match 44294
0.38036 Seconds | IsLeaf Attribute  | Match 44294
0.37459 Seconds | IsLeaf Attribute (Validation) | Match 44294
```
~~~

