# 'Are we ready for production'? Here is a preflight checklist for your USD pipeline

[ “我们准备好投入生产了吗？”以下是您USD流程的预检清单]

Now that you know the basics (if you did your homework 😉), let's make sure you are ready for your first test flight.

[ 现在您已经了解了基础知识（如果您做了功课😉），让我们确保您已为第一次试飞做好准备]

Below is a checklist that you can use to test if everything is ready to go.

[ 下面是一个清单，您可以用它来测试一切是否准备就绪]

~~~admonish warning title=""
You can never prepare a 100%, some times you gotta run before you walk as the experience (and pressure) from an actual project running on USD will be more valuable than any RnD. So make sure you have kept the points below in mind to at least some degree.

[ 你永远不可能 100% 做好准备，有时你必须先跑再走，因为在 USD 上运行的实际项目的经验（和压力）将比任何 RnD 更有价值. 因此，请确保您至少在某种程度上牢记以下几点]
~~~

Vocabulary:

[ 词汇]
- Usd comes with a whole lot of new words, as a software developer you'll get used to it quite quickly, but don't forget about your users. Having an onboarding for vocab is definitely worth it, otherwise everyone speaks a different language which can cause a lot of communication overhead.

    [ USD 带来了很多新词，作为软件开发者，你会很快习惯的，但不要忘了你的用户. 为新词汇提供入门培训绝对值得，否则每个人都说不同的语言，会造成大量的沟通成本]

Plugins (Covered in our [plugins](../core/plugins/overview.md) section):

[ 插件（在我们的[插件](../core/plugins/overview.md)部分有介绍）]
- **Kinds** (Optional, Recommended): All you need for this one, is a simple .json file that you put in your `PXR_PLUGINPATH_NAME` search path. 

    [ Kinds（可选，推荐）：您需要的只是一个简单的 .json 文件，将其放入 PXR_PLUGINPATH_NAME 搜索路径中]
- **Schemas** (Optional, Recommended): There are two flavours of creating custom schemas: Codeless (only needs a `schema.usda` + `plugInfo.json` file) and compiled schemas (Needs compilation, but gives your software devs a better UX). If you don't have the resources for a C++ developer, codeless schemas are the way to go and more than enough to get you started.

    [ Schemas（可选，推荐）：创建自定义 Schemas 有两种风格：无代码（仅需要 schema.usda + plugInfo.json 文件）和编译模式（需要编译，但为您的软件开发人员提供了帮助）更好的用户体验）. 如果您没有 C++ 开发人员的资源，无代码模式是可行的方法，并且足以帮助您入门]
- **Asset Resolver** (Mandatory): You unfortunately can't get around not using one, luckily we got you covered with our production ready asset resolvers over in our [VFX-UsdAssetResolver GitHub Repo](https://github.com/LucaScheller/VFX-UsdAssetResolver).

    [ Asset Resolver（强制）：不幸的是，您无法避免不使用资产解析器，幸运的是，我们在 [VFX-UsdAssetResolver GitHub Repo](https://github.com/LucaScheller/VFX-UsdAssetResolver). 存储库中为您提供了可用于生产的资产解析器]

Data IO and Data Flow:

[ 数据IO和数据流]
- As a pipeline/software developer the core thing that has to work is data IO. This is something a user should never have to think about. What does this mean for you:

    [ 作为流程/软件开发人员的核心工作是数据 IO .这是用户永远不应该考虑的事情.这对您意味着什么]
    - Make sure your UX experience isn't to far from what artists already know.

        [ 确保您的用户体验不要过于偏离艺术家们已经熟悉的内容]
- Make sure that your system of tracking layers and how your assets/shots are structured is solid enough to handle these cases:

    [ 确保您的跟踪层系统以及资产/镜头的结构足够可靠，能够处理以下情况]
    - Assets with different layers (model/material/fx/lighting)

        [ 资产有许多不同的层资源 (model/material/fx/lighting)]
    - FX (Asset and Shot FX, also make sure that you can also track non USD dependencies, like .bgeo, via metadata/other means)

        [ 特效（资产特效和镜头特效，同时确保你也能通过元数据/其他方式追踪非 USD 依赖项，如.bgeo文件）]
    - Assemblies (Assets that reference other assets)

        [ 组件（引用其他资产的资产）]
    - Multi-Shot workflows (Optional)

        [ 多镜头工作流程（可选）]
    - Re-times (Technically these are not possible via USD (at least over a whole layer stack), so be aware of the restrictions and communicate these!)

        [ 时间重定向（从技术上讲，通过USD（至少是整个图层堆栈）实现这一点是不可能的）所以要了解这些限制并进行沟通！）]
- It is very likely that you have to adjust certain aspects of how you handle composition at some point. In our [composition](../core/composition/overview.md) section we cover composition from an abstract implementation viewpoint, that should help keep your pipeline flexible down the line. It is one of the ways how you can be prepared for future eventualities, it does add a level of complexity though to your setups (pipeline wise, users should not have to worry about this).

    [ 您很有可能某个时候必须要重新调整层合成. 在 [合成](../core/composition/overview.md) 部分中我们从抽象实现的角度介绍了合成，这有助于让你的流程保持灵活性. 这是为未来可能发生的情况做好准备的方法之一，尽管这确实为你的设置增加了一层复杂性（从流程角度来看，用户无需担心这一点）]

