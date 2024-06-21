# Performance Optimizations [性能优化]
You can find all the .hip files of our shown examples in our [USD Survival Guide - GitHub Repo](https://github.com/LucaScheller/VFX-UsdSurvivalGuide/tree/main/files/dcc/houdini).

[ 您可以在我们的 [USD Survival Guide - GitHub Repo](https://github.com/LucaScheller/VFX-UsdSurvivalGuide/tree/main/files/dcc/houdini) 中找到我们所示示例的所有 .hip 文件]

# Table of Contents [目录]
1. [Selection Rules](#houLopSelectionRule)
1. [How to get your stage to load and open fast](#loadingMechanisms)
1. [Write full time sample ranges (with subsamples)](#timeSample) 
1. [Layer Content Size](#layerContentSize)
1. [Layer Count](#layerCount)
1. [AOV Count](#renderAOVCount)

## Selection Rules [选择的规则]<a name="houLopSelectionRule"></a>
Houdini's [LOPs Selection Rule/Prim Pattern Matching](https://www.sidefx.com/docs/houdini/solaris/pattern.html) syntax is a artist friendly wrapper around stage traversals.

[ Houdini 的 [LOPs Selection Rule/Prim Pattern Matching](https://www.sidefx.com/docs/houdini/solaris/pattern.html) 语法是对艺术家遍历 stage 的友好包装]

Pretty much any artist selectable prim pattern parm is/should be run through the selection rules. Now we won't cover how they work here, because Houdini's documentation is really detailed on this topic.

[ 几乎任何艺术家可选择的 prim pattern parm 都/应该通过选择规则. 我们不会在这里介绍它们是如何工作的，因为 Houdini 的文档关于这个主题非常详细]

Instead we'll compare it to our own [traversal section](../../../core/elements/loading_mechanisms.md#traverseData).

[ 相反，我们会将其与我们自己的 [遍历](../../../core/elements/loading_mechanisms.md#traverseData) 进行比较]
~~~admonish tip title=""
```python
import hou
rule = hou.LopSelectionRule()
# Set traversal demand, this is similar to USD's traversal predicate
# https://openusd.org/dev/api/prim_flags_8h.html#Usd_PrimFlags
hou.lopTraversalDemands.Active
hou.lopTraversalDemands.AllowInstanceProxies
hou.lopTraversalDemands.Default
hou.lopTraversalDemands.Defined
hou.lopTraversalDemands.Loaded
hou.lopTraversalDemands.NoDemands
hou.lopTraversalDemands.NonAbstract
rule.setTraversalDemands(hou.lopTraversalDemands.Default)
# Set rule pattern
rule.setPathPattern("%type:Boundable")
# Evaluate rule
prim_paths = rule.expandedPaths(lopnode=None, stage=stage)
for prim_path in prim_paths:
    print(prim_path) # Returns: Sdf.Path
```
~~~

As you can see we have a similar syntax, the predicate is "replaced" by hou.lopTraversalDemands.

[ 正如您所看到的，我们有类似的语法，断言被 hou.lopTraversalDemands“替换”]

Now the same rules apply for fast traversals:

[ 现在相同的规则适用于快速遍历]
- Fast traversals mean not going into the hierarchies we are not interested in. The equivalent to `iterator.PruneChildren` is the `~` tilde symbol (e.g. `%type:Xform ~ %kind:component`)

    [ 快速遍历意味着不进入我们不感兴趣的层级结构.与 iterator.PruneChildren 等效的是 ~ 波浪号（例如 %type:Xform ~ %kind:component ）]
- We should aim to pre-filter by type `%type:<ConcreteTypedSchemaName>` and kind `%kind:component`, before querying other data as this is fast

    [ 我们的目标是在查询其他数据之前按类型 %type:\<ConcreteTypedSchemaName\> 和种类 %kind:component 进行预先过滤，因为这样速度很快]
- Attributes lookups (via vex in the expression) are heavy to compute

    [ 属性查找（通过表达式中的 vex）计算量很大]

## How to get your stage to load and open fast <a name="loadingMechanisms"></a>

[ 怎样让你的 stage 加载和打开更快?]

As discussed in our [Loading/Traversing section](../../../core/elements/loading_mechanisms.md), we can limit how stages are opened via our three loading mechanisms (ordered from lowest to highest granularity):

[ 正如我们的[加载/遍历](../../../core/elements/loading_mechanisms.md)部分中所讨论的，我们可以通过三种加载机制（从最低到最高的排序）来限制阶段的打开方式]
- **Layer Muting**: This controls what layers are allowed to contribute to the composition result.

    [ Layer Muting：这控制允许哪些图层对合成结果做出贡献]
- **Prim Population Mask**: This controls what prim paths to consider for loading at all.

    [ Prim Population Mask：这控制要考虑加载的 prim 路径]
- **Payload Loading**: This controls what prim paths, that have payloads, to load.

    [ Payload Loading：这控制要加载哪些具有有效负载的原始路径]

Before we proceed, it is important to note, that USD is highly performant in loading hierarchies. When USD loads .usd/.usdc binary crate files, it sparsely loads the content: It can read in the hierarchy without loading in the attributes. This allows it to, instead of loading terabytes of data, to only read the important bits in the file and lazy load on demand the heavy data when requested by API queries or a hydra delegate.

[ 在我们继续之前，需要注意的是，USD 在加载层级结构方面具有很高的性能. 当 USD 加载 .usd/.usdc 二进制 crate 文件时，它会稀疏地加载内容：它可以在层次结构中读取，而无需加载属性. 这使得它可以只读取文件中的重要位，而不是加载 TB 级的数据，并在 API 查询或 Hydra 委托请求时按需延迟加载大量数据]

What does this mean for Houdini? It can often be enough to pause the scene viewer when opening the file. It can be done via this snippet:

[ 这对胡迪尼意味着什么？打开文件时暂停场景查看器通常就足够了. 可以通过以下代码片段完成]
~~~admonish tip title=""
```python
for pane in hou.ui.paneTabs():
    if pane.type == hou.paneTabType.SceneViewer:
        pane.setSceneGraphStageLocked(False)
```
~~~

Houdini exposes these three loading mechanisms in two different ways:

[ Houdini 以两种不同的方式公开这三种加载机制]
- **Configure Stage** LOP node: This is the same as setting it per code via the stage.

    [ Configure Stage LOP 节点：这与通过 stage 按代码设置相同]
- **Scene Graph Tree** panel: In Houdini, that stage that gets rendered, is actually not the stage of your node (at least what we gather from reverse engineering). Instead it is a duplicate, that has overrides in the session layer and loading mechanisms listed above.

    [ Scene Graph Tree：在 Houdini 中，渲染的 stage 实际上不是节点的 stage （至少是我们从逆向工程中收集到的）.相反，它是一个副本，在会话层和上面列出的加载机制中具有覆盖]

~~~admonish danger title="Scene Graph Tree vs Configure Stage"
Everything you set in your scene graph tree panel is a viewport **only** override to your stage. This can be very confusing when first starting out in Houdini.
If we want to apply it to the actual stage, we have to use the configure stage node. This will then also affect our scene graph tree panel.

[ 您在 scene graph tree panel 中设置的所有内容都只是覆盖到 stage 的视口. 第一次开始使用 Houdini 时，这可能会非常令人困惑. 如果我们想将其应用到实际 stage ，我们必须使用configure stage节点. 这也会影响我们的 scene graph tree 面板]

Why does Houdini do this? As mentioned hierarchy loading is fast, streaming the data to Hydra is not. This way we can still see the full hierarchy, but separately limit what the viewer sees, which is actually a pretty cool mechanism.

[ 胡迪尼为什么要这么做？正如前面提到的，层级结构加载速度很快，而将数据流式传输到 Hydra 则不然. 这样我们仍然可以看到完整的层级结构，但单独限制观看者看到的内容，这实际上是一个非常酷的机制]
~~~

Let's have a look at the differences, as you can see anything we do with the configure stage node actually affects our hierarchy, whereas scene graph tree panel edits are only for the viewport:

[ 让我们看一下差异，因为您可以看到我们对 configure stage 节点所做的任何操作实际上都会影响我们的层次结构，而 scene graph tree 面板编辑仅适用于视口]

<video width="100%" height="100%" controls autoplay muted loop>
  <source src="./loadingMechanisms.mp4" type="video/mp4" alt="Scene Graph Tree vs Configure Stage">
</video>

Another great tip is to disable tieing the scene graph panel to the active selected node:

[ 另一个很棒的技巧是禁用将 scene graph panel 绑定到活动选定节点]

![Scene Graph Tree Panel Node Displayed](houdiniSceneGraphTreePanelActiveNode.jpg)

Instead it is then tied to you active display flag, which makes things a lot faster when clicking through your network.

[ 相反，它会与您的激活显示标志绑定，这使得点击网络时速度更快]

## Write full time sample ranges (with subsamples) <a name="timeSample"></a>
In Houdini we always want to avoid time dependencies where possible, because that way the network doesn't have to recook the tree per frame.

[ 在 Houdini 中，我们总是希望尽可能避免时间依赖性，因为这样网络就不必每帧重新构建树]

We cover this in more detail for HDAs in our [HDA section](../hda/hda.md) as it is very important when building tools, but here is the short version.

[ 我们在 HDA 部分中更详细地介绍了 HDA，因为它在构建工具时非常重要，但这里是简短版本]

We have two ways of caching the animation, so that the node itself loses its time dependency.

[ 我们有两种方式来缓存动画，这样节点本身就失去了时间依赖性]

Starting with H19.5 most LOP nodes can whole frame range cache their edits. This does mean that a node can cook longer for very long frame ranges, but overall your network will not have a time dependency, which means when writing your node network to disk (for example for rendering), we only have to write a single frame and still have all the animation data. How cool is that!

[ 从 H19.5 开始，大多数 LOP 节点可以在整个帧范围内缓存其编辑.这确实意味着节点可以在很长的帧范围内烹饪更长的时间，但总的来说，您的网络不会有时间依赖性，这意味着将节点网络写入磁盘时（例如用于渲染），我们只需写入单个帧并且仍然拥有所有动画数据.多么酷啊！]

<video width="100%" height="100%" controls autoplay muted loop>
  <source src="../hda/hdaTimeDependencyPerNode.mp4" type="video/mp4" alt="Houdini Time Sample Per Node">
</video>

If a node doesn't have that option, we can almost always isolate that part of the network and pre cache it, that way we have the same effect but for a group of nodes.

[ 如果节点没有该选项，我们几乎总是可以隔离网络的该部分并预先缓存它，这样我们就可以达到相同的效果，但对于一组节点而言]

The common workflow is to link the shutter samples count to your camera/scene xform sample count and cache out the frames you need.

[ 常见的工作流程是将快门样本计数链接到相机/场景 xform 样本计数并缓存出您需要的帧]

<video width="100%" height="100%" controls autoplay muted loop>
  <source src="./timeSample.mp4" type="video/mp4" alt="Houdini Time Sample Subframes">
</video>

We recommend driving the parms through global variables or functions, that you can attach to any node via the node [onLoaded](https://www.sidefx.com/docs/houdini/hom/locations.html#scene_events) scripts.

[ 我们建议通过全局变量或函数驱动参数，您可以通过节点  [onLoaded](https://www.sidefx.com/docs/houdini/hom/locations.html#scene_events) 脚本将其附加到任何节点]

This is also how you can write out SOP geometry with deform/xform subsamples, simply put the cache node after a "SOP Import" and set the mode to "Sample Current Frame".
We usually do this in combination with enabling "Flush Data After Each Frame" on the USD render rop plus adding "$F4" to your file path.
This way we stay memory efficient and dump the layer from memory to disk every frame. After that we stitch it via [value clips](../../../core/elements/animation.md#animationValueClips) or the "UsdStitch" commandline tool/rop node.

[ 这也是您如何使用 deform/xform  子样本写出SOP几何体，只需将缓存节点放在 "SOP Import" 之后并将模式设置为“Sample Current Frame”即可.我们通常会结合在 USD render rop 上启用“Flush Data After Each Frame”以及将“$F4”添加到文件路径来执行此操作.通过这种方式，我们可以保持内存效率，并在每一帧将图层从内存转储到磁盘.之后，我们通过 [value clips](../../../core/elements/animation.md#animationValueClips) 或 "UsdStitch" commandline tool/rop 节点来缝合它]


## Layer Content Size [图层内容大小]<a name="layerContentSize"></a>
In Houdini the size of the active layer can impact performance.

[ 在 Houdini 中，激活层的大小会影响性能]

To quote from the docs:

[ 引用文档]
~~~admonish tip
As LOP layers have more and more opinions and values added, there can be slow-downs in the viewport. If you notice simple transforms are very slow in the viewport, try using a Configure Layer to start a new layer above where you're working, and see if that improves interactivity.

[ 随着 LOP 图层添加越来越多的opinions 和 values，视口中的速度可能会变慢. 如果您发现视口中的简单变换非常慢，请尝试使用 Configure Layer 在您正在工作的上方启动一个新图层，并查看是否可以提高交互性]
~~~
Let's build a test setup, where we provoke the problem on purpose:

[ 让我们构建一个测试设置，故意引发问题]

~~~admonish tip title=""
```python
from pxr import Sdf
node = hou.pwd()
layer = node.editableLayer()
with Sdf.ChangeBlock():
    root_grp_prim_path = Sdf.Path(f"/root_grp")
    root_grp_prim_spec = Sdf.CreatePrimInLayer(layer, root_grp_prim_path)
    root_grp_prim_spec.typeName = "Xform"
    root_grp_prim_spec.specifier = Sdf.SpecifierDef
    
    prim_count = 1000 * 100
    for idx in range(prim_count):
        prim_path = Sdf.Path(f"/root_grp/prim_{idx}")
        prim_spec = Sdf.CreatePrimInLayer(layer, prim_path)
        prim_spec.typeName = "Cube"
        prim_spec.specifier = Sdf.SpecifierDef
        attr_spec = Sdf.AttributeSpec(prim_spec, "debug", Sdf.ValueTypeNames.Float)
        attr_spec.default = float(idx)
        if idx != 0:
            prim_spec.SetInfo(prim_spec.ActiveKey, False)
```
~~~

~~~admonish danger
Now if we add an edit property node and tweak the "debug" attribute on prim "/root_grp/prim_0", it will take around 600 milliseconds!

[ 现在，如果我们添加一个编辑属性节点并调整 prim“/root_grp/prim_0”上的“debug”属性，大约需要 600 毫秒！]
~~~

The simple solution as stated above is to simply add a "Configure Layer" LOP node and enable "Start New Layer". Now all edits are fast again.

[ 如上所述的简单解决方案是简单地添加一个"Configure Layer" LOP 节点并启用"Start New Layer". 现在所有编辑又变得很快了]

So why is this slow? It is actually due to how Houdini makes node based editing of layers possible. Houdini tries to efficiently cache only the active layer, where all the edits go, per node (if a node did write operations). The active layer in Houdini speak is the same as the stage's edit target layer. This means every node creates a duplicate of the active layer, so that we can jump around the network and display different nodes, without recalculating everything all the time. The down side is the copying of the data, which is causing the performance hit we are seeing.

[ 那么为什么这么慢呢？这实际上是由于 Houdini基于节点的图层编辑. Houdini 尝试高效地缓存在当前激活层上每个节点的所有编辑（如果节点执行了写入操作）, 激活层在 Houdini 指的是 stage's edit target layer. 这意味着每个节点都会创建激活层的副本，以便我们可以在网络中跳转并显示不同的节点，而无需一直重新计算所有内容. 缺点是数据的复制，这导致了我们所看到的性能损失]

The solution is simple: Just start a new layer and then everything is fast again (as it doesn't have to copy all the content). Houdini colors codes its nodes every time they start a new (active) layer, so you'll also see a visual indicator that the nodes after the "Configure Layer" LOP are on a new layer.

[ 解决方案很简单：只需启动一个新层，然后一切都会变得很快（因为它不必复制所有内容）. 每次启动一个新的（激活）层时，Houdini 都会对其节点进行颜色编码，因此您还会看到一个视觉指示器，表明“Configure Layer”LOP 之后的节点位于新层上]

~~~admonish tip title="Pro Tip | Layer size in Hdas"
What does this mean for our .hda setups? The answer is simple: As there is no downside to having a large layer count (unless we start going into the thousands), each .hda can simply start of and end with creating a new layer. That way all the edits in the .hda are guaranteed to not be impacted by this caching issue.

[ 这对于我们的 .hda 设置意味着什么？答案很简单：由于拥有大量层数没有任何缺点（除非我们开始达到数千个），因此每个 .hda 都可以简单地以创建新层开始和结束.这样，.hda 中的所有编辑都保证不会受到此缓存问题的影响]
~~~

Here is a comparison video:

[ 这是一个对比视频]

<video width="100%" height="100%" controls autoplay muted loop>
  <source src="./layerContentSize.mp4" type="video/mp4" alt="Layer Size">
</video>

## Layer Count <a name="layerCount"></a>
Now as mentioned in the [layer content size](#layerContentSize) section, there is no down side to having thousands of layers. Houdini will merge these to a single (or multiple, depending on how you configured your save paths) layers on USD rop render. Since this is done on write, the described active layer stashing mechanism doesn't kick in and therefore it stays fast.

[ 现在，正如[图层内容大小](#layerContentSize)部分中提到的，拥有数千个图层没有任何缺点. Houdini 会将这些合并到 USD rop 渲染上的单个（或多个，具体取决于您如何配置保存路径）图层.由于这是在写入时完成的，因此所描述的激活层存储机制不会启动，因此它保持快速]

Does that mean we should write everything on a new layer? No, the sweet spot is somewhere in between. For example when grafting (moving a certain part of the hierarchy somewhere else) we also have to flatten the layer on purpose (Houdini's "Scene Restructure"/"Graft" nodes do this for us). At some layer count, you will encounter longer layer merging times, so don't over do it! This can be easily seen with the LOPs for loop.

[ 这是否意味着我们应该将所有内容都写在新层上？不，最佳点是介于两者之间. 例如，当嫁接（将层级结构的某个部分移动到其他地方）时，我们还必须故意压平图层（Houdini 的 "Scene Restructure"/"Graft" 节点为我们执行此操作）.在一定的层数下，您将遇到更长的层合并时间，所以不要过度！这可以通过 LOP for 循环轻松看出]

~~~admonish tip title="Pro Tip | Layer count"
So as a rule of thumb: Encapsulate heavy layer edits with newly started layers, that way the next node downstream will run with optimal performance. Everything else is usually fine to be on the active input layer.

[ 因此，根据经验：将重型层编辑放在新启动的层上，这样下游的下一个节点将以最佳性能运行.其他一切通常都可以放在激活输入层上]
~~~

In LOPs we also often work with the principle of having a "main" node stream (the stream, where your shot layers are loaded from). A good workflow would be to always put anything you merge into the main node stream into a new layer, as often these "side" streams create heavy data.

[ 在 LOP 中，我们还经常遵循“主”节点流（从其中加载镜头层的流）的原则. 一个好的工作流程是始终将合并到主节点流中的任何内容放入新层，因为这些“分支”流通常会创建大量数据]

~~~admonish danger title="Important | LOP 'For Each Loops'"
LOPs "for each loops" work a bit different: Each iteration of the loop is either merged with the active layer or kept as a separate layer, depending on the set merge style.
When we want to spawn a large hierarchy, we recommend doing it via Python, as it is a lot faster. We mainly use the "for each loop" nodes for stuff we can't do in Python code. For example for each looping a sop import.

[ “for each loops”的 LOP 工作方式略有不同：循环的每次迭代要么与激活层合并，要么保留为单独的层，具体取决于设置的合并样式.当我们想要生成一个大型层次结构时，我们建议通过 Python 来完成，因为它要快得多.我们主要使用“for each loop”节点来完成我们在Python代码中无法完成的事情.例如，对于每个循环都有一个 sop 导入]

![Houdini For Each Loop Merge Style](houdiniForEachMergeStyle.jpg)
~~~

## AOV Count <a name="renderAOVCount"></a>
Now this tip is kind of obvious, but we'd though we'd mention it anyway:

[ 这个小窍门无需过多解释，但我们还是想提一下]

When rendering interactively in the viewport, deactivating render var prims that are connected to the render outputs you are rendering, can speed up interactivity. Why? We are rendering to fewer image buffers, so less memory consumption and less work for the renderer to output your rays to pixels.

[ 在视口中进行交互渲染时，停用连接到正在渲染的渲染输出的渲染变量 prim，可以加快交互速度. 为什么？我们渲染到更少的图像缓冲区，因此内存消耗更少，渲染器将光线输出到像素的工作也更少]

Our AOVs are connected via the `orderedVars` relationship to the render product. That means we can just overwrite it to only contain render vars we need during interactive sessions. For farm submissions we can then just switch back to all.

[ 我们的 AOV 通过 orderedVars 关系连接到渲染产品. 这意味着我们可以重写,让它仅包含交互式会话期间所需的渲染变量. 对于农场提交，我们可以切换回全部]

The same goes for the `products` relationship on your render settings prim. Here you can also just connect the products you need.

[ 渲染设置上的 products relationship 也是如此. 您也可以在这里连接您需要的 products]

~~~admonish tip title=""
```python
def Scope "Render"
{
    def Scope "Products"
    {
        def RenderProduct "beauty" (
        )
        {
            rel orderedVars = [
                </Render/Products/Vars/Beauty>,
                </Render/Products/Vars/CombinedDiffuse>,
                </Render/Products/Vars/DirectDiffuse>,
                </Render/Products/Vars/IndirectDiffuse>,
            ]
            token productType = "raster"
        }
    }

    def RenderSettings "rendersettings" (
        prepend apiSchemas = ["KarmaRendererSettingsAPI"]
    )
    {
        rel camera = </cameras/render_cam>
        rel products = </Render/Products/beauty>
        int2 resolution = (1280, 720)
    }
}
```
~~~

In Houdini this is as simple as editing the relationship and putting the edit behind a switch node with a context option switch. On our render USD rop we can then set the context option to 1 and there you go, it is now always on for the USD rop write.

[ 在 Houdini 中，编辑 relationship 并将编辑放在带有上下文切换的 switch node. 当在我们的渲染 USD rop 上，我们可以将上下文选项设置为 1，然后就可以通过 USD rop 写入文件中]

<video width="100%" height="100%" controls autoplay muted loop>
  <source src="./houdiniRenderAOVCount.mp4" type="video/mp4" alt="Houdini Render AOV Count">
</video>

