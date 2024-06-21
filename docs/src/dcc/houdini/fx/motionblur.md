# Motion Blur [运动模糊]
Motion blur is computed by the hydra delegate of your choice using either the interpolated position data(deformation/xforms) or by making use of velocity/acceleration data.

[ 运动模糊由您选择的 Hydra 委托使用插值位置数据（deformation/xforms）或使用 velocity/acceleration 数据来计算]

You can find all the .hip files of our shown examples in our [USD Survival Guide - GitHub Repo](https://github.com/LucaScheller/VFX-UsdSurvivalGuide/tree/main/files/dcc/houdini).

[ 您可以在我们的 [USD Survival Guide - GitHub Repo](https://github.com/LucaScheller/VFX-UsdSurvivalGuide/tree/main/files/dcc/houdini) 中找到我们所示示例的所有 .hip 文件]

As noted in our [Motion Blur - Computing Velocities and Accelerations](../../../core/elements/animation.md#animationMotionVelocityAcceleration),
we can also easily derive the velocity and acceleration data from our position data, if the point count doesn't change.

[ 正如我们的 [Motion Blur - Computing Velocities and Accelerations](../../../core/elements/animation.md#animationMotionVelocityAcceleration) 中所述，如果点数不改变，我们还可以轻松地从位置数据导出速度和加速度数据]

![Houdini Motion Data Compute](media/motionblurDeformingVelocityAcceleration.png)

~~~admonish warning
Depending on the delegate, you will likely have to set specific primvars that control the sample rate of the position/acceleration data.

[ 根据委托，您可能必须设置特定的 primvar 来控制  position/acceleration 数据的采样率]
~~~

We can also easily derive velocities/accelerations from position data, if our point count doesn't change:

[ 如果我们的点数不改变，我们还可以轻松地从位置数据导出 velocities/accelerations]
~~~admonish tip title="Motionblur | Compute | Velocity/Acceleration | Click to expand" collapsible=true
```python
{{#include ../../../../../code/core/elements.py:animationMotionVelocityAcceleration}}
```
~~~