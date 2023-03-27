# 让戈朗的标签见鬼去吧

> 原文：<https://itnext.io/goto-hell-with-labels-in-golang-7f59a47ead8d?source=collection_archive---------1----------------------->

## Goto 经常被人诟病，但是标准库使用它们，我们能使用它们吗，它们有那么差吗？让我们探索和看看

![](img/93c053bcbb321e64baddfe6d0b09da67.png)

由[杰斯温·托马斯](https://unsplash.com/@jeswinthomas?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)在 [Unsplash](https://unsplash.com/s/photos/technology?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄的照片

许多编程语言都有某种`goto`语句。如果你不熟悉`goto`，它是一种将代码跳转到某个位置的方式。它可以用来控制执行流程。开发人员社区通常非常讨厌这种说法，因为它会导致难以遵循的代码。我一直很喜欢`goto`语句，我开始怀疑，它是不是用在了围棋的标准库中，并决定深入研究一下。

我见过很多不知道`goto`存在的围棋开发者。然而它确实如此，而且在标准库中使用得相当多。在标准库中进行简单的搜索就会发现，如果我们跳过注释和非 Go 文件，在 98 个不同的文件中**使用了`goto`语句 493 次。我们将在文章的后面看到其中的一些，让我们先了解一下这个语句是如何工作的。**

## Go 中的 Goto 语句

它的工作方式是你可以写下一个`label`，这是一个自定义的关键字，可以被`goto`语句识别。`label`是一个字符串，通过在标签后设置分号来创建。

```
// Go code
label: 
    // Code to execute in label
```

Go 在实现上有一些安全性，因为必须在与`goto`语句相同的函数中创建一个`label`。必须使用一个`label`,编译器会为你捕捉这个。如果你在`label`之后创建一个变量，情况也是如此，在`label`中使用的所有变量都必须在作用域中提前定义。

理解它的最好方法可能是看它，这里有一个超级简单的例子，展示了如何设置一个标签，然后跳转到它，跳过中间的任何代码。我们将创建一个执行 10 次的`for`循环，但是当`i`等于 5 时，我们将使用一个`goto`来中断执行。注意`label`必须在同一个函数中创建，否则编译器会失败。同样，如果不使用`label`，编译器将再次失败。在示例中，我将`label`命名为`exit`。

main.go 使用 goto 语句中断 for 循环。[尝试一下](https://play.golang.org/p/4lYpK9xGyux)

就是这样！没有别的魔法，就是这么简单。请记住，您不能跳转到同一函数之外的标签。这是一个很好的安全措施，消除了很多困惑。

尝试转到范围外的标签会崩溃— [在此尝试](https://play.golang.org/p/9TMlt_90g8D)。

需要注意的是，如果您的常规代码流遍历到标签中而没有返回，标签将被执行。如果在一个函数的末尾有多个标签，就会发生这种情况。

我将添加一条`if`语句来展示这一点，如果`I`计数器变为 6，它将打印出来。`I`永远不会变成 6 因为当它是 5 的时候我们会跳出`for`的循环，所以我们永远不会触发第二个标签，对吗？我第一次使用标签时，我以为它会这样工作，但它会运行第一个标签下的任何东西，就像正常的代码执行流。

确保不要落入多标签执行陷阱— [在此尝试](https://play.golang.org/p/psorRjkFdy6)

## 转到标准库中

让我们探索一下`goto`语句是如何在标准库中使用的，这样我们就可以更好地掌握它的用法。我将假设 std 库被认为是很好的用法。

我发现的一个很好的例子是`go/scanner`，它是一个用于扫描 Go 源代码的包。扫描文本文件可能是一项非常艰巨的任务，尤其是在你不知道里面有什么的情况下。通常使用一种叫做[词法分析](https://en.wikipedia.org/wiki/Lexical_analysis)的技术。我不会描述确切的内部工作方式，但是您可以创建称为标记的字符串，用于标识文本的当前位置。想象一下，你自己写一个文本扫描器来读取源代码并自己验证它，这不是一个简单的任务。

我们将要看的相当基础的部分是 Go 团队扫描源代码中的注释的部分。这个函数是`scanComment`，包含一个名为 exit 的`label`，就像我们之前使用的例子一样。

为了让代码更容易理解，我想给出一些背景。`s.ch`是当前被解析的字符。`s.next()`将下一个 Unicode 字符读入`s.ch`，当它是常规注释时，第一个`if`处有一个`goto`，当它是多行注释时，另一个`goto`。这是一个很好的例子，说明了如何跳过代码块而不使用多个标志来控制状态。他们可以不使用`goto`而是使用一些布尔函数，比如`isSingleLineComment`，如果这是真的，那么就跳过多行解析。但是他们避免了很多复杂的流程逻辑，而是使用了一个简单的`goto`，我喜欢这个解决方案。

第二个`goto`在解析多行注释时使用。在其中，他们不断迭代，直到找到一个标记为`*/`的注释闭包。这是一个循环，如果没有出现结束，它将一直运行到文件的末尾。当条件满足时，他们不使用`break`来取消`for`循环，而是使用`goto`来控制流量。它们也绕过用于设置发现错误的`s.error`。

go/scanner.scanComments —使用 goto 跳过状态标志的标准库示例

使用的第二个例子在`exprInternal`函数的`go/types`中，该函数用于核心转到类型检查。这是一个很好的例子，当我们有一个`switch`需要检查许多条件并且变得有点太大的时候。他们照常使用`continue`和`break`来管理流程，但是他们也使用`goto`来标准化错误处理和避免重复代码。

go/types/expr . go-使用 goto 进行标准化错误处理的示例

## 在 Go 中何时使用 goto

利用我们从标准库中获得的经验，我们可以看到用法有时是有意义的。考虑到 Go 只允许在同一个函数中使用标签，我认为这不会让代码难以理解。

我会使用它的场合是当常规的流量控制机制无法在不变得复杂的情况下削减它时，这里是一些案例的要点。

*   避免像`go/scanner`中那样存储状态和跳过代码块的条件标志
*   标准化退出计划，以防你有一个`switch`或多个`for`循环，都可能导致相同的标准退出，如`go/types/expr`
*   通过使用`goto`避免嵌套循环，需要嵌套`for`循环的复杂逻辑可以变得更容易阅读和理解
*   如`go/types/expr`中的`Error`标签一样，通过处理标签中的公共模式来避免重复代码

你对`goto`的说法有什么看法？你认为它增加了复杂性和混乱，还是你认为它是处理流程的一个好的解决方案？

本文到此为止，感谢您的阅读。和往常一样，我很欣赏反馈，或者只是接触，如果你有问题。你可以在 medium [twitter](https://twitter.com/percybolmer) 或 [linkedin](https://www.linkedin.com/in/percy-bolmer-bb223b122/) 找到我。

如果你喜欢我的作品，请随时关注我。我最近在 Go 中写了关于结构化日志的文章。

[](https://betterprogramming.pub/how-to-use-structured-json-logging-in-golang-applications-7fc5e2751dbd) [## 如何在 Golang 应用程序中使用结构化 JSON 日志记录

### 结构化日志对于软件调试非常重要。很高兴，在 Golang 中实现起来非常容易](https://betterprogramming.pub/how-to-use-structured-json-logging-in-golang-applications-7fc5e2751dbd) [](https://percybolmer.medium.com/membership) [## 阅读珀西·博尔默(以及媒体上成千上万的其他作家)的每一个故事

### 作为一个媒体会员，你的会员费的一部分会给你阅读的作家，你可以完全接触到每一个故事…

percybolmer.medium.com](https://percybolmer.medium.com/membership)