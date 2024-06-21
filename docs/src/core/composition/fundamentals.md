# Composition Fundamentals

In this section will talk about fundamental concepts that we need to know before we look at individual composition arcs.

[ 在本节中，我们将讨论在查看单个合成弧之前我们需要了解的基本概念]

~~~admonish question title="Still under construction!"
As composition is USD's most complicated topic, this section will be enhanced with more examples in the future.
If you detect an error or have useful production examples, please [submit a ticket](https://github.com/LucaScheller/VFX-UsdSurvivalGuide/issues/new), so we can improve the guide!

[ 由于合成是 USD 最复杂的主题，因此本节将来将通过更多示例来增强. 如果您发现错误或有有用的生产示例，[请提交票证](https://github.com/LucaScheller/VFX-UsdSurvivalGuide/issues/new) 以便我们改进指南！]
~~~

# Table of Contents [目录]
1. [Composition Fundamentals In-A-Nutshell](#summary)
1. [Why should I understand the editing fundamentals?](#usage)
1. [Resources](#resources)
1. [Overview](#overview)
1. [Terminology](#terminology)
1. [Composition Editing Principles - What do we need to know before we start?](#compositionFundamentals)
    1. [List-Editable Operations](#compositionFundamentalsListEditableOps)
    1. [Encapsulation](#compositionFundamentalsEncapsulation)
    1. [Layer Stack](#compositionFundamentalsLayerStack)
    1. [Edit Target](#compositionFundamentalsEditTarget)

## TL;DR - Composition Fundamentals In-A-Nutshell [概述]<a name="summary"></a>
- Composition editing works in the active [layer stack](#compositionFundamentalsLayerStack) via [list editable ops](#compositionFundamentalsListEditableOps).

  [ 合成编辑的操作通过 [list editable ops](#compositionFundamentalsListEditableOps) 在激活图层堆栈中进行]
- When loading a layer (stack) from disk via `Reference` and `Payload` arcs, the contained composition structure is immutable (USD speak [encapsulated](#compositionFundamentalsEncapsulation)). This means you can't remove the arcs within the loaded files. As for what the arcs can use for value resolution: The `Inherit` and `Specialize` arcs still target the "live" composed stage and therefore still reflect changes on top of the encapsulated arcs, the `Reference` arc is limited to seeing the encapsulated layer stack.

  [ 当通过 Reference 和 Payload 弧从磁盘加载图层（堆栈）时，所包含的组合结构是不可变的（USD中称为封装） 这意味着您不能删除已加载文件中的弧 至于什么弧可用于值解析的内容：Inherit 和 Specialize 弧仍然针对“实时”组合 stage，因此仍然反映封装弧之上的变化，而 Reference 弧仅限于查看封装的图层堆栈]

## Why should I understand the editing fundamentals? <a name="usage"></a>

[ 我为什么要了解编辑基础？]

~~~admonish tip
This section houses terminology essentials and a detailed explanation of how the underlying mechanism of editing/composing arcs works.
Some may consider it a deep dive topic, we'd recommend starting out with it first though, as it saves time later on when you don't understand why something might not work.

[ 本节包含术语要点以及编辑/组合弧的基本机制如何工作的详细说明. 有些人可能认为这是一个深入的主题，但我们建议首先从它开始，因为当您以后不明白为什么某些东西可能不起作用时，它可以节省时间]
~~~

## Resources [资源]<a name="resources"></a>
- [USD Glossary]():
    - [Layer Stack](https://openusd.org/release/glossary.html#usdglossary-layerstack)
    - [Root Layer Stack](https://openusd.org/release/glossary.html#usdglossary-rootlayerstack)
    - [Prim Index](https://openusd.org/release/glossary.html#usdglossary-index)
    - [List Editing](https://openusd.org/release/glossary.html#list-editing)
    - [LIVRPS](https://openusd.org/release/glossary.html#livrps-strength-ordering)
    - [Path Translation](https://openusd.org/release/glossary.html#usdglossary-pathtranslation)
    - [References](https://openusd.org/release/glossary.html#usdglossary-references)
- [USD FAQ - When can you delete a reference?](https://openusd.org/release/usdfaq.html#when-can-you-delete-a-reference-or-other-deletable-thing)

## Overview [概述]<a name="overview"></a>
Before we start looking at the actual composition arcs and their strength ordering rules, let's first look at how composition editing works.

[ 在我们开始查看实际的合成弧及其强度排序规则之前，让我们首先看看组合编辑的工作原理]

## Terminology [术语]<a name="terminology"></a>
USD's mechanism of linking different USD files with each other is called `composition`. Let's first clarify some terminology before we start, so that we are all on the same page:

[ USD 将不同 USD 文件相互链接的机制称为 composition 在开始之前，让我们先说明一些术语，以便我们达成共识]

- **Opinion**: A written value in a layer for a metadata field or property.

  [ Opinion：写在层中的元数据字段或属性值]
- **Layer**: A layer is an USD file on disk with [prims](../elements/prim.md) & [properties](../elements/property.md). (Technically it can also be in memory, but for simplicity on this page, let's think of it as a file on disk). More info in our [layer section](../elements/layer.md).

  [ Layer：层是磁盘上的一个 USD 文件，包含 [prims](../elements/prim.md) 和 [properties](../elements/property.md)（从技术上讲，它也可以位于内存中，但为了简化本页的内容，我们将其视为磁盘上的文件）更多信息请参阅[layer section](../elements/layer.md)]
- **Layer Stack**: A stack of layers (Hehe 😉). We'll explain it more in detail below, just remember it is talking about all the loaded layers that use the `sublayer` composition arc.

  [ Layer Stack：图层的堆栈（嘿嘿😉）下面我们会更详细地解释它，只需要记住，它讨论的是使用子图层（sublayer）组合弧加载的所有层]
- **Composition Arc**: A method of linking (pointing to) another layer or another part of the scene hierarchy. USD has different kinds of composition arcs, each with a specific behavior.

  [ Composition Arc：链接（指向）另一个图层或场景层级结构中的另一部分的方法 USD 具有不同类型的合成弧，每种弧都有特定的行为]
- **Prim Index**: Once USD has processed all of our composition arcs, it builds a `prim index` that tracks where values can come from. We can think of the `prim index` as something that outputs an ordered list of `[(<layer (stack)>, <hierarchy path>), (<layer (stack)>, <hierarchy path>)]` ordered by the composition rules.

  [ Prim Index：一旦 USD 处理完所有的合成弧，它会构建一个 prim index 索引，用于追踪值的来源. 我们可以将 prim index 索引看作输出按组合规则排序的 `[(<layer (stack)>, <hierarchy path>), (<layer (stack)>, <hierarchy path>)]` 有序列表]
- **Composed Value**: When looking up a value of a property, USD then checks each location of the `prim index` for a value and moves on to the next one if it can't find one. If no value was found, it uses a schema fallback (if the property came from a schema), other wise it falls back to not having a value (USD speak: not being `authored`).

  [ Composed Value：在查找属性值时，USD 会检查每个 `prim index` 索引，如果找不到值则继续检查下一个. 如果找不到任何值，它将使用 schema  默认值（属性如果来自 schema ），否则将返回没有值（USD术语：未设置）]

Composition is "easy" to explain in theory, but hard to master in production. It also a topic that keeps on giving and makes you question if you really understand USD. So don't worry if you don't fully understand the concepts of this page, they can take a long time to master. To be honest, it's one of those topics that you have to read yourself back into every time you plan on making larger changes to your pipeline.

[ 合成在理论上很容易解释，但在实际制作中却很难掌握. 这也是一个持续不断的话题，让你怀疑你是否真的了解 USD. 因此，如果您没有完全理解本页的概念请不要担心，它们可能需要很长时间才能掌握. 老实说这是每次您计划对流程进行较大更改时都必须重新阅读的主题之一]

We recommend really playing through as much scenarios as possible before you start using USD in production. Houdini is one of the best tools on the market that let's you easily concept and play around with composition. Therefore we will use it in our examples below.

[ 我们建议开始在生产环境中使用 USD 之前，尽可能多地尝试不同的场景. Houdini 是市场上最出色的工具之一，可以让您轻松地构思和尝试组合.因此，我们将在下面的示例中使用它]

## Composition Editing Fundamentals - What do we need to know before we start? <a name="compositionFundamentals"></a>

[ 合成编辑基础 - 在开始之前我们需要了解什么？]

Now before we talk about individual `composition arcs`, let's first focus on these different base principles composition runs on.
These principles build on each other, so make sure you work through them in order they are listed below.

[ 现在，在我们讨论单个合成弧之前，让我们先关注合成运行所依赖的这些不同的基本规则. 这些规则相互影响，所以请确保你按照以下列出的顺序依次学习]

- [List-Editable Operations](#compositionFundamentalsListEditableOps)
- [Encapsulation](#compositionFundamentalsEncapsulation)
- [Layer Stack](#compositionFundamentalsLayerStack)
- [Edit Target](#compositionFundamentalsEditTarget)

### List-Editable Operations (Ops) <a name="compositionFundamentalsListEditableOps"></a>
USD has the concept of list editable operations. Instead of having a "flat" array (`[Sdf.Path("/cube"), Sdf.Path("/sphere")]`) that stores what files/hierarchy paths we want to point to, we have wrapper array class that stores multiple sub-arrays. When flattening the list op, USD removes duplicates, so that the end result is like an ordered Python `set()`.

[ USD 有列表编辑操作的概念 我们不再使用“平面”数组（[Sdf.Path("/cube"), Sdf.Path("/sphere")]）来存储我们想要指向的文件/层级结构路径，而是使用包装数组类来存储多个子数组. 在展开列表操作时 USD 会删除重复项，因此最终结果类似于有序的 Python 集合 set()]

To make it even more confusing, composition arc list editable ops run on a different logic than "normal" list editable ops when looking at the final `composed value`.

[ 更为复杂的是，在查看最终 composed value 时，合成弧列表的可编辑操作与“正常”列表的可编辑操作遵循不同的逻辑]

We take a closer look at "normal" list editable ops in our [List Editable Ops section](./listeditableops.md), on this page we'll stay focused on the composition ones.

[ 我们在  [List Editable Ops section](./listeditableops.md) 部分中仔细研究“正常”列表可编辑操作，在此页面上，我们将继续关注组合操作]

Alright, let's have a quick primer on how these work. There are three sub-classes for composition related list editable ops:

[ 好了，让我们快速了解一下它们是如何工作的. 与合成相关的列表编辑操作分为三个子类]

- `Sdf.ReferenceListOp`: The list op for the `reference` composition arc, stores `Sdf.Reference` objects.

  [ Sdf.ReferenceListOp：用于 reference 合成弧的列表操作，存储 Sdf.Reference 对象]
- `Sdf.PayloadListOp`: The list op for the `payload` composition arc, stores `Sdf.Reference` objects.

  [ Sdf.PayloadListOp：用于 payload 合成弧的列表操作，存储 Sdf.Reference 对象]
- `Sdf.PathListOp`: The list op for `inherit` and `specialize` composition arcs, as these arcs target another part of the hierarchy (hence `path`) and not a layer. It stores `Sdf.Path` objects.

  [ Sdf.PathListOp：用于 inherit 和 specialize 合成弧的列表操作，因为这些弧指向层级结构中的另一部分（路径）而不是图层. 它存储 Sdf.Path 对象]

These are 100% identical in terms of list ordering functionality, the only difference is what items they can store (as noted above). Let's start of simple with looking at the basics:

[ 它们在列表排序功能方面 100% 相同，唯一的区别是它们可以存储哪些项目（如上所述）. 让我们从简单的基础知识开始]

~~~admonish tip title=""
```python
{{#include ../../../../code/core/composition.py:compositionListEditableOpsBasics}}
```
~~~

So far so good? Now let's look at how multiple of these list editable ops are combined. If you remember our [layer](../elements/layer.md) section, each layer stores our prim specs and property specs. The composition list editable ops are stored as metadata on the prim specs. When USD composes the stage, it combines these and then starts building the composition based on the composed result of these metadata fields.

[ 到目前为止还不错吧？现在让我们来看看如何组合多个列表编辑操作. 如果你还记得我们的 [layer](../elements/layer.md) 部分，每一层都存储着我们的 prim specs 和 property specs. 合成列表编辑操作是作为元数据存储在 prim specs 上. 当 USD 合并 stage 时它会将这些组合起来，然后根据这些元数据字段的组合结果开始构建合成]

Let's mock how USD does this without layers:

[ 让我们模拟一下 USD 如何在没有层的情况下做到这一点]

~~~admonish tip title=""
```python
{{#include ../../../../code/core/composition.py:compositionListEditableOpsMerging}}
```
~~~

When working with multiple layers, each layer can have list editable ops data in the composition metadata fields. It then gets merged, as mocked above. The result is a single flattened list, without duplicates, that then gets fed to the composition engine.

[ 使用多个层时，每个层都可以在合成元数据字段中列出可编辑操作数据然后合并，如上面所模拟的. 结果是一个没有重复元素的扁平列表，然后将其输入到合成引擎]

Here comes the fun part:

[ 有趣的部分来了]

~~~admonish danger title="List-Editable Ops | Getting the composed (combined) value"
When looking at the metadata of a prim via UIs (USD View/Houdini) or getting it via the Usd.Prim.GetMetadata() method, you will only see the list editable op of the last layer that edited the metadata, **NOT** the composed result.

[ 通过用户界面（USD View/Houdini）查看 prim 的元数据，或通过 Usd.Prim.GetMetadata() 方法获取元数据时，您只能看到最后一个编辑了元数据的图层的可编辑操作列表，而不是组合后的结果]

This is probably the most confusing part of USD in my opinion when first starting out. To inspect the full composition result, we actually have to consult the [PCP cache](pcp.md) or run a `Usd.PrimCompositionQuery`. There is another caveat though too, as you'll see in the next section: Composition is **encapsulated**. This means our edits to list editable ops only work in the active `layer stack`. More info below!

[ 在我看来这可能是 USD 初学者最容易感到困惑的部分. 要检查完整的组合结果我们实际上必须查看 [PCP cache](pcp.md) 或运行 Usd.PrimCompositionQuery 然而还有一个需要注意的点是：组合是封装的. 这意味着我们对列表可编辑操作的编辑仅在活动图层堆栈中有效.更多信息如下！]
~~~

In Houdini the list editable ops are exposed on the `reference` node. The "Reference Operation" parm sets what sub-array (prepend,append,delete) to use, the "Pre-Operation" sets it to `.Clear()` in `Clear Reference Edits in active layer` mode and to `.ClearAndMakeExplicit()` in "Clear All References" mode.

[ 在Houdini中列表编辑操作在 reference 节点上公开 “Reference Operation” 参数设置子数组要使用的方式（prepend，append，delete） 而 “Pre-Operation” 在 “Clear Reference Edits in active layer” 模式下将其设置为.Clear()，在 “Clear All References” 模式下将其设置为 .ClearAndMakeExplicit()]

<video width="100%" height="100%" controls autoplay muted loop>
  <source src="./houdiniReferenceComposition.mp4" type="video/mp4" alt="Houdini - Reference Node - List Editable Ops">
</video>

Here is how Houdini (but also the USD view) displays the references metadata field with different layers, as this is how the stage sees it.

[ 以下是 Houdini（或 USD view）如何显示不同层引用的元数据字段，因为这就是 stage 所看到的方式]

<video width="100%" height="100%" controls autoplay muted loop>
  <source src="./houdiniReferenceListEditableOpsLayers.mp4" type="video/mp4" alt="Houdini - List Editable Ops with layers">
</video>

You can see, as soon as we have our reference list editable op on different layers, the metadata only show the top most layer. To inspect all the references that are being loaded, we therefore need to look at the layer stack (the "Scene Graph Layers" panel) or perform a [compsition query](../../production/caches/composition.md).

[ 您可以看到，一旦我们在不同层上有了 reference 列表编辑操作，元数据就仅显示最顶层. 因此要检查正在加载的所有引用，我们需要查看图层堆栈（“Scene Graph Layers”面板）或执行 [合成查询](../../production/caches/composition.md)]

Also a hint on terminology: In the USD docs/glossary the `Reference` arc often refers to all composition arcs other than `sublayer`, I guess this is a relic, as this was probably the first arc. That's why Houdini uses a similar terminology.

[ 还有一个关于术语的提示：在 USD 的文档/术语表中，Reference 合成弧通常指的是除了 sublayer 之外的所有合成弧，我猜这大概是一个术语遗留问题，这可能是第一个合成弧. 这就是 houdini 使用类似术语的原因]

### Encapsulation  [封装]<a name="compositionFundamentalsEncapsulation"></a>
When you start digging through the API docs, you'll read the word "encapsulation" a few times. Here is what it means and why it is crucial to understand.

[ 当你开始深入API文档时，你会多次看到“封装”这个词. 下面就是它的含义以及为什么它至关重要]

~~~admonish danger title="Encapsulation | Why are layers loaded via references/payloads composition arc locked?"
To make USD composition fast and more understandable, the content of what is loaded from an external file via the **`Reference`** and **`Payload`** composition arcs, is **composition locked** or as USD calls it **encapsulated**. This means that you can't remove any of the composition arcs in the layer stack, that is being loaded, via the list editable ops `deletedItems` list or via the `explicitItems`.

[ 为了使 USD 的合成更快且更易于理解，通过 “Reference” 和 “Payload” 合成弧从外部文件加载的内容是锁定的(USD称之为封装) 这意味着您不能通过列表可编辑操作 “deletedItems” 列表或通过 “explicitItems” 来删除正在加载的图层堆栈中的任何合成弧]
~~~

The only way to get rid of a `payload`/`reference` is by putting it behind a `variant` in the first place and then changing the `variant` selection. This can have some unwanted side effects though. You can find a detailed explanation with an example here: [USD FAQ - When can you delete a reference?](https://openusd.org/release/usdfaq.html#when-can-you-delete-a-reference-or-other-deletable-thing)

[ 唯一删除 payload/reference 的方法是将其放在 variant 里面，然后更改 variant 选择. 然而这可能会产生一些不想要的副作用. 你可以在这里找到一个带有示例的详细解释: [USD FAQ - When can you delete a reference?](https://openusd.org/release/usdfaq.html#when-can-you-delete-a-reference-or-other-deletable-thing)]

~~~admonish important title="Encapsulation | Are my loaded layers then self contained?"
You might be wondering now, if encapsulation forces the content of `Reference`/`Payload` to be self contained, in the sense that the composition arcs within that file do not "look" outside the file. The answer is: It depends on the composition arc.

[ 您可能现在会好奇，如果封装使 Reference/Payload 的内容必须自成一体，那么在这个文件内部的合成弧是否“查看”文件外部的内容. 答案是：这取决于合成弧]

For `Inherits` and `Specializes` the arcs still evaluate relative to the composed scene. E.g. that means if you have an inherit somewhere in a referenced in layer stack, that `inherit` will still be live. So if you edit a property in the active stage, that gets inherited somewhere in the file, it will still propagate all the changes from the inherit source to all the inherit targets. The only thing that is "locked" is the composition arcs structure, not the way the composition arc evaluates. This extra "live" lookup has a performance penalty, so be careful with using `Inherits` and `Specializes` when nesting layers stacks via `References` and `Payloads`.

[ 对于 Inherits 和 Specializes 弧，它们仍然相对于组合的场景进行评估. 例如 如果您在引用的图层堆栈中的某个位置有一个继承，那么这个 inherit 仍然是激活的. 因此如果您在 active stage 编辑了一个属性，该属性在文件的某个位置被继承，那么它仍然会将来自继承源的所有更改传播到所有继承目标. 唯一“锁定”的是合成弧的结构，而不是合成弧评估的方式. 这种额外的“实时”查找会影响性能，因此在通过 References 和 Payloads 嵌套图层堆栈时使用 Inherits 和 Specializes 时要小心]

For `Internal References` this does not work though. They can only see the encapsulated layer stack and not the "live" composed stage. This makes composition faster for internal references.

[ 对于Internal References，这并不适用. 它们只能看到封装的图层堆栈，而不能看到“实时”组合的 stage 这使得内部引用合成更快]

We show some interactive examples in Houdini in our [LIVRPS](../composition/livrps.md#compositionArcReferencePayloadEncapsulation) section, as this is hard to describe in words.

[ 我们在我们的  [LIVRPS](../composition/livrps.md#compositionArcReferencePayloadEncapsulation) 部分中展示了 Houdini 中的一些交互式示例，因为这很难用语言来描述]
~~~

### Layer Stack <a name="compositionFundamentalsLayerStack"></a>
What is the layer stack, that we keep mentioning, you might ask yourself?
To quote from the [USD Glossary](https://openusd.org/release/glossary.html#usdglossary-layerstack)

[ 您可能会问自己，我们不断提到的层堆栈是什么？引自 [USD Glossary](https://openusd.org/release/glossary.html#usdglossary-layerstack)]

~~~admonish quote title=""
The ordered set of layers resulting from the recursive gathering of all SubLayers of a Layer, plus the layer itself as first and strongest.

[ 由层的所有子层递归聚集而产生的有序层集，加上作为第一个和最强层的层本身]
~~~

So to summarize, all (sub)-layers in the stage that were not loaded by `Reference` and `Payload` arcs.

[ 所以总的来说，stage上所有不是通过 Reference 和 Payload 合成弧加载的（子）层]

Now you might be thinking, isn't that the same thing as when we open a Usd file via `Usd.Stage.Open`? Well kind of, yes. When opening a stage, the USD file you open and its sublayers are the layer stack. USD actually calls this the `Root Layer Stack` (it also includes the sessions layers). So one could say, **editing a stage is process of editing a layer stack**. To extend that analogy, we could call a stage, that was written to disk and is being loaded via `Reference` and `Payload` arcs, an encapsulated layer stack.

[ 现在你可能在想，这不是和我们通过 Usd.Stage.Open 打开 Usd 文件时的情况一样吗？嗯，某种程度上是的 在打开一个 stage 时，你打开的 USD 文件及其子层就是层叠栈. 实际上称这个为 “Root Layer Stack”（它还包括会话层）因此可以说编辑 stage 就是编层堆栈的过程. 为了扩展这个类比我们可以把通过 Reference 和 Payload 合成弧写入磁盘并加载的 stage 称为一个封装的层叠栈]

These are the important things to understand (as also mentioned in the glossary):

[ 这些是需要理解的重要事项（术语表中也提到了）]

~~~admonish important title="Layer Stack | How does it affect composition?"
- Composition arcs target the layer stack, not individual layers. They recursively target the composed result (aka the result of all layers combined via the composition arc rules) of each layer they load in.

  [ 合成弧针对的是图层堆栈，而不是单个图层. 它们递归地针对加载的每个图层的组合结果（即通过合成弧规则组合的所有图层的结果）]
- We can only list edit composition arcs via list editable ops in the active layer stack. The active layer stack is usually the active stage (unless when we "hack" around it via edit targets, which you 99% of the time don't do).

  [ 我们只能通过活动图层堆栈中的列表可编辑操作来列出编辑合成弧. 活动图层堆栈通常是 acitve stage（除非我们通过编辑目标进行“修改”，但您99%的时间都不会这么做）]
~~~

So to make it clear again (as this is very important when we setup our asset/shot composition structure): **We can only update `Reference` and `Payload` arcs in the active layer stack. Once the active layer stack has been loaded via `Reference` and `Payload` arcs into another layer stack, it is encapsulated and we can't change the composition structure.**

[ 所以再次明确一下（因为这对我们设置资产/镜头合成结构非常重要）：我们只能更新激活层堆栈中的 Reference 和 Payload 合成弧. 一旦激活的层堆栈通过 Reference 和 Payload 弧加载到另一个层堆栈中，它就会被封装起来，我们就无法更改其合成结构了]

This means to keep our pipeline flexible, we usually have "only" three kind of layer stacks:

[ 这意味着为了保持我们流程的灵活性，我们通常“只”有三种类型的图层堆栈]
- **Asset Layer Stack**: When building assets, we build a packaged asset element. The end result is a (nested) layer stack that loads in different aspects of the asset (model/materials/fx/etc.). Here the main "asset.usd" file, that at the end we reference into our shots, is in control of "final" asset layer stack. We usually don't have any encapsulation issue scenarios, as the different assets layers are usually self contained or our asset composition structure is usually developed to sidestep encapsulation problems via variants.

  [ Asset Layer Stack: 在构建资源时，我们会构建一个打包的资产元素. 最终结果是一个（嵌套的）图层堆栈，它加载了资源的不同方面（模型/材质/特效/等）. 这是主要 “asset.usd” 文件，最终我们会在镜头中引用它，控制着“最终”的资产图层堆栈. 我们通常不会遇到任何封装问题，因为不同的资产层通常是自包含的或者我们的资产组合结构通常是通过变体来规避封装问题的]
- **Shot Layer Stack**: The shot layer stack is the one that sublayers in all of your different shot layers that come from different departments. That's right, since we sublayer everything, we still have access to list editable ops on everything that is loaded in via composition arcs that are generated in individual shot layers. This keeps the shot pipeline flexible, as we don't run into the encapsulation problem.

  [ Shot Layer Stack：镜头图层堆栈是包含来自不同部门的不同镜头图层进行子图层的堆栈. 没错，因为我们将所有内容都作为子图层处理，所以我们仍然可以通过各个镜头图层中生成的合成弧访问所有内容的可编辑操作列表. 这保持了镜头流程的灵活性，因为我们不会遇到封装问题]
- **Set/Environment/Assembly Layer Stack** (Optional): We can also reference in multiple assets to an assembly type of asset, that then gets referenced into our shots. This is where you might run into encapsulation problems. For example, if we want to remove a reference from the set from our shot stage, we can't as it is baked into the composition structure. The usual way around it is to: 1. Write the assembly with variants, so we can variant away unwanted assets. 2. Deactivate the asset reference 3. Write the asset reference with variants and then switch to a variant that is empty.

  [ Set/Environment/Assembly Layer Stack (Optional)：我们还可以将多个资源引用到一个 assembly  类型的资产中，然后将其引用到我们的镜头中. 这就是你可能会遇到封装问题的地方. 例如 如果我们想从 shot stage 的场景中移除一个引用，我们无法这样做，因为它已经嵌入到合成结构中了. 通常的解决方法是：1. 编写带有变体的 assembly，这样我们就可以排除掉不需要的资产. 2. Deactivate 资产引用. 3. 编写带有变体的资产引用，然后切换到一个空变体]


### Edit Target <a name="compositionFundamentalsEditTarget"></a>
To sum up edit targets in once sentence:

[ 用一句话总结编辑目标]

~~~admonish tip title="Pro Tip | Edit Targets"
A edit target defines, what layer all calls in the high level API should write to.

[ edit target 定义了高级 API 中的所有调用应写入哪一层]
~~~

Let's take a look what that means:

[ 让我们看看这意味着什么]

An edit target's job is to map from one namespace to another, we mainly use them for writing to layers in the active layer stack (though we could target any layer) and to write variants, as these are written "inline" and therefore need an extra name space injection.

[ 编辑目标的工作是将一个命名空间映射到另一个命名空间，我们主要将它们用于写入激活层堆栈中的图层（尽管我们也可以针对任何图层）以及写入变体，因为这些变体是“inline”写入的，因此需要一个额外的命名空间注入]

Setting the edit target is done on the stage, as this is our "controller" of layers in the high level API:

[ 设置编辑目标是在 stage 上完成的，因为这是高级 API 中针对层的“控制器”：]

~~~admonish tip title=""
```python
stage.SetEditTarget(layer)
# We can also explicitly create the edit target:
# Or
stage.SetEditTarget(Usd.EditTarget(layer))
# Or
stage.SetEditTarget(stage.GetEditTargetForLocalLayer(layer))
# These all have the same effect.
```
~~~

~~~admonish tip title=""
```python
{{#include ../../../../code/core/composition.py:compositionEditTarget}}
```
~~~

For convenience, USD also offers a context manager for variants, so that we don't have to revert to the previous edit target once we are done:

[ 为了方便起见，USD 还为变体提供了一个上下文管理器，这样我们就不需要在完成后返回到之前的编辑目标了]

~~~admonish tip title=""
```python
{{#include ../../../../code/core/composition.py:compositionEditTargetContext}}
```
~~~