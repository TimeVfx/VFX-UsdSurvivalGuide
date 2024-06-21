
# Prims
For an overview and summary please see the parent [Data Containers](./data_container.md) section.

[ 有关概述和摘要，请参阅 [数据容器](./data_container.md) 部分]

# Table of Contents [目录]
1. [Prim Basics](#primBasics)
    1. [Specifier](#primSpecifier)
    2. [Type Name](#primTypeName)
    3. [Kind](#primKind)
    4. [Active](#primActive)
    5. [Metadata](#primMetadata)
    6. [Tokens (Low Level API)](#primTokens)
    7. [Debugging](#primDebugging)
2. [Hierarchy (Parent/Child)](#primHierarchy)
3. [Schemas](#primSchemas)
4. [Composition](#primComposition)
5. [Loading Data (Activation/Visibility)](#primLoading)
6. [Properties (Attributes/Relationships)](#primProperties)


## Overview [概述]
The main purpose of a prim is to define and store properties. The prim class itself only stores very little data:

[ prim 的主要目的是定义和存储属性. prim 自身只存储很少的数据]

- Path/Name

    [ 路径 / 名称]
- A connection to its properties

    [ 属性之间的关联信息]
- Metadata related to composition and schemas as well as core metadata(specifier, typeName, kind,activation, assetInfo, customData)

    [ 合成和层级结构相关的元数据以及核心元数据（specifier、typeName、kind、activation、assetInfo、customData）]

This page covers the data on the prim itself, for properties check out this [section](./property.md).

[ 本页主要是 prim 本身的数据，有关 properties 请查看 [此章节](./property.md) ]

The below examples demonstrate the difference between the higher and lower level API where possible. Some aspects of prims are only available via the high level API, as it acts on composition/stage related aspects.

[ 下面的示例尽可能演示了高级别 API 和低级别 API 之间的差异. 当它用于 composition/stage 方面时, prims 只能通过高级 API 来操作]

~~~admonish warning title=""
There is a lot of code duplication in the below examples, so that each example works by itself. In practice editing data is very concise and simple to read, so don't get overwhelmed by all the examples.

[ 为了保证每个示例都能独立运行, 下面的示例中存在大量的重复代码. 在实际生产中编辑的数据非常简洁且易于阅读，所以不要被这些示例所迷惑]
~~~

### Prim Basics [Prim 基本知识]<a name="primBasics"></a>
Setting core metadata via the high level is not all exposed on the prim itself via getters/setters, instead
the getters/setters come in part from schemas or schema APIs. For example setting the kind is done via the Usd.ModelAPI.

[ 高级接口并不会暴露所有的元数据设置接口在 prim 自身的 getters/settersAPI 上，相反 getters/setters 一部分 API 来自 schemas 或 schemas API. 例如，设置 kind 是通过 Usd.ModelAPI 来完成的 ]

##### High Level
~~~admonish info title=""
```python
{{#include ../../../../code/core/elements.py:dataContainerPrimCoreHighLevel}}
```
~~~

We are also a few "shortcuts" that check specifiers/kinds (`.IsAbstract`, `.IsDefined`, `.IsGroup`, `.IsModel`), more about these in the kind section below.

[ 我们还有一些“快捷方式”用来检查 specifiers/kinds （ .IsAbstract 、 .IsDefined 、 .IsGroup 、 .IsModel ），下面有更多关于这些方面的介绍]

##### Low Level
The Python lower level Sdf.PrimSpec offers quick access to setting common core metadata via standard class instance attributes:

[ Python 低级别的 Sdf.PrimSpec API 通过类实例属性提供快速对核心元数据的访问]

~~~admonish info title=""
```python
{{#include ../../../../code/core/elements.py:dataContainerPrimCoreLowLevel}}
```
~~~

We will look at specifics in the below examples, so don't worry if you didn't understand everything just yet :)

[ 我们将在下面的示例中进行详细的介绍，所以如果您还没有理解这些内容，请不要担心]


#### Specifiers <a name="primSpecifier"></a>
Usd has the concept of [specifiers](https://openusd.org/release/glossary.html#usdglossary-specifier).

[ USD 提供了 [specifiers](https://openusd.org/release/glossary.html#usdglossary-specifier) 概念的说明]

~~~admonish important
The job of specifiers is mainly to define if a prim should be visible to hierarchy traversals. More info about traversals in our [Layer & Stage](./layer.md) section.

[ specifiers 的主要作用是定义 prim 对象是否应该对层级结构遍历可见. 有关遍历的更多信息，请参阅 [Layer & Stage](./layer.md)]
~~~

Here is an example USD ascii file with all three specifiers.

[ 以下是包含所有三个 specifiers 的 USD ascii 文件示例]

~~~admonish info title=""
```json
def Cube "definedCube" ()
{
    double size = 2
}

over Cube "overCube" ()
{
    double size = 2
}

class Cube "classCube" ()
{
    double size = 2
}
```
~~~

This is how it affects traversal:

[ 下面展示它如何影响遍历]

~~~admonish info title=""
```python
{{#include ../../../../code/core/elements.py:dataContainerPrimBasicsSpecifierTraversal}}
```
~~~


##### Sdf.SpecifierDef: `def`(define)
This specifier is used to specify a prim in a hierarchy, so that is it always visible to traversals.

[ 此 specifier 用于详细描述层级结构中的 prim，所以它对于遍历始终可见]

~~~admonish info title=""
```python
{{#include ../../../../code/core/elements.py:dataContainerPrimBasicsSpecifierDef}}
```
~~~

##### Sdf.SpecifierOver: `over`
Prims defined with `over` only get loaded if the prim in another layer has been specified with a `def`specified. It gets used when you want to add data to an existing hierarchy, for example layering only position and normals data onto a character model, where the base model has all the static attributes like topology or UVs.

[ 仅当使用 over 定义的 prim 在另一个 layer 中被定义为 `def`specified 的时候, 才会加载使用. 当你想向现有的层级结构中添加数据时，例如，要将位置和法线数据添加到角色模型上，而原始基础模型则包含所有静态属性，如拓扑结构或UV坐标, 就可以使用它]

~~~admonish important
By default stage traversals will skip over `over` only prims. Prims that only have an `over` also do not get forwarded to Hydra render delegates.

[ 默认情况下，层级结构遍历将跳过只有“over”的 prim 对象. 只有“over”的 prim 对象也不会被转发给Hydra渲染委托]
~~~

~~~admonish info title=""
```python
{{#include ../../../../code/core/elements.py:dataContainerPrimBasicsSpecifierOver}}
```
~~~

##### Sdf.SpecifierClass: `class`
The `class` specifier gets used to define "template" hierarchies that can then get attached to other prims. A typical example would be to create a set of geometry render settings that then get applied to different parts of the scene by creating an inherit composition arc. This way you have a single control point if you want to adjust settings that then instantly get reflected across your whole hierarchy.

[ class specifier 用于定义“模板”的层级结构，可以将其附加到其他 prims 上. 一个典型的例子是创建一组几何渲染设置，然后通过创建继承合成弧（inherit composition arc）将这些设置应用于场景的不同部分. 这样，如果您想调整设置，就只需控制一个点，这些调整会立即反映在整个层级结构中]

~~~admonish important
- By default stage traversals will skip over `class` prims.

    [ 默认情况下，阶段遍历将跳过 class prims ]
- Usd refers to class prims as "abstract", as they never directly contribute to the hierarchy.

    [ USD 将 class prims 对象定为“抽象”，因为它们从不直接对层级结构做出贡献]
- We target these class prims via inherits/internal references and specialize [composition arcs](../composition/livrps.md)

    [ 我们通过 inherits/internal 和 specialize [composition arcs](../composition/livrps.md) 来定位这些 class prims 对象]
~~~

~~~admonish info title=""
```python
{{#include ../../../../code/core/elements.py:dataContainerPrimBasicsSpecifierClass}}
```
~~~


#### Type Name <a name="primTypeName"></a>
The type name specifies what concrete schema the prim adheres to.
In plain english: Usd has the concept of schemas, which are like OOP classes. Each prim can be an instance of a class, so that it receives the default attributes of that class. More about schemas in our [schemas](./schemas.md) section. You can also have prims without a type name, but in practice you shouldn't do this. For that case USD has an "empty" class that just has all the base attributes called "Scope".

[ Type Name 指定 prim 遵循的具体规范 简单来说：Usd 有一个 schemas 的概念，它就像面向对象的类. 每个 prim 都可以是类的实例，因此它接收该类的默认属性. 有关 schemas 更多信息，请参阅我们的 [schemas](./schemas.md) 部分. 您也可以使用没有类型名称的 prims，但实际上您不应该这样做. 对于这种情况，USD有一个“空”类，它只具有称为“Scope”的所有基本属性]

~~~admonish info title=""
```python
{{#include ../../../../code/core/elements.py:dataContainerPrimBasicsTypeName}}
```
~~~

#### Kind <a name="primKind"></a>
The [kind](https://openusd.org/release/glossary.html#usdglossary-kind) metadata can be attached to prims to mark them what kind hierarchy level it is. This way we can quickly traverse and select parts of the hierarchy that are of interest to us, without traversing into every child prim.

[ Kind 元数据附加到 prims 用以标记它们的层级结构的级别. 这样我们就可以快速遍历并选择层级结构中我们感兴趣的部分，而无需遍历每个子 prim ]

For a full explanation we have a dedicated section: [Kinds](../plugins/kind.md)

[ 为了获得详细的解释，我们有一个专门的章节 [Kinds](../plugins/kind.md)]

Here is the reference code on how to set kinds. For a practical example with stage traversals, check out the kinds page.

[ 这是关于如何设置 Kinds 的参考代码. 有关在 stage 上进行遍历的例子，请跳转到 Kinds 页面查看]

~~~admonish info title=""
```python
{{#include ../../../../code/core/elements.py:dataContainerPrimBasicsKinds}}
```
~~~

#### Active <a name="primActive"></a>
The `active` metadata controls if the prim and its children are loaded or not.
We only cover here how to set the metadata, for more info checkout our [Loading mechansims](./loading_mechanisms.md) section. Since it is a metadata entry, it can not be animated. For animated pruning we must use [visibility](./property.md#visibility).

[ active 元数据控制是否加载 prim 及其子项. 我们在这里仅介绍如何设置元数据，有关更多信息，请查看 [Loading mechansims](./loading_mechanisms.md) 部分. 由于它是元数据，因此无法设置动画. 对于动画修剪，必须使用 [visibility](./property.md#visibility)]

~~~admonish info title=""
```python
{{#include ../../../../code/core/elements.py:metadataActive}}
```
~~~

#### Metadata <a name="primMetadata"></a>
We go into more detail about metadata in our [Metadata](./metadata.md) section.

[ 我们将在 [Metadata](./metadata.md) 的章节详细介绍元数据]

~~~admonish important
As you can see on this page, most of the prim functionality is actually done via metadata, except path, composition and property related functions/attributes.

[ 正如您在本章看到的，除了与 path, composition 和 property 相关的功能/属性之外，大部分的 prim 功能实际上是通过元数据来实现的]
~~~

#### Tokens (Low Level API) <a name="primTokens"></a>
Prim (as well as property, attribute and relationship) specs also have the tokens they can set as their metadata as class attributes ending with 'Key'.
These 'Key' attributes are the token names that can be set on the spec via `SetInfo`, for example prim_spec.SetInfo(prim_spec.KindKey, "group")

[ 可以用以 “Key” 结尾的 tokens 来设置 Prim (以及 property、attribute 和 relationship ) specs 元数据. 使用 SetInfo 设置 “Key” 属性的名称，例如 prim_spec.SetInfo(prim_spec.KindKey, "group")]

~~~admonish info title=""
```python
{{#include ../../../../code/core/elements.py:dataContainerPrimBasicsTokens}}
```
~~~

#### Debugging (Low Level API) <a name="primDebugging"></a>
You can also print a spec as its ascii representation (as it would be written to .usda files):

[ 您还可以输出 ascii 形式的 spec（就像将其写入 .usda 文件一样）]

~~~admonish info title=""
```python
{{#include ../../../../code/core/elements.py:dataContainerPrimBasicsDebugging}}
```
~~~

### Hierarchy (Parent/Child) [层级结构]<a name="primHierarchy"></a>
From any prim you can navigate to its hierarchy neighbors via the path related methods.
The lower level API is dict based when accessing children, the high level API returns iterators or lists.

[ 通过 path 相关的方法, 可以从任何 prim 访问到其附近的元素. 低级别的 API 返回的是基于字典的信息, 高级别的 API 返回的是迭代器或者列表]

~~~admonish info title=""
```python
{{#include ../../../../code/core/elements.py:dataContainerPrimHierarchy}}
```
~~~

### Schemas <a name="primSchemas"></a>

We explain in more detail what schemas are in our [schemas](./schemas.md) section.
In short: They are the "base classes" of Usd. Applied schemas are schemas that don't 
define the prim type and instead just "apply" (provide values) for specific metadata/properties

[ 我们在 [schemas](./schemas.md) 部分更详细地解释了schemas是什么. 简单来说，它是 usd 的“基类” 使用 schemas 主要是设置特定 metadata/properties 的值，而不是定义prim的基本类型 ]

~~~admonish important
The 'IsA' check is a very valueable check to see if something is an instance of a (base) class. It is similar to Python's isinstance method.

[ “IsA” 是一个非常有用的检查，用于查看某个对象是否是（基础）类的实例. 它与 Python 的 isinstance 方法类似]
~~~

~~~admonish info title=""
```python
{{#include ../../../../code/core/elements.py:dataContainerPrimSchemas}}
```
~~~

#### Prim Type Definition (High Level)
With the [prim definition](https://openusd.org/dev/api/class_usd_prim_definition.html) we can inspect what the schemas provide. Basically you are inspecting the class (as to the prim being the instance, if we compare it to OOP paradigms).
In production, you won't be using this a lot, it is good to be aware of it though.

[ 通过 [prim definition](https://openusd.org/dev/api/class_usd_prim_definition.html) 我们可以检查 schemas 都使用了些什么. 相当于检查类（如果我们将其与面向对象的范式进行比较，那么 prim 就是该类的实例）. 在生产环境中，你不会经常使用这个功能，但有必要去了解它]

~~~admonish info title=""
```python
{{#include ../../../../code/core/elements.py:dataContainerPrimTypeDefinition}}
```
~~~

#### Prim Type Info (High Level)
The [prim type info](https://openusd.org/dev/api/class_usd_prim_type_info.html) holds the composed type info of a prim. You can think of it as as the class that answers Python `type()` like queries for Usd. It caches the results of type name and applied API schema names, so that `prim.IsA(<typeName>)` checks can be used to see if the prim matches a given type.

[ [prim type info](https://openusd.org/dev/api/class_usd_prim_type_info.html) 中存放了 prim 的合成类型信息. 你可以将它看作在 usd 中类似于 Python 的 type() 的查询. 存储了 type 名称和已应用的 API schema 的名称，因此可以使用 prim.IsA(\<typeName\>) 来检查 Prim 是否与给定的类型相匹配]

~~~admonish tip
The prim's `prim.IsA(<typeName>)` checks are highly performant, you should use them as often as possible when traversing stages to filter what prims you want to edit. Doing property based queries to determine if a prim is of interest to you, is a lot slower.

[ prim 对象的 prim.IsA(\<typeName\>) 检查具有极高的性能，在遍历阶段筛选要编辑的 prim 对象时，应尽可能多地使用它们. 用基于属性进行查询以确定某个 prim 对象是否为你所关注的对象，速度会慢得多]
~~~

~~~admonish info title=""
```python
{{#include ../../../../code/core/elements.py:dataContainerPrimTypeInfo}}
```
~~~

### Composition <a name="primComposition"></a>
We discuss handling composition in our [Composition](../composition/overview.md) section as it follows some different rules and is a bigger topic to tackle.

[ 我们将在 [合成](../composition/overview.md) 章节讨论怎样处理合成的问题，因为它遵循一些不同的规则，并且需要一个大章节来深入探讨]


### Loading Data (Purpose/Activation/Visibility) <a name="primLoading"></a>
We cover this in detail in our [Loading Data](./loading_mechanisms.md) section.

[ 我们在 [Loading Data](./loading_mechanisms.md) 章节进行了详细介绍]

~~~admonish info title=""
```python
{{#include ../../../../code/core/elements.py:dataContainerPrimLoading}}
```
~~~

### Properties/Attributes/Relationships <a name="primProperties"></a>
We cover properties in more detail in our [properties](./property.md) section.

[ 我们在 [properties](./property.md) 章节进行了详细介绍]

~~~admonish important title="Deep Dive | Properties"
Technically properties are also stored as metadata on the `Sdf.PrimSpec`. So later on when we look at composition, keep in mind that the prim stack therefore also drives the property stack. That's why the prim index is on the `prim` level and not on the `property` level.

[ 从技术上讲，属性也作为元数据存储在 Sdf.PrimSpec 上. 因此，在后续查看层合成时，请记住 prim 对象堆栈也驱动着属性堆栈. 这就是为什么 prim index 位于 prim 对象级别而不是属性级别的原因]
```python
...
print(prim_spec.properties, prim_spec.attributes, prim_spec.relationships)
print(prim_spec.GetInfo("properties"))
...
```
~~~

Here are the basics for both API levels:

[ 以下是两个API级别的基础知识]

#### High Level API
~~~admonish info title=""
```python
{{#include ../../../../code/core/elements.py:dataContainerPrimPropertiesHighLevel}}
```
~~~

#### Low Level API
To access properties on `Sdf.PrimSpec`s we can call the `properties`, `attributes`, `relationships` methods. These return a dict with the {'name': spec} data.
Here is an example of what is returned when you create cube with a size attribute:

[ 要在 Sdf.PrimSpec 上访问属性，我们可以调用 properties 、 attributes 、 relationships 方法. 它返回一个 {'name': spec} 的字典数据. 以下是创建带有 size 属性​​的立方体模型时返回的示例：]


~~~admonish info title=""
```python
{{#include ../../../../code/core/elements.py:dataContainerPrimPropertiesLowLevel}}
```
~~~

~~~admonish important
Since the lower level API doesn't see the schema properties, these commands will only return what is actually in the layer, in Usd speak `authored`.

[ 由于低级别的API无法看到 schema properties，这些命令只会返回层中实际存在的内容, 用USD语言来说就是“authored”的内容]
~~~

With the high level API you can get the same/similar result by calling prim.GetAuthoredAttributes() as you can see above.
When you have multiple layers, the prim.GetAuthoredAttributes(), will give you the created attributes from all layers, where as the low level API only the ones from the active layer.

[ 使用高级API，你可以通过调用 prim.GetAuthoredAttributes() 来获得与上述相同或类似的结果. 当你拥有多个 layers 时，prim.GetAuthoredAttributes()将为你提供来自所有层已创建的属性，而低级API则仅只返回激活层的属性. ]

As mentioned in the `properties` section, properties is the base class, so the `properties` method will give you
the merged dict of the `attributes` and `relationship` dicts.

[ 正如在 `properties` 章节中所提到的，properties 是基类，因此 properties 的方法将返回将 attributes 和 relationship 两个字典的合并后的字典]





