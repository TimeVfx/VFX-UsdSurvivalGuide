# Transforms
~~~admonish question title="Still under construction!"
This section still needs some more love, we'll likely expand it more in the near future.

[ 这个部分还需要进一步完善，我们可能会在不久的将来对其进行更多的扩展]
~~~

# Table of Contents [目录]
1. [Transforms In-A-Nutshell](#summary)
1. [What should I use it for?](#usage)
1. [Resources](#resources)
1. [Overview](#overview)
    1. [Creating and animating transforms](#xformCreate)
    2. [Ignoring parent transforms by resetting the xform stack](#xformResetXformStack)
    1. [Querying transforms](#xformQuery)
1. [Transforms in production](#xformProduction):
    1. [Merging hierarchy transforms](#xformWorldSpaceToLocalSpace)
    1. [Baking transforms for constraint like behaviour](#xformBake)
    1. [Reading Xforms in Shaders](xformCoordinate)

## TL;DR - Transforms In-A-Nutshell [概述]<a name="summary"></a>
Transforms are encoded via the following naming scheme and attributes:

[ Transforms 通过以下命名控件和属性进行编码]

- **xformOpOrder**: This (non-animatable) attribute controls what **xformOp:** namespaced attributes affect the prims local space transform.

  [ xformOpOrder：此（不可动画）属性控制 xformOp：命名空间的属性影响 prims 局部空间的变换]
- **xformOp:**: Xform ops are namespaced with this namespace and can be considered by the "xformOpOrder" attribute. We can add any number of xform ops for xformable prims, the final world transform is then computed based on a prims local transform and that of all its ancestors.

  [ xformOp:: Xform ops 以此命名空间命名，并可以明确 “xformOpOrder” 的属性,我们可以为 xformable prims 添加任意数量的 xform 操作，然后根据 prims 局部变换及其所有祖先的局部变换来计算最终的世界变换]

## What should I use it for? <a name="usage"></a>

[ 我应该用它做什么？]

~~~admonish tip
We rarely write the initial transforms ourselves, this is something our DCCs excel at. We do query transforms though for different scenarios:

[ 我们通常不会自己编写初始变换这是 DCCs 所擅长的. 但是我们会针对不同的场景查询变换]
- We can bake down the transform to a single prim. This can then be referenced or inherited and used as a parent constraint like mechanism.

  [ 我们可以将变换烘焙到一个单一的prim上. 然后可以引用或继承它，并将其用作类似于父级约束的机制]
- When merging hierarchies, we often want to preserve the world transform of leaf prims. Let's say we have two stages: We can simply get the parent xform of stage A and then apply it in inverse to our leaf prim in stage B. That way the leaf prim in stage B is now in local space and merging the stages returns the expected result. We show an example of this below.

  [ 在合并层级结构时，我们通常希望保留叶子 prims 的世界变换. 假设我们有两个 stage ：我们可以简单地获取 A stage 的父级xform，然后将其逆变换应用于 B stage 中的叶子prim. 这样 B stage 中的叶子 prim 现在处于局部空间合并 stage 会返回预期的结果. 我们将在下面展示一个这样的例子]
~~~

## Resources [资源]<a name="resources"></a>
- [UsdGeom Xforms](https://openusd.org/dev/api/usd_geom_page_front.html)
- [UsdGeom.Xformable](https://openusd.org/dev/api/class_usd_geom_xformable.html)
- [UsdGeom.XformCommonAPI](https://openusd.org/dev/api/class_usd_geom_xform_common_a_p_i.html)

## Overview [概述]<a name="overview"></a>
Creating xforms is usually handled by our DCCS. Let's go over the basics how USD encodes them to understand what we are working with.

[ 创建 xforms 通常由我们的 DCC 软件来处理. 让我们回顾一下 USD 是如何编码以便了解我们正在处理的内容]

All shown examples can be found in the [xforms .hip file](https://github.com/LucaScheller/VFX-UsdSurvivalGuide/tree/main/files/dcc/houdini/transforms) in our GitHub repo.

[ 所有显示的示例都可以在我们的 GitHub 存储库的 [xforms .hip file](https://github.com/LucaScheller/VFX-UsdSurvivalGuide/tree/main/files/dcc/houdini/transforms) 文件中找到]

### Creating and animating transforms <a name="xformCreate"></a>
USD evaluates xform attributes on all sub types of the xformable schema.

[ USD 通过所有子类型上的 xformable schema 来计算 xform 属性]

![Usd Xformable](xformXformableClassInheritance.jpg)

Transforms are encoded via the following naming scheme and attributes:

[ Transforms 是通过以下命名方案和属性进行编码的]

- **xformOpOrder**: This (non-animatable) attribute controls what **xformOp:** namespaced attributes affect the prims local space transform.

  [ xformOpOrder：此（不可动画）属性控制 xformOp：命名空间的属性影响 prims 局部空间的变换]
- **xfromOp:**: Xform ops are namespaced with this namespace and can be considered by the "xformOpOrder" attribute.

    [ xformOp:: Xform ops 以此命名空间命名，并可以明确 “xformOpOrder” 的属性]
    - We can add any number of xform ops to xformable prims.

        [ 我们可以向 xformable prims 添加任意数量的 xform 操作]
    - Any xform op can be suffixed with a custom name, e.g. xformOp:translate:myCoolTranslate

        [ 任何 xform op 都可以带有自定义名称后缀，例如 xformOp:translate:myCoolTranslate]
    - Available xform Ops are:

        [ 可用的 xform 操作有]
        - xformOp:translate
        - xformOp:orient
        - xformOp:rotateXYZ, xformOp:rotateXZY, xformOp:rotateYXZ, xformOp:rotateYZX, xformOp:rotateZXY, xformOp:rotateZYX
        - xformOp:rotateX, xformOp:rotateY, xformOp:rotateZ
        - xformOp:scale
        - xformOp:transform

The final world transform is computed based on a prims local transform and that of all its ancestors.

[ 最终的世界变换是基于 prims 局部变换及其所有祖先的局部变换计算的]

~~~admonish info title=""
```python
{{#include ../../../../code/core/elements.py:xformXformableOverview}}
```
~~~

Here is the snippet in action:

[ 这是动态演示片段]

<video width="100%" height="100%" controls autoplay muted loop>
  <source src="./xformCreation.mp4" type="video/mp4" alt="Houdini Xform Creation">
</video>

### Ignoring parent transforms by resetting the xform stack <a name="xformResetXformStack"></a>
We can also set the special '!resetXformStack!' value in our "xformOpOrder" attribute to reset the transform stack. This means all parent transforms will be ignored, as well as any attribute before the '!resetXformStack!' in the xformOp order list.

[ 我们还可以在 “xformOpOrder” 属性中设置特殊的 “!resetXformStack!” 值来重置变换堆栈. 这意味着所有父级变换都将被忽略，以及 xformOp 顺序列表中 “!resetXformStack!” 之前的任何属性也将被忽略]

~~~admonish danger title=""
Resetting the xform stack is often not the right way to go, as we loose any parent hierarchy updates. We also have to make sure that we write our reset-ed xform with the correct sub-frame time samples, so that motion blur works correctly.

[ 重置 xform 堆栈通常不是正确的方法，因为我们会丢失任何父级层级结构的更新. 我们还必须确保使用正确的子帧时间样本写入重置后的xform，以确保运动模糊能够正确工作]

This should only be used as a last resort to enforce a prim to have a specific transform. We should rather re-write leaf prim xform in local space, see [Merging hierarchy transforms](#xformWorldSpaceToLocalSpace) for more info.

[ 这应仅作为最后手段来使用，以强制 prim 具有特定的变换. 我们最好以本地空间重写叶子 prim 的 xform，有关更多信息，请参阅 [Merging hierarchy transforms](#xformWorldSpaceToLocalSpace)]
~~~

~~~admonish info title=""
```python
{{#include ../../../../code/core/elements.py:xformResetXformStack}}
```
~~~

<video width="100%" height="100%" controls autoplay muted loop>
  <source src="./xformResetXformStack.mp4" type="video/mp4" alt="Houdini Xform Stack Reset">
</video>

### Querying transforms<a name="xformQuery"></a>
We can query xforms via `UsdGeom.Xformable` API or via the `UsdGeom.XformCache` cache.

[ 我们可以通过 UsdGeom.Xformable API 或通过 UsdGeom.XformCache 缓存查询 xform]

The preferred way should always be the xform cache, as it re-uses ancestor xforms in its cache, when querying nested xforms. Only when querying a single leaf transform, we should go with the Xformable API.

[ 最佳方式应该始终是使用 xform 缓存，因为它在查询嵌套 xforms 时会复用祖先 xforms 的缓存. 只有当查询单个叶节点变换时，我们才应该使用 Xformable API]

~~~admonish info title=""
```python
{{#include ../../../../code/core/elements.py:xformCache}}
```
~~~

## Transforms in production <a name="xformProduction"></a>
Let's have a look at some production related xform setups.

[ 让我们看一下一些与生产相关的 xform 设置]

### Merging hierarchy transforms <a name="xformWorldSpaceLocalSpace"></a>
In production we often need to merge different layers with different transforms at different hierarchy levels. For example when we have a cache in world space and we want to merge it into an existing hierarchy.

[ 在生产中，我们经常需要在不同的层级结构上合并具有不同变换的不同层. 例如 当我们在世界空间中有一个缓存并且希望将其合并到现有的层级结构中时]

Here's how we can achieve that (This example is a bit abstract, we'll add something more visual in the near future).

[ 以下是我们如何实现这一目标（这个示例有点抽象，我们将在不久的将来添加更直观的内容）]

~~~admonish info title=""
```python
{{#include ../../../../code/core/elements.py:xformWorldSpaceLocalSpace}}
```
~~~

<video width="100%" height="100%" controls autoplay muted loop>
  <source src="./xformWorldSpaceLocalSpace.mp4" type="video/mp4" alt="Houdini Xform Localize Xform">
</video>

### Baking transforms for constraint like behavior <a name="xformBake"></a>
If we want a parent constraint like behavior, we have to bake down the transform to a single prim. We can then inherit/internal reference/specialize this xform to "parent constrain" something.

[ 如果我们想要实现类似父级约束的行为，我们需要将变换烘焙到一个单独的 prim 上然后，我们可以 inherit/internal reference/specialize 这个变换来“父级约束”某个对象]

~~~admonish info title=""
```python
{{#include ../../../../code/core/elements.py:xformBake}}
```
~~~

<video width="100%" height="100%" controls autoplay muted loop>
  <source src="./xformBake.mp4" type="video/mp4" alt="Houdini Xform Localize Xform">
</video>


### Reading Xforms in Shaders <a name="xformCoordinate"></a>
~~~admonish question title="Still under construction!"
We'll add code examples in the future.

[ 我们将来会添加代码示例]
~~~

To read composed xforms in shaders, USD ships with the [Coordinate Systems](https://openusd.org/release/wp_coordsys.html) mechanism.

[ 为了在着色器中读取组合的 xform，USD 附带了[坐标系统](https://openusd.org/release/wp_coordsys.html) 机制]

It allows us to add a relationship on prims, that targets to an xform prim. This xform can then be queried in shaders e.g. for projections or other transform related needs.

[ 它允许我们在 prims 上添加以 xform prim 为目标链接关系. 然后可以在着色器中查询该 xform，例如 用于投影或其他与变换相关的需求]

See these links for more information:
[UsdShade.CoordSys](https://openusd.org/dev/api/class_usd_shade_coord_sys_a_p_i.html)
[Houdini CoordSys in Shaders](https://www.sidefx.com/docs/houdini/nodes/lop/coordsys.html)

[ 请参阅以下链接以获取更多信息 [UsdShade.CoordSys](https://openusd.org/dev/api/class_usd_shade_coord_sys_a_p_i.html)
[Houdini CoordSys in Shaders](https://www.sidefx.com/docs/houdini/nodes/lop/coordsys.html)]
