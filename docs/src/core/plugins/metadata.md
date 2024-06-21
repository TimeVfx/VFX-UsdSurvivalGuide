# Metadata
USD allows us to extend the base metadata that is attached to every layer, prim and property (Supported are `Sdf.Layer` and subclasses of `Sdf.Spec`). This allows us to write custom fields with a specific type, so that we don't have to rely on writing everything into the `assetInfo` and `customData` entries.

[ USD 允许我们扩展每个 layer, prim 和 property 的基本元数据（支持 Sdf.Layer 和 Sdf.Spec 的子类）. 这允许我们编写具有特定类型的自定义字段，这样我们就不必依赖将所有内容写入 assetInfo 和 customData 条目]

To get an overview of metadata in USD, check out our dedicated [metadata section](../elements/metadata.md).

[ 要了解 USD 元数据的概述，请查看 [元数据](../elements/metadata.md) 部分]

# Table of Contents [目录]
1. [Metadata Plugins In-A-Nutshell](#summary)
1. [What should I use it for?](#usage)
1. [Resources](#resources)
1. [Overview](#overview)
    1. [Installing a metadata plugin](#metadataPluginInstall)
    1. [Reading/Writing the installed custom metadata](#metadataPluginReadWrite)

## TL;DR - Metadata Plugins In-A-Nutshell [概述]<a name="summary"></a>
- Extending USD with custom metadata fields is as simple as creating a `plugInfo.json` file with entires for what custom fields you want and on what entities (Supported are `Sdf.Layer` and subclasses of `Sdf.Spec`).

    [ 使用自定义元数据字段扩展 USD 就像创建一个 plugInfo.json 文件一样简单，其中包含您想要的自定义字段和实体的完整内容（支持的有 Sdf.Layer 和 Sdf.Spec 的子类）]

## What should I use it for? <a name="usage"></a>

[ 我应该用它做什么？]

~~~admonish tip
In production, most of the sidecar metadata should be tracked via the `assetInfo` and `customData` metadata entries. It does make sense to extend the functionality with own metadata keys for:

[ 在生产中，大多数 sidecar 元数据应通过 assetInfo 和 customData 元数据条目进行跟踪. 使用自己的元数据键扩展功能确实有意义]
- Doing high performance lookups. Metadata is fast to read, as it follows simpler composition rules, so we can use it as a `IsA` replacement mechanism we can tag our prims/properties with.

    [ 进行高性能查找. 元数据的读取速度很快，因为它遵循更简单的组合规则，因此我们可以将其用作 IsA 替换机制，我们可以用它来标记我们的 prims/properties ]
- We can add [list editable ops](../composition/listeditableops.md) metadata fields, this can be used as a way to layer together different array sidecar data. For an example see our [Reading/Writing the installed custom metadata](#metadataPluginReadWrite) section.

    [ 我们可以添加 [列表编辑操作](../composition/listeditableops.md) 元数据字段，这可以作为将不同数组的辅助数据层叠在一起的一种方式. 有关示例，请参阅我们 [Reading/Writing the installed custom metadata](#metadataPluginReadWrite) ]
~~~

## Resources [资源]
- [USD Survival Guide - Metadata](../elements/metadata.md)
- [USD API - Metadata](https://openusd.org/release/api/sdf_page_front.html#sdf_plugin_metadata)
- [USD Cookbook - Extending Metadata](https://github.com/ColinKennedy/USD-Cookbook/blob/master/references/working_with_plugins.md#Extend-Metadata) 

## Overview [概述]<a name="overview"></a>
Here is the minimal plugin template with all options you can configure for your metadata:

[ 这是最小的插件模板，其中包含您可以为元数据配置的所有选项]
~~~admonish info title=""
```json
{
   "Plugins": [
       {
           "Name": "<PluginName>",
           "Type": "resource",
           "Info": {
               "SdfMetadata": {
                    "<field_name>" : {
                        "appliesTo": "<Optional comma-separated list of spec types this field applies to>",
                        "default": "<Optional default value for field>",
                        "displayGroup": "<Optional name of associated display group>",
                        "type": "<Required name indicating field type>",
                    }
               }
           }
       }
   ]
}
```
~~~


We can limit the metadata entry to the following `Sdf.Layer`/Subclasses of `Sdf.Spec`s with the `type` entry:

[ 我们可以将元数据条目限制为以下 Sdf.Layer / Sdf.Spec 的子类以及 type 条目]

| "appliesTo" token	| Spec type                    |
|-------------------|------------------------------|
| layers	        | SdfLayer (SdfPseudoRootSpec) |
| prims	            | SdfPrimSpec, SdfVariantSpec  |
| properties	    | SdfPropertySpec              |
| attributes	    | SdfAttributeSpec             |
| relationships	    | SdfRelationshipSpec          |
| variants          | SdfVariantSpec               |

You can find all the supported data types on this page in the official docs: [USD Cookbook - Extending Metadata](https://openusd.org/release/api/sdf_page_front.html#sdf_plugin_metadata).

[ 您可以在官方文档中找到此页面上所有支持的数据类型： [USD Cookbook - Extending Metadata](https://openusd.org/release/api/sdf_page_front.html#sdf_plugin_metadata) ]

### Installing a metadata plugin <a name="metadataPluginInstall"></a>

Here is an example `plugInfo.json` file for metadata, it also ships with this repo [here](https://github.com/LucaScheller/VFX-UsdSurvivalGuide/tree/main/files/plugins/metadata).

[ 这是元数据的示例 plugInfo.json 文件，它也随此存储库一起 [提供](https://github.com/LucaScheller/VFX-UsdSurvivalGuide/tree/main/files/plugins/metadata)]

~~~admonish info title=""
```python
{{#include ../../../../files/plugins/metadata/plugInfo.json}}
```
~~~

To register the above metadata plugin, copy the contents into a file called `plugInfo.json`. Then set your `PXR_PLUGINPATH_NAME` environment variable to the folder containing the `plugInfo.json` file.

[ 要注册上述元数据插件，请将内容复制到名为 plugInfo.json 的文件中. 然后将 PXR_PLUGINPATH_NAME 环境变量设置为包含 plugInfo.json 文件的文件夹]

For Linux this can be done for the active shell as follows:

[ 对于 Linux，可以对活动 shell 执行以下操作]

```bash
export PXR_PLUGINPATH_NAME=/my/cool/plugin/resources:${PXR_PLUGINPATH_NAME}
```

If you downloaded this repo, we provide the above example metadata plugin [here](https://github.com/LucaScheller/VFX-UsdSurvivalGuide/tree/main/files/plugins/metadata). All you need to do is point the environment variable there and launch a USD capable application.

[ 如果您下载了此存储库，我们会在 [此处](https://github.com/LucaScheller/VFX-UsdSurvivalGuide/tree/main/files/plugins/metadata) 提供上述示例元数据插件. 您所需要做的就是将环境变量指向那里并启动支持 USD 的应用程序]


### Reading/Writing the installed custom metadata <a name="metadataPluginReadWrite"></a>
Once the plugin is loaded, we can now read and write to the custom entry.

[ 加载插件后，我们现在可以读取和写入自定义条目]

Custom metadata fields on the `Sdf.Layer` are not exposed via Python (as far as we could find).

[ Sdf.Layer 上的自定义元数据字段不会通过 Python 公开（据我们所知）]

~~~admonish info title=""
```python
{{#include ../../../../code/core/elements.py:metadataPlugin}}
```
~~~