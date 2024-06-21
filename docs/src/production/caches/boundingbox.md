# Bounding Box Queries <a name="boundingbox"></a>
Whenever we change our geometry data, we have to update our "extents" attribute on [boundable prims](https://openusd.org/dev/api/class_usd_geom_boundable.html). The bbox cache allows us to efficiently query bounding boxes. The result is always returned in the form of a `Gf.BBox3d` object.

[ 每当我们更改几何数据时，我们都必须更新 [prim 边框](https://openusd.org/dev/api/class_usd_geom_boundable.html) 上的 “extents” 属性. bbox 缓存允许我们有效地查询边界框. 结果始终以 Gf.BBox3d 对象的形式返回]

For some production related examples, check out our [Frustum Culling](../../dcc/houdini/fx/frustumCulling.md) and [Particles](../../dcc/houdini/fx/particles.md) sections.

[ 对于一些与生产相关的示例，请查看我们的 [视锥裁剪](../../dcc/houdini/fx/frustumCulling.md) 和 [粒子](../../dcc/houdini/fx/particles.md) 部分]

Checkout the [official API docs](https://openusd.org/dev/api/class_usd_geom_b_box_cache.html) for more info.

[ 查看 [官方 API 文档](https://openusd.org/dev/api/class_usd_geom_b_box_cache.html) 以获取更多信息]

The "extents" attribute is managed via the `UsdGeom.Boundable` schema, you can find the [docs](https://openusd.org/dev/api/class_usd_geom_boundable.html) here. This has to be set per boundable prim.

[ “extents” 属性通过 UsdGeom.Boundable 架构进行管理，您可以在此处找到 [文档](https://openusd.org/dev/api/class_usd_geom_boundable.html). 这根据绑定的 prim 设置]

The "extensHint" attribute is managed via the `UsdGeom.ModeAPI`, you can find the [docs](https://openusd.org/dev/api/class_usd_geom_model_a_p_i.html) here. This can be used to accelerate lookups, by not looking into the child-hierarchy. We typically write it on prims that load payloads, to have extent data when the payload is unloaded.

[ “extensHint” 属性通过 UsdGeom.ModeAPI 进行管理，您可以在此处找到 [文档](https://openusd.org/dev/api/class_usd_geom_model_a_p_i.html). 这可以用来加速查找，而不需要查看子层级结构. 我们通常将其写入加载 payloads 的prims上，以便在卸载 payloads 时获得扩展数据]

~~~admonish info title=""
```python
{{#include ../../../../code/production/caches.py:stageQueryBBox}}
```
~~~