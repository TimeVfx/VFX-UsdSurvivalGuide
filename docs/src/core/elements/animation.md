# Animation/Time Varying Data [动画/时间数据]
Usd encodes time related data in a very simple format:

[ Usd 以非常简单的格式对时间相关数据进行编码]

```json
{
    <frame>: <value>
}
```

# Table of Contents [目录]
1. [Animation/Time Varying Data In-A-Nutshell](#summary)
2. [What should I use it for?](#usage)
3. [Resources](#resources)
4. [Overview](#overview)
    1. [Time Code](#animationTimeCode)
    2. [Layer Offset (A Non-Animateable Time Offset/Scale for Composition Arcs)](#animationLayerOffset)
    3. [Reading & Writing default values, time samples and value blocks](#animationReadWrite)
    4. [Time Metrics (Frames Per Second & Frame Range)](#animationMetadata)
    5. [Motionblur - Computing Velocities and Accelerations](#animationMotionVelocityAcceleration)
    6. [Stitching/Combining time samples](#animationStitch)
    7. [Value Clips (Loading time samples from multiple files)](#animationValueClips)

## TL;DR - Animation/Time Varying Data In-A-Nutshell [概述]<a name="summary"></a>
~~~admonish tip
- Terminology: A single time/value pair is called `time sample`, if an attribute doesn't have time samples, it has `default` value (Which just means it has a single static value).

    [ 术语：单个时间/值对被称为时间样本. 如果某个属性没有时间样本，则它具有默认值（这仅意味着它具有单个静态值）]
- Time samples are encoded in a simple {\<time(frame)\>: \<value\>} dict.

    [ 时间样本以简单的 {\<time(frame)\>: \<value\>} 字典形式进行编码 ]
- If a frame is requested where no time samples exist, it will be interpolated if the data type allows it and non changing array lengths in neighbour time samples exist. Value queries before/after the first/last time sample will be clamped to these time samples.

    [ 如果请求一个不存在时间样本的帧，数据类型允许并且相邻时间样本的数组长度不变，则会进行插值. 在第一个/最后一个时间样本之前/之后的值查询将被限制在这些时间样本内]
- Time samples are encoded unitless/per frame and not in time units. This means they have to be shifted depending on the current frames per second.

    [ 时间样本是以每帧(无单位)的方式编码的，而不是 time. 这意味着它们需要根据当前的每秒帧数进行偏移]
- Only attributes can carry time sample data. (So composition arcs can not be time animated, only offset/scaled (see the LayerOffset section on this page)).

    [ 只有属性可以携带时间样本数据（因此，合成弧不能进行时间动画处理，只能进行偏移/缩放（请参阅本页上的LayerOffset部分））]

Reading and writing is quite straight forward:

[ 读取和写入操作相对直接]
```python
{{#include ../../../../code/core/elements.py:animationOverview}}
```
~~~

## What should I use it for? <a name="usage"></a>

[ 我应该用它做什么？]

~~~admonish tip
Anything that has time varying data will be written as time samples. Usually DCCs handle the time sample creation for you, but there are situations where we need to write them ourselves. For example if you want to efficiently combine time samples from two different value sources or if we want to turn a time sample into a `default` sampled value (a value without animation).

[ 任何具有时间变化数据的内容都将以时间样本的形式写入. 通常 DCC 会为您处理时间样本的创建，但也有一些情况需要我们手动编写. 例如 如果您想要有效地组合来自两个不同值源的时间样本，或者我们想要将时间样本转换为具有默认采样值（没有动画的值）时就需要这么做]
~~~

## Resources [资源]
- [USD Time Sample Examples](https://openusd.org/release/tut_xforms.html)
- [USD Value Clips](https://openusd.org/dev/api/_usd__page__value_clips.html)
- [USD Value Clips Utils](https://openusd.org/dev/api/stitch_clips_8h.html#details) 

## Overview [概述]<a name="overview"></a>
~~~admonish important
 A single frame(time)/value pair is called `time sample`, if an attribute doesn't have time samples, it has `default` value (Which just means it has a single static value). The time value is the active frame where data is being exported on. It is not time/FPS based. This means that depending on `frames per second` set on the stage, different time samples are read. This means if you have a cache that was written in 25 FPS, you'll have to shift/scale the time samples if you want to use the cache in 24 FPS and you want to have the same result. More about this in the examples below.

 [ 单个帧（时间）/值对被称为 `time sample` 如果属性没有时间样本，则它具有 `default` 值（意味着它有一个静态值）这个值是导出数据的活动帧. 它不是基于时间/帧速率（FPS） 这意味着根据舞台上设置的每秒帧数（FPS）会读取不同的时间样本. 例如 如果你有一个以 25 FPS 写入的缓存，如果你想在 24 FPS 中使用这个缓存并希望获得相同的结果，那么你需要移动/缩放时间样本. 下面的示例中将更详细地说明这一点]
~~~

~~~admonish danger
Currently USD has no concept of animation curves or multi time value interpolation other than `linear`. This means that in DCCs you'll have to grab the time data around your frame and have the DCCs re-interpolate the data. In Houdini this can be easily done via a `Retime` SOP node.

[ 目前 USD 除了线性插值外，没有动画曲线或多时间值插值的概念. 这意味着在 DCC 中您需要抓取您帧周围的时间数据，并让 DCC 重新插值这些数据. 在Houdini中，这可以通过一个 Retime SOP 节点轻松完成]
~~~

The possible stage interpolation types are:

[ stage 中已经实现的插值类型有]

- `Usd.InterpolationTypeLinear` (Interpolate linearly (if array length doesn't change and data type allows it))

    [ Usd.InterpolationTypeLinear （线性插值（如果数组长度不变且数据类型允许））]
- `Usd.InterpolationTypeHeld` (Hold until the next time sample)

    [ Usd.InterpolationTypeHeld （保持直到下一次采样）]

These can be set via `stage.SetInterpolationType(<Token>)`. Value queries before/after the first/last time sample will be clamped to these time samples.

[ 可以通过 stage.SetInterpolationType(\<Token\>) 来设置. 当在时间线上查询第一个或最后一个时间样本之前或之后的值时，查询结果将会被限制在第一个和最后一个已知的时间样本之内. 意思是如果查询的时间点超出了时间样本的范围，那么查询结果将会被“限制”在最近的时间样本的值上，而不是返回未定义或错误的结果]

Render delegates access the time samples within the shutter open close values when motion blur is enabled. When they request a value from Usd and a time sample is not found at the exact time/frame that is requested, it will be linearly interpolated if the array length doesn't change and the data type allows it.

[ 当启用运动模糊时，渲染代理会访问快门打开和关闭时间内的时间样本. 当从 USD 查询一个值时，如果查询确切的时间/帧没有找到对应的时间样本并且数组长度不变且数据类型允许，将会返回进行线性插值的结果]

~~~admonish danger
Since value lookups in USD can only have one value source, you cannot combine time samples from different layers at run time, instead you'll have to re-write the combined values across the whole frame range. This may seem unnecessary, but since USD is a cache based format and to keep USD performant, value source lookup is only done once and then querying the data is fast. This is the mechanism what makes USD able to expand and load large hierarchies with ease.

[ 由于 USD 中的值查找只能有一个值来源，因此您不能在运行时组合来自不同层的时间样本. 相反您必须重新写入整个帧范围内的组合值. 这看似没有必要但由于 USD 是基于缓存的格式，并且为了保持 USD 的高性能值来源查找只执行一次然后查询数据会很快. 正是这种机制使 USD 能够轻松扩展和加载大型层级结构]
~~~


For example:

[ 例如]

```python
# We usually want to write time samples in the shutter open/close range of the camera times the sample count of deform/xform motion blur.

# 我们通常希望在相机快门打开/关闭的范围内写入足够的变形/运动模糊的样本数
double size.timeSamples = {
    1001: 2,
    1002: 2.274348497390747,
    1003: 3.0096023082733154,
    1004: 4.0740742683410645,
    1005: 5.336076736450195,
    1006: 6.663923263549805,
    1007: 7.9259257316589355,
    1008: 8.990397453308105,
    1009: 9.725651741027832,
    1010: 10,
}
# If we omit time samples, value requests for frames in between will get a linearly interpolated result.

# 如果我们省略时间样本，那么对于中间帧的值请求将会得到一个线性插值的结果
double scale.timeSamples = {
    1001: 2,
    1005: 5.336076736450195,
    1010: 10,
}
```

~~~admonish warning
Since an attribute can only have a single value source, we can't have a `default` value from layer A and time samples from layer B. We can however have default and time samples values from the same value source.

[ 由于一个属性只能有一个值来源，因此我们不能从图层 A 获取一个默认值，并从图层 B 获取时间样本. 但是我们可以从同一个值来源获取默认值和时间样本值]

For example:
```python
def Cube "Cube" (
)
{
    double size = 15
    double size.timeSamples = {
        1001: 1,
        1010: 10,
    }
}
```
If we now request the value without a frame, it will return `15`, if we request it with a time, then it will linearly interpolate or given the time sample if it exists on the frame.

[ 如果我们现在请求没有的帧，它将返回 15 如果我们请求一个时间，那么它将线性插值或给出时间样本（如果帧存在）]
```python
...
size_attr.Get() # Returns: 15.0
size_attr.Get(1008) # Returns: 8.0
...

```
Usually we'll only have one or the other, it is quite common to run into this at some point though. So when you query data, you should always use `.Get(<frame>)` as this also works when you don't have time samples. It will then just return the default value. Or you check for time samples, which in some cases can be quite expensive. We'll look at some more examples on this, especially with Houdini and causing node time dependencies in our [Houdini](../../dcc/houdini/hda/timedependency.md) section.

[ 通常我们只会使用其中一种，不过在实际操作中确实经常会遇到这样的情况. 因此当你查询数据时应该始终使用 .Get(\<frame\>)，因为即使在没有时间样本的情况下这种方法也有效. 如果没有时间样本它只会返回默认值. 或者你也可以检查时间样本，但在某些情况下这可能会相当耗费资源. 我们将通过一些更具体的例子来探讨这一点，尤其是在 [Houdini](../../dcc/houdini/hda/timedependency.md) 部分，我们将讨论如何在 Houdini 中造成节点时间依赖]
~~~

### Time Code <a name="animationTimeCode"></a>
The `Usd.TimeCode` class is a small wrapper class for handling time encoding. Currently it does nothing more that storing if it is a `default` time code or a `time/frame` time code with a specific frame. In the future it may get the concept of encoding in `time` instead of `frame`, so to future proof your code, you should always use this class instead of setting a time value directly.

[ Usd.TimeCode 是一个小型包装类，用于处理时间编码. 目前它除了存储是默认时间码和特定帧的 time/frame 时间码之外没有其他功能. 未来它可能会引入以 time 而不是 frame 编码的概念，因此为了使您的代码能够适应未来的变化，您应该始终使用这个类而不是直接设置时间值]

~~~admonish info title=""
```python
{{#include ../../../../code/core/elements.py:animationTimeCode}}
```
~~~

### Layer Offset (A Non-Animateable Time Offset/Scale for Composition Arcs) <a name="animationLayerOffset"></a>
The `Sdf.LayerOffset` is used for encoding a time offset and scale for composition arcs.

[ Sdf.LayerOffset 用于编码合成弧（composition arcs）的时间偏移和缩放]

~~~admonish warning
The offset and scale cannot be animated.

[ offset 和 scale无法设置动画]
~~~

Following composition arcs can use it:

[ 以下合成弧可以使用它]

- Sublayers
- Payloads
- References (Internal & file based)

The Python exposed LayerOffsets are always read-only copies, so you can't modify them in-place.
Instead you have to create new ones and re-write the arc/assign the new layer offset.

[ Python 公开的 LayerOffset 始终是只读副本，因此您无法直接修改它们. 相反您必须创建新的并重写弧/分配新的图层偏移量]

~~~admonish info title=""
```python
{{#include ../../../../code/core/elements.py:animationLayerOffset}}
```
~~~

If you are interested on how to author composition in the low level API, checkout our [composition](../composition/overview.md) section.

[ 如果您对如何在低级 API 中编辑合成感兴趣，请查看[composition](../composition/overview.md)]

### Reading & Writing default values, time samples and value blocks <a name="animationReadWrite"></a>

#### Writing data

Here are the high and low level APIs to write data.

[以下是通过高级和低级 API 写入数据]

~~~admonish tip
When writing a large amount of samples, you should use the low level API as it is a lot faster.

[ 编写大量示例时，您应该使用低级API，因为它要快得多]
~~~

~~~admonish info title=""
```python
{{#include ../../../../code/core/elements.py:animationWrite}}
```
~~~

If you are not sure if a schema attribute can have time samples, you can get the variability hint.
This is only a hint, it is up to you to not write time samples. In some parts of Usd things will fail or not work as exepcted if you write time samples for a non varying attribute.

[ 如果您不确定 schema 的属性是否可以设置时间样本您可以获取 variability 提示. 这只是一个提示是否写入时间样本取决于您自己. 在 USD 的某些部分中，如果您为非可变属性写入时间样本，可能会导致操作失败或无法按预期工作]

~~~admonish info title=""
```python
attr.GetMetadata("variability") == Sdf.VariabilityVarying
attr.GetMetadata("variability") == Sdf.VariabilityUniform 
```
~~~

#### Reading data

To read data we recommend using the high level API. That way you can also request data from value clipped (per frame loaded Usd) files. The only case where reading directly in the low level API make sense is when you need to open a on disk layer and need to tweak the time samples. Check out our [FX Houdini](../../dcc/houdini/fx/pointinstancers.md) section for a practical example.

[ 我们建议使用高级 API 来读取数据. 这样您还可以从按帧加载的 USD 中请求 value clipped 的数据文件. 唯一需要使用低级 API 直接读取数据的情况是，当您需要打开一个磁盘上的层并需要调整时间样本时. 您可以查看我们的 [FX Houdini](../../dcc/houdini/fx/pointinstancers.md) 部分以获取实际示例]

~~~admonish tip
If you need to check if an attribute is time sampled, run the following:
```python
{{#include ../../../../code/core/elements.py:animationTimeVarying}}
```
If you know the whole layer is in memory, then running GetNumTimeSamples()
is fine, as it doesn't have t open any files.

[ 如果您知道整个层都在内存中，那么运行 GetNumTimeSamples() 就可以了，因为它不需要打开任何文件]
~~~

~~~admonish info title=""
```python
{{#include ../../../../code/core/elements.py:animationRead}}
```
~~~

#### Special Values
You can also tell a time sample to block a value. Blocking means that the attribute at that frame will act as if it doesn't have any value written ("Not authored" in USD speak) to stage queries and render delegates.

[ 您还可以在时间样本 block 某个值. block 该帧的属性在舞台查询和渲染过程中将表现得像没有设定任何值一样（在USD中称为“未设定”）]

~~~admonish info title=""
```python
{{#include ../../../../code/core/elements.py:animationSpecialValues}}
```
~~~

### Time Metrics (Frames Per Second & Frame Range) <a name="animationMetadata"></a>
With what FPS the samples are interpreted is defined by the `timeCodesPerSecond`/`framesPerSecond` metadata.

[ 解析时间样本所使用的帧率（FPS）是由 timeCodesPerSecond/framesPerSecond 元数据定义的]

When working with stages, we have the following [loading order of FPS](https://openusd.org/dev/api/class_usd_stage.html#a85092d7455ae894d50224e761dc6e840):

[ stage 中 FPS [加载顺序](https://openusd.org/dev/api/class_usd_stage.html#a85092d7455ae894d50224e761dc6e840)如下]

1. timeCodesPerSecond from session layer [ 来自会话层的 timeCodesPerSecond]
1. timeCodesPerSecond from root layer [ 来自根层的 timeCodesPerSecond]
1. framesPerSecond from session layer [ 来自会话层 framesPerSecond]
1. framesPerSecond from root layer [ 来自跟层的 framesPerSecond]
1. fallback value of 24 [ 默认值]

These should match the FPS settings of your DCC. The 'framesPerSecond' is intended to be a hint for playback engines (e.g. your DCC/Usdview etc.) to set the FPS to when reading your file. The 'timeCodesPerSecond' describes the actual time sample intent. With the fallback behavior we can also only specify the 'framesPerSecond' to keep both metadata entries in sync.

[ 这些设置应该与您的 DCC 的 FPS 设置相匹配. “framesPerSecond” 旨在作为播放引擎（例如 DCC/Usdview 等）在读取文件时设置 FPS.“timeCodesPerSecond” 描述了实际的时间样本意图. 同时为了保持这两个元数据条目的同步，可以只指定 “framesPerSecond” 并使用回调行为]

When working with layers, we have the following [loading order of FPS](https://openusd.org/dev/api/class_sdf_layer.html#a8c7a1ac2e85efa2aa4831123de576b7c):

[ layer 中FPS [加载顺序](https://openusd.org/dev/api/class_sdf_layer.html#a8c7a1ac2e85efa2aa4831123de576b7c)如下]

1. timeCodesPerSecond of layer
1. framesPerSecond of layer
1. fallback value of 24

~~~admonish info
When loading samples from a sublayered/referenced or payloaded file, USD automatically uses the above mentioned metadata in the layer as a frame of reference of how to bring in the time samples. If the FPS settings mismatch it will automatically scale the time samples to match our stage FPS settings as mentioned above.

[ 通过 sublayered/referenced 或 payloaded 加载样本时，USD 会自动使用上述层中的元数据作为参考框架，以确定如何引入时间样本. 如果 FPS 设置不匹配，它将自动调整时间样本以匹配我们上面提到的 stage FPS 设置]

Therefore when writing layers, we should always write these layer metrics, so that we know what
the original intended FPS were and our caches work FPS independently.

[ 因此，在编写层时我们始终应该编写层度量，这样我们才知道最初预期的 FPS 是多少，以及我们的缓存是如何独立于 FPS 工作的]
~~~

~~~admonish warning
In VFX we often work starting from 1001 regardless of the FPS as it is easier for certain departments like FX to have pre-cache frames to init their sims as well as it also makes it easier to write frame based expressions. That means that when working with both 25 and 24 FPS caches, we have to adjust the offset of the incoming cache.

[ 在视觉特效（VFX）中无论帧率（FPS）如何，我们通常从 1001 帧开始工作，因为这样更便于某些部门（特效部门）使用预缓存帧来初始化模拟，同时也更方便编写基于帧的表达式. 这意味着当同时处理 25 FPS 和 24 FPS 的缓存时，我们必须调整传入缓存的偏移量]

Let's say we have an anim in 25 FPS starting off at 1001 that we want to bring into a 24 FPS scene. USD as mentioned above handles the scaling for us based on the metadata, but since we still want to start at 1001, we have to offset based on "frame_start * (stage_fps/layer_fps) - frame_start". See the below commented out code for a live example. That way we now have the same 25 FPS cache running in 24 FPS from the same "starting pivot" frame. If we work fully time based, we don't have this problem, as animation in the 25 FPS cache would have its time samples written at larger frames than the 24 FPS cache and USD's scaling would auto correct it.

[ 假设我们有一个以 25 FPS 开始的动画，其起始帧为1001，我们想将其导入到 24 FPS 的场景中. 如上所述，USD 会根据元数据为我们处理缩放问题，但由于我们仍然希望从 1001 帧开始，因此我们必须根据 “frame_start * (stage_fps/layer_fps) - frame_start” 进行偏移. 下面的注释代码提供了一个实时示例. 这样我们现在就可以从相同的 “起始点” 帧开始，在 24 FPS 的场景中使用相同的 25 FPS 缓存. 如果我们完全基于时间工作，就不会遇到这个问题，因为 25 FPS 缓存中的动画其时间样本的帧会大于 24 FPS 缓存中的帧，USD 的缩放功能会自动进行校正]
~~~

```python
(
    timeCodesPerSecond = 24
    framesPerSecond = 24
    metersPerUnit = 1
    startTimeCode = 1001
    endTimeCode = 1010
)
```

The `startTimeCode` and `endTimeCode` entries give intent hints on what the (useful) frame range of the USD file is. Applications can use this to automatically set the frame range of the stage when opening a USD file or use it as the scaling pivot when calculating time offsets or creating loop-able caches via value clips.

[ startTimeCode 和 endTimeCode 条目提供了关于 USD 文件（使用的）帧范围的标识. 应用程序可以使用这些标识在打开USD文件时自动设置 stage 的帧范围，或者在计算时间偏移或通过使用值剪辑创建循环缓存时将其用作缩放]

~~~admonish info title=""
```python
{{#include ../../../../code/core/elements.py:animationFPS}}
```
~~~


### Motion Blur - Computing Velocities and Accelerations <a name="animationMotionVelocityAcceleration"></a>
Motion blur is computed by the hydra delegate of your choice using either the interpolated position data or by making use of velocity/acceleration data.
Depending on the image-able schema, the attribute namings slightly differ, e.g. for meshes the names are 'UsdGeom.Tokens.points', 'UsdGeom.Tokens.velocities', 'UsdGeom.Tokens.accelerations'. Check the specific schema for the property names.

[ 运动模糊由您选择的 Hydra 委托使用位置插值数据或使用速度/加速度数据来计算. 根据 image-able schema 的不同，属性命名略有不同，例如对于mesh 网格，名称为“UsdGeom.Tokens.points”、“UsdGeom.Tokens.velocities”、“UsdGeom.Tokens.accelerations” 来检查属性名称的特定 schema]

~~~admonish warning
Depending on the delegate, you will likely have to set specific primvars that control the sample rate of the position/acceleration data.

[ 根据委托的不同您可能必须设置特定的 primvar 来控制位置/加速度数据的采样率]
~~~

We can also easily derive velocities/accelerations from position data, if our point count doesn't change:

[ 如果我们的点数不改变，我们可以轻松地从位置数据推导出速度/加速度]

~~~admonish info title=""
```python
{{#include ../../../../code/core/elements.py:animationMotionVelocityAcceleration}}
```
~~~

You can find a interactive Houdini demo of this in our [Houdini - Motion Blur](../../dcc/houdini/fx/motionblur.md) section.

[ 您可以在我们的 [Houdini - Motion Blur](../../dcc/houdini/fx/motionblur.md) 部分找到该功能的交互式 Houdini 演示]

### Stitching/Combining time samples<a name="animationStitch"></a>
When working with Usd in DCCs, we often have a large amount of data that needs to be exported per frame. To speed this up, a common practice is to have a render farm, where multiple machines render out different frame ranges of scene. The result then needs to be combined into a single file or loaded via value clips for heavy data (as described in the next section below).

[ 在 DCC 中使用 Usd 时，我们经常需要每帧导出大量数据. 为了加快速度常见的做法是拥有一个渲染场，其中多台机器渲染出场景的不同帧范围. 然后需要将结果合并到单个文件中或通过值剪辑加载大量数据（如下一节所述）]

~~~admonish tip
Stitching multiple files to a single file is usually used for small per frame USD files. If you have large (1 GB > ) files per frame, then we recommend using values clips. During stitching all data has to be loaded into memory, so your RAM specs have to be high enough to handle all the files combined.

[ 将多个文件拼接到一个文件通常用于每帧较小的 USD 文件. 如果每帧有较大 (1 GB > ) 文件，那么我们建议使用值剪辑. 在拼接过程中所有数据都必须加载到内存中，因此 RAM 规格必须足够高才能处理合并的所有文件]
~~~

A typical production use case we use it for, is rendering out the render USD files per frame and then stitching these, as these are usually a few mb per frame at most.

[ 我们使用它的一个典型的生产案例是输出每帧的 USD 文件，然后缝合这些文件. 这些文件通常每帧最多几MB]

~~~admonish warning
When working with collections, make sure that they are not to big, by selecting parent prims where possible. Currently USD stitches target path lists a bit inefficiently, which will result in your stitching either not going through at all or taking forever. See our [collections](../composition/relationships.md) section for more details.

[ 在使用集合时请确保它们不要太大并尽可能选择父级 prim. 目前 USD 在拼接目标路径列表时的效率有些低，这可能会导致你的拼接根本无法进行或耗时极长. 有关更多详细信息，请参阅我们的 [集合](../composition/relationships.md) 部分]
~~~

USD ships with a standalone `usdstitch` commandline tool, which is a small Python wrapper around the `UsdUtils.StitchLayers()` function. You can read more it in our [standalone tools](./standalone_utilities.md) section.

[ USD 附带一个独立的 usdstitch 命令行工具，它是 UsdUtils.StitchLayers() 函数的一个小型 Python 包装器。您可以在 [独立工具](./standalone_utilities.md) 部分阅读更多内容]

In Houdini you can find it in the `$HFS/bin`folder, e.g. `/opt/hfs19.5/bin`.

[ 在 Houdini 中，您可以在 $HFS/bin 文件夹中找到它，例如 /opt/hfs19.5/bin ]

Here is an excerpt:

[ 摘录如下]

~~~admonish info title=""
```python
{{#include ../../../../code/core/elements.py:animationStitchCmdlineTool}}
```
~~~

More about layer stitching/flattening/copying in our [layer](./layer.md) section.

[ 有关层 stitching/flattening/copying 的更多信息，请参阅[layer](./layer.md)]

### Value Clips (Loading time samples from multiple files)<a name="animationValueClips"></a>
~~~admonish note
We only cover value clips in a rough overview here, we might extend this a bit more in the future, if there is interest. We recommend checking out the [official docs page](https://openusd.org/dev/api/_usd__page__value_clips.html) as it is well written and worth the read!

[ 我们在这里仅粗略地概述值剪辑，如果有兴趣我们将来可能会对此进行更多扩展. 我们建议您查看 [官方文档](https://openusd.org/dev/api/_usd__page__value_clips.html) 因为它写得很好并且值得一读!]
~~~

USD [value clips](https://openusd.org/dev/api/_usd__page__value_clips.html) are USD's mechanism of loading in data per frame from different files. It has a special rule set, that we'll go over now below.

[ USD 值剪辑是 USD 每帧从不同文件加载数据的机制. 它有一个特殊的规则集我们现在将在下面讨论]

~~~admonish important
Composition wise, value clips (or the layer that specifies the value clip metadata) is right under the `local` arc strength and over `inherit` arcs.

[ 就合成而言值剪辑（或指定值剪辑元数据的图层）位于 local 弧强度下, inherit 弧强度上]
~~~

Here are some examples from USD's official docs:

[ 以下是 USD 官方文档中的一些示例]

```python
def "Prim" (
    clips = {
        dictionary clip_set_1 = {
            double2[] active = [(101, 0), (102, 1), (103, 2)]
            asset[] assetPaths = [@./clip1.usda@, @./clip2.usda@, @./clip3.usda@]
            asset manifestAssetPath = @./clipset1.manifest.usda@
            string primPath = "/ClipSet1"
            double2[] times = [(101, 101), (102, 102), (103, 103)]
        }
    }
    clipSets = ["clip_set_1"]
)
{
}
```

There is also the possibility to encode the value clip metadata via a file wild card syntax (The metadata keys start with `template`). We recommend sticking to the the above format as it is more flexible and more explicit.

[ 此外，还可以通过文件通配符语法（元数据键以 “template” 开头）来编码值剪辑元数据. 但我们推荐使用上述格式，因为它更灵活、更明确]

~~~admonish info title="Click here to expand contents" collapsible=true
```python
def "Prim" (
    clips = {
        dictionary clip_set_2 = {
            string templateAssetPath = "clipset2.#.usd"
            double templateStartTime = 101
            double templateEndTime = 103
            double templateStride = 1
            asset manifestAssetPath = @./clipset2.manifest.usda@
            string primPath = "/ClipSet2"
        }
    }
    clipSets = ["clip_set_2"]
)
{
}
```
~~~

As you can see it is pretty straight forward to implement clips with a few key metadata entries:

[ 正如您所见，使用几个关键的元数据条目实现剪辑是非常简单的]

- `primPath`: Will substitute the current prim path with this path, when looking in the clipped files. This is similar to how you can specify a path when creating references (when not using the `defaultPrim` metadata set in the layer metadata).

    [ primPath ：在查看剪辑的文件时，将用此路径替换当前的 prim 路径. 这类似于创建引用时指定路径的方式（当不使用图层元数据中设置的 defaultPrim 元数据时）]
- `manifestAssetPath`: A asset path to a file containing a hierarchy of attributes that have time samples without any default or time sample data.

    [ manifestAssetPath：一个指向包含具有时间样本（无默认或时间样本数据）的属性层级结构的文件的资产路径]
- `assetPaths`: A list of asset paths that should be used for the clip.

    [ assetPaths ：剪辑应使用的资源路径列表]
- `active`: A list of (\<stage time\>, \<asset path list index\>) pairs, that specify on what frame what clip is active.

    [ active ： (\<stage time\>, \<asset path list index\>) 的列表，指定在哪个帧上哪个剪辑处于激活状态]
- `times`: A list of (\<stage time\>, \<asset path time\>) pairs, that map how the current time should be mapped into the time that should be looked up in the active asset path file.

    [ times：一个包含 (\<stage time\>, \<asset path time\>) 的列表，用于映射当前时间应如何在激活资产路径文件中查找给定时间的值]
- `interpolateMissingClipValues` (Optional): Boolean that activates interpolation of time samples from surrounding clip files, should the active file not have any data on the currently requested time.

    [ interpolateMissingClipValues（可选）：布尔值，如果当前激活的文件在请求的时间点上没有数据，系统将启用插值功能，从周围的剪辑文件中提取时间样本来估计或填充缺失的数据]

~~~admonish warning
The content of individual clip files must be the raw data, in other words anything that is loaded in via composition arcs is ignored.

[ 各个剪辑文件的内容必须是原始数据，换句话说，通过合成弧加载的任何内容都将被忽略]
~~~

The other files that are needed to make clips work are:

[ 使剪辑工作所需的其他文件包括]

- The `manifest` file: A file containing a hierarchy of attributes that have time samples  without any default or time sample data.

    [ manifest 文件：包含了场景中对象的动态属性，这些属性随时间变化而有所不同，但它们没有默认的数据或时间样本数据.(这意味着它们的具体值需要在播放过程中通过其他方式（如采样或计算）来确定)]
- The `topology` file: A file containing all the attributes that only have static `default` data.

    [ topology 文件：包含了场景中对象的静态属性，这些属性不随时间变化，而是具有默认值. (这些属性可能包括对象的形状、结构或其他不随时间改变的属性)]

Here is how you can generate them:
USD ships with a `usdstitchclips` commandline tool that auto-converts multiple clip (per frame) files to a value clipped main file for you. This works great if you only have a single root prim you want to load clipped data on.

[ 下面是如何生成它们的方法：USD 附带了一个名为 usdstitchclips 的命令行工具，该工具可以自动将多个（每帧一个）剪辑文件转换为一个带有值剪辑的主文件. 如果你只想在单个 root prim 上加载剪辑数据，那么这个方法非常有效]

Unfortunately this is often not the case in production, so this is where the value clip API comes into play. The `usdstitchclips` tool is a small Python wrapper around that API, so you can also check out the Python code there.

[ 不幸的是，在实际制作中情况往往并非如此，这正是值剪辑 API 发挥作用的地方 usdstitchclips 工具是围绕该 API 的一个小型 Python 包装器，因此您也可以在那里查看 Python 代码]

Here are the basics, the main modules we will be using are `pxr.Usd.ClipsAPI` and `pxr.UsdUtils`:

[ 以下是一些基础知识，我们主要使用的模块包括：pxr.Usd.ClipsAPI 和 pxr.UsdUtils ]

~~~admonish tip
Technically you can remove all default attributes from the per frame files after running the topology layer generation. This can save a lot of disk space,but you can't partially re-render specific frames of the cache though. So only do this if you know the cache is "done".

[ 从技术上来说，在 topology laye r生成之后你可以从每帧文件中移除所有默认属性. 这可以节省大量的磁盘空间，但你无法对缓存中的特定帧进行部分重新渲染. 因此只有在你确定缓存 “已完成” 时才进行此操作]
~~~

~~~admonish info title=""
```python
{{#include ../../../../code/core/elements.py:animationStitchClipsUtils}}
```
~~~

Since in production (see the next section), we usually want to put the metadata at the asset roots, we'll usually only want to run

[ 由于在生产中（请参阅下一节），我们通常希望将元数据放在资产根目录中，因此我们通常只想运行]

- `UsdUtils.StitchClipsTopology(topology_layer, time_sample_files)`
- `UsdUtils.StitchClipsManifest(manifest_layer, topology_layer, time_sample_files, clip_prim_path)`

And then create the clip metadata in the cache_layer ourselves:

[ 然后我们自己在 cache_layer 中创建剪辑元数据]

~~~admonish info title=""
```python
{{#include ../../../../code/core/elements.py:animationStitchClipsAPI}}
```
~~~


#### How will I use it in production?

[ 我将如何在生产中使用它？]

Value clips are the go-to mechanism when loading heavy data, especially for animation and fx. They are also the only USD mechanism for looping data.

[ 值剪辑是加载大量数据时的首选机制，尤其是动画和特效. 它也是唯一用于循环数据的 USD 机制]

As discussed in more detail in our [composition](../composition/overview.md) section, caches are usually attached to asset root prim. As you can see above, the metadata must always specify a single root prim to load the clip on, this is usually your asset root prim. You could also load it on a prim higher up in the hierarchy, this makes your scene incredibly hard to debug though and is not recommended.

[ 正如我们的 [合成](../composition/overview.md) 部分中详细讨论的，缓存通常附加到资产 root prim. 正如您在上面看到的元数据必须始终指定一个 root prim 来加载剪辑这通常是您的资产 root prim. 您还可以将其加载到层级结构中更高的 prim 上，但这会使您的场景难以调试，不建议这样做]

This means that if you are writing an fx cache with a hierarchy that has multiple asset roots, you'll be attaching the metadata to each individual asset root. This way you can have a single value clipped cache, that is loaded in multiple parts of you scene. You can then payload/reference in this main file, with the value clip metadata per asset root prim, per asset root prim, which allows you to partially load/unload your hierarchy as usual.

[ 这意味着，如果你正在编写具有多个资产根节点的层级结构的特效缓存时，你需要将元数据附加到每个单独的资产根节点上. 这样你就可以拥有一个单一的值剪辑缓存，该缓存可以在场景的多个部分中加载. 你可以在这个主文件中使用 payload/reference ，每个资产根节点的 prim 都带有值剪辑元数据，这允许你像往常一样部分加载/卸载你的层级结构]

Your file structure will look as follows:

[ 您的文件结构将如下所示]

- Per frame(s) files with time sample data:

    [ 包含时间样本数据的每帧文件]
    - /cache/value_clips/time_sample.1001.usd
    - /cache/value_clips/time_sample.1002.usd
    - /cache/value_clips/time_sample.1003.usd
- Manifest file (A lightweight Usd file with attributes without values that specifies that these are attributes are with animation in clip files):

    [ Manifest file（一个轻量级的 USD 文件，其中的属性不包含值，而是作为一种标记或声明存在，指示系统在其他地方（即剪辑文件中）查找这些属性的动画数据]
    - /cache/value_clips/manifest.usd
- Topology file (A USD file that has all attributes with default values):

    [ Topology file （一个USD文件，其中包含的所有属性都设置了默认值）这意味着, 当这个USD文件被加载或解析时，不需要额外的数据或信息来填充这些属性，因为它们已经具有预设的默认值]
    - /cache/value_clips/topology.usd
- Value clipped file (It sublayers the topology.usd file and writes the value clip metadata (per asset root prim)):

    [ Value clipped file（它会对 topology.usd 文件进行子层划分，并写入值剪辑元数据（每个资产 root prim）]
    - /cache/cache.usd

Typically your shot or asset layer USD files will then payload or reference in the individual asset root prims from the cache.usd file.

[ 通常，您的镜头或资产层 USD 文件将从cache.usd文件中 payload 或 reference 各个资产根节点 prims]

~~~admonish important
Since we attach the value clips to asset root prims, our value clipped caches can't have values above asset root prims.

[ 由于我们将值剪辑附加到资产 root prims，因此我们的值剪辑缓存的值不能高于资产 root prims]
~~~

If this all sounds a bit confusing don't worry about it for now, we have a hands-on example in our
[composition](../composition/overview.md) section.

[ 如果这一切听起来有点令人困惑，现在不用担心，我们的 [composition](../composition/overview.md) 部分有一个实践案例]

~~~admonish tip title="Value Clips and Instanceable Prims"
For more info on how value clips affect instancing, check out our [composition](../composition/overview.md) section. There you will also find an example with multiple asset roots re-using the same value clipped cache.

[ 有关值剪辑如何影响实例的更多信息请查看 [合成](../composition/overview.md) 在那里您可以找到一个示例，其中多个资产根重新使用相同的值剪切缓存]
~~~


#### How does it affect attribute time samples and queries? [它如何影响时间采样和查询?]

When working with time samples in value clips there are two important things to keep in mind:

[ 在处理值剪辑中的时间样本时，需要记住两件重要的事情]

##### Subframes
The `active` and `times` metadata entries need to have sub-frames encoded. Let's look at our example:

[ active 和 times 元数据需要对子帧进行编码. 让我们看看例子]

Three per frame files, with each file having samples around the centered frame:

[ 三个单帧文件，每个文件在中心帧周围都有样本]

- /cache/value_clips/time_sample.1001.usd": (1000.75, 1001, 1001.25)
- /cache/value_clips/time_sample.1002.usd": (1001.75, 1002, 1002.25)
- /cache/value_clips/time_sample.1003.usd": (1002.75, 1003, 1003.25)

They must be written as follows in order for subframe time sample to be read.

[ 必须按如下方式编写它们才能读取子帧时间样本]

```python
double2[] active = [(1000.5, 0), (1001.75, 1), (1002.75, 2)]
double2[] times = [(1000.5, 1000.5), (1001.75, 1001.75), (1002.75, 1003)]
```
As you may have noticed, we don't need to specify the centred or .25 frame, these will be interpolated linearly to the next entry in the list.

[ 正如您可能已经注意到的，我们不需要指定中心帧或 .25 帧，这些帧将通过线性插值计算得到列表中的下一个]

##### Queries
When we call attribute.GetTimeSamples(), we will get the interval that is specified with the `times` metadata.
For the example above this would return:

[ 当我们调用 attribute.GetTimeSamples() 时，我们将获取使用 times 元数据指定的间隔. 对于上面的例子，这将返回]

```python
(1000.75, 1001, 1001.25, 1001.75, 1002, 1002.25, 1002.75, 1003, 1003.25)
```

If we would only write the metadata on the main frames:

[ 如果我们只在主文件中写入元数据]

```python
double2[] active = [(1001, 0), (1002, 1), (1003, 2)]
double2[] times = [(1001, 1001), (1002, 1002), (1003, 1003)]
```
It will return:

[ 它将返回]

```python
(1001, 1001.25, 1002, 1002.25, 1003, 1003.25)
```
~~~admonish important
With value clips it can be very expensive to call `attribute.GetTimesamples()` as this will open all layers to get the samples in the interval that is specified in the metadata. It does not only read the value clip metadata. If possible use `attribute.GetTimeSamplesInInterval()` as this only opens the layers in the interested interval range.

[ 使用值剪辑时调用 attribute.GetTimesamples() 可能会非常昂贵，因为它会打开所有层以获取在元数据中指定的时间间隔内的样本. 它不仅仅是读取值剪辑元数据. 如果可能的话请使用 attribute.GetTimeSamplesInInterval() 因为它只会在感兴趣的时间间隔范围内打开层]
~~~

