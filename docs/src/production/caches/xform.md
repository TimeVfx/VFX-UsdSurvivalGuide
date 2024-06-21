# Xform Queries <a name="xform"></a>
As mentioned in our [transforms section](../../core/elements/transform.md), we can batch query transforms via the `UsdGeom.XformCache()`.

[ 正如我们的 [转换部分](../../core/elements/transform.md) 中提到的，我们可以通过 UsdGeom.XformCache() 批量查询转换]

It caches ancestor parent xforms, so that when we query leaf prims under the same parent hierarchy, the lookup retrieves the cached parent xforms. The cache is managed per time code, so if we use `XformCache.SetTime(Usd.TimeCode(<someOtherFrame>))`, it clears the cache and re-populates it on the next query for the new time code.

[ 它缓存祖先父 xform，以便当我们在同一父层级结构下查询叶 prim 时，查找会检索缓存的父 xform. 缓存是按时间代码管理的，因此如果我们使用 XformCache.SetTime(Usd.TimeCode(\<someOtherFrame\>)) ，它将清除缓存并在下一个查询中重新填充新时间代码]

Checkout the [official API docs](https://openusd.org/dev/api/class_usd_geom_xform_cache.html) for more info.

[ 查看 [官方 API 文档](https://openusd.org/dev/api/class_usd_geom_xform_cache.html) 以获取更多信息]

~~~admonish info title=""
```python
{{#include ../../../../code/production/caches.py:stageQueryTransform}}
```
~~~