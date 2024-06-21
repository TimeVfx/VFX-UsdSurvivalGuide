# Composition Arcs
In this section we'll cover how to create composition arcs via code. To check our how composition arcs interact with each other, check out our [Composition Strength Ordering (LIVRPS)](./livrps.md) section.

[ 在本节中，我们将介绍如何通过代码创建合成弧. 要检查合成弧如何相互作用，请查看[Composition Strength Ordering (LIVRPS)](./livrps.md)]

Please read out [fundamentals section](./fundamentals.md) as we often refer to it on this page.

[ 请阅读我们经常在此页面上引用的[基础](./fundamentals.md)部分]


# Table of Contents [目录]
1. [Composition Arcs In-A-Nutshell](#summary)
1. [What should I use it for?](#usage)
1. [Resources](#resources)
1. [Overview](#overview)
1. [Composition Arcs](#compositionArcs)
    1. [Sublayers / Local Opinions](#compositionArcSublayer)
        1. [Value Clips](#compositionArcValueClips)
    1. [Inherits](#compositionArcInherit)
    1. [Variants](#compositionArcVariant)
    1. [References](#compositionArcReference)
        1. [References File](#compositionArcReferenceExternal)
        1. [References Internal](#compositionArcReferenceInternal)
    1. [Payloads](#compositionArcPayload)
    1. [Specializes](#compositionArcSpecialize)

## TL;DR - Composition Arcs In-A-Nutshell [概述]<a name="summary"></a>
- Creating composition arcs is straight forward, the high level API wraps the low level list editable ops with a thin wrappers. We can access the high level API via the `Usd.Prim.Get<ArcName>` syntax.

    [ 创建合成弧非常简单，高级 API 用包装器封装了低级列表编辑操作.我们可以通过 Usd.Prim.Get\<ArcName\> 语法访问高级 API]
- Editing via the low level API is often just as easy, if not easier, especially when [creating (nested) variants](#compositionArcVariantCopySpec). The list editable ops can be set via `Sdf.PrimSpec.SetInfo(<arcType>, <listEditableOp>)` or via the properties e.g. `Sdf.PrimSpec.<arcType>List`.

    [ 使用低级 API 进行编辑通常更容易，尤其是在创建（嵌套）变体时. 列表编辑操作可以通过 Sdf.PrimSpec.SetInfo(\<arcType\>, \<listEditableOp\>) 或通过属性（例如 Sdf.PrimSpec.\<arcType\>List）来设置]

## What should I use it for? <a name="usage"></a>

[ 我应该用它做什么？]

~~~admonish tip
We'll be using the below code, whenever we write composition arcs in USD.

[ 每当我们以 USD 编写合成弧时，我们会使用以下代码]
~~~

## Resources [资源]<a name="resources"></a>
- [Sdf.Layer](https://openusd.org/dev/api/class_sdf_layer.html)
- [Sdf.LayerOffset](https://openusd.org/dev/api/class_sdf_layer_offset.html)
- [Sdf.\<Arc\>ListOp](https://www.sidefx.com/docs/hdk/list_op_8h.html)
- [Usd.Inherits](https://openusd.org/dev/api/class_usd_inherits.html)
- [Usd.VariantSets](https://openusd.org/dev/api/class_usd_variant_sets.html)
- [Usd.VariantSet](https://openusd.org/dev/api/class_usd_variant_set.html)
- [Usd.References](https://openusd.org/dev/api/class_usd_references.html)
- [Sdf.VariantSetSpec](https://openusd.org/dev/api/class_sdf_variant_set_spec.html)
- [Sdf.VariantSpec](https://openusd.org/dev/api/class_sdf_variant_spec.html)
- [Sdf.Reference](https://openusd.org/dev/api/class_sdf_reference.html)
- [Usd.Payloads](https://openusd.org/dev/api/class_usd_payloads.html)
- [Sdf.Reference](https://openusd.org/dev/api/class_sdf_payload.html)
- [Usd.Specializes](https://openusd.org/dev/api/class_usd_specializes.html)

## Overview [概述]<a name="overview"></a>
This section will focus on how to create each composition arc via the high and low level API.

[ 本节将重点介绍如何通过高级和低级 API 创建每个合成弧]

## Composition Arcs
All arcs that make use of [list-editable ops](./fundamentals.md#list-editable-operations-ops), take of of these tokens as an optional `position` keyword argument via the high level API.

[ 所有使用列表编辑操作的合成弧，都可以通过高级 API 将这些 tokens 作为可选的位置参数来使用]

- `Usd.ListPositionFrontOfAppendList`: Prepend to append list, the same as `Sdf.<Type>ListOp`.appendedItems.insert(0, item)

    [ Usd.ListPositionFrontOfAppendList ：添加到前置列表之后，与 Sdf.\<Type\>ListOp.appishedItems.insert(0, item) 相同]
- `Usd.ListPositionBackOfAppendList`: Append to append list, the same as `Sdf.<Type>ListOp`.appendedItems.append(item)

    [ Usd.ListPositionBackOfAppendList ：添加到后置列表之后，与 Sdf.\<Type\>ListOp.appishedItems.append(item) 相同]
- `Usd.ListPositionFrontOfPrependList`: Prepend to prepend list, the same as `Sdf.<Type>ListOp`.appendedItems.insert(0, item)

    [ Usd.ListPositionFrontOfPrependList ：添加到前置列表之前，与 Sdf.\<Type\>ListOp.appishedItems.insert(0, item) 相同]
- `Usd.ListPositionBackOfPrependList`: Append to prepend list, the same as `Sdf.<Type>ListOp`.appendedItems.append(item)

    [ Usd.ListPositionBackOfPrependList ：添加到后置列表之后，与 Sdf.\<Type\>ListOp.appishedItems.append(item) 相同]

As we can see, all arc APIs, except for sublayers, in the high level API, are thin wrappers around the [list editable op](./fundamentals.md#list-editable-operations-ops) of the arc.

[ 正如我们所看到的，在高级 API 中，除子层之外的所有 arc API 都是围绕 arc 的 [list editable op](./fundamentals.md#list-editable-operations-ops) 包装器]


### Sublayers / Local Opinions <a name="compositionArcSublayer"></a>
~~~admonish tip title="Pro Tip | What do I use sublayer arcs for?"
In our [Composition Strength Ordering (LIVRPS)](./livrps.md#compositionArcSublayer) section we cover in detail, with production related examples, what the sublayer arc is used for.

[ 在我们的 [合成强度排序 (LIVRPS)](./livrps.md#compositionArcSublayer) 部分，我们通过与生产相关的示例详细介绍了 sublayer 合成弧的用途]
~~~

~~~admonish tip title=""
```python
{{#include ../../../../code/core/composition.py:compositionArcSublayer}}
```
~~~

When working in Houdini, we can't directly sublayer onto the root layer as with native USD, due to Houdini's layer caching mechanism, that makes node based stage editing possible. Layering on the active layer works as usual though.

[ 在 Houdini 中工作时，由于 Houdini 的图层缓存机制,使得基于节点的 stage 编辑成为可能，因此我们无法像原生 USD 那样直接将子图层添加到 root layer. 不过在激活图层上进行图层叠加操作还是照常进行]

~~~admonish tip title=""
```python
{{#include ../../../../code/core/composition.py:compositionArcSublayerHoudini}}
```
~~~

Here is the result:

![Alt text](houdiniCompositionSublayerPython.jpg)

#### Value Clips <a name="compositionArcValueClips"></a>
~~~admonish tip title="Pro Tip | What do I use value clips for?"
In our [Composition Strength Ordering (LIVRPS)](./livrps.md#compositionArcValueClips) section we cover in detail, with production related examples, what value clips are used for.

[ 在我们的 [合成强度排序 (LIVRPS)](./livrps.md#compositionArcValueClips) 部分中，我们通过与生产相关的示例详细介绍了值剪辑的用途]
~~~

We cover value clips in our [animation section](../elements/animation.md). Their opinion strength is lower than direct (sublayer) opinions, but higher than anything else.

[ 我们在[动画](../elements/animation.md) 部分介绍了值剪辑. 它们的权重低于直接（sublayer）权重，但高于其他任何权重]

The write them via metadata entries as covered here in our [value clips](../elements/animation.md#value-clips-loading-time-samples-from-multiple-files) section.

[ 通过元数据条目写入它们，如 [value clips](../elements/animation.md#value-clips-loading-time-samples-from-multiple-files) 部分所述]

### Inherits <a name="compositionArcInherit"></a>
~~~admonish tip title="Pro Tip | What do I use inherit arcs for?"
In our [Composition Strength Ordering (LIVRPS)](./livrps.md#compositionArcInherit) section we cover in detail, with production related examples, what the inherit arc is used for.

[ 在我们的 [合成强度排序 (LIVRPS)](./livrps.md#compositionArcInherit) 部分中，我们通过与生产相关的示例详细介绍了继承弧的用途]
~~~

Inherits, like specializes, don't have a object representation, they directly edit the list-editable op list.

[ Inherits 和 specializes 一样，没有对象，它们直接操作 list-editable 列表]

~~~admonish tip title=""
```python
{{#include ../../../../code/core/composition.py:compositionArcInherit}}
```
~~~

### Variants <a name="compositionArcVariant"></a>
~~~admonish tip title="Pro Tip | What do I use variant arcs for?"
In our [Composition Strength Ordering (LIVRPS)](./livrps.md#compositionArcVariant) section we cover in detail, with production related examples, what the variant arc is used for.

[ 在我们的 [合成强度排序 (LIVRPS)](./livrps.md#compositionArcVariant) 部分中，我们通过与生产相关的示例详细介绍了变体弧的用途]
~~~

Variant sets (the variant set->variant name mapping) are also managed via list editable ops.
The actual variant set data is not though. It is written "in-line" into the prim spec via the `Sdf.VariantSetSpec`/`Sdf.VariantSpec` specs, so that's why we have dedicated specs.

[ 变体集（变体集->变体名称映射）也通过列表编辑操作进行管理. 但实际的变体集数据并非如此. 它通过 Sdf.VariantSetSpec / Sdf.VariantSpec specs “内联”写入 prim specs，因此我们有专用规范]

This means we can add variant data, but hide it by not adding the variant set name to the `variantSets`metadata.

[ 这意味着我们可以添加变体数据，同时不将变体集名称添加到 variantSets 元数据中，就可以隐藏它]

For example here we added it:

~~~admonish tip title=""
```python
def Xform "car" (
    variants = {
        string color = "colorA"
    }
    prepend variantSets = "color"
)
{
    variantSet "color" = {
        "colorA" {
            def Cube "cube"
            {
            }
        }
        "colorB" {
            def Sphere "sphere"
            {
            }
        }
    }
}
```
~~~

Here we skipped it, by commenting out the:
`car_prim_spec.SetInfo("variantSetNames", Sdf.StringListOp.Create(prependedItems=["color"]))` line in the below code.
This will make it not appear in UIs for variant selections.

[ 这里我们通过注释掉下面代码中的 car_prim_spec.SetInfo("variantSetNames", Sdf.StringListOp.Create(prependedItems=["color"])) 行来跳过它. 这将使其不会出现在用于变体选择的 UI 中]

~~~admonish tip title=""
```python
def Xform "car" (
    variants = {
        string color = "colorA"
    }
)
{
    variantSet "color" = {
        "colorA" {
            def Cube "cube"
            {
            }
        }
        "colorB" {
            def Sphere "sphere"
            {
            }
        }
    }
}
```
~~~

~~~admonish tip title=""
```python
{{#include ../../../../code/core/composition.py:compositionArcVariant}}
```
~~~

<a name="compositionArcVariantCopySpec"></a>
~~~admonish tip title="Pro Tip | Copying layer content into a variant"
When editing variants, we can also move layer content into a (nested) variant very easily via the `Sdf.CopySpec` command. This is a very powerful feature!

[ 编辑变体时，我们还可以通过 Sdf.CopySpec 命令非常轻松地将图层内容移动到（嵌套）变体中. 这是一个非常强大的功能！]

```python
{{#include ../../../../code/core/composition.py:compositionArcVariantCopySpec}}
```
~~~

Here is how we can created nested variant sets via the high level and low level API. As you can see it is quite a bit easier with the low level API.

[ 以下是我们如何通过高级和低级 API 创建嵌套变体集. 正如您所看到的，使用低级 API 会更容易一些]

~~~admonish tip title=""
```python
{{#include ../../../../code/core/composition.py:compositionArcVariantNested}}
```
~~~


### References <a name="compositionArcReference"></a>
~~~admonish tip title="Pro Tip | What do I use reference arcs for?"
In our [Composition Strength Ordering (LIVRPS)](./livrps.md#compositionArcReference) section we cover in detail, with production related examples, what the reference arc is used for.

[ 在我们的 [合成强度排序 (LIVRPS)](./livrps.md#compositionArcReference) 部分中，我们通过与生产相关的示例详细介绍了引用弧的用途]
~~~

The `Sdf.Reference` class creates a read-only reference description object:

[ Sdf.Reference 类创建一个只读引用对象]

~~~admonish tip title=""
```python
{{#include ../../../../code/core/composition.py:compositionArcReferenceClass}}
```
~~~

#### References File <a name="compositionArcReferenceExternal"></a>
Here is how we add external references (references that load data from other files):

[ 以下是我们添加外部引用（从其他文件加载数据的引用）的方法]

~~~admonish tip title=""
```python
{{#include ../../../../code/core/composition.py:compositionArcReferenceExternal}}
```
~~~


#### References Internal <a name="compositionArcReferenceInternal"></a>
Here is how we add internal references (references that load data from another part of the hierarchy) :

[ 以下是我们添加内部引用（从层级结构的另一部分加载数据的引用）的方法]

~~~admonish tip title=""
```python
{{#include ../../../../code/core/composition.py:compositionArcReferenceInternal}}
```
~~~

### Payloads <a name="compositionArcPayload"></a>
~~~admonish tip title="Pro Tip | What do I use payload arcs for?"
In our [Composition Strength Ordering (LIVRPS)](./livrps.md#compositionArcPayload) section we cover in detail, with production related examples, what the payload arc is used for.

[ 在我们的 [合成强度排序 (LIVRPS)](./livrps.md#compositionArcPayload) 部分中，我们通过与生产相关的示例详细介绍了 payload 弧的用途]
~~~

The `Sdf.Payload` class creates a read-only payload description object:

[ Sdf.Payload 类创建一个只读对象]

~~~admonish tip title=""
```python
{{#include ../../../../code/core/composition.py:compositionArcPayloadClass}}
```
~~~

Here is how we add payloads. Payloads always load data from other files:

[ 这是我们添加 payloads 的方法 payloads 总是从其他文件加载数据]

~~~admonish tip title=""
```python
{{#include ../../../../code/core/composition.py:compositionArcPayload}}
```
~~~

### Specializes <a name="compositionArcSpecialize"></a>
~~~admonish tip title="Pro Tip | What do I use specialize arcs for?"
In our [Composition Strength Ordering (LIVRPS)](./livrps.md#compositionArcSpecialize) section we cover in detail, with production related examples, what the specialize arc is used for.

[ 在我们的 [合成强度排序 (LIVRPS)](./livrps.md#compositionArcSpecialize) 部分中，我们通过与生产相关的示例详细介绍 specialize 弧的用途]
~~~

Specializes, like inherits, don't have a object representation, they directly edit the list-editable op list.

[ Specializes 就像继承一样，没有对象，它们直接操作列表编辑的列表]

~~~admonish tip title=""
```python
{{#include ../../../../code/core/composition.py:compositionArcSpecialize}}
```
~~~