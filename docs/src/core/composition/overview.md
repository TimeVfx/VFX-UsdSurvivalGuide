# Composition (Combining layers) 组合(合并层)
Composition is the "art" of combining layers in USD. (Yes "art" as everyone does it a bit differently 😉)

[ 合成是 USD 中“艺术化”地组合图层的技巧.(没错，就是“艺术化”，因为每个人做的方式都会有所不同😉)]

Composition requires some base knowledge that we cover in our fundamentals section. You can also skip it, as it is a bit of a deep dive, but we highly recommend it, as the subsequent pages refer to it quite often.

[ 合成需要一些我们在基础部分中介绍的基础知识. 你也可以跳过这部分，因为它有点深入，但我们强烈推荐你阅读，因为后面的页面经常会引用到这部分内容]

- [Composition Fundamentals](./fundamentals.md): Here we cover the basic terminology and principles behind composition.

    [ [Composition Fundamentals](./fundamentals.md)：介绍合成背后的基本术语和规则]
- [Composition Arcs](./arcs.md): Here we look at how to create composition arcs via code.

    [ [Composition Arcs](./arcs.md)：探讨如何通过代码创建合成弧]
- [Composition Strength Ordering (LIVRPS)](./livrps.md): Here we discuss each arc's usage and give tips and tricks on how to best utilize it in production.

    [ [Composition Strength Ordering (LIVRPS)](./livrps.md)：讨论每个合成弧的用法，并提供在实际生产中如何最好地利用这些弧线的技巧和建议]
- [List Editable Ops](./listeditableops.md): Here we look at list editable ops. These are used to give every arc a specific load order. We also take a look at other aspects of USD that use these.

    [ [List Editable Ops](./listeditableops.md)：研究列表的编辑操作. 这些操作用于为每个合成弧指定特定的加载顺序. 还会研究在 USD 中其他层面上使用这些操作]
- [Inspecting Composition (Prim Cache Population (PCP))](./pcp.md): Here we take a look at how to debug and inspect composition.

    [ [Inspecting Composition (Prim Cache Population (PCP))](./pcp.md)：研究如何调试和检查合成]

This is probably USD's most dreaded topic, as it is also the most complex. But don't worry, we'll start slow and explain everything with a lot of examples, so that you'll feel comfortable with the topic in no time!

[ 这可能是 USD 中最令人畏惧的主题，因为它也是最复杂的. 但不用担心，我们会从简单的开始，并用大量的例子来解释一切，这样你就能很快对这个主题感到得心应手]