# Plugin System [插件系统]
Usd has a plugin system over which individual components are loaded.

[ Usd 有一个插件系统，可以在其中加载各个组件]

~~~admonish Abstract title="Usd Pipeline Plugins"
In this guide we will cover how to create these minimal set of plugins needed to setup a production ready Usd pipeline:

[ 在这份指南中，我们将介绍如何创建一套最小化的插件集，建立一个可用于生产的 USD 流程]
- **Kinds Plugin** (No compiling needed): For this plugin all you need is a simple .json file that adds custom kinds. Head over to our [kind](./kind.md) section to get started.

    [ Kinds Plugin（无需编译）：对于此插件，您需要的只是一个添加自定义种类的简单 .json 文件. 请前往 [kind](./kind.md) 部分开始]
- **Asset Resolver Plugin** (Compiling needed or use pre-packed resolvers): Head over to our [asset resolver](./assetresolver.md) section to get started. The actual [code](https://github.com/LucaScheller/VFX-UsdAssetResolver) and [guide](https://lucascheller.github.io/VFX-UsdAssetResolver/) is hosted here as it is a big enough topic of its own.

    [ Asset Resolver Plugin（编译所需的或使用预打包的解析器）：前往[资产解析器](./assetresolver.md) 部分开始. 实际的 [代码](https://github.com/LucaScheller/VFX-UsdAssetResolver)和 [指南](https://lucascheller.github.io/VFX-UsdAssetResolver/)托管在此处，因为它本身就是一个足够大的主题]
- **Schema Plugin** (Optional, compiling needed if you want Python C++/Bindings): A schema plugin allows you to create own prim types/API schemas. This is useful when you need to often create a standardized set of attributes on prims that are relevant for your pipeline. Head over to our [schemas](./schemas.md) section to get going or to our [schemas overview](../elements/schemas.md) section to get an overview of what schemas are.

    [ Schema Plugin（可选，如果您需要 Python C++/绑定，则需要编译）：Schema Plugin 允许您创建自己的 prim types/API schemas 当您需要经常在与流程相关的 prim 上创建一组标准化属性时，这非常有用. 请转至 [schemas](./schemas.md) 部分以开始操作，或前往  [schemas overview](../elements/schemas.md)  部分]
- **Metadata Plugin** (Optional, No compiling needed): A metadata plugin allows you to create own metadata entries, so that you don't have to use the `assetInfo`/`customData` dict fields for transporting custom metadata. Schema plugins can also specify metadata, it is limited to the prim/applied schema though, so a standalone metadata plugin allows us to make metadata available on all prims/properties regardless of prim type/schema.

    [ Metadata Plugin（可选，无需编译）：元数据插件允许您创建自己的元数据条目，这样您就不必使用 assetInfo / customData 字典字段来传输自定义元数据.  Schema plugins 还可以指定元数据，但它仅限于 prim/applied schema，因此独立的元数据插件允许我们使元数据在所有 prims/properties 上可用，而不管 prim type/schema 如何]
~~~

~~~admonish important title="Compiling against USD"
As listed above, some plugins need to be compiled. Most DCCs ship with a customized USD build, where most vendors adhere to the [VFX Reference Platform](https://vfxplatform.com/) and only change USD with major version software releases. They do backport important production patches though from time to time. That's why we recommend using the USD build from the DCC instead of trying to self compile and link it to the DCC, as this guarantees the most stability. This does mean though, that you have to compile all plugins against each (major version) releases of each individual DCC.

[ 如上所述一些插件需要编译. 大多数 DCC 都附带定制的 USD 版本，其中大多数供应商都遵循 [VFX Reference Platform](https://vfxplatform.com/) 并且仅随着主要版本软件发布而更改 USD. 他们有时会向后移植重要的生产补丁. 这就是为什么我们建议使用 DCC 的 USD 构建，而不是尝试自编译并将其链接到 DCC，因为这可以保证最大的稳定性. 但这确实意味着，您必须针对每个 DCC 的每个（主要版本）版本来编译所有插件]
~~~

Typical plugins are:

[ 典型的插件有]

- Schemas
- Metadata
- Kinds
- Asset Resolver
- Hydra Delegates (Render Delegates)
- File Format Plugins (.abc/.vdb)

You can inspect if whats plugins were registered by setting the `TF_DEBUG` variable as mentioned in the [debugging](../profiling/debug.md) section:

[ 您可以通过设置 TF_DEBUG 变量来检查插件是否已注册，如调试部分中所述]

```bash
export TF_DEBUG=PLUG_REGISTRATION
```

If you want to check via Python, you have to know under what registry the plugin is installed. There are several (info shamelessly copied from the below linked USD-CookBook page ;)):

[ 如果您想通过 Python 进行检查，您需要知道插件安装在哪个注册表下. 有几个注册表可以选择（以下信息是从 USD-CookBook 页面复制过来的，感谢提供这些信息）：]

- KindRegistry
- PlugRegistry
- Sdf_FileFormatRegistry
- ShaderResourceRegistry
- UsdImagingAdapterRegistry

Colin Kennedy's USD-Cookbook has an excellent overview on this topic:
[USD Cook-Book Plugins](https://github.com/ColinKennedy/USD-Cookbook/blob/33eac067a0a62578934105b19a2b9d8e4ea0646c/references/working_with_plugins.md)

[ Colin Kennedy 的 USD-Cookbook 对这个主题有很好的概述：[USD Cook-Book Plugins](https://github.com/ColinKennedy/USD-Cookbook/blob/33eac067a0a62578934105b19a2b9d8e4ea0646c/references/working_with_plugins.md)]

Plugins are detected by looking at the `PXR_PLUGINPATH_NAME` environment variable for folders containing a`plugInfo.json` file.

[ 插件的检测是通过查看名为 PXR_PLUGINPATH_NAME 的环境变量来实现的，该环境变量指定了包含 plugInfo.json 文件的文件夹路径]

To set it temporarily, you can run the following in a shell and then run your Usd application:

[ 要临时设置它，您可以在 shell 中运行以下命令，然后运行您的 Usd 应用程序]

```
// Linux
export PXR_PLUGINPATH_NAME=/my/cool/plugin/resources:${PXR_PLUGINPATH_NAME}
// Windows
set PXR_PLUGINPATH_NAME=/my/cool/plugin/resources:${PXR_PLUGINPATH_NAME}
```

If you search you Usd installation, you'll find a few of these for different components of Usd. They are usually placed in a <Plugin Root>/resources folder as a common directory convention.

[ 如果您在 USD 安装目录中搜索，您会发现针对不同 USD 组件的多个此类文件. 它们通常放置在 \<Plugin Root\>/resources 文件夹中，这是常见的目录约定]

Via Python you can also partially search for plugins (depending on what registry they are in) and also print their plugInfo.json file content via the `.metadata` attribute.

[ 通过 Python 您也可以部分搜索插件（具体取决于它们位于哪个注册表中）并通过 .metadata 属性打印它们的 plugInfo.json 文件内容]

~~~admonish info title=""
```python
{{#include ../../../../code/core/elements.py:pluginsRegistry}}
```
~~~

We can also use the plugin registry to lookup from what plugin a specific type/class (in this case a schema) is registered by:

[ 我们还可以使用插件注册表来查找特定类型/类（在本例中为 schema）注册的插件]

~~~admonish info title=""
```python
{{#include ../../../../code/core/elements.py:schemasPluginRegistry}}
```
~~~

