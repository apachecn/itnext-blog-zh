# 私有 PowerShell 模块

> 原文：<https://itnext.io/private-powershell-modules-76f51a1bf893?source=collection_archive---------1----------------------->

## 打包和部署您的 DevOps 微框架

*第四部分* [*声明性 DevOps 微框架*](https://medium.com/@cjkuech/declarative-devops-microframeworks-9908c8d05332)

PowerShell 模块(如果设计正确)是完全模块化和可移植的——打包 DevOps 微框架的理想解决方案。虽然您*可以*尝试使用私有的 PowerShell Gallery 实例或 [Azure 工件](https://github.com/PowerShell/PowerShellGet/issues/492)来托管您的模块，但是开发和发布您的模块以供私人使用的最简单和最容易的方式是将它们存储在`git`中，并在部署时复制文件。

本指南将向您展示如何构建 PowerShell 模块，在您的其他项目中引用它，以及私下部署它。

![](img/3b9d5cced46c9299952d1e80411e2f85.png)

模块化、独立、可移植的代码是最好的代码

# 创建 PowerShell 模块

## 创建一个文件夹，并将光盘放入该文件夹

```
PS /projects> mkdir MyModule
PS /projects> cd MyModule
PS /projects/MyModule>
```

## 添加所需的文件

我们将添加以下文件:

*   `MyModule.psd1`，模块清单。
*   `MyModule.psm1`，这个脚本包含了我们导出的所有变量、函数、别名等。
*   `MyModule.tests.ps1`，我们的[纠缠](https://github.com/pester/Pester)单元测试。
*   `README.md`，我们的文档模块。这个文档是给我们的队友*开发*模块的。使用模块的人*将转而参考嵌入在我们模块中的[基于注释的帮助](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_comment_based_help)。*
*   `.gitignore`，告诉`git`不要跟踪我们的`.vscode`文件夹，因为 VS 代码可能会在你不知情的情况下将工作站特定的设置放到文件夹中。

```
PS /projects/MyModule> New-ModuleManifest ./MyModule.psd1
PS /projects/MyModule> "" > ./MyModule.psm1
PS /projects/MyModule> "" > ./MyModule.tests.ps1
PS /projects/MyModule> "" > ./README.md
PS /projects/MyModule> ".vscode" > ./.gitignore
```

## 向文件中添加内容

向每个文件添加相关内容。

对于模块清单，

*   将`RootModule`设置为`'MyModule.psm1'`
*   将`FunctionsToExport`、`CmdletsToExport`、`VariablesToExport`和`AliasesToExport`设置为从模块中导出的项目名称
*   将`ScriptsToProcess`设置为包含`class`和`enum`定义的脚本(如果有)
*   如果您的模块有 PowerShell 版本约束，设置`CompatiblePSEditions`以限制支持的[版本](https://docs.microsoft.com/en-us/windows-server/get-started/powershell-on-nano-server)
*   如果您的模块需要依赖关系，则设置`RequiredModules`。
*   有关配置模块清单的更多信息，请参见[官方指南](http://• https://docs.microsoft.com/en-us/powershell/developer/module/how-to-write-a-powershell-module-manifest)。

# 引用模块

## 按名称引用

您总是希望通过名称而不是路径来引用模块，以避免不必要地将模块耦合到它在磁盘上的路径。

## 告诉`Import-Module`如何找到你的模块

PowerShell 使用`PSModulePath`环境变量来确定在哪个文件夹中查找模块，因此我们必须修改这个环境变量。如果你使用 Windows，`PSModulePath`包含一个用`;`分隔的路径列表；如果使用*NIX，`PSModulePath`包含一个用`:`分隔的路径列表。在本指南中，我们将使用*NIX 分隔符。

要将路径`$Path`添加到我们的`PSModulePath`中，我们可以将`":$Path"`添加到`PSModulePath`字符串中；每当您在`$Path`开始开发需要该模块的项目时，在您的 PowerShell 会话中运行`$env:PSModulePath += ":$Path"`。在修改环境变量后，您可以通过在同一个会话中从命令行启动 VS 代码[来确保 VS 代码使用正确的路径。注意，我们可以全局地修改我们的环境变量，但是为开发独立的项目而进行全局工作站更改是不好的做法。](https://code.visualstudio.com/docs/editor/command-line)

# 部署模块

## 通常

部署模块就像将文件夹复制到目标机器的一个模块文件夹中一样简单。

## 码头工人

您可以在 Dockerfile 文件中轻松实现这一点。

## 安装依赖项

如果您在模块清单的`RequiredModules`中指定了依赖项，那么您可以使用这个代码片段来安装所有的依赖项。

# 后续步骤

## 了解有关扩展 PowerShell 的更多信息

本文是关于大规模管理 PowerShell 代码库的系列文章的一部分。阅读本系列的[部分，了解更多关于为大型 DevOps 代码库设计和编写更少代码的信息。](https://medium.com/@cjkuech/declarative-devops-microframeworks-9908c8d05332)