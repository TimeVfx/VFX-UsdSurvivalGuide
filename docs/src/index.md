![](../UsdSurvivalGuide.png#center)

# Usd Survival Guide [ Usd帮助指南 ]

[![Deploy Documentation to GitHub Pages](https://github.com/LucaScheller/VFX-UsdAssetResolver/actions/workflows/mdbook.yml/badge.svg)](https://github.com/LucaScheller/VFX-UsdSurvivalGuide/actions/workflows/mdbook.yml)

This repository aims to be a practical onboarding guide to [USD](https://openusd.org/release/index.html) for software developers and pipeline TDs.
For more information what makes this guide unique, see the [motivation](./introduction/motivation.md) section.

[ 该存储库旨在为软件开发人员和流程TD 提供实用的 USD 入门指南。关于本指南更多的独特之处信息，请参阅 [序言](./introduction/motivation.md) 部分]

```admonish success title="Siggraph Presentation"
This guide was officially introduced at [Siggraph 2023 - Houdini Hive](https://www.sidefx.com/houdini-hive/siggraph-2023/#usd). Special thanks to SideFX for hosting me and throwing such a cool Houdini lounge and presentation line up!

<iframe width="100%" height="540px" src="https://www.youtube-nocookie.com/embed/VSE7ayrRStg?si=Dnp1ozZH_fOwjn6O" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>
```

{{#include contributors.md}}

## License [ 授权许可 ]
This guide is licensed under the **Apache License 2.0**. For more information as to how this affects copyright and distribution see our [license](https://github.com/LucaScheller/VFX-UsdSurvivalGuide/blob/main/LICENSE) page.

[ 本指南是 Apache License 2.0 许可。有关版权和分发的更多信息，请参阅 [授权许可](https://github.com/LucaScheller/VFX-UsdSurvivalGuide/blob/main/LICENSE) ]

## Prerequisites [ 前期准备 ]
Before starting our journey, make sure you are all packed:

[ 在我们开始学习之前，请确保您已准备好 ]
- It is expected to have a general notion of what Usd is. If this is your first contact with the topic, check out the links in [motivation](./introduction/motivation.md). The introduction resources there take 1-2 hours to consume for a basic understanding and will help with understanding this guide.

    [ 确保对USD有一个基本的了解. 如果这是你第一次接触此课题,请先查看 [序言](./introduction/motivation.md) 来帮助理解本指南, 大概需要1-2个小时能获得一个初步的了解]
- A background in VFX industry. Though not strictly necessary, it helps to know the general vocabulary of the industry to understand this guide. This guide aims to stay away from Usd specific vocabulary, where possible, to make it more accessible to the general VFX community.

    [ 有视觉特效行业背景. 虽然不是必须的, 但了解行业的专有名词有助于帮助理解本指南. 本指南尽可能少用USD特定的词汇, 来保证大多数的视觉特效人员更易理解]
- Motivation to learn new stuff. Don't worry to much about all the custom Usd terminology being thrown at you at the beginning, you'll pick it up it no time once you start working with Usd!

    [ 保持学习新事物的热情. 不用担心一开始就向您介绍的USD专业术语,当您开始使用USD，您很快就会学会它]

## Next Steps [ 下一章 ]
To get started, let's head over to the [Core Elements](./core/overview.md) section!

[ 首先, 让我们转到 [核心](./core/overview.md) 部分]

```admonish tip
This guide primarly uses [Houdini](https://www.sidefx.com/) to explain concepts, as it is one of the most accessible and easiest ways to learn the ways of Usd with. You can install a non-commercial version for free from their website. It is highly recommended to use it when following this guide.

[ 本指南主要使用 [Houdini](https://www.sidefx.com/) 来解释概念，因为它是学习 Usd 的最容易、最简单的方法之一. 您可以从他们的网站免费安装非商业版本. 强烈建议在遵循本指南时使用它]
```