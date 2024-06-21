# FAQ (Frequently Asked Questions) [FAQ（常见问题解答）]

# Table of Contents [目录]
1. [Should I prefer assets with a lot of prims or prefer combined meshes?](#faqPrimCount)

    [ 我应该选择具有大量 prim 的资产还是选择合并的网格？](#faqPrimCount)
1. [How is "Frames Per Second" (FPS) handled in USD?](#faqFPS)

    [ USD中是如何处理“每秒帧数”(FPS)的？](#faqFPS)
1. [How is the scene scale unit handled in USD?](#faqSceneScale)

    [ USD中是如何处理场景比例单位的？](#faqSceneScale)

## Should I prefer assets with a lot of prims or prefer combined meshes? <a name="faqPrimCount"></a>
When working in hierarchy based formats, an important influence factor of performance is the hierarchy size.

[ 当使用基于层级结构的格式时，层级结构的大小是一个影响性能的重要因素]

~~~admonish important title="Pro Tip | Hierarchy Size"
Basically it boils down to these rules:
Keep hierarchies as small as possible at all times, only start creating separates meshes when:

[ 基本上它可以归结为以下规则：始终保持层级结构尽可能小，仅在以下情况下开始创建单独的网格]
- your mesh point/prim count starts going into the millions

    [ 您的 mesh point/prim  数量开始达到数百万]
- you need to assign different render geometry settings

    [ 您需要分配不同的渲染几何设置]
- you need to add different transforms

    [ 你需要添加不同的转换]
- you need to hide the prims individually

    [ 你需要单独隐藏prims]
- you need separate materials (We can also use `UsdGeom.Subset`s, which are face selections per mesh, to assign materials, to workaround this)

    [ 您需要单独的材质（我们还可以使用 UsdGeom.Subset s，即按面赋予材质​​，以解决此问题）]
~~~

At the end of the day it is a balancing act of **What do I need to be able to access separately in the hierarchy** vs **I have a prim that is super heavy (100 Gbs of data) and takes forever to load**.
A good viewpoint is the one of a lighting/render artist, as they are the ones that need to often work on individual (sub-)hierarchies and can say how it should be segmented.

[ 归根结底，这是一个平衡行为，即我需要能够在层级结构中单独访问什么与我有一个超重的 prim（100 Gbs 数据）并且需要永远加载. 一个好的视角是 lighting/render 艺术家的视角，因为他们经常需要单独处理（子）层级结构，并且可以说出应该如何对其进行分段]

## How is "frames per second" (FPS) handled in USD? <a name="faqFPS"></a>
Our time samples that are written in the time unit-less `{<frame>: <value> }` format are interpreted based on the `timeCodesPerSecond`/`framesPerSecond` metadata set in the session/root layer.

[ 我们以无时间单位 {\<frame\>: \<value\> } 格式编写的时间样本是根据会话/根层中设置的 timeCodesPerSecond / framesPerSecond 元数据进行解释的]

```python
(
    endTimeCode = 1010
    framesPerSecond = 24
    metersPerUnit = 1
    startTimeCode = 1001
    timeCodesPerSecond = 24
)
```
You can find more details about the specific metadata priority and how to set the metadata in our [animation section](../core/elements/animation.html#animationMetadata).

[ 您可以在[动画](../core/elements/animation.html#animationMetadata) 部分找到有关特定元数据优先级以及如何设置元数据的更多详细信息]

## How is the scene scale unit and up axis handled in USD? <a name="faqSceneScale"></a>
We can supply an up axis and scene scale hint in the layer metadata, but this does not seem to be used by most DCCs or in fact Hydra itself when rendering the geo. So if you have a mixed values, you'll have to counter correct via transforms yourself.

[ 我们可以在图层元数据中提供向 up axis 和 scene scale 提示，但这似乎不会被大多数 DCC 或实际上 Hydra 本身在渲染时使用. 因此，如果您有混合值，则必须自己通过转换来纠正]

The default scene `metersPerUnit` value is centimeters (0.01) and the default `upAxis` is `Y`.

[ 默认场景 metersPerUnit 值为厘米 (0.01)，默认 upAxis 为 Y ]

You can find more details about how to set these metrics see our [layer metadata section](../core/elements/metadata.md#readingwriting-stage-and-layer-metrics-fpsscene-unit-scaleup-axis-highlow-level-api).

[ 您可以找到有关如何设置这些指标的更多详细信息，请参阅[图层元数据](../core/elements/metadata.md#readingwriting-stage-and-layer-metrics-fpsscene-unit-scaleup-axis-highlow-level-api)部分]