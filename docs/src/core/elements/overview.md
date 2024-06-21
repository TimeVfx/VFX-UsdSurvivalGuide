# Elements [基本单元]
In this sub-section we have a look at the basic building blocks of Usd.

[ 本小节中，我们将了解 USD 的基本组成单元]

Our approach is incrementally going from the smallest building blocks to the larger ones (except for metadata we squeeze that in later as a deep dive), so the recommended order to work through is as follows:

[ 我们的方法是从最小的组成单元逐步过渡到较大的组成单元（元数据除外，我们稍后将进行深入研究）, 因此建议的顺序如下]

- [Paths](./path.md)
- [Data Containers (Prims & Properties)](./data_container.md)
    - [Prims](./prim.md)
    - [Properties (Attributes/Relationships)](./property.md)
- [Data Types](./data_type.md)
- [Schemas ('Classes' in OOP terminology)](./schemas.md)
- [Metadata](./metadata.md)
- [Layers & Stages (Containers of actual data)](./layer.md)
- [Traversing/Loading Data (Purpose/Visibility/Activation/Population)](./loading_mechanisms.md)
- [Animation/Time Varying Data](./animation.md)
- [Materials](./materials.md)
- [Transforms](./transform.md)
- [Collections](./collection.md)
- [Notices/Event Listeners](./notice.md)
- [Standalone Utilities](./standalone_utilities.md)

This will introduce you to the core classes you'll be using the most and then increasing the complexity step by step to see how they work together with the rest of the API.

[ 本章将向您介绍最常使用的核心类, 然后逐步增加复杂性, 以了解它们如何与 API 的其余部分一起工作]

~~~admonish tip title="Vocabulary/Terminology"
We try to stay terminology agnostic as much as we can, but some vocab you just have to learn to use USd. We compiled a small [cheat sheet](../glossary.md) here, that can assist you with all those weird Usd words.

[ 我们尽量保持术语的通用性，但有些词汇你必须学习才能使用USD. 我们在这里整理了一份[小抄](../glossary.md)，可以帮助你了解所有那些奇怪的USD术语]
~~~

Getting down the basic building blocks down is crucial, so take your time! In the current state the examples are a bit "dry", we'll try to make it more entertaining in the future.

[ 弄清楚基本的组成单元至关重要，所以慢慢来! 在目前的状态下，这些例子有点“枯燥”，我们将来会尽力让它变得更有趣]

Get yourself comfortable and let's get ready to roll! You'll master the principles of Usd in no time!

[ 放松身心，让我们做好准备吧！您很快就会掌握 USD 的原理]