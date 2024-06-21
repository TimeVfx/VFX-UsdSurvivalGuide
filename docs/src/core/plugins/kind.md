# Kinds
The [kind](https://openusd.org/release/glossary.html#usdglossary-kind) metadata is a special metadata entry on prims that can be written  to mark prims with data what "hierarchy level type" it is. This way we can quickly traverse and select parts of the hierarchy that are of interest to us, without traversing into every child prim. The most common types are `component` and `group`. We use these (or sub-kinds) of these to make our stage more easily browse-able, so that we can visually/programmatically easily detect where assets start.

[ [kind](https://openusd.org/release/glossary.html#usdglossary-kind) 元数据是 prims 上的特殊元数据条目，可以用它来编写数据标记 prims 的“层级结构类型”. 这样我们就可以快速遍历并选择层次结构中我们感兴趣的部分，而无需遍历每个子 prim. 最常见的类型是 component 和 group . 我们使用这些（或子类型）来使我们的 stage 更容易浏览，以便我们可以以视觉/编程方式轻松检测资产的开始位置]

# Table of Contents [目录]
1. [Kinds In-A-Nutshell](#summary)
1. [What should I use it for?](#usage)
1. [Resources](#resources)
1. [Overview](#overview)
    1. [Reading and Writing Kinds](#kindAuthoring)
    1. [Kind Registry](#kindRegistry)
    1. [Kind IsA Checks/Travesal](#kindTraversal)

## TL;DR - Kinds In-A-Nutshell [概述]<a name="summary"></a>
~~~admonish tip
The kind metadata is mainly used to for two things:

[ kind 元数据主要用于两件事]
- Traversal: You can quickly detect (and stop traversal to children) where the assets are in your hierarchy by marking them as a a `model` subtype like `component`. A typical use case would be "find me all `fx` assets": In your pipeline you would define a 'fx' model kind subtype and then you can traverse for all prims that have the 'fx' kind set.

    [ 遍历：您可以通过将资产标记为 model 子类型（如 component ）来快速检测（并停止遍历子级）资产在层级结构中的位置. 一个典型的用例是“找到所有 fx 资产”：在您的流程中，您定义一个 “fx” 子类型，然后您可以遍历所有设置了 “fx” 类型的 prim]
- DCCs use this to drive user selection in UIs. This way we can quickly select non-leaf prims that are of interest. For example to transform an asset in the hierarchy, we only want to select the asset root prim and not its children, so we can tell our DCC to select only `model` kind prims. This will limit the selection to all sub-kinds of `model`.

    [ DCC 使用它来驱动 UI 中的用户选择. 这样我们就可以快速选择感兴趣的非叶prims. 例如，要转换层级结构中的资产，我们只想选择资产 root prim 而不是其子项，因此我们可以告诉 DCC 仅选择 model 类型 prim. 这会将选择限制为 model 的所有子类型]
~~~
~~~admonish tip title="Pro Tip | Houdini Kind Icons"
If you have created custom kinds, you can place icons (png/svg) in your `$HOUDINI_PATH/config/Icons/SCENEGRAPH_kind_<name>.<ext>` folder and they will be shown in your scene graph tree panel.

[ 如果您创建了自定义类型，则可以将图标 (png/svg) 放置在 $HOUDINI_PATH/config/Icons/SCENEGRAPH_kind_\<name\>.\<ex\t> 文件夹中，它们将显示在场景图树面板中]
~~~

## What should I use it for? <a name="usage"></a>

[ 我应该用它做什么？]

~~~admonish tip
In production, you'll use kinds as a way to filter prims when looking for data. As kind data is written in prim metadata, it is very fast to access and suited for high performance queries/traversals. For more info on traversals, take a look at the [traversal]() section.

[ 在生产中，您将在查找数据时使用 kind 作为过滤 prim 的方法. 由于类型数据写入 prim 元数据中，因此访问速度非常快，适合高性能查询/遍历. 有关遍历的更多信息，请查看遍历部分]
~~~
~~~admonish important
You should always tag all prims in the hierarchy with kinds at least to the asset level. Some Usd methods as well as DCCs will otherwise not traverse into the child hierarchy of a prim if they come across a prim without a kind being set.
So this means you should have at least `group` kind metadata set for all parent prims of `model` sub-kind prims.

[ 您应该始终将层级结构中的所有 prims 标记为至少到资产级别的类型. 否则，如果某些 Usd 方法和 DCC 遇到未设置类型的 prim，它们将不会遍历 prim 的子层级结构. 因此，这意味着您应该至少为 model 子类 prims 的所有父 prims 设置 group 类元数据]
~~~

## Resources [资源]
- [Kind Glossary Definition](https://openusd.org/release/glossary.html#usdglossary-kind)
- [Extending Kinds](https://openusd.org/dev/api/kind_page_front.html#kind_extensions)
- [Kind Registry](https://openusd.org/dev/api/class_kind_registry.html) 

## Overview [概述]<a name="overview"></a>
Usd ships with these kinds by default, to register your own kinds, see the below examples for more details:

[ USD 默认情况下附带这些种类，要注册您自己的种类，请参阅以下示例了解更多详细信息]

- `model`: The base kind for all model kinds, don't use it directly.

    [ model ：所有模型类型的基础类型，不要直接使用它]
    - `group`: A group of model prims.

        [ group ：prims model 的组]
        - `assembly`: A collection of model prims, typically used for sets/a assembled collection of models or environments.

            [ assembly ：prims 模型的集合，通常用于模型或场景的集合/组装集合]
    - `component`: A sub-kind of 'model' that has no other child prims of type 'model'

        [ component ：“model” 的子类型，没有其他 “model” 类型的 child prims]
- `subcomponent`: An important subhierarchy of an component.

    [ subcomponent ：组件的重要子层级结构]


~~~admonish important
You should always tag all prims with kinds at least to the asset level in the hierarchy. Some DCCs will otherwise not traverse into the hierarchy if they come across a prim without a hierarchy.
So this means you should have `group` kinds set for all parent prims of `model` prims.

[ 您应该始终将所有 prim 的类型至少标记到层级结构中的资产级别. 否则，如果某些 DCC 遇到没有层级结构的 prim，它们将不会遍历层次结构. 因此，这意味着您应该为 model prims 的所有 parent prims 设置 group 类型]
~~~

### Reading and Writing Kinds <a name="kindAuthoring"></a>
Kinds can be easily set via the high and low level APIs:

[ 可以通过高级和低级 API 轻松设置 kind]
~~~admonish info title=""
```python
{{#include ../../../../code/core/elements.py:dataContainerPrimBasicsKinds}}
```
~~~

An example Usd file could look likes this

[ USD 文件示例如下所示]
~~~admonish info title=""
```python
def Xform "set" (
    kind = "set"
)
{
    def Xform "garage" (
        kind = "group"
    )
    {
        def Xform "bicycle" (
            kind = "prop"
        )
        {
        }
    }

    def Xform "yard" (
        kind = "group"
    )
    {
        def Xform "explosion" (
            kind = "fx"
        )
        {
        }
    }
}
```
~~~

### Creating own kinds <a name="kindPlugin"></a>

[ 创建自己的 kinds]

We can register kinds via the [plugin system](./overview.md).

[ 我们可以通过插件系统注册种类]

~~~admonish info title=""
```python
{{#include ../../../../files/plugins/kinds/plugInfo.json}}
```
~~~

To register the above kinds, copy the contents into a file called `plugInfo.json`. Then set your `PXR_PLUGINPATH_NAME` environment variable to the folder containing the `plugInfo.json` file.

[ 要注册上述类型，请将内容复制到名为 plugInfo.json 的文件中. 然后将 PXR_PLUGINPATH_NAME 环境变量设置为包含 plugInfo.json 文件的文件夹]

For Linux this can be done for the active shell as follows:

[ 对于 Linux，可以对活动 shell 执行以下操作]

```bash
export PXR_PLUGINPATH_NAME=/my/cool/plugin/resources:${PXR_PLUGINPATH_NAME}
```

If you downloaded this repo, we provide an example kind plugin [here](https://github.com/LucaScheller/VFX-UsdSurvivalGuide/tree/main/files/plugins/kinds). All you need to do is point the environment variable there and launch a USD capable application.

[ 如果您下载了此存储库，我们会在 [此处](https://github.com/LucaScheller/VFX-UsdSurvivalGuide/tree/main/files/plugins/kinds) 提供示例插件. 您所需要做的就是将环境变量指向那里并启动支持 USD 的应用程序]

### Kind Registry <a name="kindRegistry"></a>
We can also check if a plugin with kind data was registered via Python.

[ 我们还可以检查带有类型数据的插件是否是通过 Python 注册的]

~~~admonish info title=""
```python
{{#include ../../../../code/core/elements.py:kindRegistry}}
```
~~~

### Kind IsA Checks & Traversal <a name="kindTraversal"></a>
We can then also use our custom kinds for traversal checks.
Usd offers the `prim.IsModel` and `prim.IsGroup` checks as convenience methods to check if a kind is a sub-kind of the base kinds.

[ 然后我们还可以使用自定义类型进行遍历检查. Usd 提供 prim.IsModel 和 prim.IsGroup 检查作为检查某种类型是否是基本类型的子类型的便捷方法]

~~~admonish important
Make sure that your whole hierarchy has kinds defined (to the prim you want to search for), otherwise your `prim.IsModel` and `prim.IsGroup` checks will fail. This also affects how DCCs implement traversal: For example when using Houdini's LOPs selection rules with the `%kind:component` syntax, the selection rule will not traverse into the prim children and will stop at the parent prim without a kind. Manually checking via `registry.IsA(prim.GetKind(), "component")` still works though, as this does not include the parents in the check. (See examples below)
#ToDo Add icon

[ 确保您的整个层级结构已定义 kinds（针对您要搜索的 prim），否则您的 prim.IsModel 和 prim.IsGroup 检查将失败. 这也会影响 DCC 实现遍历的方式：例如，当使用 Houdini 的 LOP 选择规则和 %kind:component 语法时，选择规则不会遍历到 prim 子级，而是在没有类型的父 prim 处停止. 不过，通过 registry.IsA(prim.GetKind(), "component") 手动检查仍然有效，因为这不包括检查中的父母. （参见下面的示例）]
~~~

~~~admonish info title=""
```python
{{#include ../../../../code/core/elements.py:kindTraversal}}
```
~~~