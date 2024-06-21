# Point Instancers ('Copy To Points') [点实例化 ('Copy To Points')]
We have four options for mapping Houdini's packed prims to USD:

[ 我们有四种选项可以将 Houdini 的打包 prims 映射到 USD]

- As transforms
- As point instancers
- As deformed geo (baking down the xform to actual geo data)
- As skeletons, more info in our [RBD section](./rbd.md)

~~~admonish tip title="Pro Tip | When to use PointInstancer prims?"
We'll always want to use USD's PointInstancer prims, when representing a "replicate a few unique meshes to many points" scenario.
In SOPs we usually do this via the "Copy To Points" node.

[ 在表示“将几个独特的网格复制到多个点”的场景时，我们始终希望使用USD的 PointInstancer prims. 在SOP中，我们通常通过"Copy To Points"节点来完成此操作]
~~~

You can find all the .hip files of our shown examples in our [USD Survival Guide - GitHub Repo](https://github.com/LucaScheller/VFX-UsdSurvivalGuide/tree/main/files/dcc/houdini).

[ 您可以在我们的[USD Survival Guide - GitHub Repo](https://github.com/LucaScheller/VFX-UsdSurvivalGuide/tree/main/files/dcc/houdini) 中找到我们所示示例的所有 .hip 文件]

For all options for SOP to LOP importing, check out the official [Houdini docs](https://www.sidefx.com/docs/houdini/solaris/sop_import.html).

[ 有关 SOP 到 LOP 导入的所有选项，请查看官方 [Houdini docs](https://www.sidefx.com/docs/houdini/solaris/sop_import.html)]

In the below examples, we use the `path`/`name` attributes to define the prim path. You can actually configure yourself what attributes Houdini should use for defining our prim paths on the "SOP import node" LOP/"USD Configure" SOP node.

[ 在下面的示例中，我们使用 path / name 属性来定义 prim 路径. 实际上，您可以自己配置 Houdini 应使用哪些属性来定义 prim path .通过 LOP 中的 "SOP import node" 或 SOP 中的 "USD Configure node" 节点来设置 ]

## Houdini Native Import (and making it convenient for artists)
To import our geometry to "PointInstancer" USD prims, we have to have it as packed prims in SOPs. If you have nested packed levels, they will be imported as nested point instancers. We do not recommend doing this, as it can cause some confusing outputs. The best practice is to always have a "flat" packed hierarchy, so only one level of packed prims.

[ 要将几何图形导入 USD prims 中 “PointInstancer” ，我们必须将 prims 在 SOP 中完成打包. 如果您有嵌套的打包级别，它们将作为嵌套点实例器导入.我们不建议这样做，因为它可能会导致一些令人困惑的输出. 最佳实践是始终具有“扁平”打包层级结构，即只有一层的打包 prim]

Houdini gives us the following options for importing:

[ Houdini 为我们提供了以下导入选项]

- The `usdinstancerpath` attribute defines the location of our `PointInstancer` prim.

    [ usdinstancerpath 属性定义 PointInstancer prim 的位置]
- The `path`/`name` attribute defines the location of the prototype prims. Prototypes are the unique prims that should get instances, they are similar to the left input on your "copy to points" node.

    [ path / name 属性定义 prototype prims 的位置. Prototypes 是获取 instances 的特殊 prims，它类似于 "copy to points" 节点上的左侧输入]

An important thing to note is, that if your `path`/`name` attribute does not have any `/` slashes or starts with `./`, the prototypes will be imported with the following path: `<usdinstancerpath>/Prototypes/<pathOrName>`. Having the `/Prototypes` prim is just a USD naming convention thing.

[ 需要注意的重要一点是，如果您的 path / name 属性没有任何 / 斜杠或以 ./ 开头，则 prototypes 将通过以下路径导入： \<usdinstancerpath\>/Prototypes/\<pathOrName\> 拥有 /Prototypes prim 只是 USD 命名约定的事情]

To make it easy to use for artists, we recommend mapping the traditional path attribute value to `usdinstancerpath` and making sure that the `name` attribute is relative.

[ 为了方便艺术家使用，我们建议将传统 path 的属性值映射到 usdinstancerpath 并确保 name 属性是相对的]

Another important thing to know about packed prims is, that the `path`/`name` attributes are also used to define the hierarchy within the packed prim content. So before you pack your geometry, it has to have a valid path value.

[ 关于打包 prim 需要了解的另一件重要事情是， path / name 属性还用于定义打包 prim 内容中的层级结构. 因此，在打包几何图形之前，它必须具有有效的路径值]

Good, now that we know the basics, let's have a look at a not so expectable behavior:
If you remember, in our [Basic Building Blocks of Usd](../../../core/elements/overview.md) section, we explained that relationships can't be animated. Here's the fun part:

[ 好，现在我们已经了解了基础知识，让我们来看看一个不太令人期待的行为：如果您还记得，在我们的 [Basic Building Blocks of Usd](../../../core/elements/overview.md) 中，我们解释了 relationships  无法动画化. 这是有趣的部分]

~~~admonish danger title="PointInstancer | Varying Prototypes | Problem"
The mapping of what point maps to what prototype prim is stored via the `protoIndices` attribute. This maps an index to the prim paths targetd by the `prototypes` relationship. Since relationships can't be animated, the `protoIndices`/`prototypes` properties has to be aware of all prototypes, that ever get instanced across the whole cache.

[ 哪个点到哪个 prototype prim 的映射通过 protoIndices 属性存储.这会将 index <==> prim paths 的关系通过 prototypes relationship 映射. 由于 relationship 无法进行动画处理，因此 protoIndices / prototypes 属性必须知道整个缓存中所有 instanced  的 prototypes]
~~~

This is the reason, why in our LOPs instancer node, we have to predefine all prototypes. The problem is in SOPs, it kind of goes against the usual artist workflow. For example when we have debris instances, we don't want to have the artist managing to always have at least one copy of all unique prims we are trying to instance.

[ 这就是为什么在我们的 LOP instancer 节点中，我们必须预定义所有 prototypes .问题出在 SOP 中，它有点违背艺术家通常的工作流程.例如，当我们处理破碎实例，在尝试实例化的所有特定 prim 时候,我们不希望艺术家来管理任何副本.]


~~~admonish tip title="PointInstancer | Varying Prototypes | Solution"
The artist should only have to ensure a unique `name` attribute value per unique instance and a valid `usdinstancerpath` value.
Making sure the protoIndices don't jump, as prototypes come in and out of existence, is something we can fix on the pipeline side.

[ 艺术家只需确保每个唯一实例都有唯一的 name 属性值和有效的 usdinstancerpath 值. 确保 protoIndices 不会随着原型的出现和消失而跳转，这是我们可以在流程方面修复的问题]
~~~

Luckily, we can fix this behavior, by tracking the prototypes ourselves per frame and then re-ordering them as a post process of writing our caches.

[ 幸运的是，我们可以通过每帧跟踪原型来修复此行为，然后在写入缓存的后期过程中对它们重新排序]

Let's take a look at the full implementation:

[ 我们看一下完整的实现]

<video width="100%" height="100%" controls autoplay muted loop>
  <source src="./media/pointInstancerPrototypeReorder.mp4" type="video/mp4" alt="Houdini Prototype Re-Order">
</video>

As you can see, all looks good, when we only look at the active frame, because the active frame does not know about the other frames. As soon as we cache it to disk though, it "breaks", because the protoIndices map to the wrong prototype.

[ 正如您所看到的，当我们只查看活动框架时，一切看起来都很好，因为活动框架不知道其他框架。但是，一旦我们将其缓存到磁盘，它就会“崩溃”，因为 protoIndices 映射到错误的原型]

All we have to do is create an attribute, that per frame stores the relation ship targets as a string list. After the cache is done, we have to map the wrong prototype index to the write one.

[ 我们所要做的就是创建一个属性，该属性每帧将关系目标存储为字符串列表. 缓存完成后，我们必须将错误的原型索引映射到写入索引]

Here is the tracker script:

[ 这是跟踪器脚本]

~~~admonish tip title="PointInstancer | Re-Order Prototypes | Track Prototypes | Click to expand" collapsible=true
```python
{{#include ../../../../../code/dcc/houdini.py:houdiniPointInstancerReorderTracker}}
```
~~~

And here the post processing script. You'll usually want to trigger this after the whole cache is done writing. It also works with value clips, you pass in all the individual clip files into the layers list. This is also another really cool demo, of how numpy can be used to get C++ like performance.

[ 这里是后处理脚本.您通常希望在整个缓存写入完成后触发此操作.它也适用于值剪辑，您将所有单独的剪辑文件传递到图层列表中.这也是另一个非常酷的演示，展示了如何使用 numpy 获得类似 C++ 的性能]

~~~admonish tip title="PointInstancer | Re-Order Prototypes | Track Prototypes | Click to expand" collapsible=true
```python
{{#include ../../../../../code/dcc/houdini.py:houdiniPointInstancerReorderPostProcess}}
```
~~~

Phew, now everything looks alright again!

[ 唷，现在一切看起来都恢复正常了！]

## Performance Optimizations [性能优化]
You may have noticed, that we always have to create packed prims on SOP level, to import them as PointInstancer prims. If we really want to go all out on the most high performance import, we can actually replicate a "Copy To Points" import. That way we only have to pass in the prototypes and the points, but don't have the overhead of spawning the packed prims in SOPs.

[ 您可能已经注意到，我们总是必须在 SOP 级别创建打包 prim，以将它们导入为 PointInstancer prim. 如果我们真的想全力以赴实现最高性能的导入，我们实际上可以复制"Copy To Points" 导入. 这样我们只需要传入原型和点，但没有在 SOP 中生成打包 prim 的开销]

Is this something you need to be doing? No, Houdini's LOPs import as well as the packed prim generation are highly performant, the following solution is really only necessary, if you are really picky about making your export a few hundred milliseconds faster with very large instance counts.

[ 这是你需要做的事情吗？不，Houdini 的 LOP 导入以及打包的 prim 生成都具有高性能，如果您真的很挑剔在实例数量非常大的情况下使导出速度加快几百毫秒，那么以下解决方案实际上是必要的]

<video width="100%" height="100%" controls autoplay muted loop>
  <source src="./media/pointInstancerPerformance.mp4" type="video/mp4" alt="Houdini Prototype Re-Order">
</video>

As you can see we are at a factor 20 (1 seconds : 50 milliseconds). Wow! Now what we don't show is, that we actually have to conform the point instances attributes to what the PointInstancer prim schema expects. So the ratio we just mentioned is the best case scenario, but it can be a bit slower, when we have to map for example `N`/`up` to `orientations`. This is also only this performant because we are importing a single PointInstancer prim, which means we don't have to segment any of the protoIndices.

[ 正如您所看到的，我们的速度是 20 倍（1 秒：50 毫秒）.哇！现在我们没有展示的是，我们实际上必须使点实例属性符合 PointInstancer prim 模式所期望的. 因此，我们刚刚提到的比率是最好的情况，但当我们必须将 N / up 映射到 orientations 时，它可能会慢一些. 这也是唯一的性能，因为我们正在导入单个 PointInstancer prim，这意味着我们不必对任何 protoIndices 进行分段]

We also loose the benefit of being able to work with our packed prims in SOP level, for example for collision detection etc.

[ 我们还失去了能够在 SOP 级别使用我们的打包 prim 的好处，例如用于碰撞检测等]

Let's look at the details:

[ 我们来看看细节]

<video width="100%" height="100%" controls autoplay muted loop>
  <source src="./media/pointInstancerPerformanceDetails.mp4" type="video/mp4" alt="Houdini Prototype Re-Order">
</video>

On SOP level:

[ 在SOP层面]
- We create a "protoIndices" attribute based on all unique values of the "name" attribute

    [ 我们根据“name”属性的所有唯一值创建“protoIndices”属性]
- We create a "protoHash" attribute in the case we have multiple PointInstancer prim paths, so that we can match the prototypes per instancer

    [ 如果我们有多个 PointInstancer prim 路径，我们创建一个“protoHash”属性，以便我们可以匹配每个实例的原型]
- We conform all instancing related attributes to have the correct precision. This is very important, as USD does not allow other precisions types than what is defined in the PointInstancer schema.

    [ 我们使所有实例相关的属性都具有正确的精度. 这非常重要，因为 USD 不允许使用除 PointInstancer 模式中定义的精度类型之外的其他精度类型]
- We conform the different instancing attributes Houdini has to the attributes the PointInstancer schema expects. (Actually the setup in the video doesn't do this, but you have to/should, in case you want to use this in production)

    [ 我们将 Houdini 具有的不同实例属性与 PointInstancer 模式期望的属性相一致. （实际上视频中的设置并没有这样做，但你必须/应该这样做，以防你想在生产中使用它）]

On LOP level:

[ 在LOP级别]
- We import the points as a "Points" prim, so now we have to convert it to a "PointInstancer" prim. For the prim itself, this just means changing the prim type to "PointInstancer" and renaming "points" to "positions".

    [ 我们将点作为“Points”prim 导入，因此现在我们必须将其转换为“PointInstancer”prim. 对于 prim 本身，这仅意味着将 prim 类型更改为“PointInstancer”并将“points”重命名为“positions”]
- We create the "prototypes" relationship property.

    [ 我们创建“prototypes”关系属性]

~~~admonish tip title="PointInstancer | Custom Import | Click to expand" collapsible=true
```python
{{#include ../../../../../code/dcc/houdini.py:houdiniPointInstancerNativeStream}}
```
~~~