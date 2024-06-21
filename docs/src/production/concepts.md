# Advanced Concepts [高级概念]

# Table of Contents [目录]
1. [Edit Targets](#editTargets)
1. [Utility functions in the Usd.Utils module](#usdUtils)
1. [Utility functions in the Sdf module](#sdf)
    1. [Moving/Renaming/Removing prim/property/variant specs with Sdf.BatchNamespaceEdit()](#sdfBatchNamespaceEdit)
    1. [Copying data with Sdf.CopySpec](#sdfCopySpec)
    1. [Delaying change notifications with the Sdf.ChangeBlock](#sdfChangeBlock)
1. [Relationships](#relationship)
    1. [Special Relationships](#relationshipSpecial)
    1. [Relationship Forwarding (Binding post)](#relationshipForwarding)
    1. [Collections](#relationshipCollection)

## Edit Targets [编辑目标]
~~~admonish tip title="Pro Tip | Edit Targets"
A edit target defines, what layer all calls in the high level API should write to.

[ 编辑目标定义了高级 API 中的所有调用应写入哪一层]
~~~

An edit target's job is to map from one namespace to another, we mainly use them for writing to layers in the active layer stack (though we could target any layer) and to write variants, as these are written "inline" and therefore need an extra name space injection.

[ 编辑目标的工作是将一个名称空间映射到另一个名称空间，我们主要使用它们写入激活层堆栈中的层（尽管我们可以针对任何层）并编写变体，因为这些是“内联”写入的，因此需要一个额外的名称空间注入]

We cover edit targets in detail in our [composition fundamentals](../core/composition/fundamentals.md#compositionFundamentalsEditTarget) section.

[ 我们在[合成基础](../core/composition/fundamentals.md#compositionFundamentalsEditTarget) 部分详细介绍了编辑目标]

## Utility functions in the Usd.Utils module <a name="utilsUsdUtils"></a>

[ Usd.utils 模块中的实用函数]

Usd provides a bunch of utility functions in the `UsdUtils` module ([USD Docs](https://openusd.org/dev/api/flatten_layer_stack_8h.html)):

[ Usd 在 UsdUtils 模块中提供了一堆实用函数([USD Docs](https://openusd.org/dev/api/flatten_layer_stack_8h.html))]

For retrieving/upating dependencies:

[ 用于检索/更新依赖项]

- **UsdUtils.ExtractExternalReferences**: This is similar to `layer.GetCompositionAssetDependencies()`, except that it returns three lists: `[<sublayers>], [<references>], [<payloads>]`. It also consults the assetInfo metadata, so result might be more "inclusive" than `layer.GetCompositionAssetDependencies()`.

    [ UsdUtils.ExtractExternalReferences：这与 layer.GetCompositionAssetDependencies() 类似，只不过它返回三个列表： [\<sublayers\>], [\<references\>], [\<payloads\>] .它还查阅 assetInfo 元数据，因此结果可能比 layer.GetCompositionAssetDependencies() 更具“包容性”]
- **UsdUtils.ComputeAllDependencies**: This recursively calls `layer.GetCompositionAssetDependencies()` and gives us the aggregated result.

    [ UsdUtils.ComputeAllDependency：这会递归调用 layer.GetCompositionAssetDependencies() 并为我们提供聚合结果]
- **UsdUtils.ModifyAssetPaths**: This is similar to Houdini's output processors. We provide a function that gets the input path and returns a (modified) output path.
    [ UsdUtils.ModifyAssetPaths：这类似于Houdini的输出处理器. 我们提供一个函数来获取输入路径并返回（修改后的）输出路径]

For animation and value clips stitching:

[ 对于动画和值剪辑拼接]

- Various tools for stitching/creating value clips. We cover these in our [animation section](../core/elements/animation.md#animationValueClips). These are also what the commandline tools that ship with USD use.

    [ 用于拼接/创建值剪辑的各种工具. 我们将在[动画](../core/elements/animation.md#animationValueClips)部分介绍这些内容. 这些也是 USD 附带的命令行工具所使用的]

For collection authoring/compression:

[ 对于集合创作/压缩]

- We cover these in detail in our [collection section](../core/elements/collection.md#collectionQuery).

    [ 我们在[集合](../core/elements/collection.md#collectionQuery)部分详细介绍了这些内容]

## Utility functions in the Sdf module <a name="sdf"></a>

[ Sdf 模块中的实用函数]

### Moving/Renaming/Removing prim/property/variant specs with Sdf.BatchNamespaceEdit() <a name="sdfBatchNamespaceEdit"></a>

[ 使用 Sdf.BatchNamespaceEdit() Moving/Renaming/Removing  prim/property/variant]

We've actually used this quite a bit in the guide so far, so in this section we'll summarize its most important uses again:

[ 我们实际上已经在指南中多次使用了它，因此在本节中我们将再次总结它最重要的用途]

#### Using Sdf.BatchNamespaceEdit() moving/renaming/removing prim/property (specs)
We main usage is to move/rename/delete prims. We can only run the name space edit on a layer, it does not work with stages.
Thats means if we have nested composition, we can't rename prims any more. In production this means we'll only be using this
with the "active" layer, that we are currently creating/editing. All the edits added are run in the order they are added,
we have to be careful what order we add removes/renames if they interfere with each other.

[ 我们的主要用途是 move/rename/delete prims . 我们只能在 layer 运行 NamespaceEdit，它不适用于 stage. 这意味着如果我们有嵌套合成，我们就不能再重命名 prims 了.这意味在生产中，着我们只会将其与我们当前正在创建/编辑的“激活”层一起使用. 添加编辑按层添加的顺序运行，如果它们相互干扰，我们必须小心添加删除/重命名的顺序]

~~~admonish tip title="Sdf.BatchNamespaceEdit | Moving/renaming/removing prim/property specs | Click to expand!" collapsible=true
```python
{{#include ../../../code/production/production.py:productionConceptsSdfBatchNamespaceMoveRenameDelete}}
```
~~~

#### Using Sdf.BatchNamespaceEdit() for variant creation
We can create variant via the namespace edit, because variants are in-line USD namespaced paths.

[ 我们可以通过命名空间编辑创建变体，因为变体是内联的 USD 命名空间路径]

~~~admonish tip title="Sdf.BatchNamespaceEdit | Moving prim specs into variants | Click to expand!" collapsible=true
```python
{{#include ../../../code/production/production.py:productionConceptsSdfBatchNamespaceEditVariant}}
```
~~~

We also cover variants in detail in respect to Houdini in our [Houdini - Tips & Tricks](../dcc/houdini/faq/overview.md) section.

[ 我们还在 [Houdini - Tips & Tricks](../dcc/houdini/faq/overview.md) 部分详细介绍了 Houdini 的变体]

### Copying data with Sdf.CopySpec <a name="sdfCopySpec"></a>
We use the `Sdf.CopySpec` method to copy/duplicate content from layer to layer (or within the same layer).

[ 我们使用 Sdf.CopySpec 方法在图层之间（或在同一图层内）复制内容]

#### Copying specs (prim and properties) from layer to layer with Sdf.CopySpec()
The `Sdf.CopySpec` can copy anything that is representable via the `Sdf.Path`. This means we can copy prim/property/variant specs.
When copying, the default is to completely replace the target spec.

[ Sdf.CopySpec 可以复制通过 Sdf.Path 表示的任何内容. 这意味着我们可以复制 prim/property/variant 复制时，默认是完全替换目标spec]

We can filter this by passing in filter functions. Another option is to copy the content to a new anonymous layer and then
merge it via `UsdUtils.StitchLayers(<StrongLayer>, <WeakerLayer>)`. This is often more "user friendly" than implementing
a custom merge logic, as we get the "high layer wins" logic for free and this is what we are used to when working with USD.

[ 我们可以通过传入过滤函数来过滤它. 另一种选择是将内容复制到新的匿名层，然后通过 UsdUtils.StitchLayers(\<StrongLayer\>, \<WeakerLayer\>) 合并它. 这通常比实现自定义合并逻辑更“用户友好”，因为我们免费获得“高层获胜”逻辑，这就是我们在使用 USD 时所习惯的]

~~~admonish question title="Still under construction!"
We'll add some examples for custom filtering at a later time.

[ 我们稍后将添加一些自定义过滤的示例]
~~~

~~~admonish tip title="Sdf.CopySpec | Copying prim/property specs | Click to expand!" collapsible=true
```python
{{#include ../../../code/production/production.py:productionConceptsSdfCopySpecStandard}}
```
~~~

#### Using Sdf.CopySpec() for variant creation
We can also use `Sdf.CopySpec` for copying content into a variant.

[ 我们还可以使用 Sdf.CopySpec 将内容复制到变体中]

~~~admonish tip title="Sdf.CopySpec | Copying prim specs into variants | Click to expand!" collapsible=true
```python
{{#include ../../../code/production/production.py:productionConceptsSdfCopySpecVariant}}
```
~~~

We also cover variants in detail in respect to Houdini in our [Houdini - Tips & Tricks](../dcc/houdini/faq/overview.md) section.

[ 我们还在 [Houdini - Tips & Tricks](../dcc/houdini/faq/overview.md) 部分详细介绍了 Houdini 的变体]

### Delaying change notifications with the Sdf.ChangeBlock <a name="sdfChangeBlock"></a>

[ 使用 Sdf. ChangeBlock 延迟更改通知]

Whenever we edit something in our layers, change notifications get sent to all consumers (stages/hydra delegates) that use the layer. This causes them to recompute and trigger updates.

[ 每当我们在图层中编辑某些内容时，更改通知都会发送给使用该图层的所有使用者（stage/Hydra 委托）. 这会导致它们重新计算并触发更新]

When performing a large edit, for example creating large hierarchies, we can batch the edit, so that the change notification gets the combined result.

[ 当执行大型编辑时，例如创建大型层次结构，我们可以批量编辑，以便更改通知获得组合结果]

~~~admonish danger title="Pro Tip | When/How to use Sdf.ChangeBlocks"
In theory it is only safe to use the change block with the lower level Sdf API.
We can also use it with the high level API, we just have to make sure that we don't accidentally query an attribute, that we just overwrote or perform ops on deleted properties.

[ 理论上，只有将更改块与较低级别的 Sdf API 一起使用才是安全的. 我们还可以将它与高级 API 一起使用，我们只需要确保我们不会意外查询属性，我们只是覆盖或对已删除的属性执行操作]

We therefore recommend work with a read/write code pattern:

[ 因此，我们建议使用 read/write 代码模式]

- We first query all the data via the Usd high level API

    [ 我们首先通过 Usd 高级 API 查询所有数据]
- We then write our data via the Sdf low level API

    [ 然后我们通过 Sdf 低级 API 写入数据]

When writing data, we can also write it to a temporary anonymous layer, that is not linked to a stage and then merge the result back in via `UsdUtils.StitchLayers(anon_layer, active_layer)`. This is a great solution when it is to heavy to query all data upfront.

[ 写入数据时，我们还可以将其写入未链接到 stage 的临时匿名层，然后通过 UsdUtils.StitchLayers(anon_layer, active_layer) 将结果合并回来. 当预先查询所有数据过于繁重时，这是一个很好的解决方案]
~~~

For more info see the [Sdf.ChangeBlock](https://openusd.org/dev/api/class_sdf_change_block.html) API docs.

[ 有关更多信息，请参阅 [Sdf.ChangeBlock](https://openusd.org/dev/api/class_sdf_change_block.html) API 文档]

~~~admonish tip title=""
```python
{{#include ../../../code/production/production.py:productionConceptsSdfChangeBlock}}
```
~~~

## Relationships <a name="relationship"></a>

### Special Relationships <a name="relationshipSpecial"></a>
~~~admonish question title="Still under construction!"
This sub-section is still under development, it is subject to change and needs extra validation.

[ 该子部分仍在开发中，可能会发生变化并且需要额外的验证]
~~~
USD has a few "special" relationships that infer information on our hierarchy. These are:
- `proxyPrim`: This is a relationship from a prim with a render purpose to a prim with a proxy purpose. It can be used by clients to get low-resolution proxy representations (for example for simulations/collision detection etc.). Setting it is optional.

[ USD 有一些“特殊”关系，可以推断出有关我们的层次结构的信息. 它们是： - `proxyPrim`：这是从渲染 prim 到代理 prim 的关系. 客户端可以使用它来获取低分辨率代理（例如用于模拟/碰撞检测等）. 这个设置是可选的]

These special relationships have primvar like inheritance from parent to child prims:

[ 这些特殊关系类似 primvar 的从父 prims 到子 prims 的继承]

- `material:binding`: This controls the material binding.

    [ material:binding ：控制材质绑定]
- `coordSys:<customName>`: Next to collections, this is currently the only other multi-apply API schema that ships natively with USD. It allows us to target an xform prim, whose transform we can then read into our shaders at render time.

    [ coordSys:\<customName\> ：除了集合之外，这是目前唯一与 USD 一起原生提供的其他多应用 API 模式. 它允许我们以 xform prim 为目标，然后我们可以在渲染时将其转换读入着色器]
- `skel:skeleton`/`skel:animationSource`:
    - `skel:skeleton`: This defines what skeleton prims with the skelton binding api schema should bind to.

        [ skel:skeleton ：这定义了骨架绑定 api 模式的骨架 prims 应绑定到的对象]
    - `skel:animationSource`: This relationship can be defined on mesh prims to target the correct animation, but also on the skeletons themselves to select the skeleton animation prim.

        [ skel:animationSource ：可以在 mesh prims 上定义此关系以定位正确的动画，也可以在骨骼本身上定义选择的骨骼动画物体]

### Relationship Forwarding (Binding post) <a name="relationshipForwarding"></a>
~~~admonish question title="Still under construction!"
This sub-section is still under development, it is subject to change and needs extra validation.

[ 该子部分仍在开发中，可能会发生变化并且需要额外的验证]
~~~

### Collections <a name="relationshipCollection"></a>
We cover collections in detail in our [collection section](../core/elements/collection.md#collectionQuery) with advanced topics like inverting or compressing collections.

[ 我们在 [集合](../core/elements/collection.md#collectionQuery) 部分有详细介绍，其中包括 inverting or compressing collections 等高级主题]