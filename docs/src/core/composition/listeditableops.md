# List Editable Ops (Operations)
On this page we will have a look at list editable ops when not being used in composition arcs.

[ 在这一页上，我们将讨论在不使用组合弧的情况下进行可编辑列表操作]

~~~admonish danger
As mentioned in our [fundamentals](./fundamentals.md#compositionFundamentalsListEditableOps) section, list editable ops play a crucial role to understanding composition. Please read that section before this one, as we build on what was written there.

[ 正如我们的 [基础知识](./fundamentals.md#compositionFundamentalsListEditableOps) 部分中提到的，列表可编辑操作对于理解组合起着至关重要的作用. 请先阅读该部分，因为我们是在该部分所写内容的基础上构建的]
~~~

# Table of Contents [目录]
1. [List Editable Ops In-A-Nutshell](#summary)
1. [What should I use it for?](#usage)
1. [Resources](#resources)
1. [Overview](#overview)
1. [Composition Arcs](#listOpCompositionArc)
1. [Relationships](#listOpRelationship)
1. [Metadata](#listOpMetadata)

## TL;DR List Editable Ops - <Topic> In-A-Nutshell [概述]<a name="summary"></a>
- USD has the concept of list editable operations. Instead of having a "flat" array (`[Sdf.Path("/cube"), Sdf.Path("/sphere")]`) that stores data, we have wrapper array class that stores multiple sub-arrays (`prependedItems`, `appendedItems`, `deletedItems`, `explicitItems`). When flattening the list op, it merges the prepended and appended items and also removes items in deletedItems as well as duplicates, so that the end result is like an ordered Python `set()`. When in explicit mode, it only keeps the elements in explicitItems and ignores previous layers. This merging is done per layer, so that for example an `appendedItems` op in a higher layer, gets added to an `explicitItems` from a lower layer. This allows us to average the array data over multiple layers.

    [ USD 有列表可编辑操作的概念. 与传统的“平面”数组（如 [Sdf.Path("/cube"), Sdf.Path("/sphere")]）存储数据不同，而是存储多个子数组（如prependedItems、appendedItems、deletedItems、explicitItems）在将列表操作展平时，它会合并前置和后置添加的项目，同时删除已删除的项目以及重复项，最终的结果类似于一个有序的 Python 集合. 在显式模式下它仅保留 explicitItems 中的元素并忽略之前的层. 这种合并是按层进行的. 因此较高层中的 appendedItems 操作会被添加到较低层的 explicitItems 中. 这使我们能够对多个层上的数组数据进行平均]
- List editable ops behave differently based on the type:

    [ 列表可编辑操作的行为会根据其类型而有所不同]
    - **Composition**: When using list editable ops to define composition arcs, we can only edit them in the [active layer stack](./fundamentals.md#compositionFundamentalsLayerStack). Once referenced or payloaded, they become [encapsulated](./fundamentals.md#compositionFundamentalsEncapsulation).

        [ 合成（Composition）：当使用列表可编辑操作来定义组合弧时，我们只能在[激活层堆栈](./fundamentals.md#compositionFundamentalsLayerStack)中编辑它们. 一旦它们被 referenced 或者 payloaded 它们就会被封装.]
    - **Relationships/Metadata**: When making use of list editable ops when defining relationships and metadata, we do not have encapsulation. This means that any layer stack can add/delete/set explicit the list editable type. See the examples below for more info.

        [ 关联关系/元数据（Relationships/Metadata）：在定义关系和元数据时使用列表可编辑操作没有封装. 这意味着任何层堆栈都可以添加、删除或显式设置列表可编辑类型. 请参阅下面的示例以获取更多信息]

## What should I use it for? <a name="usage"></a>

[ 我应该用它做什么？]

~~~admonish tip
Using list editable ops in non composition arc scenarios is rare, as we often want a more attribute like value resolution behavior. It is good to know though that the mechanism is there. A good production use case is making metadata that tracks asset dependencies list editable, that way all layers can contribute to the sidecar data.

[ 在非组合弧场景中使用列表可编辑操作很少见，因为我们通常需要更多属性，例如 值解析行为. 不过很高兴知道该机制已经存在. 一个好的生产用例是让跟踪资产依赖项列表的元数据可编辑，这样所有层都可以为 sidecar 数据做出贡献]
~~~

## Resources [资源]<a name="resources"></a>
- [USD Glossary - List Editing](https://openusd.org/release/glossary.html#list-editing)

## Overview [概述]<a name="overview"></a>
Let's first go other how list editable ops are edited and applied:

[ 让我们首先了解如何编辑和应用列表可编辑操作]

These are the list editable ops that are available to us:

[ 这些是我们可用的可编辑操作列表]

- Composition:
    - `Sdf.PathListOp`
    - `Sdf.PayloadListOp`
    - `Sdf.ReferenceListOp`
- Base Data Types:
    - `Sdf.PathListOp`
    - `Sdf.StringListOp`
    - `Sdf.TokenListOp`
    - `Sdf.IntListOp`
    - `Sdf.Int64ListOp`
    - `Sdf.UIntListOp`
    - `Sdf.UInt64ListOp`

USD has the concept of list editable operations. Instead of having a "flat" array (`[Sdf.Path("/cube"), Sdf.Path("/sphere")]`) that stores data, we have wrapper array class that stores multiple sub-arrays (`prependedItems`, `appendedItems`, `deletedItems`, `explicitItems`). When flattening the list op, it merges the prepended and appended items and also removes items in deletedItems as well as duplicates, so that the end result is like an ordered Python `set()`. When in explicit mode, it only keeps the elements in explicitItems and ignores previous layers. This merging is done per layer, so that for example an `appendedItems` op in a higher layer, gets added to an `explicitItems` from a lower layer. This allows us to average the array data over multiple layers.

[ USD 有列表可编辑操作的概念. 与传统的“平面”数组（如 [Sdf.Path("/cube"), Sdf.Path("/sphere")]）存储数据不同，而是存储多个子数组（如prependedItems、appendedItems、deletedItems、explicitItems） 在将列表操作展平时它会合并前置和后置添加的项目，同时删除已删除的项目以及重复项，最终的结果类似于一个有序的 Python 集合. 在显式模式下它仅保留 explicitItems 中的元素并忽略之前的层. 这种合并是按层进行的，因此较高层中的 appendedItems 操作会被添加到较低层的 explicitItems 中. 这使我们能够对多个层上的数组数据进行平均]

All list editable ops work the same way, the only difference is what data they can hold.

[ 所有列表可编辑操作的工作方式都是相同的，唯一的区别是它们可以保存哪些数据]

These are 100% identical in terms of list ordering functionality, the only difference is what items they can store (as noted above). Let's start of simple with looking at the basics:

[ 它们在列表排序功能方面 100% 相同，唯一的区别是它们可以存储哪些项目（如上所述）.让我们从简单的基础知识开始]

~~~admonish tip title=""
```python
{{#include ../../../../code/core/composition.py:listEditableOpsLowLevelAPI}}
```
~~~

When working with the high level API, all the function signatures that work on list-editable ops usually take a position kwarg which corresponds to what list to edit and the position (front/back):

[ 使用高级 API 时，所有在可编辑列表操作上工作的函数签名通常采用一个位置参数，它对应于要编辑的列表和位置（前/后）]

- `Usd.ListPositionFrontOfAppendList`: Prepend to append list, the same as `Sdf.<Type>ListOp`.appendedItems.insert(0, item)
- `Usd.ListPositionBackOfAppendList`: Append to append list, the same as `Sdf.<Type>ListOp`.appendedItems.append(item)
- `Usd.ListPositionFrontOfPrependList`: Prepend to prepend list, the same as `Sdf.<Type>ListOp`.appendedItems.insert(0, item)
- `Usd.ListPositionBackOfPrependList`: Append to prepend list, the same as `Sdf.<Type>ListOp`.appendedItems.append(item)


~~~admonish tip title=""
```python
{{#include ../../../../code/core/composition.py:listEditableOpsHighLevelAPI}}
```
~~~

Now let's look at how multiple of these list editable ops are combined.

[ 现在让我们看看如何组合多个列表可编辑操作]

~~~admonish important title="Pro Tip | List Editable OPs in Metadata "
Again it is very important, that composition arc related list editable ops get combined with a different rule set. We cover this extensively in our [fundamentals](./fundamentals.md) section.

[ 再次强调，合成弧相关的列表可编辑操作与不同的规则集相结合是非常重要的. 我们在 [基础](./fundamentals.md) 部分对此进行了详细的介绍]

Non-composition related list editable ops do not make use of [encapsulation](./fundamentals.md#compositionFundamentalsEncapsulation). This means that any layer can contribute to the result, meaning any layer can add/remove/set explicit. When getting the value of the list op for non-composition arc list ops, we get the absolute result, in the form of an explicit list editable item list.

[ 与组合无关的列表可编辑操作不使用封装. 这意味着任何层都可以对结果做出贡献，即任何层都可以添加、删除或设置显式项. 对于非组合弧列表操作当我们获取列表操作的值时，我们得到的是绝对结果以显式列表可编辑项列表的形式呈现]

In contrast: When looking at composition list editable ops, we only get the value of the last layer that edited the value, and we have to use composition queries to get the actual result.

[ 相比之下：在查看组合列表可编辑操作时，我们只获取最后一个编辑该值的层的值，而且我们必须使用组合查询来获取实际结果]

This makes non-composition list editable ops a great mechanism to store averaged side car data. Checkout our Houdini example below, to see this in action.

[ 这使得非组合列表可编辑操作成为存储平均辅助数据的优秀机制. 请查看下面的 Houdini 示例以了解它的实际运作情况]
~~~

Let's mock how USD does this (without using `Sdf.Layer`s to keep it simple):

[ 让我们模拟一下 USD 如何做到这一点（为了简单起见，不使用 Sdf.Layer ）]

~~~admonish tip title=""
```python
{{#include ../../../../code/core/composition.py:listEditableOpsMerging}}
```
~~~

When working with multiple layers, each layer can have list editable ops data in (composition-) metadata fields and relationship specs. It then gets merged, as mocked above. The result is a single flattened list, without duplicates.

[ 使用多个层时，每个层都可以在（组合）元数据字段和关系规范中列出可编辑的操作数据然后它会被合并. 如上面所模拟的结果是一个单一的扁平列表没有重复项]

## Composition Arcs <a name="listOpCompositionArc"></a>
For a detailed explanation how list editable ops work in conjunction with composition arcs, please check out our [composition fundamentals](./fundamentals.md) section.

[ 有关列表可编辑操作如何与合成弧结合使用的详细说明，请查看[composition fundamentals](./fundamentals.md)]

## Relationships <a name="listOpRelationship"></a>
As with list editable metadata, relationships also show us the combined results from multiple layers. Since it is not a composition arc list editable op, we also don't have the restriction of encapsulation. That means, calling `GetTargets`/`GetForwardedTargets` can combine appended items from multiple layers. Most DCCs go for the `.SetTargets` method though, as layering relationship data can be confusing and often not what we want as an artist using the tools.

[ 与列表可编辑元数据一样，关联关系也向我们显示多个层的组合结果. 由于它不是一个组合弧列表可编辑操作因此我们也没有封装的限制. 这意味着调用 GetTargets / GetForwardedTargets 可以组合来自多个层的后置添加项. 不过大多数 DCC 工具更倾向于使用.SetTargets方法因为分层关系数据可能会令人困惑，而且通常不是我们作为艺术家在使用工具时所期望的]

~~~admonish important title="Pro Tip | List Editable OPs in Collections "
Since [collections](../elements/collection.md) are made up of relationships, we can technically list edit them too. Most DCCs set the collection relationships as an explicit item list though, as layering paths can be confusing.

[ 由于集合是由关系组成的，因此从技术上讲我们也可以对它们进行列表编辑. 不过大多数 DCC 将集合关系设置为显式项目列表因为分层路径可能会令人困惑]
~~~

## Metadata <a name="listOpMetadata"></a>
The default metadata fields that ship with USD, are not of the list editable type. We can easily extend this via a metadata plugin though.

[ USD 附带的默认元数据字段不是列表可编辑类型. 不过，我们可以通过元数据插件轻松扩展它]

Here is a showcase from our composition example file that edits a custom string list op field which we registered via a [custom meta plugin](../plugins/metadata.md).

[ 这是我们的组合示例文件的展示，它编辑通过[自定义插件](../plugins/metadata.md)注册的自定义字符串列表操作字段]

<video width="100%" height="100%" controls autoplay muted loop>
  <source src="./houdiniListEditableOpsMetadata.mp4" type="video/mp4" alt="Houdini Custom ListEditableOps Metadata">
</video>

As you can see the result is dynamic, even across encapsulated arcs and it always returns an explicit list op with the combined results.

[ 正如您所看到的，结果是动态的，即使在封装后的合成弧中也是如此，并且它始终返回一个包含组合结果的显式列表操作]

This can be used to combine metadata non destructively from multiple layers.

[ 这可以用于从多个层非破坏性地组合元数据]