# Particles
Importing particles (points) is the simplest form of geometry import.

[ 导入粒子（点）是几何导入的最简单形式]

Let's see how we can make it complicated 🤪. We'll take a look at these two cases:

[ 让我们看看如何让它变得复杂🤪.我们来看看这两个案例]
- Houdini native point import
- Render overrides via (Numpy) Python wrangles

You can find all the .hip files of our shown examples in our [USD Survival Guide - GitHub Repo](https://github.com/LucaScheller/VFX-UsdSurvivalGuide/tree/main/files/dcc/houdini).

[您可以在我们的 [USD Survival Guide - GitHub Repo](https://github.com/LucaScheller/VFX-UsdSurvivalGuide/tree/main/files/dcc/houdini) 中找到我们所示示例的所有 .hip 文件]

For all options for SOP to LOP importing, check out the official [Houdini docs](https://www.sidefx.com/docs/houdini/solaris/sop_import.html).

[ 有关 SOP 到 LOP 导入的所有选项，请查看官方 [Houdini docs](https://www.sidefx.com/docs/houdini/solaris/sop_import.html)]

## Houdini Native Import [Houdini 原生导入]
When importing points, all you need to set is a path attribute on your points (rather than on prims as with polygon meshes), because we don't have any prims on sop level. (Thanks captain obvious).

[ 导入点时，您需要设置的只是点上的路径属性（而不是像多边形网格那样在 prims 上），因为我们在 sop 级别上没有任何 prims（Thanks captain obvious）]

For an exercise, let's build a simple SOP import ourselves. Should we use this in production: No, Houdini's geo conversion is a lot faster, when it comes to segmenting your input based on the path attribute. Nonetheless it is a fun little demo:

[ 作为练习，我们自己构建一个简单的 SOP 导入. 我们是否应该在生产中使用它：不，当涉及到基于路径属性对输入进行分段时，Houdini 的geo conver要快得多.尽管如此，它还是一个有趣的小演示]

~~~admonish tip title=""
```python
{{#include ../../../../../code/dcc/houdini.py:houdiniPointsNativeStream}}
```
~~~

<video width="100%" height="100%" controls autoplay muted loop>
  <source src="./media/pointsNativeStream.mp4" type="video/mp4" alt="Houdini Python Wrangle">
</video>

Why should we do it ourselves, you might ask? Because there are instances, where we can directly load in the array, because we know we are only editing a single prim. Some of Houdini's nodes actually use this mechanism in a few of the point instancer related nodes.

[ 您可能会问，为什么我们要自己做呢？因为在某些情况下，我们可以直接加载到数组中，因为我们知道我们只编辑一个 prim. Houdini的一些节点实际上在一些与点实例器相关的节点中使用了这种机制]

## Render-Time Overrides via (Numpy) Python Wrangles [通过 (Numpy) Python Wrangles 进行渲染时覆盖]
Now you might be thinking, is Python performant enough to actually manipulate geometry?

[ 现在您可能会想，Python 的性能是否足以实际操作几何图形？]

In the context of points (also point instancers), we answer is yes. As we do not have to do geometry operations, manipulating points is "just" editing arrays. This can be done very efficiently via numpy, if we use it for final tweaking. So don't expect to have the power of vex, the below is a "cheap" solution to adding render time overrides, when you don't have the resources to write your own compiled language (looking at your [DNEG (OpenVDB AX)](https://www.openvdb.org/documentation/doxygen/openvdbax.html)).

[ 在点（也是点实例）的背景下，我们的答案是肯定的. 由于我们不必进行几何操作，因此操作点“只是”编辑数组. 如果我们使用 numpy 进行最终调整，这可以非常有效地完成.因此，不要指望拥有 vex 的强大功能，当您没有资源编写自己的编译语言时（查看您的 [DNEG (OpenVDB AX)](https://www.openvdb.org/documentation/doxygen/openvdbax.html)），下面是添加渲染时间覆盖的“廉价”解决方案]

In the near future, this can probably also be done by Houdini's render procedurals (it actually already can be). The method we show here, is DCC independent though, so it does have its benefits, because you don't need a Houdini (engine) license.

[ 在不久的将来，这可能也可以通过 Houdini 的渲染程序来完成（实际上已经可以了）.我们在这里展示的方法虽然独立于 DCC，但它确实有其优点，因为您不需要 Houdini（引擎）许可证]

To showcase how we manipulate arrays at render time, we've built a "Python Wrangle" .hda. Here is the basic .hda structure:

[ 为了展示我们如何在渲染时操作数组，我们构建了一个“Python Wrangle”.hda 这是基本的 .hda 结构]

<video width="100%" height="100%" controls autoplay muted loop>
  <source src="./media/pointsPythonWrangleOverview.mp4" type="video/mp4" alt="Houdini Python Wrangle">
</video>

As discussed in our [Creating efficient LOPs .Hdas](../hda/overview.md) section, we start and end the Hda with a new layer to ensure that we don't "suffer" from the problem of our layer getting to data heavy. Then we have two python nodes: The first one serializes the Hda parms to a json dict and stores it on our point prims, the second one modifies the attributes based on the parm settings. Why do we need to separate the data storage and execution? Because we want to only opt-in into the python code execution at render-time. So that's why we put down a switch node that is driven via a context variable. Context variables are similar to global Houdini variables, the main difference is they are scoped to a section of our node graph or are only set when we trigger a render.

[ 正如我们在创建高效的 [Creating efficient LOPs .Hdas](../hda/overview.md) 部分中所讨论的，我们以新层开始和结束 Hda，以确保我们不会因层数据量过大而“遭受”问题. 然后我们有两个 python 节点：第一个将 Hda parms 序列化为 json 字典并将其存储在我们的点 prims 上，第二个节点根据 parm 设置修改属性. 为什么我们需要将数据存储和执行分开？因为我们只想在渲染时选择加入 python 代码执行.这就是为什么我们放置一个通过上下文变量驱动的开关节点.上下文变量与全局 Houdini 变量类似，主要区别在于它们的范围仅限于节点图的一部分，或者仅在我们触发渲染时设置]

<video width="100%" height="100%" controls autoplay muted loop>
  <source src="./media/pointsPythonWrangleHusk.mp4" type="video/mp4" alt="Houdini Python Wrangle Husk">
</video>

This means that when rendering the USD file to disk, all the points will store is our wrangle code (and the original point data, in production this usually comes from another already cached USD file that was payloaded in). In our pre render scripts, we can then iterate over our stage and execute our code.

[ 这意味着，当将 USD 文件渲染到磁盘时，所有点将存储我们的 wrangle 代码（以及原始点数据，在生产中这通常是通过 payload 加载的另一个已缓存的 USD 文件）.在我们的预渲染脚本中，我们可以通过迭代 stage 并执行我们的代码]

Let's talk about performance: The more attributes we manipulate, the slower it will get. To stress test this, let's try building a point replicator with a constant seed. To "upres" from 1 million to 10 million points, it takes around 30 seconds. For this being a "cheap" solution to implement, I'd say that is manageable for interactivity. Now we could also do a similar thing by just using a point instancer prim and upres-ing our prototypes, using this method allows for per point overrides though, which gives us more detail.

[ 让我们谈谈性能：我们操作的属性越多，速度就越慢.为了对此进行压力测试，让我们尝试构建一个具有恒定种子的点复制器,从 100 万点“升级”到 1000 万点，大约需要 30 秒.由于这是一个“廉价”的实施解决方案，我认为这对于交互性来说是可以管理的.现在我们也可以通过仅使用点实例器 prim 并升级我们的原型来完成类似的事情，但使用此方法允许每个点覆盖，这为我们提供了更多细节]

Here is a demo of a point replicator:

[ 这是点复制器的演示]

<video width="100%" height="100%" controls autoplay muted loop>
  <source src="./media/pointsPythonWranglePointReplicate.mp4" type="video/mp4" alt="Houdini Python Wrangle Husk">
</video>

Another cool thing is, that this is actually not limited to points prims (We lied in our intro 😱). Since all attributes are is arrays of data, we can run the python wrangle on any prim. For example if we just wan't to increase our pscale width or multiply our velocities, operating on an array via numpy is incredibly fast, we're talking a few 100 milliseconds at most for a few millions points. As mentioned in our [data types](../../../core/elements/data_type.md) section, USD implements the buffer protocol, so we don't actually duplicate any memory until really needed and mapping `Vt.Array`s to numpy is as straight forward as casting the array to a numpy array.

[ 另一个很酷的事情是，这实际上不限于 points prims（我们在介绍中撒了谎😱）. 由于所有属性都是数组数据，因此我们可以在任何 prim 上运行 python wrangle. 例如，如果我们只是不想增加 pscale width 或 multiply velocities，通过 numpy 对数组进行操作的速度非常快，我们所说的数百万个点最多需要 100 毫秒.正如我们的[数据类型](../../../core/elements/data_type.md)部分中提到的，USD 实现了缓冲区协议，因此在真正需要用到之前我们实际上不会复制任何内存数据，并且将 Vt.Array 映射到 numpy 就像将数组转换为 numpy 数组一样简单]


Now the below code might look long, but the import bits are:

[ 现在下面的代码可能看起来很长，但是重要的部分]

- Getting the data: `np.array(attr.Get(frame)`
- Setting the data: `attr.Set(attr.GetTypeName().type.pythonClass.FromNumpy(output_data[binding.property_name]), frame))`
- Updating the extent hint: `UsdGeom.Boundable.ComputeExtentFromPlugins(boundable_api, frame)`

~~~admonish tip title="Python Wrangle Hda | Summary |  Click to expand!" collapsible=true
```python
    ...
    # Read data
    input_data[binding.property_name] = np.array(attr.Get(frame))
    ...
    # Write data
    for binding in bindings:
        attr = prim.GetAttribute(binding.property_name)
        if len(output_data[binding.property_name]) != output_point_count:
            attr.Set(pxr.Sdf.ValueBlock())
            continue
        attr_class = attr.GetTypeName().type.pythonClass
        attr.Set(attr_class.FromNumpy(output_data[binding.property_name]), frame)
    ...
    # Re-Compute extent hints
    boundable_api = pxr.UsdGeom.Boundable(prim)
    extent_attr = boundable_api.GetExtentAttr()
    extent_value = pxr.UsdGeom.Boundable.ComputeExtentFromPlugins(boundable_api, frame)
    if extent_value:
        extent_attr.Set(extent_value, frame)
```
~~~


The code for our "python kernel" executor:

[ 我们的“python kernel”执行器的代码]

~~~admonish tip title="Python Wrangle Hda | Python Kernel | Click to expand!" collapsible=true
```python
{{#include ../../../../../files/dcc/houdini/points/pythonWrangle.py}}
```
~~~

The code for our pre render script:

[ 我们的预渲染脚本的代码]

~~~admonish tip title="Python Wrangle Hda | Pre-Render Script | Click to expand!" collapsible=true
```python
{{#include ../../../../../files/dcc/houdini/points/renderPreFrame.py}}
```
~~~
