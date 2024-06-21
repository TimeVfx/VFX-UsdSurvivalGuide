# Collection Membership
Let's have a look at how we can query if a prim path is in a collection. For more info about how collections work, check out our [Collections](../../core/elements/collection.md) section.

[ 让我们看看如何查询 prim 路径是否在集合中. 有关集合如何工作的更多信息，请查看我们的 [集合](../../core/elements/collection.md) 部分]

### Creating & querying collections <a name="collectionQuery"></a>
We interact with collectins via the `Usd.CollectionAPI` class [API Docs](https://openusd.org/release/api/class_usd_collection_a_p_i.html). The collection api is a multi-apply API schema, so we can add multiple collections to any prim. We can them access them via the collection API. The `UsdUtils` module also offers some useful functions to recompute collections so that they don't consume to much disk storage.

[ 我们通过 Usd.CollectionAPI [API文档](https://openusd.org/release/api/class_usd_collection_a_p_i.html) 与 collectins 交互. 集合 API 是一个多应用 API 模式，因此我们可以向任何 prim 添加多个集合. 我们可以让他们通过集合 API 访问它们. UsdUtils 模块还提供了一些有用的函数来重新计算集合，以便它们不会消耗太多磁盘存储]

Here are the UsdUtils.ComputeCollectionIncludesAndExcludes [API docs](https://openusd.org/dev/api/authoring_8h.html#ad2939a973bd544ff30e4828ff09765db).

[ 以下是 UsdUtils.ComputeCollectionIncludesAndExcludes [API 文档](https://openusd.org/dev/api/authoring_8h.html#ad2939a973bd544ff30e4828ff09765db) ]

~~~admonish tip title=""
```python
{{#include ../../../../code/core/elements.py:collectionOverview}}
```
~~~

### Inverting a collection <a name="collectionInvert"></a>
When we want to isolate a certain part of the hierarchy (for example to pick what to render), a typical thing to do, is to give users a "render" collection which then gets applied by setting all prims not included to be [inactive](./prim.md#active). Here is an example of how to iterate a stage by pruning (skipping the child traversal) and deactivating anything that is not in the specific collection.

[ 当我们想要隔离层级结构的某个部分（例如选择要渲染的内容）时，典型的做法是为用户提供一个“渲染”集合，然后通过将所有未包含的 prim 设置为非活动状态来应用该集合. 下面是如何通过修剪（跳过子遍历）并停用不在特定集合中的任何内容来迭代 stage 的示例]

This is very fast and "sparse" as we don't edit leaf prims, instead we find the highest parent and deactivate it, if no children are part of the target collection.

[ 这样做非常快速且“稀疏”，因为我们不编辑叶子基本体，而是找到最高的父级，如果其没有子级属于目标集合，则将其停用]

~~~admonish tip title=""
```python
{{#include ../../../../code/core/elements.py:collectionInvert}}
```
~~~
