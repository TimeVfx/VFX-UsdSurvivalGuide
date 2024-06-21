# Material Binding <a name="materialBinding"></a>
~~~admonish question title="Still under construction!"
This sub-section is still under development, we'll add more advanced binding lookups in the near future!

[ 该子部分仍在开发中，我们将在不久的将来添加更高级的绑定查找]
~~~

Looking up material bindings is as simple as running `materials, relationships = UsdShade.MaterialBindingAPI.ComputeBoundMaterials([<list of prims>]`.
This gives us the bound material as a `UsdShade.Material` object and the relationship that bound it.
That means if the binding came from a parent prim, we'll get the `material:binding` relationship from the parent.

[ 查找材质绑定就像运行 materials, relationships = UsdShade.MaterialBindingAPI.ComputeBoundMaterials([\<list of prims\>]) 一样简单. 这为我们提供了作为 UsdShade.Material 对象的绑定材料以及绑定它的关系. 这意味着如果绑定来自父 prim，我们将从父项获得 material:binding 关系]

~~~admonish info title=""
```python
{{#include ../../../../code/production/caches.py:stageQueryMaterialBinding}}
```
~~~