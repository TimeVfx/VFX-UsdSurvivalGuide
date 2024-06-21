# Paths [路径]
As Usd is a hierarchy based format, one of its core functions is handling paths.
In order to do this, Usd provides the pxr.Sdf.Path class. You'll be using quite a bunch, so that's why we want to familiarize ourselves with it first.

[ USD 是一种基于层级结构的设计，因此其核心功能之一是处理路径. 为了做到这一点，USD 提供了 pxr.Sdf.Path 类. 您将会经常使用，所以我们首先要熟悉它]

~~~admonish info title=""
```python
pxr.Sdf.Path("/My/Example/Path")
```
~~~

# Table of Contents [目录]
1. [API Overview In-A-Nutshell](#summary)
2. [What should I use it for?](#usage)
3. [Resources](#resources)
4. [Overview](#overview)
    1. [Creating a path & string representation](#pathBasics)
    2. [Special Paths: emptyPath & absoluteRootPath](#pathSpecialPaths)
    3. [Variants ](#pathVariants)
    4. [Properties](#pathProperties)

## TL;DR - Paths In-A-Nutshell [概述] <a name="summary"></a>
Here is the TL;DR version:
Paths can encode the following path data:

[ 这是 TL;DR 摘要版本. Paths 可以按以下路径数据进行操作]

- `Prim`: "/set/bicycle" - Separator `/`
- `Property`:
    - `Attribute`: "/set/bicycle.size"  - Separator `.`
    - `Relationship`: "/set.bikes[/path/to/target/prim]"  - Separator `.` / Targets `[]`  (Prim to prim target paths e.g. collections of prim paths)
- `Variants` ("/set/bicycle{style=blue}wheel.size")

~~~admonish info title=""
```python
{{#include ../../../../code/core/elements.py:pathSummary}}
```
~~~

## What should I use it for? <a name="usage"></a>

[ 我应该用它做什么？]

~~~admonish tip
Anything that is path related in your hierarchy, use Sdf.Path objects. It will make your life a lot easier than if you were to use strings.

[ 在您的层级结构中，任何与路径相关的内容，都请使用Sdf.Path对象. 这将使您的工作比使用字符串时轻松得多]
~~~

## Resources [资源]<a name="resources"></a>
- [Sdf.Path](https://openusd.org/release/api/class_sdf_path.html#sec_SdfPath_Overview)

## Overview [概述]<a name="overview"></a>
Each element in the path between the "/" symbol is a [prim](https://openusd.org/release/glossary.html#usdglossary-prim) similar to how on disk file paths mark a folder or a file.

[ 路径中 "/" 分割的每一个 prim 元素, 类似与磁盘文件路径中标记文件夹或文件的方式]

Most Usd API calls that expect Sdf.Path objects implicitly take Python strings as well, we'd recommend using Sdf.Paths from the start though, as it is faster and more convenient.

[ 大多数 USD API 调用都期望得到 Sdf.Path 对象，但它们也隐式地接受 Python 字符串, 但我们建议从一开始就使用 Sdf.Paths，因为它更快、更方便]

We recommend going through these small examples (5-10 min), just to get used to the Path class.

[ 我们建议您阅读这些示例（5-10 分钟），以便习惯 Path 类。]

### Creating a path & string representation <a name="pathBasics"></a>

~~~admonish info title=""
```python
{{#include ../../../../code/core/elements.py:pathBasics}}
```
~~~

### Special Paths: emptyPath & absoluteRootPath <a name="pathSpecialPaths"></a>

~~~admonish info title=""
```python
{{#include ../../../../code/core/elements.py:pathSpecialPaths}}
```
~~~

### Variants <a name="pathVariants"></a>
We can also encode variants into the path via the {variantSetName=variantName} syntax.

[ 我们还可以将变体编码到路径中.通过{variantSetName=variantName}语法]

~~~admonish info title=""
```python
{{#include ../../../../code/core/elements.py:pathVariants}}
```
~~~

### Properties <a name="pathProperties"></a>
Paths can also encode properties (more about what these are in the next section).
Notice that attributes and relationships are both encoded with the "." prefix, hence the name `property` is used to describe them both.

[ 路径还可以对属性进行编码（下一节将详细介绍这些属性）。请注意，属性和关系都用“.”前缀进行编码. 所以 property 是用来描述他们俩的]

~~~admonish tip
When using Usd, we'll rarely run into the relationship `[]` encoded targets paths. Instead we use the `Usd.Relationship`/`Sdf.RelationshipSpec` methods to set the path connections. Therefore it is just good to know they exist.

[ 在使用USD时，我们很少会遇到关系链接使用 [] 来编码的目标路径.相反，我们使用Usd.Relationship/Sdf.RelationshipSpec方法来设置路径连接. 因此，只需知道它们存在即可]
~~~

~~~admonish info title=""
```python
{{#include ../../../../code/core/elements.py:pathProperties}}
```
~~~

