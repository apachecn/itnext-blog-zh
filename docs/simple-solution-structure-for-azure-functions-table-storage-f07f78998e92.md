# Azure 函数和表存储的简单解决方案结构

> 原文：<https://itnext.io/simple-solution-structure-for-azure-functions-table-storage-f07f78998e92?source=collection_archive---------1----------------------->

![](img/09ebe4e72fbba7b13e7bf65b050411e5.png)

由[万花筒](https://unsplash.com/@kaleidico?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

自从我开始倾向于 Azure，我发现创建和管理资源是多么容易。然而，我不得不承认，云上的大多数选项对于大中型企业来说都是负担得起的。每个云提供商也为他们的一些服务提供免费层，但他们并不打算持有重要的东西(他们中的大多数是出于沙盒的目的)。

我希望我的项目能够轻松实现云计算。并且，我认为如果我使用 **Azure Functions** 作为 Web Api & **表存储**作为数据库，我的钱包会很安全。但是我也不想依赖 Azure。因此，万一出现意外情况，我需要能够将我的数据库和应用程序切换到其他地方。

我基本上实现了一个类似于*干净架构的*结构，将业务从数据库和 API 端点中分离出来。

> **TLDR；**这篇文章解释了一个解决方案结构，我使用来实现 Azure 功能，使用表存储作为基于 Clean 架构和中介模式的数据库。

*→关于“清洁架构”的详细信息，请查看 Github 上的这个* [*回购*](https://github.com/jasontaylordev/CleanArchitecture)

最后，我计划有一个这样的解决方案:

```
Solution
|--App.Functions
|--App.Application
|--App.Domain
|--App.Infrastructure
```

## 1) App。功能

将名为`App.Functions`的新 **Azure Function** 项目添加到解决方案中。在向导中选择`Http Trigger`。它将为我们创建一个示例函数。但是我们需要为解决方案做一些安排。

**—依赖注入**

一如既往，我们需要做的第一件事是准备依赖注入。在模板中，没有启动类。首先，我们要创建一个名为`Startup.cs`的文件

看看下面的例子:

属性`[assembly:FunctionStartup(typeof(App.Function.Startup))]`告诉应用程序这个类将被用作初始化器(web API 项目中的`startup.cs` 也是如此)。

您可以在`builder.Services`对象上使用`.AddScoped<>`、`.AddSingletion<>`和其他方法来准备您的服务。我们将在接下来的步骤中装饰这个类。

**—中介**

MediatR 是一个帮助我们在 Clean 架构中实现 CQRS 模式的库。我们将定义命令、查询&处理程序来实现应用程序业务。

使用 NuGet 包管理器或使用 dotnet CLI 添加包`MediaTr`

```
dotnet add package MediatR
```

## 2) App。应用

我们将使用`App.Application`项目来支持我们的应用业务。这个项目将保留 MediatR 对象的实现。

创建一个名为`DIConfig.cs`的类，创建一个扩展方法来注册服务&mediator 对象。

添加这些库以使用 MediatR

```
dotnet add package MediatR
dotnet add package MediatR.Extensions.Microsoft.DependencyInjection
```

它应该看起来像下面这样。

第 9 行:向服务注册 MediatR 元素。(需要安装`MediatR.Extensions.Microsoft.DependencyInjection`包才能使用`.AddMediatR`扩展。

然后使用这个扩展将`App.Application`注册到函数中。

## 3) App。基础设施

我们将在这里保留表存储库，并且我们将在这里使用 *Azure 表存储*。

将以下包添加到您的项目中

```
dotnet add package Azure.Data.Tables
```

表存储的基本用法如下所示:

```
**TableServiceClient** serviceClient 
                = new **TableServiceClient**("<connection-string>");var **tableClient** = client.**GetTableClient**("<table-name>");tableClient.AddEntity<User>(user);
```

但是我们要让它看起来很好&有条理。

首先，创建一个`DIConfig.cs`来注册服务&准备`TableServiceClient`

然后，我将为*表存储*创建一个通用存储库

## 4) App。领域

正如你所猜测的，这是我们开赌场的地方。我们在这里没有什么花招。

我只有这样的`EntityBase.cs`和`TRef.cs`类型:

`EntityBase`保持从 ITableEntity 实现的属性。我不打算使用 API 模型& DTO 在这里。我最期望的是，我可能需要一个用于表示层的 API 模型&你知道这个练习的其余部分。

`TRef`只是防止我在一个关系的情况下拥有两个不同的属性。

**—结论**

也许创建一个分层结构是不合适的，因为 azure 函数的原因是保持它的简单&原子性。但正如我在开始时所说的，这是关于使用云的能力和灵活性，如果它以某种方式开始威胁我的钱包，我必须能够从功能切换到 web API &开始使用另一个数据库。这只是我的实践，我想解释一下。

你可以在这个 [GitHub 库](https://github.com/mehmetural/FunctionDemo)上查看我的演示。并且，如果你想使用这个结构，你可以在 Visual Studio 上安装[这个解决方案模板](https://github.com/mehmetural/FunctionDemo/blob/master/SolutionTemplate.zip)来启动你自己的项目(1)。

(1)安装 zip 文件并将其移动到 `C:\Users<your-user>\Documents\Visual Studio 2022\Templates\ProjectTemplates`，然后搜索“*Clean architecture for Azure functions*”。在一些默认模板之后应该会看到。