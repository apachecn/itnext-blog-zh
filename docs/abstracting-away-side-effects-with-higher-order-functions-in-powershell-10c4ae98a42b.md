# 用 PowerShell 中的高阶函数抽象出副作用

> 原文：<https://itnext.io/abstracting-away-side-effects-with-higher-order-functions-in-powershell-10c4ae98a42b?source=collection_archive---------4----------------------->

## 使用功能抽象的更干净、更安全的 PowerShell 代码

我通常带着对状态和副作用的病态厌恶来写 PowerShell。实际上，这些不受欢迎的模式通常是不可避免的，尤其是在 DevOps 中。像`git`和编译器这样不可避免的 CLI 要求有状态外壳光标在 CLI 被调用之前移动到不同的目录；必须维护可变集合；集成测试和基础设施代码通常需要在云中提供临时资源——所有这些操作都会改变我们系统的状态，并随后将系统置于未知、难以管理的状态。

在本文中，我们回顾了一些利用 [*高阶函数*](https://medium.com/swlh/functional-programming-in-powershell-876edde1aadb) 从我们的脚本中抽象出副作用管理的例子。

![](img/a9edbd5c276a4d852f23b4fed27a3e55.png)

保持无国籍，我的朋友。

# 调用间接目录

## 问题是

通常在 DevOps 中，您需要在特定的目录中运行命令。例如，`git`(与大多数 PowerShell 命令不同)不接受目录路径参数，并期望您在执行之前`cd`到正确的目录。类似的，应用建设 CLIs 有`npm`、`docker`、`dotnet`等。，通常在根目录下运行效果最好。

我们总是可以将`Push-Location`或`Set-Location`放入我们需要的目录中，但是当脚本完成时，我们总是要记得返回。更糟糕的是，我们需要添加样板`try` / `finally`语句，以防脚本失败——否则，运行脚本的人会意外地发现他们的终端位于与预期不同的位置。

## 解决方案

我们可以编写一个名为`Invoke-InDirectory`的[高阶函数](https://medium.com/swlh/functional-programming-in-powershell-876edde1aadb)来抽象出调用给定目录中的命令所需的样板文件，而不会产生副作用(如果命令出错，终端将位于意外的位置)。

现在我们可以用[声明性地编写 *what*](/declarative-devops-30788ddd43cd) (调用目录中的命令)而不是 *how* ( `try` / `finally`语句和`Push-Location` / `Pop-Location`调用)。

# 在上下文中调用

## 问题是

在大型 PowerShell 脚本中，脚本输出可能会变得一塌糊涂。几乎不可能准确地说出哪一行代码发出了给定的日志行，或者哪一行抛出了错误。

## 解决方案

我们可以实现一个上下文记录器，它维护一个上下文字符串堆栈，并在脚本发出输出时将它们记录下来，这样您就总能确切地知道是哪个上下文块抛出了错误。为此，我通常使用`[Requirements](https://github.com/microsoft/Requirements)`框架；然而，我们可以在单个函数中实现相同的模式，维护一个堆栈来跟踪我们在程序中的位置。

现在，我们可以将一致的日志记录添加到我们的脚本中，例如本例中的 React 应用程序构建脚本。

所有记录的输出都将以时间戳和上下文堆栈为前缀，您可以很容易地将它们与代码中的`Invoke-InContext`调用树关联起来。

# 使用<insert your="" resource="">调用</insert>

## 问题是

在部署服务时，您通常想要分配临时存储空间来存放工件，但是在部署之后，应该释放这个空间以节省成本。实现提供和删除存储的代码可能很繁琐，如果您的脚本不处理故障，您的存储可以保持分配。

这个问题的一般情况在 DevOps 中经常出现。例如，集成测试管道可能希望在运行集成测试之前构建昂贵的生产基础设施资源，然后在测试完成后清除这些资源。

## 解决方案

我们不像前面两个例子那样使用高阶函数模式来维护堆栈，而是在 scriptblock 执行完毕后使用我们的模式来运行清理代码。在这个例子中，我们创建了一个函数，它自动向用户提供一个新的存储上下文。用户不需要了解如何创建存储帐户和管理清理，这都是抽象的。

# 后续步骤

既然您已经看到了简化代码的高阶抽象的实际例子，那么请深入理解 PowerShell 中的[函数式编程，以便在您的代码库中更深入地应用这些概念。你也可以利用现有的框架，比如依赖于这种模式的](https://medium.com/swlh/functional-programming-in-powershell-876edde1aadb)`[Requirements](https://github.com/microsoft/Requirements)`和 Pester。