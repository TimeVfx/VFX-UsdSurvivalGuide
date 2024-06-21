# Data Types/Roles [数据类型/数据角色]

# Table of Contents [目录]
1. [Data Types/Roles In-A-Nutshell](#summary)
1. [What should I use it for?](#usage)
1. [Resources](#resources)
1. [Overview](#overview)
    1. [Data Types](#dataTypes)
    1. [Data Roles](#dataRoles)

## TL;DR - Metadata In-A-Nutshell [概述]<a name="summary"></a>
When reading and writing data in USD, all data is of a specific data type. USD extends the standard base types (`float`, `int`) with computer graphics related classes (e.g. `Gf.Matrix3d`, `Gf.Vec3h`) that make it easy to perform common 3d math operations and Usd related types (e.g. `Sdf.Asset`). To give a hint about how a data type should be used, we also have the concept of data roles (`Position`, `Normal`, `Color`). All data value type names are stored in the `Sdf.ValueTypeNames` registry, from these we can get the base classes stored in `Gf` (math related classes), `Sdf`(USD related classes) and `Vt` (array classes that carry `Gf` data types).

[ 在USD中读取写入数据时, 所有的数据都是有特定的数据类型. USD 扩展了标准基础类型（如 float、int）加入了与计算机图形相关的类（例如 Gf.Matrix3d、Gf.Vec3h） 让常见的 3D 数学运算变得容易，同时 USD 也增加了与其自身相关的类型（例如 Sdf.Asset） 此外 USD 还有数据角色（Position、Normal、Color）的概念，用于提示数据类型应如何使用. 所有的数据值类型名称都存储在 Sdf.ValueTypeNames 注册表中, 从注册表中我们可以获取存储在 Gf（与数学相关的类）、Sdf（USD相关类）和 Vt（携带Gf数据类型的数组类）中的基础类]

~~~admonish tip title="Pro Tip | Creating arrays"
The constructors for arrays allow us to input tuples/lists instead of the explicit base classes.

[ 数组的构造函数允许我们输入元组/列表，而不是明确的基类]

```python
# Explicit
pxr.Vt.Vec3dArray([Gf.Vec3d(1,2,3)])
# Implicit
pxr.Vt.Vec3dArray([(1,2,3)])
```
Instead of searching for the corresponding `Gf` or `Vt` array class, we can get it from the type name and instantiate it:

[ 我们可以从类型名称中获取并实例化它，而不是搜索相应的 Gf 或 Vt 数组类]

```python
vec3h_array = Sdf.ValueTypeNames.Point3hArray.type.pythonClass([(0,1,2)])
# Or: Usd.Attribute.GetTypeName().type.pythonClass([(0,1,2)]) / Sdf.AttributeSpec.typeName.type.pythonClass([(0,1,2)])
print(type(vec3h_array)) # Returns: <class 'pxr.Vt.Vec3hArray'>
```
As `Vt.Array` support the buffer protocol, we can map the arrays to numpy without data duplication and perform high performance 
value editing.

[ 由于 Vt.Array 支持缓冲区协议，我们可以将数组映射到 numpy 无需复制数据 执行高性能的值编辑]

```python
from pxr import Vt
import numpy as np
from array import array
# Python Arrays
vt_array = Vt.Vec3hArray.FromBuffer(array("f", [1,2,3,4,5,6])) # Returns: Vt.Vec3hArray(2, (Gf.Vec3h(1.0, 2.0, 3.0),Gf.Vec3h(4.0, 5.0, 6.0),))
# From Numpy Arrays 
Vt.Vec3hArray.FromNumpy(np.ones((10, 3)))
Vt.Vec3hArray.FromBuffer(np.ones((10, 3)))
# To Numpy arrays
np.array(vt_array)
```
~~~

## What should I use it for? <a name="usage"></a>

[我应该用它做什么？]

~~~admonish tip
When creating attributes we always have to specify a `Sdf.ValueTypeName`, which defines the data type & role.
USD ships with great computer graphics related classes in the `Gf` module, which we can use for calculations.
The `Vt.Array` wrapper around the `Gf` data base classes implements the buffer protocol, so we can easily map these arrays to Numpy arrays and perform high performance value edits. This is great for adjusting geometry related attributes.

[ 在创建属性时，我们总是需要指定一个 Sdf.ValueTypeName，它定义了数据类型和角色. USD 在 Gf 模块中附带了很棒的计算机图形相关类，我们可以使用这些类进行计算. Gf 数据基类的 Vt.Array 包装器实现了缓冲区协议，因此我们可以轻松地将这些数组映射到 Numpy 数组并执行高性能的值编辑. 这对于调整与几何相关的属性非常有用]
~~~

## Resources [资源]<a name="resources"></a>
- [USD API Data Type Docs](https://openusd.org/dev/api/_usd__page__datatypes.html)
- [Nvidia USD Data Types Docs](https://docs.omniverse.nvidia.com/dev-guide/latest/dev_usd/quick-start/usd-types.html)

## Overview [概述]<a name="overview"></a>
When reading and writing data in USD, all data is of a specific data type. USD extends the standard base types (`float`, `int`) with computer graphics related classes (`Gf.Matrix3d`, `Gf.Vec3h`) that make it easy to perform common 3d math operations. To give a hint about how a data type should be used, we also have the concept of data roles.

[ 在 USD 中读写数据时，所有数据都属于特定的数据类型. USD扩展了标准基础类型（如float、int）增加了与计算机图形相关的类（如Gf.Matrix3d、Gf.Vec3h）使得执行常见的3D数学操作变得简单. 为了提供关于数据类型应如何使用的提示，我们还引入了数据角色的概念]

### Data Types <a name="dataTypes"></a>
Let's first talk about data types. These are the base data classes all data is USD is stored in.

[ 首先，我们来谈谈数据类型. 数据类型是 USD 中存储所有数据的基础数据类]

To access the base data classes there are three modules:

[ 要访问基础数据类，有三个模块可供选择]

- `Sdf`(Scene Description Foundations): Here you'll 99% of the time only be using `Sdf.AssetPath`, `Sdf.AssetPathArray` and the [list editable ops](../composition/listeditableops.md) that have the naming convention `<Type>ListOp` e.g. `Sdf.PathListOp`, `Sdf.ReferenceListOp`, `Sdf.ReferenceListOp`, `PayloadListOp`, `StringListOp`, `TokenListOp`

    [ Sdf（基础场景描述）：在99%的情况下，您只会使用到 Sdf.AssetPath、Sdf.AssetPathArray以及 \<Type\>ListOp 约定命名的 [list editable ops](../composition/listeditableops.md)，例如 Sdf.PathListOp, Sdf.ReferenceListOp, Sdf.ReferenceListOp, PayloadListOp, StringListOp, TokenListOp]
- `Gf`(Graphics Foundations): The `Gf` module is basically USD's math module. Here we have all the math related data classes that are made up of the base types (base types being `float`, `int`, `double`, etc.). Commonly used classes/types are `Gf.Matrix4d`, `Gf.Quath`(Quaternions), `Gf.Vec2f`, `Gf.Vec3f`, `Gf.Vec4f`. Most classes are available in different precisions, noted by the suffix `h`(half),`f`(float), `d`(double).

    [ Gf（基础图形）：Gf 模块基本上就是 USD 的数学模块 由基础类型（float、int、double等）组成的所有数学相关的数据类. 常用的  classes/types 包括 Gf.Matrix4d、Gf.Quath（四元数）、Gf.Vec2f、Gf.Vec3f、Gf.Vec4f 大部分都会提供不同的操作精度，通过后缀 h(half),f(float), d(double) 来区分 ]
- `Vt` (Value Types): Here we can find all the `Array` (list) classes that capture the base classes from the `Gf` module in arrays. For example `Vt.Matrix4dArray`. The `Vt` arrays implement the [buffer protocol](https://docs.python.org/3/c-api/buffer.html), so we can convert to Numpy/Python without data duplication. This allows for some very efficient array editing via Python. Checkout our [Houdini Particles](../../dcc/houdini/fx/particles.md) section for a practical example. The value types module also houses wrapped base types (`Float`, `Int`, etc.), we don't use these though, as with Python everything is auto converted for us.

    [ Vt（值类型）：我们可以将 Gf 模块中的所有基础类放到数组（列表）中. 例如 Vt.Matrix4dArray  Vt数组实现了 [缓冲区协议](https://docs.python.org/3/c-api/buffer.html) 因此我们可以将其转换为 Numpy/Python，而无需复制数据. 这允许我们通过 Python 进行一些非常高效的数组编辑操作. 相关实例在 [Houdini Particles](../../dcc/houdini/fx/particles.md) 部分中. 值类型模块还包含封装的基础类型（如Float、Int等）但我们并不使用这些, 因为在 Python 中所有的东西都会自动为我们进行转换 ]

The `Vt.Type` registry also handles all on run-time registered types, so not only data types, but also custom types like `Sdf.Spec`,`SdfReference`, etc. that is registered by plugins.

[ Vt.Type 注册表还负责处理所有运行时的注册类型，不仅限于数据类型，还包括由插件注册的自定义类型，如 Sdf.Spec、SdfReference等 ]

~~~admonish info title=""
```python
{{#include ../../../../code/core/elements.py:tfTypeRegistry}}
```
~~~


### Data Roles [数据角色]<a name="dataRoles"></a>
Next let's explain what data roles are. Data roles extend the base data types by adding an hint about how the data should be interpreted. For example `Color3h`is just `Vec3h` but with the `role` that it should be treated as a color, in other words it should not be transformed by xforms ops, unlike `Point3h`, which is also `Vec3h`. Having roles is not something USD invented, you can find something similar in any 3d application.

[ 接下来，让我们解释一下数据角色是什么. 数据角色通过添加关于如何解析数据的标识来扩展基础数据类型. 例如，Color3h 本质上只是 Vec3h 但带有角色标签后它应被视为颜色. 换句话说，它不应像 Point3h（也就是 Vec3h）那样可以通过 xforms ops 进行移动操作. 角色这一概念并不是 USD 发明的，您可以在任何 3D 应用程序中找到类似的东西]

### Working with data classes in Python  [ 在 Python 中使用数据类]
You can find all values roles (and types) in the `Sdf.ValueTypeNames` module, this is where USD keeps the value type name registry.

[ 您可以在 Sdf.ValueTypeNames 模块中找到所有值角色（和类型）这是 USD 保存数据类型名称注册表的地方]

The `Sdf.ValueTypeName` class gives us the following properties/methods:

[ Sdf.ValueTypeName 类为我们提供了以下属性/方法]

- `aliasesAsStrings`: Any name aliases e.g. `texCoord2f[]`

    [ aliasesAsStrings ：名称的别名，例如 texCoord2f[] ]
- `role`: The role (intent hint), e.g. "Color", "Normal", "Point"

    [ role ：角色（意图标识），例如 “颜色”、“法线”、“点”]
- `type`: The actual Tf.Type definition. From the type definition we can retrieve the actual Python data type class e.g. `Gf.Vec3h` and instantiate it.

    [ type ：具体的 Tf.Type 定义. 从类型定义中我们可以找到具体的 Python 数据类型类，例如 Gf.Vec3h 并实例化它]
- `cppTypeName`: The C++ data type class name, e.g `GfVec2f`. This is the same as `Sdf.ValueTypeNames.TexCoord2f.type.typeName`

    [ cppTypeName ：C++ 数据类型类名称，例如 GfVec2f 这与 Sdf.ValueTypeNames.TexCoord2f.type.typeName 相同]
- `defaultUnit`: The default unit. As USD is unitless (at least per when it comes to storing the data), this is `Sdf.DimensionlessUnitDefault` most of the time.

    [ defaultUnit ：默认单位. 由于 USD 是无单位的（至少在存储数据时是这样），因此大多数情况下都是 Sdf.DimensionlessUnitDefault]
- `defaultValue`: The default value, e.g. `Gf.Vec2f(0.0, 0.0)`. For arrays this is just an empty Vt.Array. We can use `.scalarType.type.pythonClass` or `.scalarType.defaultValue` to get a valid value for a single element.

    [ defaultValue ：默认值，例如 Gf.Vec2f(0.0, 0.0) 对于数组来说，这只是一个空的 Vt.Array. 我们可以使用 .scalarType.type.pythonClass 或 .scalarType.defaultValue 来获取单个元素的有效值]
- Check/Convert scalar <-> array value type names:

    [ 检查/标量转换 <-> 数组值类型名称：]
    - `isArray`: Check if the type is an array.

        [ isArray ：检查类型是否为数组]
    - `arrayType`: Get the vector type, e.g. `Sdf.ValueTypeNames.AssetArray.arrayType`gives us `Sdf.ValueTypeNames.AssetArray`

        [ arrayType ：获取向量类型，例如 Sdf.ValueTypeNames.AssetArray.arrayType 返回给我们 Sdf.ValueTypeNames.AssetArray]
    - `isScalar`: Check if the type is scalar.

        [ isScalar ：检查类型是否为标量]
    - `scalarType`: Get the scalar type, e.g. `Sdf.ValueTypeNames.AssetArray.scalarType`gives us `Sdf.ValueTypeNames.Asset`

        [ scalarType ：获取标量类型，例如 Sdf.ValueTypeNames.AssetArray.scalarType 返回给我们 Sdf.ValueTypeNames.Asset]

We can also search based on the string representation or aliases of the value type name, e.g. `Sdf.ValueTypeNames.Find("normal3h")`.

[ 我们还可以根据值类型名称的字符串标识或别名进行搜索，例如 Sdf.ValueTypeNames.Find("normal3h")]

~~~admonish tip title="Pro Tip | Creating arrays"
The constructors for arrays allow us to input tuples/lists instead of the explicit base classes.

[ 数组的构造函数允许我们输入元组/列表作为参数，而不需要直接使用明确的基类实例]

```python
# Explicit
pxr.Vt.Vec3dArray([Gf.Vec3d(1,2,3)])
# Implicit
pxr.Vt.Vec3dArray([(1,2,3)])
```
Instead of searching for the corresponding `Gf` or `Vt` array class, we can get it from the type name:

[ 与其搜索对应的 Gf 或 Vt 数组类，我们可以从类型名称中获取它]

```python
vec3h_array =  Sdf.ValueTypeNames.Point3hArray.type.pythonClass((0,1,2))
print(type(vec3h_array)) # Returns: <class 'pxr.Vt.Vec3hArray'>
```
~~~


We won't be looking at specific data classes on this page, instead we'll have a look at how to access the data types from value types and vice versa as usually the data is generated by the DCC we use and our job is to validate the data type/role or e.g. create attributes with the same type when copying values.

[ 在本章节中，我们不会深入探讨具体的数据类，而是会查看如何从值类型访问数据类型. 因为通常数据是由我们使用的 DCC 软件生成的，而我们的工作则是验证数据类型/角色，或者在复制值时创建具有相同类型的属性]


Let's look at this in practice:

[ 让我们看看实践中的情况]

~~~admonish info title=""
```python
{{#include ../../../../code/core/elements.py:dataTypeRoleOverview}}
```
~~~
