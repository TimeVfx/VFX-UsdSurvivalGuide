# Tips & Tricks [提示与技巧]
You can find all the .hip files of our shown examples in our [USD Survival Guide - GitHub Repo](https://github.com/LucaScheller/VFX-UsdSurvivalGuide/tree/main/files/dcc/houdini).

[ 您可以在我们的[USD Survival Guide - GitHub Repo](https://github.com/LucaScheller/VFX-UsdSurvivalGuide/tree/main/files/dcc/houdini) 中找到我们所示示例的所有 .hip 文件]

# Table of Contents [目录]
1. [Composition](#compositionOverview)
    1. [Extracting payloads and references from an existing layer stack with anonymous layers](#compositionReferencePayloadLayerStack)
    1. [Efficiently re-writing existing hierarchies as variants](#compositionArcVariantReauthor)
    1. [Adding overrides via inherits](#compositionArcInherit)
1. [How do I check if an attribute has time samples (if there is only a single time sample)?](#timeSampleValueMightBeTimeVarying)
1. [Where are Houdini's internal lop utils stored?](#houLopUtils)
1. [How do I get the LOPs node that last edited a prim?](#houLastEditedPrim)
1. [How do I store side car data from node to node?](#houSidecarData)

## Composition [合成]<a name="compositionOverview"></a>
Now we've kind of covered these topics in our [A practical guide to composition](../../../production/composition.md) and [Composition - LIVRPS](../../../core/composition/livrps.md) sections.

[ 现在，我们已经在 [A practical guide to composition](../../../production/composition.md) and [Composition - LIVRPS](../../../core/composition/livrps.md) 部分中涵盖了这些主题]

We strongly recommend reading these pages before this one, as they cover the concepts in a broader perspective.

[ 我们强烈建议您先阅读这几页，因为它们从更广泛的角度涵盖了这些概念]

### Extracting payloads and references from an existing layer stack with anonymous layers <a name="compositionReferencePayloadLayerStack"></a>
[ 使用匿名层从现有层堆栈中提取payloads and reference]

When building our composition in USD, we always have to make sure that layers that were generated in-memory are loaded via the same arc as layers loaded from disk.
If we don't do this, our composition would be unreliable in live preview vs cache preview mode.

[ 当以USD构建我们的合成时，我们始终必须确保在内存中生成的图层通过与从磁盘加载的图层相同的弧加载.如果我们不这样做，我们的合成在实时预览与缓存预览模式下将不可靠]

Composition arcs always reference a specific prim in a layer, therefore we usually attach our caches to some sort of predefined root prim (per asset).
This means that if we import SOP geometry, with multiple of these root hierarchies, we should also create multiple references/payloads so that each root prim can be unloaded/loaded via the payload mechanism.

[ 组合弧始终引用层中的特定 prim，因此我们通常将缓存附加到某种预定义的根 prim（每个资产）这意味着，如果我们导入具有多个根层次结构的 SOP 几何体，我们还应该创建多个 references/payloads，以便每个根 prim 都可以通过 payload 机制卸载/加载]

Instead of having a single SOP import or multiple SOP imports that are flattened to a single layer, we can put a SOP import within a for loop. Each loop iteration will then carry only the data of the current loop index (in our example box/sphere/tube) its own layer, because we filter the sop level by loop iteration.

[ 我们可以将 SOP import 放入 for 循环中，而不是将单个 SOP import 或多个 SOP import 扁平化为单个层. 然后，每次循环迭代将仅携带当前循环索引（在我们的示例中为框/球体/管）其自己的层的数据，因为我们通过循环迭代过滤 sop 级别]

The very cool thing then is that in a Python node, we can then find the layers from the layer stack of the for loop and individually payload them in. 

[ 非常酷的事情是，在 Python 节点中，我们可以从 for 循环的层堆栈中找到层并单独将它们加载进去]

Again you can also do this with a single layer, this is just to demonstrate that we can pull individual layers from another node.

[ 同样，您也可以使用单个层来执行此操作，这只是为了演示我们可以从另一个节点提取各个层]

<video width="100%" height="100%" controls autoplay muted loop>
  <source src="./houdiniCompositionReferencePayloadForLoop.mp4" type="video/mp4" alt="Houdini Reference/Payload For Loop">
</video>

You can find this example in the composition.hipnc file in our [USD Survival Guide - GitHub Repo](https://github.com/LucaScheller/VFX-UsdSurvivalGuide/tree/main/files/dcc/houdini).

[ 您可以在我们的[USD Survival Guide - GitHub Repo](https://github.com/LucaScheller/VFX-UsdSurvivalGuide/tree/main/files/dcc/houdini) 的composition.hipnc 文件中找到此示例]

## Efficiently re-writing existing hierarchies as variants <a name="compositionArcVariantReauthor"></a>
[ 有效地将现有层级结构重写为变体]

Via the low level API we can also copy or move content on a layer into a variant. This is super powerful to easily create variants from caches.

[ 通过低级 API，我们还可以将图层上的内容复制或移动到变体中. 这非常强大，可以轻松地从缓存创建变体]

Here is how it can be setup in Houdini:

[ 以下是在 Houdini 中的设置方法]

<video width="100%" height="100%" controls autoplay muted loop>
  <source src="../../../core/composition/houdiniCompositionVariantCopyMove.mp4" type="video/mp4" alt="Houdini Composition Variant Copy/Move">
</video>

Here is the code for moving variants:

[ 这是移动变体的代码]
~~~admonish tip title=""
```python
{{#include ../../../../../code/core/composition.py:compositionArcVariantMoveHoudini}}
```
~~~

And for copying:

[ 这是复制变体的代码]
~~~admonish tip title=""
```python
{{#include ../../../../../code/core/composition.py:compositionArcVariantCopyHoudini}}
```
~~~

## Adding overrides via inherits <a name="compositionArcInherit"></a>
[ 通过继承添加覆盖]

We can add inherits as explained in detail in our [composition - LIVRPS](../../../core/composition/livrps.md#inherits) section.

[ 我们可以添加继承，如我们的 [composition - LIVRPS](../../../core/composition/livrps.md#inherits) 部分中详细解释的那样]

We typically drive the prim selection through a user defined [prim pattern/lop selection rule](../performance/overview.md#houLopSelectionRule). In the example below, for simplicity, we instead iterate over all instances of the prototype of the first pig prim.

[ 我们通常通过用户定义的 [prim pattern/lop selection rule](../performance/overview.md#houLopSelectionRule) 来驱动 prim 选择. 在下面的示例中，为了简单起见，我们改为迭代第一个 pig prim 的原型的所有实例]

~~~admonish tip title=""
```python
{{#include ../../../../../code/dcc/houdini.py:houdiniCompositionInheritInstanceable}}
```
~~~

<video width="100%" height="100%" controls autoplay muted loop>
  <source src="./houdiniCompositionInheritInstanceable.mp4" type="video/mp4" alt="Houdini Composition Inherit">
</video>

## How do I check if an attribute has time samples (if there is only a single time sample)? <a name="timeSampleValueMightBeTimeVarying"></a>
[ 如何检查属性是否有时间样本（如果只有一个时间样本）？]

We often need to check if an attribute has animation or not. Since time samples can come through many files when using value clips, USD ships with the `Usd.Attribute.ValueMightBeTimeVarying()` method. This checks for any time samples and exists as soon as it has found some as to opening every file like `Usd.Attribute.GetTimeSamples` does.

[ 我们经常需要检查一个属性是否有动画. 由于使用值剪辑时时间样本可以来自许多文件，因此 USD 附带 Usd.Attribute.ValueMightBeTimeVarying() 方法. 这会检查任何时间样本，并且一旦发现像 Usd.Attribute.GetTimeSamples 那样打开每个文件的样本，就会立即存在]

The issue is that if there is only a single time sample (not default value) it still returns False, as the value is not animated per se. (By the way, this is also how the .abc file did it). Now that kind of makes sense, the problem is when working with live geometry in Houdini, we don't have multiple time samples, as we are just looking at the active frame.
So unless we add a "Cache" LOP node afterwards that gives us multiple time samples, the `GetTimeSamples` will return a "wrong" value.

[ 问题是，如果只有一个时间样本（不是默认值），它仍然返回 False，因为该值本身不是动画的. （顺便说一下.abc 文件也是这样做的）现在这是有道理的，问题是在 Houdini 中处理实时几何体时，我们没有多个时间样本，因为我们只是查看激活帧. 因此，除非我们之后添加一个“Cache”LOP 节点来为我们提供多个时间样本，否则 GetTimeSamples 将返回一个“错误”值]

Here is how we get a correct value:

[ 这是我们获得正确值的方法]

~~~admonish tip title=""
```python
{{#include ../../../../../code/dcc/houdini.py:houdiniTimeDependency}}
```
~~~

The logic is relatively simple: When looking at in-memory layers, use the usual command of `GetNumTimeSamples` as in-memory layers are instant when querying data.
When looking at on disk files, use the `ValueMightBeTimeVarying`, as it is the fastest solution.

[ 逻辑相对简单：查看内存层时，使用常用命令 GetNumTimeSamples ，因为内存层在查询数据时是即时的. 查看磁盘文件时，请使用 ValueMightBeTimeVarying，因为它是最快的解决方案]

You can find the shown file here: [UsdSurvivalGuide - GitHub](https://github.com/LucaScheller/VFX-UsdSurvivalGuide/tree/main/files/dcc/houdini/timeSamples)

[ 您可以在此处找到显示的文件 [UsdSurvivalGuide - GitHub](https://github.com/LucaScheller/VFX-UsdSurvivalGuide/tree/main/files/dcc/houdini/timeSamples)]

![Houdini Attribute Value Might Be Time Varying](houdiniAttributeValueMightBeTimeVarying.jpg)


## Where are Houdini's internal lop utils stored? <a name="houLopUtils"></a>
[ Houdini的内部lop utils存储在哪里？]

You can find Houdini's internal loputils under the following path:

[ 您可以在以下路径下找到Houdini的内部loputils]
~~~admonish tip title=""
$HFS/houdini/python3.9libs/loputils.py
~~~
It is not an official API module, so use it with care, it may be subject to change.

[ 它不是官方 API 模块，因此请小心使用，它可能会发生变化]

You can simply import via `import loputils`. It is a good point of reference for UI related functions, for example action buttons on parms use it at lot.

[ 您只需通过 import loputils 导入即可. 对于 UI 相关功能来说，这是一个很好的参考点，例如 parms 上的操作按钮就经常使用它]

Here you can find the [loputils.py - Sphinx Docs](https://ikrima.github.io/houdini_additional_python_docs/loputils.html) online.

[ 在这里您可以找到 [loputils.py - Sphinx Docs](https://ikrima.github.io/houdini_additional_python_docs/loputils.html)]

## How do I get the LOPs node that last edited a prim? <a name="houLastEditedPrim"></a>
[ 如何获取上次编辑 prim 的 LOP 节点？]

When creating data on your layers, Houdini attaches some custom data to the `customData` prim metadata. Among the data is the `HoudiniPrimEditorNodes`. This stores the internal [hou.Node.sessionId](https://www.sidefx.com/docs/houdini/hom/hou/nodeBySessionId.html) and allows you to get the last node that edited a prim.

[ 在图层上创建数据时，Houdini 会将一些自定义数据附加到 customData prim 元数据. 数据中有 HoudiniPrimEditorNodes .这存储了内部 hou.Node.sessionId 并允许您获取编辑 prim 的最后一个节点]

This value is not necessarily reliable, for example if you do custom Python node edits, this won't tag the prim (unless you do it yourself). Most Houdini nodes track it correctly though, so it can be useful for UI related node selections.

[ 该值不一定可靠，例如，如果您进行自定义 Python 节点编辑，这不会标记 prim（除非您自己执行）.大多数 Houdini 节点都能正确跟踪它，因此它对于 UI 相关节点选择很有用]

~~~admonish tip title=""
```Python
...
def Xform "pig" (
    customData = {
        int[] HoudiniPrimEditorNodes = [227]
    }
    kind = "component"
)
...
```
~~~
Here is how you retrieve it:

[ 以下是检索它的方法]
~~~admonish tip title=""
```Python
import hou
from pxr import Sdf
stage = node.stage()
prim = stage.GetPrimAtPath(Sdf.Path("/pig"))
houdini_prim_editor_nodes = prim.GetCustomDataByKey("HoudiniPrimEditorNodes")
edit_node = None
if houdini_prim_editor_nodes:
    edit_node = hou.nodeBySessionId(houdini_prim_editor_nodes[-1])
```
~~~

You can also set it:

[ 您还可以设置它]
~~~admonish tip title=""
```Python
import hou
from pxr import Sdf, Vt
node = hou.pwd()
stage = node.editableStage()
prim = stage.GetPrimAtPath(Sdf.Path("/pig"))
houdini_prim_editor_nodes = prim.GetCustomDataByKey("HoudiniPrimEditorNodes") or []
houdini_prim_editor_nodes = [i for i in houdini_prim_editor_nodes]
houdini_prim_editor_nodes.append(node.sessionId())
prim.SetCustomDataByKey("HoudiniPrimEditorNodes", Vt.IntArray(houdini_prim_editor_nodes))
```
~~~

The Houdini custom data gets stripped from the file, if you enable it on the USD rop (by default it gets removed).

[ 如果您在 USD rop 上启用它，则 Houdini 自定义数据将从文件中删除（默认情况下它会被删除）]

![Alt text](houdiniNodeBySessionId.jpg)

## How do I store side car data from node to node? <a name="houSidecarData"></a>
[ 如何在节点之间存储边车数据？]

To have a similar mechanism like SOPs detail attributes to track data through your network, we can write our data to the `/HoudiniLayerInfo` prim. This is a special prim that Houdini creates (and strips before USD file write) to track Houdini internal data. It is hidden by default, you have to enable "Show Layer Info Primitives" in your scene graph tree under the sunglasses icon to see it. We can't track data on the root or session layer customData as Houdini handles these different than with bare bone USD to enable efficient node based stage workflows.

[ 为了拥有类似 SOPs detail attributes 的机制来跟踪网络中的数据，我们可以将数据写入 /HoudiniLayerInfo prim. 这是 Houdini 创建的特殊 prim（并在 USD 文件写入之前剥离）来跟踪 Houdini 内部数据. 默认情况下它是隐藏的，您必须在太阳镜图标下的场景图树中启用“Show Layer Info Primitives”才能看到它.我们无法跟踪根或会话层 customData 上的数据，因为 Houdini 处理这些数据的方式与使用裸露 USD 不同，以实现基于节点的高效阶段工作流程]

You can either do it via Python:

[ 您可以通过 Python 来完成]
~~~admonish tip title=""
```Python
import hou
import json
from pxr import Sdf
node = hou.pwd()
stage = node.editableStage()
prim = stage.GetPrimAtPath(Sdf.Path("/HoudiniLayerInfo"))
custom_data_key = "usdSurvivalGuide:coolDemo"
my_custom_data = json.loads(prim.GetCustomDataByKey(custom_data_key) or "{}")
print(my_custom_data)
prim.SetCustomDataByKey(custom_data_key, json.dumps(my_custom_data))
```
~~~

Or with Houdini's [Store Parameters Values](https://www.sidefx.com/docs/houdini/nodes/lop/storeparametervalues.html) node. See the docs for more info (It also uses the loputils module to pull the data).

[ 或者使用 Houdini 的 [Store Parameters Values](https://www.sidefx.com/docs/houdini/nodes/lop/storeparametervalues.html) 节点.有关更多信息，请参阅文档（它还使用 loputils 模块来提取数据）]
