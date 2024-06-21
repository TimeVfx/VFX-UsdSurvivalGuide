# Frustum Culling [视锥剔除]
USD also ships with 3d related classes in the `Gf` module. These allow us to also do bounding box intersection queries.

[ USD 还在 Gf 模块中附带了 3d 相关类. 这些还允许我们进行边界框交叉查询]

We also have a frustum class available to us, which makes implementing frustum culling quite easy! The below code is a great exercise that combines using numpy, the core USD math modules, cameras and time samples. We recommend studying it as it is a great learning resource.

[ 我们还有一个可用的视锥类，这使得实现视锥体剔除变得非常容易！下面的代码是一个很好的练习，结合了 numpy、核心 USD 数学模块、相机和时间样本的使用. 我们建议您学习它，因为它是一个很好的学习资源]

The code also works on "normal" non point instancer boundable prims.

[ 该代码也适用于“正常”非点实例器可绑定prims]

You can find all the .hip files of our shown examples in our [USD Survival Guide - GitHub Repo](https://github.com/LucaScheller/VFX-UsdSurvivalGuide/tree/main/files/dcc/houdini).

[ 您可以在我们的 [USD Survival Guide - GitHub Repo](https://github.com/LucaScheller/VFX-UsdSurvivalGuide/tree/main/files/dcc/houdini) 中找到我们所示示例的所有 .hip 文件]

<video width="100%" height="100%" controls autoplay muted loop>
  <source src="./media/frustumCullingPointIInstancer.mp4" type="video/mp4" alt="Houdini Frustum Culling">
</video>

If you look closely you'll notice that the python LOP node does not cause any time dependencies. This is where the power of USD really shines, as we can sample the full animation range at once. It also allows us to average the culling data.

[ 如果仔细观察，您会发现 python LOP 节点不会导致任何时间依赖性. 这就是美元真正发挥作用的地方，因为我们可以立即采样完整的动画范围. 它还允许我们对剔除数据进行平均]

For around 10 000 instances and 100 frames, this takes around 2 seconds.

[ 对于大约 10000 个实例和 100 个帧，这大约需要 2 秒]

Here is the code shown in the video.

[ 这是视频中显示的代码]

~~~admonish tip title=""
```python
{{#include ../../../../../code/dcc/houdini.py:houdiniFrustumCulling}}
```
~~~