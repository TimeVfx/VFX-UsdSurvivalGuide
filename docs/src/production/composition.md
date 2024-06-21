# A practical guide to composition [合成实用指南]
~~~admonish question title="Still under construction!"
As composition is USD's most complicated topic, this section will be enhanced with more examples in the future.
If you detect an error or have useful production examples, please [submit a ticket](https://github.com/LucaScheller/VFX-UsdSurvivalGuide/issues/new), so we can improve the guide!

[ 由于合成是 USD 最复杂的主题，因此本节将通过更多示例来增强理解. 如果您发现错误或者有更好的生产案例，[please submit a ticket](https://github.com/LucaScheller/VFX-UsdSurvivalGuide/issues/new)，以便我们改进指南！]

In the near future, we'll add examples for:

[ 在不久的将来，我们将添加以下示例]
- Best practice asset structures
- Push Vs Pull / FullPin Opt-In pipelines
~~~

~~~admonish tip
We have a supplementary Houdini scene, that you can follow along with, available in this [site's repository](https://github.com/LucaScheller/VFX-UsdSurvivalGuide/tree/main/files/composition). All the examples below will walk through this file as it easier to prototype and showcase arcs in Houdini via nodes, than writing it all in code.

[ 我们有一个补充的 Houdini 场景，您可以按照该场景进行操作，该场景可在[本站点的存储库](https://github.com/LucaScheller/VFX-UsdSurvivalGuide/tree/main/files/composition)中找到. 下面的所有示例都将遍历此文件，因为通过节点在 Houdini 中制作 prototype  和展示合成弧比将其全部写入代码更容易]
~~~

### How to approach shot composition [如何处理镜头合成]
When prototyping your shot composition, we recommend an API like approach to loading data:

[ 在设计你的镜头组合原型时，我们推荐采用类似于API的方法来加载数据]

We create a `/__CLASS__/assets` and a `/__CLASS__/shots/<layer_name>` hierarchy. All shot layers first load assets via references and shot (fx/anim/etc.) caches via payloads into their respective class hierarchy. We then inherit this into the actual "final" hierarchy. This has one huge benefit:

[ 我们创建一个 /\_\_CLASS\_\_/assets 和一个 /\_\_CLASS\_\_/shots/\<layer_name\> 层级结构. 所有 shot layers 首先通过 references load assets 并通过 payloads 将镜头（fx/anim/等）的缓存加载到各自的 class 的层级结构中. 然后我们将其继承到实际的“最终”层级结构中.这有一个巨大的好处]

The class hierarchy is a kind of "API" to your scene hierarchy. For example if we want to time shift (in USD speak layer offset) an asset that has multiple occurrences in our scene, we have a single point of control where we have to change the offset. Same goes for any other kind of edit.

[ class 是场景层级结构的 “API” 类型. 例如我们想要对场景中多次出现的资产进行时间偏移（用USD概念来说是 layer offset），我们只需在一个控制点更改偏移量即可. 对于任何其他类型的编辑也是如此]

We take a look at variations of this pattern in our [Composition Strength Ordering (LIVRPS)](../core/composition/livrps.md) section in context of different arcs, we highly recommend looking through the "Pro Tip" sections there.

[ 在 [Composition Strength Ordering (LIVRPS)](../core/composition/livrps.md) 部分中，我们针对不同合成弧研究了各种变体，强烈建议您查阅其中的“专业提示”部分]

This approach solves composition "problems": When we want to payload something over an asset reference, we can't because the payload arc is weaker than the reference arc. By "proxying" it to a class prim and then inheriting it, we guarantee that it always has the strongest opinion. This makes it easier to think about composition, as it is then just a single list-editable op rather than multiple arcs coming from different sources.

[ 这种方法解决了组合“问题”：当我们想要通过 payload 覆盖资产引用时，由于 payload 弧比 reference 弧弱，所以无法实现. 通过将其 “proxying” 到 class  prim，然后继承它，我们可以确保它始终具有最强的优先级. 这使得思考组合变得更加简单，因为它只是一个可编辑的列表操作，而不是来自不同源的多个弧]

The downside to this approach is that we (as pipeline devs) need to restructure all imports to always work this way. The cache files themselves can still write the "final" hierarchy, we just have to reference/payload it all in to the class hierarchy and then inherit it. This may sound like a lot of work, but it is actually quick to setup and definitely helps us/artists keep organized with larger scenes.

[ 这种方法的缺点是，我们作为流程开发人员需要重构所有的导入，以确保始终以这种方式工作. 缓存文件本身仍然可以写入“最终”的层次结构，我们只需要将其全部 reference/payload  到 class 层级结构中，然后继承它. 这听起来像是一项艰巨的工作，但实际上设置起来非常快，并且绝对可以帮助我们/艺术家在更大的场景中保持组织性]

It also keeps our whole setup [instanceable](../core/composition/livrps.md#compositionInstance), so that we have the best possible performance.

[ 它还使我们的整个设置保持[可实例化](../core/composition/livrps.md#compositionInstance)，以便我们拥有最佳的性能]

~~~admonish danger title="Pro Tip | Instanceable Prims"
When creating our composition structure concept, we should always try to keep everything instanceable until the very last USD file (the one that usually gets rendered). This way we ensure optimal performance and scalability.

[ 在创建我们的合成结构概念时，我们应该始终尝试保持所有内容均可实例化，直到最后的 USD 文件（通常是用于渲染的文件）这样我们才能确保最佳的性能和可扩展性]
~~~

We also show an example for this approach in our [Composition Payloads section](../core/composition/livrps.md#compositionArcPayloadLoadWorkflow) as well as bellow in the next bullet point.

[ 我们还在 [Composition Payloads section](../core/composition/livrps.md#compositionArcPayloadLoadWorkflow) 部分以及下一点中展示了这种方法的示例]

### Loading heavy caches into your shots [将大量缓存加载到您的镜头中]
When writing heavy caches, we usually write per frame/chunk files and load them via value clips. Let's have a look how to best do this:
As a best practice we always want to keep everything [instanceable](../core/composition/livrps.md#compositionInstance), so let's keep that in mind when loading in the data.

[ 当写入大量缓存时，我们通常写入每帧/块文件并通过值剪辑加载它们. 让我们看看如何最好地做到这一点：作为最佳实践，我们始终希望保持所有内容[可实例化](../core/composition/livrps.md#compositionInstance)，所以在加载数据时请记住这一点]

~~~admonish tip title="Pro Tip | How do we load value clipped files?"
- When making prims instanceable, the value clip metadata has to be under the instanceable prim, as the value clip metadata can't be read from outside of the instance (as it would then mean each instance could load different clips, which would defeat the purpose of instanceable prims).

  [在使 prim 可实例化时，值剪辑元数据必须位于可实例化的 prim 之下，因为无法从实例外部读取值剪辑元数据（因为如果这样做，则意味着每个实例都可以加载不同的剪辑，这将违背可实例化 prim 的目的）]
- Value clip metadata can't be inherited/internally referenced/specialized in. It must reside on the prim as a direct opinion.

  [ 值剪辑元数据无法 inherited/internally referenced/specialized. 它必须直接作为 prim 直接存在]
- We can't have data above the prims where we write our metadata. In a typical asset workflow, this means that all animation is below the asset prims (when using value clips).

  [ 我们不能在编写元数据的 prim 之上存储数据. 在典型的资产工作流中，这意味着当使用值剪辑时，所有动画都位于资产 prim 之下]
~~~

Let's have a look how we can set it up in Houdini while prototyping, in production you should do this via code, see our [animation](../core/elements/animation.md#animationValueClips) section for how to do this.

[ 让我们看看如何在原型设计时在 Houdini 中进行设置，在生产中您应该通过代码来完成此操作，请参阅[动画](../core/elements/animation.md#animationValueClips)部分以了解如何执行此操作]

<video width="100%" height="100%" controls autoplay muted loop>
  <source src="compositionValueClipAbstraction.mp4" type="video/mp4" alt="Houdini Value Clip Composition">
</video>

The simplest version is to write the value clips, stitch them and load the resulting file as a payload per asset root prim. This doesn't work though, if you have to layer over a hierarchy that already exists, for example an asset.

[ 最简单的版本是编写值剪辑，将它们拼接起来并将生成的文件作为每个资产 root prim 的 payload 进行加载. 然而如果你必须在一个已经存在的层级结构（例如一个资产）上进行分层，这种方法就不可行了]

We therefore go for the "API" like approach as discussed above: We first load the cache via a payload into a class hierarchy and then inherit it onto its final destination. This is a best-practise way of loading it. By "abstracting" the payload to class prims, we have a way to load it via any arc we want, for example we could also payload it into variants. By then inherting it to the "final" hierarchy location, we ensure that no matter what arc, the cache gets loaded. This way we can load the cache (as it has heavy data) as payloads and then ensure it gets loaded with the highest opinion (via inherits).

[ 因此，我们采用了上述讨论的类似“API”的方法：首先通过 payload 将缓存加载到 class 层级结构中，然后将其继承到最终目的地. 这是加载它的最佳实践方式. 通过将 payload “抽象化”为 calss prim，我们可以通过任何想要的弧将其加载，例如我们也可以将其 payload 到变体中. 然后通过将其继承到“最终”层级结构位置，我们确保无论使用什么弧，缓存都会被加载. 这样我们可以将缓存（因为它包含大量数据）作为 payload 加载，然后确保它以最高的优先级（通过继承）被加载]

Let's also take a look at why this doesn't work when trying to write value clips at non asset root prims:

[ 我们来看看为什么当尝试在非资产根 prims 写入值剪辑时这不起作用]

<video width="100%" height="100%" controls autoplay muted loop>
  <source src="compositionValueClipAbstractionStageRoot.mp4" type="video/mp4" alt="Houdini Value Clip Composition">
</video>

Loading a payload for a whole cache file works, the problem you then run into though is, that the inherit arcs don't see the parent prims value clip metadata. So we'd have to load the whole layer as a value clip. While this is possible, we highly don't recommend it, as it is not clear where the data source is coming from and we are making the scene structure messy by not having clear points of data loading. We also can't make it instanceable (unles we intend on making the whole hierarchy instanceable, which in shots doesn't work because we need per prim overrides (on at least asset (parent) prims)).

[ 加载整个缓存文件的 payload 是可行的，但随后你会遇到的问题是，继承弧无法看到父级 prim 的值剪辑元数据. 因此我们需要将整个层作为值剪辑加载. 虽然这是可能的，但我们强烈不推荐这样做，因为这不清楚数据源来自哪里，而且我们没有明确的数据加载点，这使得场景结构变得混乱. 我们也无法使其可实例化（除非我们打算使整个层次结构可实例化，这在镜头中是不可行的，因为我们需要每个 prim（至少资产（父级）prim）重写）]