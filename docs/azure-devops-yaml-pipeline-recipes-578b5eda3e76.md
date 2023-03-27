# Azure DevOps yaml 管道配方

> 原文：<https://itnext.io/azure-devops-yaml-pipeline-recipes-578b5eda3e76?source=collection_archive---------2----------------------->

第 1 部分:简介和创建 NuGet 包

![](img/efa0c643da8e62de66d6589f568fb01c.png)

来源:https://pxhere.com/en/photo/925035

# 介绍

在我们的团队中，我们相当广泛地使用 Azure DevOps。我们管理着相当多的项目，大多数都有持续集成和部署设置。我们使用了 Azure DevOps 管道的许多功能，从 Azure Artifacts Nuget 包到本地代理，以在发布和本地部署期间运行本地工具。

自从引入 yaml 管道定义以来，我们已经开始在小项目上使用它们，但是等待它们变得更加成熟，以便将我们最重要的项目迁移到那个新的定义。我们利用这次迁移的机会重新设计了管道，并利用了“旧”UI 风格构建的改进。例如，我们使用多阶段来更好地将构建从不同级别的发布中分离出来。

在这一系列文章中，我想描述一些我们的方法，我们如何为每一种类型的项目定义我们的构建。

# Yaml 管道

关于如何设置 yml 管道，我不会讲太多细节。您应该已经对如何创建和配置新的 yml 管道有了基本的了解。如果你在开始时需要帮助，请访问 [Azure Pipelines 文档](https://docs.microsoft.com/en-us/azure/devops/pipelines/get-started/pipelines-get-started?view=azure-devops)。

与旧的基于“UI”的管道相比，Yaml 管道有很多优点。最突出的是，由于它们是存储库的一部分，它们是简单的源代码，与应用程序代码的其余部分一起。如果源代码中的变化要求管道发生变化，您可以在同一个提交/分支中完成。它可以是拉请求的一部分，甚至可以在拉请求期间执行更改。

# 。Net 库来获取包

现在让我们讨论我们的第一个食谱。我们有很多。Net 类库项目，我们将它们转换成 NuGet 包以包含在我们的主应用程序中。

## 代码

我的介绍可能已经让我失去了一半的读者，所以让我们开门见山吧。下面是代码，我将在后面详细解释。

## 使用模板

迁移到 Yaml 管道的一个缺点是我们失去了“任务组”。顾名思义，任务组允许创建一组可以在多个管道中使用的任务，并且可以使用变量来设置属性。这样，我们可以在这些任务组中保存配方，在多个管道中使用它们，如果需要更改，我们可以在所有管道中同时进行。

补救的一个方法是使用模板将 yaml 管道文件分成多个文件。

模板并不复杂，它们是一个简单的 yaml 文件，您可以将它添加到您的“普通”yaml 管道文件(或子文件夹)中，该文件将定义作业定义的“步骤”部分。

使用和定义使用模板的管道的示例

```
azure-pipeline.ymljobs:
- job: demo build
  displayName: 'Build project'
  steps:
  - template: build-steps.yml build-steps.ymlsteps:
- checkout: self
  clean: true
  lfs: true- task: DotNetCoreCLI@2
  displayName: 'NuGet Restore'
  inputs:
    command: restore
    projects: '**/*.csproj'- task: DotNetCoreCLI@2
  displayName: Build all projects
  inputs:
    projects: '**/*.csproj'
    arguments: '--configuration $(buildConfiguration) --no-restore'
```

在不同项目之间共享步骤的一种方法是将“公共的”构建模板存储在存储库中，并使用 git 子模块将公共模板包含到所有项目中。

## 版本控制

对于那些包，我们使用 [GitHub 流](https://guides.github.com/introduction/flow/)并进行自动版本控制。简单地说，这意味着存储库的*主*分支被锁定，我们授权使用 PR 将工作推给它。分支上的工作要么是新特性，要么是错误修复。我们这样做是为了通过代码评审不断改进我们的代码。

当 PR 通过并且代码被合并到 master 时，我们自动创建一个 beta 版本。一旦我们对我们的版本有了信心，想要创建一个新的官方版本，我们就在 master 上创建一个标签，它触发一个新的构建并发布主版本。

为了自动化版本管理，我们使用 [GitVersion](https://gitversion.net/docs/) 。它是一个工具，根据当前检出的 git 分支的当前状态和一组可重写的规则，可以计算版本号。它可以从 [Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=gittools.usegitversion) 安装。它添加了一个 GitVersion 任务，该任务将运行计算并创建一系列包含不同“样式”版本的环境变量。然后，这些环境变量可用于构建步骤或更新程序集版本。

## 更新程序集版本

GitVersion 支持更新 AssemblyInfo 文件，但方式非常有限。它不支持修改现在存储在“新”csproj 文件中的程序集信息。

为了克服这个问题，我们使用另一个 DevOps 扩展，称为[assembly Info](https://marketplace.visualstudio.com/items?itemName=bleddynrichards.Assembly-Info-Task&targetId=81fedd7b-ac69-4753-8129-f37a56066d21&utm_source=vstsproduct&utm_medium=ExtHubManageList)。它提供了两个任务，一个用于 Net Framework(带有 AssemblyInfo 文件),一个用于新的 Net core / standard 项目。

您只需在管道中添加任务，并使用 GitVersion 的环境变量来修改文件信息或程序集信息。正如您在示例中看到的，我对`VersionNumber`和`FileVersionNumber`使用了`$(GitVersion.AssemblySemVer)`变量，但是`$(GitVersion.InformationalVersion)`包含了信息版本的 git 提交的 SHA1。

## 构建项目

构建过程非常标准，只需调用 nuget restore，然后进行构建。我使用通配符一次恢复/构建所有的 csproj。

## 发布构建二进制文件

神器有两种用途。首先，它们可以在构建结果页面上下载。这允许我们检查构建阶段是否设置正确。第二个用途是它们将在第二个阶段，即部署阶段被自动下载。

我正在将编译后的二进制文件复制到工件暂存目录中。这是完全可选的，您也可以忽略它，只发布 nupkg 文件。我喜欢在工件中复制二进制文件，因为这有助于确保构建按预期工作。

## 创建 nuget 包并发布

最后，我使用。Net Core CLI 任务来打包项目，并将其直接复制到工件暂存目录的子目录中。我还将`versionEnvVar`参数设置为`GitVersion.NuGetVersionV2`环境变量，使其版本与 nuget 版本控制方案兼容。

最后，我使用`PublishBuildArtifacts`步骤将两个文件夹作为工件发布。

## 部署

部署阶段被配置为仅在`build_pack`阶段成功且触发管道的提交在主分支上时触发。

它有一个单独的步骤，一个使用`push`命令的`NuGetCommand`将所有`.nupkg`文件推入一个内部 feed。

显然，对于公共 NuGet 包，您可能想要添加其他步骤或其他部署，例如基于一个标签来部署在公共 NuGet 提要上。

# 结论

如你所见，使用 Azure DevOps pipeline 来自动化 NuGet 包的部署相当容易。使用 GitVersion 还允许自动化版本号。使用模板也是简化在多个项目中重用同一部分管道的好方法。

在下一部分中，我们将看到一个使用 WiX 构建桌面应用程序、创建和部署安装程序的方法。

如果您有任何问题、意见或改进，请随时添加评论。