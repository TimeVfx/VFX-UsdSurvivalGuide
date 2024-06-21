# Schemas
~~~admonish important
This page only covers how to compile/install custom schemas, as we cover what schemas are in our [schemas](../elements/schemas.md) basic building blocks of USD section.

[ 本页面仅介绍如何编译/安装自定义 schemas，因为我们已在 USD 的基本构建块 [schemas](../elements/schemas.md) 部分中介绍了 schemas 的定义]
~~~

As there is a very well written documentation in the [official docs](https://openusd.org/release/api/_usd__page__generating_schemas.html), we only cover compilation(less) schema creation and installation here as a hands-on example and won't go into any customization details. You can also check out Colin's excellent [Usd-Cook-Book](https://github.com/ColinKennedy/USD-Cookbook/tree/master/plugins/custom_schemas_with_python_bindings) example.

[ 由于 [官方文档](https://openusd.org/release/api/_usd__page__generating_schemas.html) 中有写得很好的文档，因此我们在这里仅介绍编译（较少）schema 的创建和安装作为实践示例，并且不会讨论任何自定义细节. 您还可以查看 Colin 的优秀 [Usd-Cook-Book](https://github.com/ColinKennedy/USD-Cookbook/tree/master/plugins/custom_schemas_with_python_bindings) 示例]

# Table of Contents [目录]
1. [API Overview In-A-Nutshell](#summary)
2. [What should I use it for?](#usage)
3. [Resources](#resources)
4. [Overview](#overview)
5. [Generate Codeless Schema](#usdGenSchemaCodelessSchema)
    1. [Edit 'GLOBAL' prim 'customData' dict](#usdGenSchemaCodelessSchemaStep1)
    2. [Run usdGenSchema](#usdGenSchemaCodelessSchemaStep2)
    3. [Add the generated pluginInfo.json director to 'PXR_PLUGINPATH_NAME' env var.](#usdGenSchemaCodelessSchemaStep3)
    4. [Run your Usd (capable) application.](#usdGenSchemaCodelessSchemaStep4)
6. [Generate Compiled Schema](#usdGenSchemaCompiledSchema)
    1. [Run usdGenSchema](#usdGenSchemaCompiledSchemaStep1)
    2. [Compile schema](#usdGenSchemaCompiledSchemaStep2)
    3. [Update environment variables.](#usdGenSchemaCompiledSchemaStep3)
    4. [Run your Usd (capable) application.](#usdGenSchemaCompiledSchemaStep4)


## TL;DR - Schema Creation In-A-Nutshell [概述]<a name="summary"></a>
- Generating schemas in Usd is as easy as supplying a customized `schema.usda` file to the `usdGenSchema` commandline tool that ships with Usd. That's right, you don't need to code!

    [ 在 Usd 中生成架构就像向 Usd 附带的 usdGenSchema 命令行工具提供自定义的 schema.usda 文件一样简单. 没错，您不需要编码！]
- Custom schemas allow us to create custom prim types/properties/metadata (with fallback values) so that we don't have to repeatedly re-create it ourselves.

    [ Custom schemas 允许我们创建自定义 prim types/properties/metadata（带有默认值），所以我们不需要重复造轮子]
- In OOP speak: It allows you to create your own subclasses that nicely fit into Usd and automatically generates all the `Get<PropertyName>`/`Set<PropertyName>` methods, so that it feels like you're using native USD classes.

    [ 用面向对象编程的语言来说：它允许您创建自己的子类，这些子类能够很好地融入 USD 中，并自动生成所有 Get\<PropertyName\>/Set\<PropertyName\> 方法，从而让您感觉就像在使用 USD 原生类一样]
- We can also create `codeless` schemas, these don't need to be compiled, but we won't get our nice automatically generated getters and setters and schema C++/Python classes.

    [ 我们还可以创建 codeless 模式，这些模式不需要编译，但我们不会得到自动生成的 getter 和 setter 以及模式 C++/Python 类]

~~~admonish tip
Codeless schemas are ideal for smaller studios or when you need to prototype a schema. The result only consists of a `plugInfo.json` and `generatedSchema.usda` file and is instantly created without any need for compiling.

[ 无代码架构非常适合小型工作室或当您需要对架构进行原型设计时. 结果仅包含 plugInfo.json 和 generatedSchema.usda 文件，并且无需编译即可立即创建]
~~~

~~~admonish important title="Compiling against USD"
Most DCCs ship with a customized USD build, where most vendors adhere to the [VFX Reference Platform](https://vfxplatform.com/) and only change USD with major version software releases. They do backport important production patches though from time to time. That's why we recommend using the USD build from the DCC instead of trying to self compile and link it to the DCC, as this guarantees the most stability. This does mean though, that you have to compile all plugins against each (major version) releases of each individual DCC.

[ 大多数 DCC 都附带定制的 USD 版本，其中大多数供应商都遵循 [VFX 参考平台](https://vfxplatform.com/) ，并且仅随着主要版本软件发布而更改 USD. 他们有时会向后移植重要的生产补丁. 这就是为什么我们建议使用 DCC 的 USD 构建，而不是尝试自编译并将其链接到 DCC，因为这可以保证最大的稳定性. 但这确实意味着，您必须针对每个 DCC 的每个（主要版本）版本编译所有插件]
~~~

## What should I use it for? <a name="usage"></a>

[ 我应该用它做什么？]

~~~admonish tip
We'll usually want to generate custom schemas, when we want to have a set of properties/metadata that should always exist (with a fallback value) on certain prims. A typical use case for creating an own typed/API schema is storing common render farm settings or shot related data.

[ 当我们想要拥有一组始终存在于某些 prims 上（具有默认值）的属性/元数据时,我们通常会想要 custom schemas. 创建自定义 typed/API schema 的一个典型用例是存储渲染农场通用设置或镜头相关数据]
~~~

# Resources [资源]<a name="resources"></a>
- [API Docs](https://openusd.org/release/api/_usd__page__generating_schemas.html)

## Overview [概述]<a name="overview"></a>
For both examples we'll start of with the example schema that USD ships with in its official repo.

[ 对于这两个示例，我们将从 USD 在其官方存储库中附带的示例模式开始]

You can copy and paste the [content](https://github.com/PixarAnimationStudios/OpenUSD/blob/release/extras/usd/examples/usdSchemaExamples/schema.usda) into a file and then follow along or take the prepared files from [here](https://github.com/LucaScheller/VFX-UsdSurvivalGuide/tree/main/files/plugins/schemas) that ship with this repo.

[ 您可以将 [内容](https://github.com/PixarAnimationStudios/OpenUSD/blob/release/extras/usd/examples/usdSchemaExamples/schema.usda) 复制并粘贴到文件中，然后继续操作或从 [此处](https://github.com/LucaScheller/VFX-UsdSurvivalGuide/tree/main/files/plugins/schemas) 获取随此存储库附带的准备好的文件]

Our guide focuses on working with Houdini, therefore we use the `usdGenSchema` that ships with Houdini. You can find it in your Houdini /bin directory.

[ 我们的指南侧重于使用 Houdini，因此我们使用 Houdini 附带的 usdGenSchema 您可以在 Houdini /bin 目录中找到它]
~~~admonish tip title=""
```bash
$HFS/bin/usdGenSchema
# For example on Linux:
/opt/hfs19.5/bin/usdGenSchema
```
~~~

If you download/clone this repo, we ship with .bash scripts that automatically runs all the below steps for you.

[ 如果您下载/克隆此存储库，我们会附带 .bash 脚本，该脚本会自动为您运行以下所有步骤]

You'll first need to `cd` to the root repo dir and then run `./setup.sh`. Make sure that you edit the `setup.sh` file to point to your Houdini version. By default it will be the latest Houdini major release symlink, currently `/opt/hfs19.5`, that Houdini creates on install.

[ 您首先需要 cd 到根存储库目录，然后运行 ​​./setup.sh 确保编辑 setup.sh 文件以指向您的 Houdini 版本. 默认情况下，它将是 Houdini 在安装时创建的最新 Houdini 主要版本符号链接，当前为 /opt/hfs19.5 ]

Then follow along the steps as mentioned below.

[ 然后按照下面提到的步骤进行操作]

## Codeless Schema [无代码模式]<a name="usdGenSchemaCodelessSchema"></a>
Codeless schemas allow us to generate schemas without any C++/Python bindings. This means your won't get fancy `Schema.Get<PropertyName>`/`Schema.Set<PropertyName>` getters and setters. On the upside you don't need to compile anything.

[ 无代码模式允许我们在没有任何 C++/Python 绑定的情况下生成 schemas. 这意味着您不会得到 Schema.Get\<PropertyName\> / Schema.Set\<PropertyName\> getter 和 setter 方法. 从好的方面来说，你不需要编译任何东西]

~~~admonish tip
Codeless schemas are ideal for smaller studios or when you need to prototype a schema. The result only consists of a `plugInfo.json` and `generatedSchema.usda` file.

[ 无代码模式非常适合小型工作室或需要对模式进行原型设计时. 结果仅包含 plugInfo.json 和 generatedSchema.usda 文件]
~~~

To enable codeless schema generation, we simply have to add `bool skipCodeGeneration = true` to the customData metadata dict on the global prim in our schema.usda template file.

[ 要启用无代码模式生成，我们只需将 bool skipCodeGeneration = true 添加到 schema.usda 模板文件中全局 prim 上的 customData 元数据字典中]

~~~admonish tip title=""
```python
over "GLOBAL" (
    customData = {
        bool skipCodeGeneration = true
    }
) {
}
```
~~~

Let's do this step by step for our example schema.

[ 让我们针对示例架构逐步执行此操作]

#### Step 1: Edit 'GLOBAL' prim 'customData' dict <a name="usdGenSchemaCodelessSchemaStep1"></a>
Update the global prim custom data dict from:

[ 更新全局 prim 自定义数据字典]

~~~admonish tip title=""
```python
over "GLOBAL" (
    customData = {
        string libraryName       = "usdSchemaExamples"
        string libraryPath       = "."
        string libraryPrefix     = "UsdSchemaExamples"
    }
) {
}
```
~~~
to:
~~~admonish tip title=""
```python
over "GLOBAL" (
    customData = {
        string libraryName       = "usdSchemaExamples"
        string libraryPath       = "."
        string libraryPrefix     = "UsdSchemaExamples"
        bool skipCodeGeneration = true
    }
) {
}
```
~~~

~~~admonish info title="Result | Click to expand content" collapsible=true
```python
{{#include ../../../../files/plugins/schemas/codelessSchema/schema.usda}}
```
~~~

#### Step 2: Run usdGenSchema <a name="usdGenSchemaCodelessSchemaStep2"></a>
Next we need to generate the schema.

[ 接下来我们需要生成架构]

Make sure that you first sourced you Houdini environment by running `$HFS/houdini_setup` so that it find all the correct libraries and python interpreter.

[ 确保您首先通过运行 $HFS/houdini_setup 获取 Houdini 环境，以便它找到所有正确的库和 python 解释器]

~~~admonish tip title="usdGenSchema on Windows"
On Windows you can also run `hython usdGenSchema schema.usda dst` to avoid having to source the env yourself.

[ 在 Windows 上，您还可以运行 hython usdGenSchema schema.usda dst 以避免必须自己获取环境]
~~~

Then run the following

[ 然后运行以下命令]

~~~admonish tip title=""
```bash
cd /path/to/your/schema # In our case: .../VFX-UsdSurvivalGuide/files/plugins/schemas/codelessSchema
usdGenSchema schema.usda dst
```
~~~
Or if you use the helper bash scripts in this repo (after sourcing the `setup.sh` in the repo root):

[ 或者，如果您在此存储库中使用帮助器 bash 脚本（在存储库根目录中获取 `setup.sh` ）]

~~~admonish tip title=""
```bash
cd ./files/plugins/schemas/codelessTypedSchema/
chmod +x build.sh # Add execute rights
source ./build.sh # Run usdGenSchema and source the env vars for the plugin path
```
~~~

~~~admonish bug
Not sure if this is a bug, but the `usdGenSchema` in codeless mode currently outputs a wrong `plugInfo.json` file. (It leaves in the cmake @...@ string replacements).

[ 不确定这是否是一个错误，但无代码模式下的 usdGenSchema 当前输出错误的 plugInfo.json 文件（它留在 cmake @...@ 字符串替换中）]

The fix is simple, open the `plugInfo.json` file and replace:

[ 修复很简单，打开 plugInfo.json 文件并替换]

```python
...
    "LibraryPath": "@PLUG_INFO_LIBRARY_PATH@", 
    "Name": "usdSchemaExamples", 
    "ResourcePath": "@PLUG_INFO_RESOURCE_PATH@", 
    "Root": "@PLUG_INFO_ROOT@", 
    "Type": "resource"
...
```
To:
```python
...
    "LibraryPath": ".", 
    "Name": "usdSchemaExamples", 
    "ResourcePath": ".", 
    "Root": ".", 
    "Type": "resource"
...
```
~~~

~~~admonish info title="Result | Click to expand content" collapsible=true
```python
{{#include ../../../../files/plugins/schemas/codelessSchema/dist/plugInfo.json}}
```
~~~


#### Step 3: Add the generated pluginInfo.json director to 'PXR_PLUGINPATH_NAME' env var. <a name="usdGenSchemaCodelessSchemaStep3"></a>
Next we need to add the pluginInfo.json directory to the `PXR_PLUGINPATH_NAME` environment variable.

[ 接下来我们需要将 pluginInfo.json 目录添加到 PXR_PLUGINPATH_NAME 环境变量中]

~~~admonish tip title=""
```bash
// Linux
export PXR_PLUGINPATH_NAME=/Enter/Path/To/dist:${PXR_PLUGINPATH_NAME}
// Windows
set PXR_PLUGINPATH_NAME=/Enter/Path/To/dist;%PXR_PLUGINPATH_NAME%
```
~~~
If you used the helper bash script, it is already done for us.

[ 如果您使用了 helper bash 脚本，那么它已经为我们完成了]

#### Step 4: Run your Usd (capable) application. <a name="usdGenSchemaCodelessSchemaStep4"></a>
Yes, that's right! It was that easy. (Puts on sunglass, ah yeeaah! 😎)

[ 恩，那就对了！就是这么简单 （戴上墨镜耍帅，啊耶！😎）]

If you run Houdini and then create a primitive, you can now choose the `ComplexPrim` as well as assign the `ParamAPI` API schema.

[ 如果您运行 Houdini 然后创建一个 prim，您现在可以选择 ComplexPrim 并分配 ParamAPI API 架构]

!["test"](./schemaCodelessHoudini.jpg#center)

Or if you want to test it in Python:

[ 或者如果你想在 Python 中测试它]

~~~admonish info title=""
```python
{{#include ../../../../code/core/elements.py:schemasPluginCodelessTest}}
```
~~~

## Compiled Schema <a name="usdGenSchemaCompiledSchema"></a>
Compiled schemas allow us to generate schemas with any C++/Python bindings. This means we'll get `Schema.Get<PropertyName>`/`Schema.Set<PropertyName>` getters and setters automatically which gives our schema a very native Usd feeling. You can then also edit the C++ files to add custom features on top to manipulate the data generated by your schema. This is how many of the schemas that ship with USD do it.

[ 编译模式允许我们使用任何 C++/Python 生成 schemas. 这意味着我们将自动获得 Schema.Get\<PropertyName\> / Schema.Set\<PropertyName\> getter 和 setter，这给我们的模式带来了非常原生的 Usd 感觉. 然后，您还可以编辑 C++ 文件以在顶部添加自定义功能，以操作架构生成的数据. USD 附带的许多模式都是这样做的]

~~~admonish tip title="usdGenSchema on Windows"
Currently these instructions are only tested for Linux. We might add Windows support in the near future. (We use CMake, so in theory it should be possible to run the same steps in Windows too.)

[ 目前这些指令仅针对 Linux 进行了测试. 我们可能会在不久的将来添加 Windows 支持. (我们使用 CMake，因此理论上也应该可以在 Windows 中运行相同的步骤)]
~~~

Let's get started step by step for our example schema.

[ 让我们逐步开始我们的 schema 演示]

We also ship with a `build.sh` for running all the below steps in one go. Make sure you first run the `setup.sh` as described in the [overview](#overview) section and then navigate to the compiledSchema folder.

[ 我们还附带了一个 build.sh ，用于一次性运行以下所有步骤. 确保您首先按照概述部分中的说明运行 setup.sh ，然后导航到 CompiledSchema 文件夹]

~~~admonish tip title=""
```bash
cd .../VFX-UsdSurvivalGuide/files/plugins/schemas/compiledSchema
chmod +x build.sh # Add execute rights
source ./build.sh # Run usdGenSchema and source the env vars for the plugin path
```
~~~

This will completely rebuild all directories and set the correct environment variables. You can then go straight to the last step to try it out.

[ 这将完全重建所有目录并设置正确的环境变量. 然后您可以直接进入最后一步进行尝试]

#### Step 1: Run usdGenSchema <a name="usdGenSchemaCompiledSchemaStep1"></a>
First we need to generate the schema.

[ 首先我们需要生成 schema]

Make sure that you first sourced you Houdini environment by running `$HFS/houdini_setup` so that it can find all the correct libraries and python interpreter.

[ 确保您首先通过运行 $HFS/houdini_setup 获取 Houdini 环境，以便它可以找到所有正确的库和 python 解释器]

~~~admonish tip title="usdGenSchema on Windows"
On Windows you can also run `hython usdGenSchema schema.usda dst` to avoid having to source the env yourself.

[ 在 Windows 上，您还可以运行 hython usdGenSchema schema.usda dst 以避免必须自己获取环境]
~~~

Then run the following

[ 然后运行以下命令]

~~~admonish tip title=""
```bash
cd /path/to/your/schema # In our case: ../VFX-UsdSurvivalGuide/files/plugins/schemas/compiledSchema
rm -R src
usdGenSchema schema.usda src
```
~~~

Currently `usdGenSchema` fails to generate the following files:

[ 目前 usdGenSchema 无法生成以下文件]

- `module.cpp`
- `moduleDeps.cpp`
- `__init__.py`

We needs these for the Python bindings to work, so we supplied them in the `VFX-UsdSurvivalGuide/files/plugins/schemas/compiledSchema/auxiliary` folder of this repo. Simply copy them into the `src` folder after running `usdGenSchema`.

[ 为了让 Python 绑定能够正常工作，我们需要在本存储库的 VFX-UsdSurvivalGuide/files/plugins/schemas/compiledSchema/auxiliary 文件夹中提供这些文件. 只需在运行 usdGenSchema 之后将它们复制到src文件夹中即可]

It does automatically detect the boost namespace, so the generated files will automatically work with Houdini's `hboost` namespace.

[ 它会自动检测 boost 命名空间，因此生成的文件将自动与 Houdini 的 hboost 命名空间一起使用]

~~~admonish important
If you adjust your own schemas, you will have edit the following in these files:

[ 如果您调整自己的 schemas，您将在这些文件中编辑以下内容]

- `module.cpp`: Per user define schema you need to add a line consisting of `TF_WRAP(<SchemaClassName>);`

    [ module.cpp ：对于用户定义的模式，您需要添加一行，该行包含 TF_WRAP(\<SchemaClassName\>)]
- `moduleDeps.cpp`: If you add C++ methods, you will need to declare any dependencies what your schemas have. This file also contains the namespace for C++/Python where the class modules will be accessible. We change `RegisterLibrary(TfToken("usdSchemaExamples"), TfToken("pxr.UsdSchemaExamples"), reqs);` to `RegisterLibrary(TfToken("usdSchemaExamples"), TfToken("UsdSchemaExamples"), reqs);` as we don't want to inject into the default pxr namespace for this demo.

    [ moduleDeps.cpp：如果您添加了C++方法，则需要声明 schemas 所依赖的任何内容. 此文件还包含 C++/Python 的命名空间，其中可以访问类模块. 我们将 RegisterLibrary(TfToken("usdSchemaExamples"), TfToken("pxr.UsdSchemaExamples"), reqs) 更改为 RegisterLibrary(TfToken("usdSchemaExamples"), TfToken("UsdSchemaExamples"), reqs)，因为我们不希望在此演示中注入默认的pxr命名空间]

~~~

#### Step 2: Compile schema <a name="usdGenSchemaCompiledSchemaStep2"></a>
Next up we need to compile the schema. You can check out our asset resolver guide for more info on [system requirements](https://lucascheller.github.io/VFX-UsdAssetResolver/installation/requirements.html). In short you'll need a recent version of:

[ 接下来我们需要编译架构. 您可以查看我们的资产解析器指南，了解有关 [系统要求](https://lucascheller.github.io/VFX-UsdAssetResolver/installation/requirements.html)的更多信息.简而言之，您需要以下版本]

- gcc (compiler)
- cmake (build tool).

To compile, we first need to adjust our `CMakeLists.txt` file.

[ 要编译，我们首先需要调整 CMakeLists.txt 文件]

USD actually ships with a `CMakeLists.txt` file in the [examples](https://github.com/PixarAnimationStudios/OpenUSD/blob/release/extras/usd/examples/usdSchemaExamples/CMakeLists.txt) section. It uses some nice USD CMake convenience functions generate the make files.

[ USD 实际上在示例部分附带了一个 CMakeLists.txt 文件. 它使用一些不错的 USD CMake 便利函数来生成 make 文件]

We are not going to use that one though. Why? Since we are building against Houdini and to make things more explicit, we prefer showing how to explicitly define all headers/libraries ourselves. For that we provide the `CMakeLists.txt` file [here](https://github.com/LucaScheller/VFX-UsdSurvivalGuide/tree/main/files/plugins/schemas/compiledSchema).

[ 不过我们并不会使用那个. 为什么呢？因为我们正在针对Houdini进行构建，并且为了让事情更加明确，我们更倾向于展示如何显式地定义所有头文件和库. 为此，我们在这里提供了CMakeLists.txt文件]

Then run the following

[ 然后运行以下命令]

~~~admonish tip title=""
```bash
# Clear build & install dirs
rm -R build
rm -R dist
# Build
cmake . -B build
cmake --build build --clean-first # make clean all
cmake --install build             # make install
```
~~~

Here is the content of the CMakeLists.txt file. We might make a CMake intro later, as it is pretty straight forward to setup once you know the basics.

[ 这是 CMakeLists.txt 文件的内容. 我们稍后可能会做一个 CMake 介绍，因为一旦您了解了基础知识，设置就非常简单]

~~~admonish info title="CMakeLists.txt | Click to expand content" collapsible=true
```python
{{#include ../../../../files/plugins/schemas/compiledSchema/CMakeLists.txt}}
```
~~~

#### Step 3: Update environment variables. <a name="usdGenSchemaCompiledSchemaStep3"></a>
Next we need to update our environment variables. The cmake output log actually has a message that shows what to set:

[ 接下来我们需要更新我们的环境变量. cmake 输出日志实际上有一条消息显示要设置的内容]

- `PXR_PLUGINPATH_NAME`: The USD plugin search path variable.

    [ PXR_PLUGINPATH_NAME ：USD 插件搜索路径变量]
- `PYTHONPATH`: This is the standard Python search path variable.

    [ PYTHONPATH ：这是标准的 Python 搜索路径变量]
- `LD_LIBRARY_PATH`: This is the search path variable for how `.so` files are found on Linux.

    [ LD_LIBRARY_PATH ：这是用于在 Linux 上查找 .so 文件的搜索路径变量]

~~~admonish tip title=""
```bash
// Linux
export PYTHONPATH=..../VFX-UsdSurvivalGuide/files/plugins/schemas/compiledSchema/dist/lib/python:/opt/hfs19.5/python/lib/python3.9/site-packages:$PYTHONPATH
export PXR_PLUGINPATH_NAME=.../VFX-UsdSurvivalGuide/files/plugins/schemas/compiledSchema/dist/resources:$PXR_PLUGINPATH_NAME
export LD_LIBRARY_PATH=.../VFX-UsdSurvivalGuide/files/plugins/schemas/compiledSchema/dist/lib:/python/lib:/dsolib:$LD_LIBRARY_PATH
// Windows
set PYTHONPATH=..../VFX-UsdSurvivalGuide/files/plugins/schemas/compiledSchema/dist/lib/python;/opt/hfs19.5/python/lib/python3.9/site-packages;%PYTHON_PATH%
set PXR_PLUGINPATH_NAME=.../VFX-UsdSurvivalGuide/files/plugins/schemas/compiledSchema/dist/resources;%PXR_PLUGINPATH_NAME%
```
For Windows, specifying the linked .dll search path is different. We'll add more info in the future.

[ 对于 Windows，指定链接的 .dll 搜索路径有所不同。我们将来会添加更多信息]
~~~

#### Step 4: Run your Usd (capable) application. <a name="usdGenSchemaCompiledSchemaStep4"></a>
If we now run Houdini and then create a primitive, you can now choose the `ComplexPrim` as well as assign the `ParamAPI` API schema.

[ 如果我们现在运行 Houdini，然后创建一个 prim，您现在可以选择 ComplexPrim 并分配 ParamAPI API 架构]

![""](./schemaCodelessHoudini.jpg#center)

Or if you want to test it in Python:

[ 或者如果你想在 Python 中测试它]

~~~admonish info title=""
```python
{{#include ../../../../code/core/elements.py:schemasPluginCompiledTest}}
```
~~~

As you can see we now get our nice `Create<PropertyName>`/`Get<PropertyName>`/`Set<PropertyName>` methods as well as full Python exposure to our C++ classes.

[ 正如您所看到的，我们现在获得了很好的 Create\<PropertyName\> / Get\<PropertyName\> / Set\<PropertyName\> 方法以及对 C++ 类的完整 Python 暴露]