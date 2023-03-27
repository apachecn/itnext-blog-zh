# PowerShell 新手？使用类

> 原文：<https://itnext.io/new-to-powershell-use-classes-ab7b1e6f72ec?source=collection_archive---------7----------------------->

## 使用类避免 PowerShell 的常见缺陷

*在继续之前，先学习使用和定义类的基本语义:* [*关于类*](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_classes)

PowerShell 具有强大但有时不直观的编程功能，这可能会让新用户感到头疼。这些特性使 PowerShell 成为几乎所有 DevOps 任务的理想候选，但经常会威胁 PowerShell 开发新手选择更麻烦的语言，如 C#或 Python。通过使用类，新的 PowerShell 开发人员可以迈出他们进入 PowerShell 生态系统的第一步，而不会从不受欢迎的惊喜中退缩

# 类比函数简单？

在大多数语言中，函数比类简单。静态类基本上只是组织函数的方法，但是 PowerShell 已经支持用于组织函数的模块，所以当我们可以使用模块和函数时，为什么我们要使用静态类呢？

![](img/1174a062d847e9a645c973e442745ea6.png)

使用静态方法和函数的平凡函数的等效实现

## 简化输入

PowerShell 功能极其强大。通过几行参数属性，您可以为您的函数拥有一个全功能的命令行界面。这些接口非常适合面向用户的功能，但是这种面向用户的表示逻辑不应该保留在我们的核心实现逻辑中。

方法输入要简单得多，默认情况下每个参数都是强制性的，因此可以保持内部业务逻辑简单。通过隔离业务逻辑，您可以只围绕您的核心业务逻辑实现[单元测试](https://github.com/pester/Pester)，并依靠声明性参数验证来保证您的表示性 cmdlets 上的代码健康。

## 简化输出

对于学习 PowerShell 的人来说，最大的惊喜之一是`return`语句并不是唯一从函数中输出值的语句——每个非 void 表达式都向输出中写入一个值。在方法中，只有`return`语句输出一个值，这使得它们更容易测试、调试和理解。

方法还支持方法头中的简单返回类型声明，确保您的函数总是返回您期望的类型(或者抛出类型错误)。

# 毕业

因此，您已经使用静态类完成了最后期限，现在您已经熟悉了基本的 PowerShell 语义，并准备将您的脚本提升一个档次。以下是你接下来如何成长的方法。

## 深入研究函数

我们避免使用函数，因为它们处理参数和返回的方法可能会令人困惑，但是一旦您了解了这些特性的细微差别，它们会极大地提升您的代码。

*   了解[函数参数](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_parameters)，这样您就可以快速地以声明方式创建强大的命令行界面，或者通过参数验证使您的脚本更加稳定。
*   不要把输出看作奇异值，把它们看作一个流。学习如何[重定向流](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_redirection)并在[管道](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_pipelines)中对其进行操作。

## 让课程更上一层楼

课程不仅仅是给初学者的。学会利用高级特性，比如通过类实现的隐式和显式类型转换函数，这样你就可以用类来扩展你的功能代码。