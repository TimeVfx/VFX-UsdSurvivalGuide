# Attribute Queries <a name="attribute"></a>
When we query attribute via `Usd.Attribute.Get()`, the value source resolution is re-evaluated on every call.
This can be cached by using `Usd.AttributeQuery(<prim>, <attributeName>)`. This small change can already bring a big performance boost, when looking up a large time sample count. In our example below, it doubled the speed.

[ 当我们通过 Usd.Attribute.Get() 查询属性时，每次调用时都会重新评估值源解析. 可以使用 Usd.AttributeQuery(\<prim\>, \<attributeName\>) 对其进行缓存. 当查找大量时间样本时，这个小小的改变已经可以带来很大的性能提升. 在下面的示例中，速度提高了一倍]

For value clips the speed increase is not as high, as the value source can vary between clips.

[ 对于值剪辑，速度提升并不那么高，因为值源可能因剪辑而异]

The `Usd.AttributeQuery` object has a very similar signature to the `Usd.Attribute`. We can also get the time sample count, bracketing time samples and time samples within an interval.
For more information, check out the [official API docs](https://openusd.org/dev/api/class_usd_attribute_query.html).

[ Usd.AttributeQuery 对象具有与 Usd.Attribute 非常相似的签名. 我们还可以获得时间样本计数、包围时间样本和某个时间间隔内的时间样本. 有关更多信息，请查看 [官方 API 文档](https://openusd.org/dev/api/class_usd_attribute_query.html)]

~~~admonish info title=""
```python
{{#include ../../../../code/production/caches.py:stageQueryAttribute}}
```
~~~

# Primvars Queries <a name="primvars"></a>
As mentioned in our [properties](../../core/elements/property.md#reading-inherited-primvars) section, we couldn't get the native `FindIncrementallyInheritablePrimvars` primvars API method to work correctly. That's why we implemented it ourselves here, which should be nearly as fast, as we are not doing any calls into parent prims and tracking the inheritance ourselves.

[ 正如我们的属性部分中提到的，我们无法使本机 FindIncrementallyInheritablePrimvars primvars API 方法正常工作. 这就是为什么我们在这里自己实现它，这应该几乎一样快，因为我们没有对父 prim 进行任何调用并自己跟踪继承]

It's also a great example of when to use `.PreAndPostVisit()` prim range iterators.

[ 这也是何时使用 .PreAndPostVisit() prim range 迭代器的一个很好的例子]

~~~admonish info title=""
```python
{{#include ../../../../code/production/caches.py:stageQueryInheritedPrimvars}}
```
~~~