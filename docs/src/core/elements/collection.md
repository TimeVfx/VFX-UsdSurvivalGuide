# Collections [集合]
Collections are USD's mechanism of storing a set of prim paths. We can nest/forward collections to other collections and relationships, which allows for powerful workflows. For example we can forward multiple collections to a light linking relationship or forwarding material binding relationships to a single collection on the asset root prim, which then in return forwards to the material prim.

[ 集合是 USD 存储一组 prim 路径的机制. 我们可以将集合嵌套/转换到其他集合和关系，从而实现强大的工作流程. 例如 我们可以将多个集合转换成轻型链接关系，或者将材质绑定关系转换到资产 root prim 上的单个集合，然后将其转发到材质 prim]

# Table of Contents [目录]
1. [Collections In-A-Nutshell](#summary)
1. [What should I use it for?](#usage)
1. [Resources](#resources)
1. [Overview](#overview)
    1. [Creating & querying collections](#collectionQuery)
    1. [Inverting a collection](#collectionInvert)

## TL;DR - Metadata In-A-Nutshell [概述]<a name="summary"></a>
- Collections are encoded via a set of properties. A prim can store any number of collections.

    [ 集合通过一组属性进行编码. prim 可以存储任意数量的集合]
    - `collection:<collectionName>:includes` relationship: A list of target `Sdf.Path`s to include, we can als target other collections.

        [ collection:\<collectionName\>:includes 关系：包含以 Sdf.Path 为目标的列表，我们也可以定位其他集合]
    - `collection:<collectionName>:excludes` relationship: A list of target `Sdf.Path`s to exclude. These must be below the include paths.

        [ collection:\<collectionName\>:excludes 关系：排除以Sdf.Path为目标的列表. 这些必须位于包含路径下方]
    - `collection:<collectionName>:expansionRule`attribute: Controls how collections are computed, either by running `includes` - `excludes` (mode `explicitOnly`) or by expanding all child prims and then doing `includes` - `excludes` (mode `expandPrims`).

        [ collection:\<collectionName\>:expansionRule 属性：控制集合的计算方式，可以通过运行 includes - excludes （ explicitOnly 模式）或通过展开所有子 prims 和然后执行 includes - excludes （ expandPrims 模式）]
- Collections can link to other collections, which gives us a powerful mechanism of forwarding hierarchy structure information.

    [ 集合可以链接到其他集合，这为我们提供了转发层级结构信息的强大机制]
- Collections can easily be accessed and queried via the [Usd.CollectionAPI](https://openusd.org/release/api/class_usd_collection_a_p_i.html). The query can be limited via USD [filter predicates](https://openusd.org/dev/api/prim_flags_8h.html#Usd_PrimFlags), e.g. to defined prims only.

    [ 通过 [Usd.CollectionAPI](https://openusd.org/release/api/class_usd_collection_a_p_i.html) 可以轻松访问和查询集合. 可以通过 USD [filter predicates](https://openusd.org/dev/api/prim_flags_8h.html#Usd_PrimFlags) 限制查询，例如 仅限定义的 prims]
- To help speed up collection creation, USD also ships with util functions:

    [ 为了帮助加快集合创建速度，USD 还附带了 util 函数]
    - Collection creation: `UsdUtils.AuthorCollection(<collectionName>, prim, [<includePathList>], [<excludePathList>])`

        [ 集合创建： UsdUtils.AuthorCollection(\<collectionName\>, prim, [\<includePathList\>], [\<excludePathList\>])]
    - Re-writing a collection to be as sparse as possible: `include_paths, exclude_paths = UsdUtils.ComputeCollectionIncludesAndExcludes(target_paths, stage)`

        [ 重新编写集合，使其尽可能稀疏： include_paths, exclude_paths = UsdUtils.ComputeCollectionIncludesAndExcludes(target_paths, stage)]

## What should I use it for? <a name="usage"></a>

[ 我应该用它做什么？]

~~~admonish tip
We use collections for multiple things:

[ 我们使用集合来完成多项任务]
- Creating a group of target paths, that are of interest to other departments, e.g. mark prims that are useful for FX/layout/lighting selections (for example character vs. background). Another common thing to use them for is storing render layer selections, that then get applied in our final render USD file.

    [ 创建一个目标路径组，这些路径对其他部门来说很重要，例如 标记对 FX/layout/lighting 有用的 prims（例如 角色与背景）. 使用它们的另一个常见目的是存储渲染层选择，然后将其应用于我们最终的渲染USD文件中]
- Faster navigation of hierarchies by isolating to collections that interest us.

    [ 通过隔离我们感兴趣的集合，更快速地浏览层次结构]
- As collections can contain other collections, they are a powerful mechanism of forwarding and aggregating selections.

    [ 由于集合可以包含其他集合，因此它们是转发和聚合选择的强大机制]
~~~

## Resources [资源]<a name="resources"></a>
- [Usd.CollectionAPI](https://openusd.org/release/api/class_usd_collection_a_p_i.html)
- [UsdUtils Collection Convenience Functions](https://openusd.org/dev/api/authoring_8h.html#ad2939a973bd544ff30e4828ff09765db)


## Overview [概述]<a name="overview"></a>
Collections are made up of relationships and attributes:

[ 集合由关系和属性组成]

- `collection:<collectionName>:includes` relationship: A list of target `Sdf.Path`s to include, we can als target other collections.

    [ collection:\<collectionName\>:includes 关系：包含以 Sdf.Path 为目标的列表，我们也可以定位其他集合]
- `collection:<collectionName>:excludes` relationship: A list of target `Sdf.Path`s to exclude. These must be below the include paths. Excluding another collection does not work.

    [ collection:\<collectionName\>:excludes 关系：排除以 Sdf.Path 为目标的列表. 这些必须位于包含路径下方. 排除另一个集合是行不通的]
- `collection:<collectionName>:expansionRule`attribute: This controls how collections are expanded:

    [ collection:\<collectionName\>:expansionRule 属性：控制集合的扩展方式]
    - `explicitOnly`: Do not expand to any child prims, instead just do an explicit diff between include and exclude paths. This is like a Python `set().difference()`.

        [ explicitOnly ：不要扩展到任何子 prims，而只是在包含和排除路径之间进行显式比较. 这就像 Python set().difference()]
    - `expandPrims`: Expand the include paths to all children and subtract the exclude paths.

        [ expandPrims ：将包含路径扩展到所有子项并减去排除路径]
    - `expandPrimsAndProperties`: Same as `expandPrims`, but expand properties too. (Not used by anything at the moment).

        [ expandPrimsAndProperties ：与 expandPrims 相同，但也扩展属性（目前没有被任何东西使用）]
- (Optional) `collection:<collectionName>:includeRoot` attribute: When using `expandPrims`/`expandPrimsAndProperties` this bool attribute enables the includes to target the `/` pseudo root prim.

    [ （可选） collection:\<collectionName\>:includeRoot 属性：当使用 expandPrims / expandPrimsAndProperties 时，此布尔属性使包含能够定位 / 伪根 prim。]

~~~admonish danger title="Collection Size"
Make sure that you write your collections as sparsely as possible, as otherwise they can take a long time to combine when stitching multiple files, when writing per frame USD files.

[ 确保您编写的集合尽可能稀疏，否则在拼接多个文件时以及写入每帧 USD 文件时，它们可能需要很长时间才能组合]
~~~

### Creating & querying collections <a name="collectionQuery"></a>
We interact with them via the `Usd.CollectionAPI` class [API Docs](https://openusd.org/release/api/class_usd_collection_a_p_i.html). The collection api is a multi-apply API schema, so we can add multiple collections to any prim.

[ 我们通过 Usd.CollectionAPI [API 文档](https://openusd.org/release/api/class_usd_collection_a_p_i.html)）与它们进行交互. 集合 API 是一种多应用 API 架构，因此我们可以将多个集合添加到任何 prim 上]

Here are the UsdUtils.ComputeCollectionIncludesAndExcludes [API docs](https://openusd.org/dev/api/authoring_8h.html#ad2939a973bd544ff30e4828ff09765db).

[ 以下是 UsdUtils.ComputeCollectionIncludesAndExcludes [API 文档](https://openusd.org/dev/api/authoring_8h.html#ad2939a973bd544ff30e4828ff09765db)]

~~~admonish tip title=""
```python
{{#include ../../../../code/core/elements.py:collectionOverview}}
```
~~~

### Inverting a collection <a name="collectionInvert"></a>
When we want to isolate a certain part of the hierarchy (for example to pick what to render), a typical thing to do, is to give users a "render" collection which then gets applied by setting all prims not included to be [inactive](./prim.md#active). Here is an example of how to iterate a stage by pruning (skipping the child traversal) and deactivating anything that is not in the specific collection.

[ 当我们想要隔离层级结构中的某个部分（例如 选择哪些内容进行渲染）时，一个典型的做法是给用户提供一个“渲染”集合，然后通过设置所有未包含的 prim 为不激活状态来实现应用. 下面是一个如何遍历 stage 并通过修剪（跳过子遍历）来停用不在特定集合中的任何基本体的示例]

This is very fast and "sparse" as we don't edit leaf prims, instead we find the highest parent and deactivate it, if no children are part of the target collection.

[ 这样做非常快速且“稀疏”，因为我们不编辑 leaf prims，而是找到最高的父级 prim，如果它的子级 prim 中没有属于目标集合的，就将其停用]

~~~admonish tip title=""
```python
{{#include ../../../../code/core/elements.py:collectionInvert}}
```
~~~
