# General Approach [一般策略]
~~~admonish question title="Still under construction!"
We'll expand our Houdini section in the future with topics such as:

[ 未来我们将扩展 Houdini 部分，主题包括]
- lighting
- rendering (render products/render vars (USD speak for AOVs)/render procedurals)
- asset/shot templates
~~~

This page will focus on the basics of what you need to know before getting started in LOPs.

[ 本页将重点介绍开始使用 LOP 之前需要了解的基础知识]

Currently this is limited to LOPs basics and SOP geometry importing/exporting, we'll expand this in the future to other topics.

[ 目前这仅限于 LOP 基础知识和 SOP 几何导入/导出，我们将来会将其扩展到其他主题]

# Table of Contents [目录]
1. [Houdini LOPs In-A-Nutshell](#summary)
1. [What should I use it for?](#usage)
1. [Resources](#resources)
1. [Overview](#overview)
1. [Artist Vs Pipeline](#artistVsPipeline)
1. [Path Structure](#path)
1. [How to convert between LOPs and SOPs](#IO)
    1. [Importing from LOPs to SOPs](#IOLopsToSops)
    1. [Exporting from SOPs to LOPs](#IOSopsToLops)
    1. [Stage/Layer Metrics](#IOLayerMetrics)
1. [Composition](#composition)
    1. [Asset Resolver](#compositionAssetResolver)
    1. [Creating Composition Arcs](#compositionAssetResolver)

## TL;DR - Approach In-A-Nutshell [概述]<a name="summary"></a>
When working in Houdini, the basics our pipeline has to handle is USD import/export as well as setting up the correct composition.

[ 在Houdini中工作时，我们流程的基本任务是需要处理 USD 的导入/导出以及设置正确的合成]

As covered in our composition section, composition arcs are centered around loading a specific prim (and its children) in the hierarchy. We usually design our path structure around "root" prims. That way we can load/unload a specific hierarchy selection effectively. With value clips (USD speak for per frame/chunk file loading) we also need to target a specific root prim, so that we can keep the hierarchy reference/payloadable and instanceable.

[ 正如我们在合成部分所介绍的那样，合成弧主要围绕在层级结构中加载特定的 prim （及其子项）进行. 我们通常会围绕 "root" prims 来设计我们的路径结构. 这样，我们就可以有效地加载/卸载特定的层级结构. 使用值剪辑（在USD中指的是按帧/块文件加载）时，我们也需要针对特定的 "root" prims ，以确保我们可以保持层级结构的 reference/payloadable 和 instanceable]

To make it convenient for our artists to use USD, we should therefore make handling paths and composition as easy as possible. Our job is to abstract away composition, so that we use its benefits as best as possible without inconveniencing our artists.

[ 为了使我们的艺术家能够更方便地使用USD，我们应该尽可能简化路径和合成的处理. 我们的工作是抽象化合成的过程，以便我们能够尽可能好地利用它的优势，同时不给我们的艺术家带来不便]

~~~admonish tip title="Houdini | SOPs to LOPs Path | Evaluation Order"
As for paths, Houdini's SOPs to LOPs mechanism in a nutshell is:

[ 至于 paths，简单来说，Houdini中的SOPs到LOPs机制是]
1. Check what "path" attribute names to consult

    [ 检查需要参考的 “paths” 属性名称]
1. Check (in order) if the attribute exists and has a non-empty value

    [ 检查(按顺序)属性是否存在且具有非空值]
1. If the value is relative (starts with "./some/Path", "some/Path", "somePath", so no `/`), prefix the path with the setting defined in "Import Path Prefix" (unless it is a point instance prototype/volume grid path, see exceptions below).

    [ 如果值是相对的（以 "./some/Path"、"some/Path"、"somePath" 开头，即不以 “/” 开头），使用 "Import Path Prefix" 中定义的值作为路径前缀（除非它是 point instance prototype/volume path，请参阅下面的例外情况）]
1. If no value is found, use a fallback value with the `<typeName>_<idx>` syntax.

    [ 如果没有找到任何值，则返回按\<typeName\>_\<idx\>语法的返回值]

There are two special path attributes, that cause a different behavior for the relative path anchoring.

[ 有两个特殊的  path attributes ，它们会在相对路径处理时产生不同的行为]
- **usdvolumesavepath**: This defines the path of your "Volume" prim.

  [ usdvolumesavepath：这个属性定义了你的“Volume” prim 的路径]
- **usdinstancerpath**: This defines the path of your "PointInstancer" prim (when exporting packed prims as point instancers).

  [ usdinstancerpath：这个属性定义了你的“PointInstancer” prim 的路径（当将打包的 prim 导出为 point instancers 时）]
~~~

## What should I use it for? <a name="usage"></a>

[ 我应该用它做什么？]
~~~admonish tip
We'll be using LOPs to create awesome VFX via USD!

[ 我们将使用 LOP 通过 USD 创建出色的视觉特效！]
~~~

## Resources [资源]<a name="resources"></a>
- [Importing SOP geometry into USD](https://www.sidefx.com/docs/houdini/solaris/sop_import.html)
- [Solaris Performance](https://www.sidefx.com/docs/houdini/solaris/performance.html)
- [Houdini Configure Layer](https://www.sidefx.com/docs/houdini/nodes/lop/configurelayer.html)

## Overview [概述]<a name="overview"></a>
<video width="100%" height="100%" controls autoplay muted loop>
  <source src="./houdiniNodeCategories.mp4" type="video/mp4" alt="Houdini Node Categories">
</video>

You can find all the examples we take a look at in our [USD Survival Guide - GitHub Repository](https://github.com/LucaScheller/VFX-UsdSurvivalGuide/blob/main/files/dcc/houdini)

[ 您可以在 [USD Survival Guide - GitHub Repository](https://github.com/LucaScheller/VFX-UsdSurvivalGuide/blob/main/files/dcc/houdini) 中找到我们查看的所有示例]

We have a lot of really cool nodes available for us what ship natively with Houdini. Quite a few of them are actually bare-bone wrappers around USD API commands, so that we don't have to master the API to use them.

[ 我们有很多非常酷的节点可供我们使用，它们是由 Houdini 原生提供的. 实际上相当多的是 USD API 命令的简单包装器，因此我们不必掌握 API 即可使用它们]

Now for pipeline developers, these are the nodes you'll primarily be interacting with:

[ 现在对于流程开发人员来说，这些是您将主要与之交互的节点]

![Houdini Pipeline Nodes](houdiniNodePipeline.jpg)

You favorite node will be the Python LOP node, as we have exposure to the full USD API and can modify the stage to our needs.

[ 您最喜欢的节点将是 Python LOP 节点，因为我们已经接触到完整的 USD API，并且可以根据我们的需要修改 stage]

Ready!?! Let's goooooooo!

[ 准备好！？！让我们开始吧！]

## Artist Vs Pipeline <a name="artistVsPipeline"></a>
When switching to a USD backed pipeline, an important thing to not gloss over, is how to market USD to artists.

[ 在切换到以USD为基础的流程时，一个不容忽视的重要问题是如何向艺术家推广USD]

Here are the most important things to keep in mind of what a pipeline should abstract away and what it should directly communicate to artists before getting started:

[ 在开始之前，以下是关于工作流程应该抽象掉哪些内容以及应该直接向艺术家传达哪些内容的最重要事项]
- As USD is its own data format, we will not have the native file IO speeds of .bgeo(.sc). The huge benefit is that we can use our caches in any DCC that offers a USD implementation. The downside is, that we have to be more "explicit" of what data we are exporting. For example for packed geometry we have to define how to map it to USD, e.g. as "PointInstancers", "Xforms" or flattened geometry. This means there is now an additional step, that has to be made aware of, before each export.

  [ 由于USD是其自己的数据格式，因此我们不会拥有 .bgeo(.sc) 的本机文件 IO 速度. 巨大的好处在于，我们可以在任何提供USD实现的 DCC 工具中使用我们的缓存. 然而，缺点是我们必须更“明确”地指出我们要导出哪些数据. 例如，对于打包的几何体，我们必须定义如何将其映射到 USD，比如作为“PointInstancers”、“Xforms”或辗平的几何体. 这意味着现在有一个额外的步骤，需要在每次导出之前让所有人了解]
- USD exposes other departments work to us directly through its layering mechanism. This a very positive aspect, but it also comes with more communication overhead. Make sure you have setup clear communication structures beforehand of who to contact should questions and issues arise.

  [ USD通过其分层机制直接将其他部门的工作暴露给我们. 这是一个非常积极的方面，但也带来了更多的沟通成本. 请确保您已经提前设置了清晰的沟通结构，以便在出现问题和疑问时知道该联系谁]
- USD comes with its own terminology. While we do recommend teaching and learning it, when first transitioning to USD, we should try to keep in familiar waters where possible to soften the blow.

  [ USD带有自己的术语. 虽然我们推荐在首次过渡到USD时进行教学和学习，但我们也应该尽可能保持在熟悉的领域，以减轻冲击]

Here are a few things pipeline can/should cover to make things as easy going as possible:

[ 以下是工作流程可以/应该涵盖的一些内容，以使事情尽可能顺利进行]
- Provide learning material in the form of documentation, follow along workshops and template scenes. We recommend putting a strong focus on this before going "live" as once a show is running, it can cause a high demand in one-on-one artist support. The more you prepare in advance, the more things will flow smoothly. We also recommend tieing artists into your development process, as this keeps you on your toes and also helps ease the transition.

  [ 以文档形式提供学习材料，遵循研讨会和模板场景. 我们建议在“上线”之前将重点放在这方面，因为一旦项目开始运行，可能会产生艺术家一对一支持的高需求. 您准备得越充分，事情就会越顺利. 我们还建议将艺术家纳入您的开发过程，因为这可以让您保持警觉，并有助于平稳过渡]
- A core element pipeline should always handle is data IO. We provide various tips on how to make exporting to LOPs similar to SOP workflows in this guide.

  [ 流程的核心应该始终处理数据 IO. 我们在本指南中提供了有关如何导出到类似于 SOP 工作流程的 LOP 的各种提示]
- We recommend being more restrictive in different workflow aspects rather than allowing to mix'n'match all different styles of geometry import/export and node tree flow "designs". What we mean with this is, that we should "lock" down specific use cases like "What geo am I exporting (characters/water/debris/RBD/etc.)" and build specific HDAs around these. This way there is no ambiguity to how to load in geometry to USD. It also makes pre-flight checks easy to implement, because we know in advance what to expect.

  [ 我们建议在不同的工作流程方面采取更多限制，而不是允许混合使用所有不同的几何体导入/导出和节点树流程“设计”. 我们的意思是，我们应该“锁定”特定的用例，比如“我正在导出哪种几何体（角色/水/碎片/刚体动力学/等）”，并围绕这些用例构建特定的 HDA .这样，如何将几何图形加载到 USD 就没有歧义了. 同时，这也使得预检查更容易实现，因为我们提前知道会发生什么]
- In LOPs, we can often stick to building a "monolithic" node stream (as to SOPs where we often branch and merge). As order of operation in LOPs is important, there are fewer ways to merge data. This means we can/should pre-define how our node tree should be structured (model before material, material before lighting etc.). A good approach is to segment based on tasks and then merge their generated data into the "main" input stream. For example when creating lights, we can create a whole light rig and then merge it in.

  [ 在 LOPs 中，我们通常可以坚持构建一个“整体的”节点流（就像SOPs中我们经常进行分支和合并一样）。由于LOPs中的操作顺序很重要，所以合并数据的方式较少。这意味着我们可以/应该预先定义节点树的结构（模型在材质之前，材质在灯光之前等）. 一个好的方法是根据任务进行分段，然后将它们生成的数据合并到“主”输入流中。例如，在创建灯光时，我们可以创建一个完整的灯光设置，然后将其合并进来]

These tips may sound like a lot of work, but the benefits of USD are well worth it!

[ 这些建议可能听起来需要很多工作，但 USD 的好处绝对值得我们付出努力！]

~~~admonish question title="Still under construction!"
We'll likely expand on this sub-section in the future.

[ 我们将来可能会扩展此小节]
~~~

## Path Structure <a name="path"></a>
As covered in our [composition section](../../core/composition/overview.md), composition arcs are centered around loading a specific prim (and its children) in the hierarchy. We usually design our path structure around "root" prims. That way we can load/unload a specific hierarchy selection effectively. With value clips (USD speak for per frame/chunk file loading) we also need to target a specific root prim, so that we can keep the hierarchy reference/payloadable and instanceable.

[ 正如我们在[合成](../../core/composition/overview.md)部分所提到的，合成弧主要围绕在层级结构中加载特定的 prim（及其子项）.我们通常围绕 "root" prims 设计路径结构. 这样，我们就可以有效地加载/卸载特定的层级结构选择. 使用值剪辑（USD术语，用于按帧/块文件加载）时，我们也需要针对特定的 "root" prims 进行目标定位，以保持层次结构引用/可加载和可实例化]

As pipeline developers, we therefore should make it as convenient as possible for artists to not have to worry about these "root" prims.

[ 作为流程开发人员，我们应该尽可能地为艺术家提供便利，让他们不必担心这些 "root" prims]

We have two options:

[ 我们有两个选择：]
- We give artists the option to not be specific about these "root" prims. Everything that doesn't have one in its name, will then be grouped under a generic "/root" prim (or whatever we define as a "Import Path Prefix" on our import configure nodes). This makes it hard for pipeline to re-target it into a specific (shot) hierarchy. It kind of breaks the USD principle of layering data together.

  [ 我们给艺术家提供选项，让他们不必特意关注这些 "root" prims 然后 名称中没有的所有内容项，将被归组到一个通用的 "/root" prim 下（或我们在导入配置节点上定义为“导入路径前缀”的任何内容）. 这会使 流程 难以将其重新定位到特定的（镜头）层次结构中. 这有点违背了USD将数据分层组合的原则]
- We enforce to always have these root prims in our path. This looses the flexibility a bit, but makes our node network easier to read as we always deal with absolute(s, like the Sith) prim paths.

  [ 我们强制执行在路径中始终包含这些 root prims. 这稍微损失了一些灵活性，但使我们的节点网络更容易阅读，因为我们总是处理 absolute(s, like the Sith) prim paths]

When working in SOPs, we don't have sidecar metadata per path segment (prim) as in LOPs, therefore we need to define a naming convention upfront, where we can detect just based on the path, if a root is defined or not. There is currently no industry standard (yet), but it might be coming sooner than we might think! Say goodbye to vendor specific asset structures, say hello to globally usable assets.

[ 在SOPs中工作时，我们没有像LOPs中那样为每个路径（prim）提供辅助元数据，因此我们需要预先定义一个命名约定，这样我们就可以根据路径来检测是否定义了根. 目前还没有行业标准（尚待制定），但它的到来可能比我们想象的要早！告别供应商特定的资产结构，迎接全球可用的资产]

As also mentioned in our composition section, this means that only prims under the root prims can have data (as the structure above is not payloaded/referenced). Everything we do in SOPs, affects only the leaf prims in world space. So we are all good on that side.

[ 正如我们在合成部分也提到的，这意味着只有 root prims 下的 prims 才能包含数据（因为上面的结构不是 payloaded/referenced ）.我们在SOPs中所做的一切只影响世界空间中的leaf prims 所以我们在这一方面没有问题]

## How to convert between LOPs and SOPs <a name="IOLopsToSops"></a>
To handle the SOPs to LOPs conversion we can either configure the import settings on the LOPs sop import node or we can use the SOPs USD configure node, which sets the exact same settings, but as detail attributes. For pipeline tools, we recommend using the SOPs detail attributes, as we can dynamically change them depending on what we need.

[ 为了处理 SOP 到 LOP 的转换，我们可以在 LOP 中 sop import 节点上配置导入设置，也可以使用 SOP 下 USD configure节点，该节点设置完全相同的设置，但对于 detail attributes. 作为流程工具，我们建议使用 SOP detail attributes，因为我们可以根据需要动态更改它们]

| LOPs Sop Import                                  | SOPs USD Configure Name                            |
|--------------------------------------------------|----------------------------------------------------|
| ![LOPs SOP Import](houdiniLOPsSOPImportPath.jpg) | ![SOPs Usd Configure](houdiniSOPsUsdConfigure.jpg) |

We strongly recommend reading the official docs [Importing SOP geometry into USD](https://www.sidefx.com/docs/houdini/solaris/sop_import.html) as supplementary reading material.

[ 我们强烈建议阅读官方文档  [Importing SOP geometry into USD](https://www.sidefx.com/docs/houdini/solaris/sop_import.html) 作为补充阅读材料]

In our [Basic Building Blocks of Usd](../../core/elements/overview.md) section, the first thing we covered was how to handle paths.
Before we look at our import/export nodes let's do the same for Houdini.

[ 在 [Basic Building Blocks of Usd](../../core/elements/overview.md) 部分中，我们首先讨论的是如何处理路径. 在查看导入/导出节点之前，让我们对 Houdini 做同样的事情]

In Houdini we can define what attributes Houdini consults for defining the `Sdf.Path` for our prims. By default it is the `path` and `name` attribute. When looking up the path, it looks through the path attributes in the order we define on the sop import/USD configure node. If the value is empty it moves onto the next attribute. If it doesn't find a valid value, it will fallback to defaults (`<typeName>_<idx>`, e.g. "mesh_0").

[ 在Houdini中，我们可以在为我们的 prims 定义 Sdf.Path 时参考哪些 Houdini 属性. 默认情况下，这些属性是 path 和 name . 在查找路径时，它会按照我们在 sop import/USD configure node 上定义的顺序查看路径属性. 如果值是空的，它就会移动到下一个属性. 如果找不到有效的值，它将回退到默认值（\<typeName\>_\<idx\>, e.g. "mesh_0"）]

We can also specify relative paths. These are any paths, that don't start with `/`. These will be prefixed with the prefix defined via the "Import Path Prefix" parm on either ones of the configure nodes.

[ 我们还可以指定相对路径,不以 / 开头的任何路径. 它们将被加上配置节点上的“导入路径前缀”参数定义的前缀]

~~~admonish tip title="Houdini | SOPs to LOPs Path | Evaluation Order"
1. Check what "path" attribute names to consult

    [ 检查需要参考的 “paths” 属性名称]
1. Check (in order) if the attribute exists and has a non-empty value

    [ 检查(按顺序)属性是否存在且具有非空值]
1. If the value is relative (starts with "./some/Path", "some/Path", "somePath", so no `/`), prefix the path with the setting defined in "Import Path Prefix" (unless it is a point instance prototype/volume grid path, see exceptions below).

    [ 如果值是相对的（以 "./some/Path"、"some/Path"、"somePath" 开头，即不以 “/” 开头），使用 "Import Path Prefix" 中定义的值作为路径前缀（除非它是 point instance prototype/volume path，请参阅下面的例外情况）]
1. If no value is found, use a fallback value with the `<typeName>_<idx>` syntax.

    [ 如果没有找到任何值，则返回按\<typeName\>_\<idx\>语法的返回值]
~~~

When working with packed prims (or nested levels of packed prims), the relative paths are anchored to the parent packed level for the nested levels. The top level packed prim is then anchored as usual against the import settings prefix.

[ 使用打包 prims（或打包 prims 的嵌套级别）时，相对路径将锚定到嵌套级别的父打包级别. 然后，顶级打包的 prim 像往常一样锚定在导入设置前缀上]

For example:
1. Packed: "/level_0"
    1. Packed: "./level_1"
        1. Mesh: "myCoolMesh"

The resulting path will be "/level_0/level_1/myCoolMesh". Be careful, using "../../../myPath" works too, strongly **not** recommended as it breaks out of the path (like a `cd ../../path/to/other/folder``)!

[ 生成的路径将为“/level_0/level_1/myCoolMesh”. 请小心，使用“../../../myPath”也可以，强烈不推荐，因为它会跳出路径（如 `cd ../../path/to/other/folder``） ！]

~~~admonish danger title="Important | Paths and Packed Prims"
When working with paths in SOPs and packed prims, we have to set the path before we pack the geometry. We can't adjust it afterwards, without unpacking the geometry. If we define absolute paths within packed geometry, they will not be relative to the parent packed level. This can cause unwanted behaviours (as your hierarchy "breaks" out of its packed level). We therefore recommend not having absolute paths inside packed prims.

[ 当使用 SOP 和打包 prims 中的路径时，我们必须在打包几何体之前设置路径. 之后，如果不解压几何体，我们之后就无法调整它. 如果我们在打包的几何体内部定义绝对路径，它们将不会相对于父级打包层级. 这可能会导致不期望的行为（例如，你的层级“跳出”其打包层级）. 因此，我们建议不要在打包的 prim 中使用绝对路径]

We can easily enforce this, by renaming our "outside" packed attributes to something else and using these as "path" import attributes. That way the inside get's the fallback behavior, unless explicitly set to our "outer" path attributes.

[ 我们可以通过将“外部”打包属性重命名为其他内容并将它们用作“路径”导入属性来轻松地强制执行此操作. 这样，除非明确设置为我们的“外部”路径属性，否则内部将使用后备行为]
~~~

There are two special path attributes, that cause a different behavior for the relative path anchoring.

[ 有两个特殊的 path attributes ，它们会在相对路径处理时产生不同的行为]
- **usdvolumesavepath**: This defines the path of your "Volume" prim. As for volumes we usually use the "name" attribute to drive the volume grid/field name, this gives us "/my/cool/explosionVolume/density", "/my/cool/explosionVolume/heat", etc, as long as we don't define another path attribute that starts with "/". So this gives us a convenient way to define volumes with (VDB) grids.

  [ usdvolumesavepath：这个属性定义了你的“Volume” prim 的路径. 对于体积，我们通常使用 “name” 属性来驱动 volume grid/field name，这样我们就可以得到“/my/cool/explosionVolume/density”、“/my/cool/explosionVolume/heat”等，只要我们没有定义另一个以“/”开头的路径属性. 这为我们提供了一种使用（VDB）网格定义体积的便捷方法]
- **usdinstancerpath**: This defines the path of your "PointInstancer" prim (when exporting packed prims as point instancers). When we define a relative path, it is anchored under "/my/cool/debrisInstancer/Prototypes/ourRelativePathValue". Collecting our prototypes under the "/Prototypes" prim is a common USD practice, as it visually is clear where the data is coming as well as it make the hierarchy "transportable" as we don't point to random other prims, that might be unloaded.

  [ usdinstancerpath：这个属性定义了你的“PointInstancer” prim 的路径（当将打包的 prim 导出为 point instancers 时）, 当我们定义相对路径时，它会被锚定在“/my/cool/debrisInstancer/Prototypes/ourRelativePathValue”下. 将我们的原型收集在“/Prototypes” prim下是USD的一个常见做法，因为从视觉上可以清楚地看到数据的来源，同时也使层级结构“可移植”，因为我们不指向可能已卸载的随机其他 prim]

When these two attributes are active, our path attributes that are relative, are anchored differently:

[ 当这两个属性处于激活状态时，我们的相对路径处理时产生不同的行为]
- **usdvolumesavepath**: "/my/cool/explosionVolume/**relativePathValue**".
- **usdinstancerpath**: "/my/cool/debrisInstancer/Prototypes/**relativePathValue**"

Here's a video showing all variations, you can find the file, as mentioned above, in our GitHub repo:

[ 这个视频显示所有的变体，您可以在我们的 GitHub 存储库中找到该文件，如上所述]

<video width="100%" height="100%" controls autoplay muted loop>
  <source src="./houdiniPathAbsoluteVsRelative.mp4" type="video/mp4" alt="Houdini Node Path">
</video>

### Importing from LOPs to SOPs <a name="IOLopsToSops"></a>
Importing from LOPs is done via two nodes:

[ 从 LOP 导入是通过两个节点完成的]
- **LOP Import**: This imports packed USD prims.

  [ LOP Import：导入打包的 USD prims]
- **USD Unpack**: This unpacks the packed USD prims to polygons.

  [ USD Unpack：这会将打包的 USD prims 解包为多边形]

<video width="100%" height="100%" controls autoplay muted loop>
  <source src="./houdiniSOPsLOPImportTraversal.mp4" type="video/mp4" alt="Houdini LOP Import Traversal">
</video>

~~~admonish danger title="Selecting Parent & Child Prims"
If we select a parent and child prim at the same time, we will end up with duplicated geometry on import (and the import will take twice as long).

[ 如果我们同时选择父项和子项 prim，我们最终会在导入时得到重复的几何体（并且导入时间将增加两倍）]

By default the "USD Unpack" node traverses to "GPrims" (Geometry Prims), so it only imports anything that is a geo prim. That means on LOP import, we don't have to select our leaf mesh prims ourselves, when we want the whole model.

[ 默认情况下，“USD Unpack”节点会遍历到“GPrims”（Geometry Prims），因此它只导入任何属于 geo prims 的内容. 这意味着在 LOP 导入时，当我们需要整个模型时，我们不必自己选择叶子网格 prims]
~~~

As loading data takes time, we recommend being as picky as possible of what you want to import. We should not import our whole stage as a "USD Packed Prim" and then traverse/break it down, instead we should pick upfront what to load. This avoids accidentally creating doubled imports and keeps things fast.

[ 由于加载数据需要时间，因此我们建议对要导入的内容尽可能精简. 我们不应该将整个stage作为“USD Packed Prim”导入，然后遍历/拆分. 而是应该预先选择要加载的内容. 这可以避免意外创建重复的导入并保持快速]

~~~admonish tip title="Pro Tip | Import Xforms"
Our packed USD prims, carry the same transform information as standard packed geo. We can use this to parent something in SOPs by extracting the packed intrinsic transform to the usual point xform attributes.

[ 我们打包的 USD prims 带有与标准打包 geo 相同的转换信息. 通过将打包的内在变换提取到点上 xform 属性，我们可以使用它来为 SOP 中的某些内容创建父级]
~~~

If we want to import data from a LOPs PointInstancer prim, we can LOP import the packed PointInstancer prim and then unpack it polygons. This will give us a packed USD prim per PoinstInstancer prin (confusing right?). Be careful with displaying this in the viewport, we recommend extracting the xforms and then adding a add node that only keeps the points for better performance.

[ 如果我们想从 LOP PointInstancer prim 导入数据，我们可以 LOP 导入打包的 PointInstancer prim，然后将其解包为多边形. 这将为我们提供每个 PoinstInstancer prin 的打包 USD prim（令人困惑，对吗？）在视口中显示此内容时要小心，我们建议提取 xform，然后添加一个仅保留点的添加节点，以获得更好的性能]

~~~admonish danger title="USD Packed Prims"
Displaying a lot of packed prims in SOPS can lead to Houdini being unstable. We're guessing because it has to draw via Hydra and HoudiniGL and because USD packed prims draw a stage partially, which is a weird intermediate filter level.

[ 在 SOPS 中显示大量 packed prims 可能会导致 Houdini 不稳定. 我们猜测是因为它必须通过 Hydra 和 HoudiniGL 进行绘制，而且 USD packed prims 只绘制 stage 的一部分，这是一个奇怪的中间过滤器级别]

We recommend unpacking to polygons as "fast" as possible and/or pre-filtering the LOP import as best as possible.

[ 我们建议尽可能“快”地解压到多边形或尽可能地预先过滤 LOP 导入]
~~~

<video width="100%" height="100%" controls autoplay muted loop>
  <source src="./houdiniSOPsLOPImportPointInstancer.mp4" type="video/mp4" alt="Houdini LOP Import PointInstancer">
</video>

### Exporting from SOPs to LOPs <a name="IOSopsToLops"></a>
Export to LOPs is as simple as creating a SOP Import node and picking the SOP level geo.

[ 导出到 LOP 就像创建 SOP Import 节点并选择 SOP 级别物体一样简单]

As described above, we can drive the SOP import settings via the "USD Configure" SOPs level node or the "SOP Import" LOPs node.

[ 如上所述，我们可以通过"USD Configure”节点或“SOP Import”节点设置 SOP 导入设置]

We go into detail on how to handle the different (FX) geometry types in our [Geometry IO/FX section](../houdini/fx/overview.md).

[我们将在 [Geometry IO/FX section](../houdini/fx/overview.md) 部分详细介绍如何处理不同的 (FX) 几何类型]

We recommend building workflows for each geo type (points/deformed meshes/transforms/copy-to-points/RBD/crowds) as this makes it easy to pre-flight check as we know exactly what to export and we can optimize (the heck) out of it.

[ 我们建议为每种 geo 类型（points/deformed meshes/transforms/copy-to-points/RBD/crowds）构建工作流程，这样可以轻松进行预先检查，因为我们确切地知道要导出什么并且可以优化]

~~~admonish tip title="Pro Tip | Working Against Stages"
When working LOPs, we like to use the terminology: "We are working against a stage."

[ 在处理 LOP 时，我们喜欢使用这样的术语：“我们正在针对一个stage进行工作"]

What do we mean with that? When importing from or editing our stage, we are always making the edits relative to our current stage.
When importing to SOPs, we can go out of sync, if our SOP network intermediate-caches geometry. For example if it writes SOP imported geometry to a .bgeo.sc cache and the hierarchy changes in LOPs, you SOPs network will not get the correct hierarchy until it is re-cached.

[ 这是什么意思？当从 stage 导入或 editing stage 时，我们总是相对于当前 stage 进行编辑. 当导入到 SOP 时，如果我们的 SOP network 中间缓存几何图形，可能不会同步更新. 例如，如果它将 SOP 导入的几何图形写入 .bgeo.sc 缓存，并且 LOP 中的层次结构发生更改，则 SOP network 将无法获得正确的层级结构，直到重新缓存为止]

This can start being an issue, when you want to "over" the data from SOPs onto an existing hierarchy. Therefore we should always try to write our caches "against a stage". Instead of just caching our USD to disk and then composition arc-ing it into our "main" node stream.

[ 当您想要将 SOP 中的数据“覆盖”到现有层级结构时，这可能会成为一个问题. 因此，我们应该始终尝试“针对 stage ”写入缓存. 而不是仅仅将我们的 USD 缓存到磁盘，然后将其合成到我们的“主”节点流中]

This means that we can validate our hierarchy and read stage metrics like shutter sample count or local space transforms on USD export.
This ensures that the resulting cache is valid enough to work downstream in our pipeline.  Houdini's "SOP Import" node is also constructed to work against the input connected stage and does alot of this for you (e.g. material binding/local space correction ("Adjust Transforms for Input Hierarchy")).

[ 这意味着我们可以验证我们的层级结构并读取 stage 指标，例如 快门样本计数或 USD 导出的局部空间变换. 这确保了生成的缓存足够有效，可以在我们的流程下游工作. Houdini 的"SOP Import"节点也被构造为针对输入连接 stage 工作，并为您完成很多工作（例如材质绑定/局部空间校正（“调整输入层次结构的变换”））]
~~~

Another thing we have to keep in mind is that SOPs only represents the active frame, where as a written USD cache file represents the whole frame range.

[ 我们必须记住的另一件事是，SOP 仅代表激活帧，而写入的 USD 缓存文件代表整个帧范围]

To emulate this we can add a cache LOPs node. If we have geometry "popping" in and out across the frame range, it will by default always be visible in the whole frame range cached file. When we write USD files on different machines (for example on a farm), each frame that is being cooked does not know of the other frames.

[ 为了模拟这一点，我们可以添加一个 LOP cache 节点. 如果我们在帧范围内“popping”几何图形，则默认情况下它将始终在整个帧范围缓存文件中可见. 当我们在不同的机器上（例如在农场上）写入 USD 文件时，正在烘焙的每个帧都不知道其他帧]

To solve this USD has two solutions:

[ 为了解决这个问题，美元有两种解决方案]
- If our output file is a single stitched USD file, then we have to manually check the layers before stitching them and author visibility values if we want to hide the prims on frames where they produced no data.

  [ 如果我们的输出文件是单个缝合的 USD 文件，那么如果我们想要隐藏不产生数据的帧上的 prims，则必须在缝合图层之前手动检查图层并编写可见性值]
- With value clips we have to write an "inherited" visibility time sample per prim for every time sample. If the hierarchy does not exist, it will fallback to the manifest, where we have to write a value of "invisible".

  [ 对于值剪辑，我们必须为每个时间样本的每个 prim 编写一个“inherited”可见性时间样本. 如果层级结构不存在，它将会使用我们设置为“invisible”的基本配置]

Houdini has the option to track visibility for frame ranges (that are cooked in the same session) to prevent this. For large production scenes, we usually have to resort to the above to have better scalability.

[ Houdini 可以选择跟踪帧范围（在同一会话中生成）的可见性，以防止出现这种情况. 对于大型的制作场景，我们通常不得不借助以上的方式来获得更好的扩展性]

<video width="100%" height="100%" controls autoplay muted loop>
  <source src="./houdiniLOPsSOPImportTimeVisibility.mp4" type="video/mp4" alt="Houdini SOP Import Time Visibility">
</video>

### Stage/Layer Metrics <a name="IOLayerMetrics"></a>
As mentioned in our [stage/layer](../../core/elements/layer.md#layerMetrics) and [animation](../../core/elements/animation.md#animationMetadata) sections, we can/should setup layer related metrics.

[ 正如我们的 [stage/layer](../../core/elements/layer.md#layerMetrics) 和 [animation](../../core/elements/animation.md#animationMetadata) 部分中提到的，我们可以/应该设置图层相关的指标]

They work the same way in Houdini, the only difference is we have to use Houdini's [Configure Layer](https://www.sidefx.com/docs/houdini/nodes/lop/configurelayer.html) node to set them, if we want to set them on the root layer/stage (due to how Houdini manages stages).

[ 它们在 Houdini 中的工作方式相同，唯一的区别是，如果我们想在 root layer/stage 上设置它们（由于 Houdini 管理 stage 的方式），我们必须使用 Houdini 的 [Configure Layer](https://www.sidefx.com/docs/houdini/nodes/lop/configurelayer.html) 节点来设置它们]

![Houdini Stage/Layer metrics](houdiniLayerMetrics.jpg)

## Composition <a name="composition"></a>

### Asset Resolver <a name="compositionAssetResolver"></a>
In Houdini our asset resolver is loaded as usual via the plugin system. If we don't provide a context, the default context of the resolver is used.

[ 在 Houdini 中，我们的资源解析器照常通过插件系统加载. 如果我们不提供上下文，则使用解析器的默认上下文]

We can pass in our [asset resolver context](../../core/plugins/assetresolver.md#assetResolverContext) in two ways:

[ 我们可以通过两种方式传递[资产解析器上下文](../../core/plugins/assetresolver.md#assetResolverContext)]
- Via the the configure stage node. When set via the stage node, the left most node stream with a context "wins", when we start merging node trees.

  [ 通过 configure stage 节点. 当通过 stage 节点设置后，当我们开始合并节点树时，具有上下文的最左边的节点流“获胜”]
- Globally via the scene graph tree panel settings.

  [ 通过场景图树面板设置进行全局设置]

| Node Tree                                               | Global Context                                               |
|---------------------------------------------------------|--------------------------------------------------------------|
| ![LOPs SOP Import](houdiniAssetResolverContextNode.jpg) | ![SOPs Usd Configure](houdiniAssetResolverContextGlobal.jpg) |

For more info about resolvers, check out our [asset resolver](../../core/plugins/assetresolver.md) section.
We provide reference resolver implementations that are ready to be used in production.

[ 有关解析器的更多信息，请查看[资产解析器](../../core/plugins/assetresolver.md)部分. 我们提供可供在生产中使用的参考解析器实现]

### Creating Composition Arcs <a name="compositionAssetResolver"></a>
Creating composition arcs in LOPs can be done via Houdini's sublayer, variant and reference (for inherit/payload/specializes) nodes.

[ 在 LOP 中创建合成弧可以通过 Houdini 的 sublayer, variant and reference (for inherit/payload/specializes) 节点来完成]

We recommend using these for prototyping composition. Once we've figured out where we want to go, we should re-write it in Python, as it is easier and a lot faster when working on large hierarchies.

[ 我们建议使用它们来制作合成原型. 一旦我们弄清楚了我们想要的结构，我们应该用 Python 重新编写它，因为在处理大型层级结构时它更容易、更快]

This guide comes with an extensive [composition fundamentals](../../core/composition/overview.md)/[composition in production](../../production/composition.md) section with a lot of code samples.

[ 本指南提供了[composition fundamentals](../../core/composition/overview.md)/[composition in production](../../production/composition.md)大量代码示例]

We also cover advanced composition arc creation, specifically for Houdini, in our [Tips & Tricks](../houdini/faq/overview.md#compositionOverview) section.

[ 我们还在[提示与技巧](../houdini/faq/overview.md#compositionOverview)部分介绍了专门针对 Houdini 的高级合成弧创建]