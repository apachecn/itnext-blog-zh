# 编写可维护的 PowerShell

> 原文：<https://itnext.io/writing-maintainable-powershell-503e5b680ed9?source=collection_archive---------0----------------------->

## 应用流行的模式使我们的代码更易维护

*第五部分* [*声明性 DevOps 微框架*](https://medium.com/@cjkuech/declarative-devops-microframeworks-9908c8d05332)

PowerShell 比大多数语言更难在大型代码库中管理。虽然 REST APIs 和应用程序有你可以遵循的标准设计模式，但这并不一定适用于 DevOps 代码库。因此，许多代码库在面向用户的界面和内部逻辑之间缺乏强有力的分离，从而变得难以阅读和维护。我们可以利用其他语言中的一些常见模式与 PowerShell 一起工作，以保持大型 PowerShell 代码库的可维护性。

![](img/6acd269cc27ae06aa7f5642d536d540c.png)

大事情最容易分成小部分来处理(照片由 Raphael Koh 在 Unsplash 上拍摄)

# 手动音量调节

[MVC](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller) 对于大型面向用户的应用程序来说是一种非常常见的软件工程模式，无论它们是[ASP.NET 网站](https://dotnet.microsoft.com/apps/aspnet/mvc)还是 iOS 应用程序。MVC 通过将应用程序代码划分为模型、视图和控制器，将表示逻辑与业务逻辑完全分离。就像图形用户界面一样，命令行界面也可以从这种分离中受益。

## 模型

模型是表示应用程序核心结构的模式化数据对象。它们可以用于存储业务逻辑的状态，或者包含与模型直接相关的业务逻辑。它永远不能包含表示逻辑，除了一个`ToString()`方法。

在 PowerShell 中，我们通常将模型定义为类。[类可以让你的代码更加直观](https://medium.com/@cjkuech/new-to-powershell-use-classes-ab7b1e6f72ec)并且让开发者能够[更好的对与他们的模型直接相关的代码进行分组](https://medium.com/@cjkuech/functional-powershell-with-classes-820c8e9acd8f)。它们还支持[验证属性](https://medium.com/@cjkuech/defensive-powershell-with-validation-attributes-8e7303e179fd)，帮助你最小化代码。

## 视图

视图只包含与向最终用户展示模型相关的代码。在 GUI 中，视图可以是按钮、页面、文本编辑器或任何其他面向用户的 UI 元素。视图只能接收和显示模型，或者调用控制器——它们不能直接更新模型。

在 PowerShell 中，视图是 Cmdlets。Cmdlets 允许开发人员以声明方式[为他们的功能](https://medium.com/@cjkuech/defensive-powershell-with-validation-attributes-8e7303e179fd)定义[富于表现力的面向用户的命令行界面](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_functions_advanced_methods)，并通过将`[CmdletBinding()]`属性应用到他们的`Param`块来添加对类似于`-ErrorAction`参数的特性的支持。视图是从我们的模块中导出的唯一 cmdlets。

有两类视图 cmdlets

*   **数据视图**—数据视图输出模型。不能从 PowerShell 中的模块干净地导出类，所以我们需要使用数据视图函数来抽象任何类构造函数或静态类方法调用。数据视图应该干净地与管道集成，如果可能的话，遵循[功能范例](https://medium.com/@cjkuech/functional-programming-in-powershell-876edde1aadb)，并且在运行时不会对执行产生副作用。在下面的例子中，`Test-Node`是一个数据视图。
*   **用户视图** —用户视图不是输出值，而是向用户提供用户友好的格式化输出，例如使用`Format-Table`或`Write-Host`。尽管`Write-Host`通常被 PSScriptAnalyzer 禁止，但是`Write-Host`在用户视图中使用是可以接受的，因为用户视图应该只由用户直接运行。任何与用户的交互，例如与`Read-Host`的交互，必须包含在用户视图中，以及任何进度指示器或其他 UI 元素中。在下面的例子中，`Start-Troubleshooter`是一个用户视图。

## 控制器

为了确保表示逻辑和业务逻辑完全分离，视图永远不能修改模型，只能与控制器交互。控制器是模型和视图之间的粘合剂。

视图很难进行单元测试，因为它们是为人类而不是计算机优化的，并且有许多可能的参数组合。如果我们干净地将业务逻辑分成具有更简单接口的控制器和模型，我们可以向控制器和模型添加单元测试，同时主要依靠声明性验证属性来最小化视图代码中的错误。我们还可以通过将输入验证合并到视图中，并设计我们的控制器来假设来自视图的有效输入，从而简化我们的控制器和模型代码。

在 PowerShell 中，控制器被优化定义为函数。为了区分视图函数和控制器函数，使用 param list(`function MyFunc([int]$x, [string]$y) {`)而不是`Param`块来定义控制器函数，并使用 PascalCase 或 camelCase 来区分控制器和动词名词命名的视图。控制器的简化界面将迫使你防御性地编程，并在你的视图中尽早验证输入。参数列表也将迫使你保持控制器接口尽可能简单，以最小化你的控制器测试覆盖所需的单元测试用例的数量。

# 模块

模块是组织代码的一种方式。模块被一次导入到你的会话中(除非你使用`-Force`)，所以你可以避免由于重新声明代码而损失的执行时间。模块还允许您在模块范围的变量中存储状态。因此，模块相当于纯面向对象语言(如 C#和 Java)中用于管理代码的静态类。[将 PowerShell 放入私有模块](https://medium.com/@cjkuech/private-powershell-modules-76f51a1bf893)将迫使你将代码分解成更易管理的部分，这也可以推动你将代码抽象成[微框架](https://medium.com/@cjkuech/declarative-devops-microframeworks-9908c8d05332)，并确保你的代码具有定义良好的接口。因此，模块有一些坚实的优势—

## 可试验的

模块具有定义良好的导出 cmdlets 接口。接口可以很容易地用 Pester 测试来维护和验证。相比之下，没有模块实现的等效代码不会保持严格的接口，并且将不可避免地开始与代码库的其余部分纠缠在一起，由于运行该函数所需的难以再现的状态，以及由于运行该函数而导致的难以验证的副作用，使得测试变得困难。

## 轻便的

模块与你的代码库的其余部分相隔离，确保关注点的分离。因此，它们可以被[便携地复制并在系统的其他地方](https://medium.com/@cjkuech/private-powershell-modules-76f51a1bf893)使用。随着代码库的发展，模块也可以被干净地替换——模块迫使您为您单独关心的代码创建一个定义良好的接口，因此如果您需要删除它，您完全知道如何替换它。

## 比脚本更好

模块还支持按名称导入，因此可以避免硬编码相对路径:`Import-Module MyModule`。相反，必须用`&"$PSScriptRoot\path\to\MyScript.ps1"`调用脚本，将脚本耦合到它的相对路径。

接受用户输入的脚本必须使用`Param`块作为脚本中的第一个代码，禁止声明`enum`或其他可能简化脚本参数的类型。如果运行多次，它们的效率也很低，因为每次运行脚本时，都必须重新定义内部定义的任何帮助函数或其他常量值。人们可能会尝试将下面的代码实现为一个单独的`Get-BadNode`脚本，但是`$someDynamicSetting`和`myHelperFunction`会在每次调用时被初始化，这不必要地损害了性能；此外，`Get-BadNode`的脚本实现不支持使用`enum`来帮助完成`-Values`参数的值。

# 后续步骤

开始将这些概念应用到您自己的代码库中—

*   将您的大型脚本分解成业务逻辑和表示逻辑。
*   将您的脚本分组分解成具有良好定义的接口的模块。对这些接口进行单元测试。
*   识别代码中难以测试或经常需要手动测试的区域，将难以测试的接口从核心中分离出来，并围绕核心添加单元测试。
*   考虑如何使您的演示视图 cmdlets 更加用户友好。
*   开始将你的代码模块化成[声明性 DevOps 微框架](https://medium.com/@cjkuech/declarative-devops-microframeworks-9908c8d05332)。