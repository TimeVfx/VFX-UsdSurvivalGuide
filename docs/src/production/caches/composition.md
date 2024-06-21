# Composition Query
Next let's look at prim composition queries. Instead of having to filter the prim index ourselves, we can use the `Usd.PrimCompositionQuery` to do it for us. For more info check out our [Inspecting Composition](../../core/composition/pcp.md) section.

[ 接下来让我们看看 prim 组合查询. 我们可以使用 Usd.PrimCompositionQuery 来帮我们过滤 prim 索引，而不必自己过滤 prim 索引. 欲了解更多信息，请查看我们的 [检查成分](../../core/composition/pcp.md) 部分]

The query works by specifying a filter and then calling `GetCompositionArcs`.

[ 该查询通过指定过滤器然后调用 GetCompositionArcs 来工作]

USD provides these convenience filters, it returns a new `Usd.PrimCompositionQuery` instance with the filter applied:

[ USD 提供了这些方便的过滤器，它返回一个应用了过滤器的新 Usd.PrimCompositionQuery 实例]

- `Usd.PrimCompositionQuery.GetDirectInherits(prim)`: Returns all non ancestral inherit arcs

    [ Usd.PrimCompositionQuery.GetDirectInherits(prim) ：返回所有非祖先继承弧]
- `Usd.PrimCompositionQuery.GetDirectReferences(prim)`: Returns all non ancestral reference arcs

    [ Usd.PrimCompositionQuery.GetDirectReferences(prim) ：返回所有非祖先参考弧]
- `Usd.PrimCompositionQuery.GetDirectRootLayerArcs(prim)`: Returns arcs that were defined in the active layer stack.

    [ Usd.PrimCompositionQuery.GetDirectRootLayerArcs(prim) ：返回在激活层堆栈中定义的弧]

These are the sub-filters that can be set. We can only set a single token value per filter:

[ 这些是可以设置的子过滤器. 我们只能为每个过滤器设置一个标记值]

- **ArcTypeFilter**: Filter based on different arc(s).

    [ ArcTypeFilter：根据不同的弧进行过滤]
- **DependencyTypeFilter**: Filter based on if the arc was introduced on a parent prim or on the prim itself.

    [ DependencyTypeFilter：根据弧是在父 prim 上还是在 prim 本身上引入进行过滤]
- **ArcIntroducedFilter**: Filter based on where the arc was introduced.

    [ ArcIntroducedFilter：根据引入弧的位置进行过滤]
- **HasSpecsFilter**: Filter based if the arc has any specs (For example an inherit might not find any in the active layer stack)

    [ HasSpecsFilter：如果弧具有任何 specs，则基于过滤器（例如，继承可能在激活层堆栈中找不到任何 specs）]

~~~admonish tip title=""
```python
{{#include ../../../../code/core/composition.py:pcpPrimCompositionQuery}}
```
~~~

The returned filtered `Usd.CompositionArc` objects, allow us to inspect various things about the arc. You can find more info in the [API docs](https://openusd.org/dev/api/class_usd_prim_composition_query_arc.html)

[ 返回的过滤后的 Usd.CompositionArc 对象使我们能够检查有关弧的各种内容. 您可以在 [API 文档](https://openusd.org/dev/api/class_usd_prim_composition_query_arc.html) 中找到更多信息]
