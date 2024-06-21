# Stage API Query Caches [Stage API 缓存查询]
When inspecting stages, we often want to query a lot of data.

[ 在检查 stages 时，我们经常想要查询大量数据]

Some types of data, like transforms/material bindings or primavars, are inherited down the hierarchy. Instead of re-querying the ancestor data for each leaf prim query, USD ships with various query classes that cache their result, so that repeated queries have faster look ups.

[ 某些类型的数据（例如变换/材质绑定或 primavar）会沿层级结构继承. USD 不需要为每个叶 prim 查询重新查询祖先数据，而是附带了各种缓存其结果的查询类，以便重复查询具有更快的查找速度]

For currently cover these query caches:

[ 目前已有这些查询缓存]
- [Xforms](./xform.md)
- [BoundingBox](./boundingbox.md)
- [Attribute/(Inherited) Primvars](./attribute.md)
- [Material Binding](./materialbinding.md)
- [Collection Membership](./collection.md)
- [Composition](./composition.md)