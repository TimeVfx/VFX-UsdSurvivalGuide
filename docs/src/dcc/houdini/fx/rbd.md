# RBD (Rigid Body Dynamics) [RBD（刚体动力学）]
As mentioned in our [Point Instancer](./pointinstancers.md) section, we have four options for mapping Houdini's packed prims to USD:

[ 正如我们的 Point Instancer 部分中提到的，我们有四个选项用于将 Houdini 的打包 prims 映射到 USD]
- As transforms
- As point instancers
- As deformed geo (baking down the xform to actual geo data)
- As skeletons

Which confronts us with the question:

[ 这给我们提出了一个问题]

~~~admonish tip title="Pro Tip | How should I import my RBD(Rigid Body Dynamics) simulation?"
As mentioned in our [production FAQ](../../../production/faq.md#faqPrimCount), large prim counts cause a significant decrease in performance.
That's why we should avoid writing RBD sims as transform hierarchies and instead either go for deformed geo (not memory efficient) or skeletons 
(memory efficient).

[ 正如我们的[生产常见问题解答](../../../production/faq.md#faqPrimCount)中提到的，大量 prim 计数会导致性能显着下降. 这就是为什么我们应该避免将 RBD 模拟编写为变换层次结构，而是选择变形几何（内存效率不高）或骨架（内存效率低）]
~~~

~~~admonish danger title="Skeletons and Custom Normals"
Currently Hydra does not yet read in custom normals, when calculating skinning. This will likely be resolved in the very near future, until then we have to bake the skinning pre-render. This is very fast to do via the skeleton API, more details below.

[ 目前，Hydra 在计算蒙皮时尚未读取自定义法线. 这个问题可能会在不久的将来得到解决，在那之前我们必须烘焙蒙皮预渲染. 通过骨架 API 可以非常快地完成此操作，更多详细信息如下]
~~~

On this page we'll focus on the skeleton import, as it is the only viable solution for large hierarchy imports. It is also the most efficient one, as skeletons only store the joint xforms, just like the RBD simulation point output.

[ 在本页中，我们将重点关注骨架导入，因为它是大型层级结构导入的唯一可行的解​​决方案. 它也是最有效的一种，因为骨架只存储关节 xform，就像 RBD 模拟点输出一样]

Using skeletons does have one disadvantage: We can't select a mesh separately and hide it as skeletons deform subsets of each individual prim based on weighting.

[ 使用骨架确实有一个缺点：我们无法单独选择网格并将其隐藏，因为骨架会根据权重对每个单独的 prim 的子集进行变形]

The solution is to set the scale of the packed piece to 0. For hero simulations, where we need to have a separate hierarchy entry for every piece, we'll have to use the standard packed to xform import. We can also do a hybrid model: Simply change the hierarchy paths to your needs and import them as skeletons. This way, we can still disable meshes based on what we need, but get the benefits of skeletons.

[ 解决方案是将打包块的比例设置为 0 . 对于角色模拟，我们需要为每个块都有一个单独的层级结构条目，我们必须使用标准的打包到 xform 导入. 我们还可以制作混合模型：只需根据您的需求更改层次结构路径并将它们作为骨架导入.这样，我们仍然可以根据需要禁用网格，但可以获得骨架的好处]

You can find all the .hip files of our shown examples in our [USD Survival Guide - GitHub Repo](https://github.com/LucaScheller/VFX-UsdSurvivalGuide/tree/main/files/dcc/houdini).

[ 您可以在我们的 [USD Survival Guide - GitHub Repo](https://github.com/LucaScheller/VFX-UsdSurvivalGuide/tree/main/files/dcc/houdini) 中找到我们所示示例的所有 .hip 文件]


## Houdini Native Import via KineFX/Crowd Agents [通过 KineFX/Crowd Agents 进行 Houdini 原生导入]
Let's first take a look at what Houdini supports out of the box:

[ 我们先来看看Houdini开箱即用的支持什么]
- We can import KineFX characters and crowd agents as skeletons. This does have the overhead of creating KineFX characters/crowd agents in a way that they map to our target output hierarchy we want in LOPs. It is also harder to setup with a lot of different skeletons (Or we couldn't figure, input is always welcome).

    [ 我们可以导入 KineFX 角色和人群代理作为骨架. 这确实会产生创建 KineFX 角色/人群代理的开销，因为它们映射到我们在 LOP 中想要的目标输出层级结构. 设置许多不同的骨架也更困难（或者我们无法弄清楚，欢迎大家指出）]
- The "RBD Destruction" LOP node has a reference implementation of how to import packed prims as skeletons animations. It relies on Python for the heavy data lifting and doesn't scale well, we'll take a look at a high performance solution in the next section below, that uses Houdini's native import for the data loading and Python to map everything to its correct place.

    [“RBD Destruction”LOP 节点有一个关于如何将打包的 prims 作为骨架动画导入的参考实现.它依赖于 Python 来进行繁重的数据提升，并且扩展性不佳，我们将在下面的下一节中介绍一个高性能解决方案，该解决方案使用 Houdini 的本机导入进行数据加载，并使用 Python 将所有内容映射到正确的位置地方]

Here is a video showing both methods:

[ 这是显示这两种方法的视频]

<video width="100%" height="100%" controls autoplay muted loop>
  <source src="./media/rbdKineFXCrowdAgent.mp4" type="video/mp4" alt="Houdini Character Skeleton Import">
</video>

This works great, if we have a character like approach.

[ 如果我们有类似角色的方法，这会非常有效]

## Houdini High Performance RBD Skeleton Import [Houdini 高性能 RBD 骨架导入]

The problem is with RBD sims this is not the case: We usually have a large hierarchy consisting of assets, that we fracture, that should then be output to the same hierarchy location as the input hierarchy. As skeletons are only evaluated for child hierarchies, where we have a "SkelRoot" prim as an ancestor, our approach should be to make all asset root prims (the ones that reference/payload all the data), be converted to have the "SkelRoot" prim type. This way we can nicely attach a skeleton to each asset root, which means we can selectively payload (un)load our hierarchy as before, even though each asset root pulls the skeleton data from the same on disk cache.

[ 问题在于 RBD sims 的情况并非如此：我们通常有一个由资产组成的大型层级结构，我们将其破碎，然后应将其输出到与输入层级结构相同的位置. 由于skeletons 仅针对子层级结构评估，其中我们有一个 “SkelRoot” prim 作为祖先，因此我们的方法应该是将所有资产 root prims（ reference/payload 所有数据）转换成 SkelRoot prim type. 通过这种方式，我们可以很好地将骨架附加到每个资产根，这意味着我们可以像以前一样有选择地 payload（卸载）我们的层级结构，即使每个资产根从磁盘上同一缓存中提取数据]

Let's take a look how that works:

[ 让我们看看它是如何工作的]

<video width="100%" height="100%" controls autoplay muted loop>
  <source src="./media/rbdSkeleton.mp4" type="video/mp4" alt="Houdini Skeleton Custom Import">
</video>

Now we won't breakdown how skelton's themselves work, as this is a topic on its own. You can read up on it in the [official API docs](https://openusd.org/dev/api/_usd_skel__intro.html).

[ 现在我们不会详细介绍 skelton 本身的工作原理，因为这本身就是一个主题。您可以在 [official API docs](https://openusd.org/dev/api/_usd_skel__intro.html) 中阅读它]

What we'll do instead is break down our approach. We have three inputs from LOPs, where we are pulling data from:

[ 相反，我们要做的是分析我们的方法. 我们从 LOP 获得三个输入，从中提取数据]
- Geometry data: Here we import our meshes with:

  [ Geometry data：这里我们将导入网格]
    - their joint weighting (RBD always means a weight of 1, because we don't want to blend xforms).

        [ their joint weighting（RBD 始终意味着权重为 1，因为我们不想混合 xforms）]
    - their joint index. Skeleton animations in USD can't have time varying joint index to joint name mapping. This is designed on purpose, to speed up skeleton mesh binding calculation.

        [ their joint index. USD 格式的骨骼动画不能具有随时间变化的关节索引到关节名称的映射. 这是有意设计的，以加速骨架网格物体绑定计算]
    - their target skeleton. By doing our custom import we can custom build our skeleton hierarchy. In our example we choose to group by asset, so one skeleton per asset.

        [ their target skeleton. 通过进行自定义导入，我们可以自定义构建骨架层次结构. 在我们的示例中，我们选择按资产分组，因此每个资产有一个骨架]

- Rest data: As with a SOP transform by points workflow, we have to provide:

  [ cRest data：与 SOP 按点变换工作流程一样，我们必须提供]
    - the rest position

        [ 初始位置]
    - the bind position (We can re-use the rest position)

        [ bind position（我们可以重新使用 rest position）]
    - the joint names. We could also import them via the animation data, there is no benefit though, as joint idx to name mappings can't be animated (even though they are attributes and not relationships).

        [ the joint names. 我们还可以通过动画数据导入它们，但这没有任何好处，因为 joint idx 到名称映射无法进行动画处理（即使它们是属性而不是关系）]
- Animation data: This pulls in the actual animation data as:

    [ Animation data：这会将实际动画数据导入]
    - translations
    - rotations
    - scales (Houdini's RBD destruction .hda optionally disables animation import for scales as an performance opt-in)

After the initial import, we then "only" have to apply a few schemas and property renames. That's it, it's as simple as that!

[ 初始导入后，我们“只需”应用一些模式和属性重命名. 就是这样，就这么简单！]

So why is this a lot faster, than the other methods?

[ 那么为什么这比其他方法快很多呢？]

We use Houdini's super fast SOP import to our benefit. The whole thing that makes this work are the following two pieces of our node network:

[ 我们使用 Houdini 的超快速 SOP 导入对我们有利. 使这项工作成功的全部因素是我们节点网络的以下两个部分]

Our joint name to index mapping is driven by a simple "Enumerate" node that runs per skeleton target prim path.

[ 我们的 joint name to index mapping 由一个简单的“枚举”节点驱动，该节点在每个骨架目标 prim 路径上运行]

 <video width="100%" height="100%" controls autoplay muted loop>
  <source src="./media/rbdSkeletonJointIndexName.mp4" type="video/mp4" alt="Houdini Skeleton Joint Index Name">
</video>

Our joint xforms then create the same skeleton target prim path. We don't have to enumerate, because on LOP import our geometry is segmented based on the skeleton prim path. This slices the arrays the exact same way as the enumerate node.

[ 然后我们的 joint xforms 创建相同的骨架目标 prim 路径. 我们不必枚举，因为在 LOP 导入时，我们的几何体是根据骨架 prim 路径进行分段的. 这以与枚举节点完全相同的方式对数组进行切片]

 <video width="100%" height="100%" controls autoplay muted loop>
  <source src="./media/rbdSkeletonJointXforms.mp4" type="video/mp4" alt="Houdini Skeleton Joint Xforms">
</video>

The outcome is that we let Houdini do all the heavy data operations like mesh importing and attribute segmentation by path.
We then only have to remap all the data entries to their correct parts. As USD is smart enough to not duplicate data for renames, this is fast and memory efficient.

[ 结果是我们让 Houdini 完成所有繁重的数据操作，例如网格导入和按路径进行属性分割. 然后我们只需将所有数据条目重新映射到其正确的部分即可. 由于 USD 足够聪明，不会在重命名时重复数据，因此速度快且内存效率高]
