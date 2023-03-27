# 我在 Go 项目中见过的最常见的 10 个错误

> 原文：<https://itnext.io/the-top-10-most-common-mistakes-ive-seen-in-go-projects-4b79d4f6cd65?source=collection_archive---------0----------------------->

![](img/6bf87105470b9f96526d5e96e123d6ea.png)

信用:[teachertoolkit.co.uk](http://teachertoolkit.co.uk)

# 开始前的注意事项

随着这篇文章的成功，我想到这样的信息可以很好地适合这本书。因此，这是我未来的书: **100 个 Go 错误**。它将于 2022 年出版，但已经可以作为 MEAP (Manning Early Access Program)使用:

[](https://www.manning.com/books/100-go-mistakes-how-to-avoid-them) [## 100 个 Go 错误:如何避免它们

### 在你的 Go 代码中发现你自己都不知道的错误，并通过避免常见的…

www.manning.com](https://www.manning.com/books/100-go-mistakes-how-to-avoid-them) 

# 介绍

这篇文章是我在围棋项目中看到的最常见的错误列表。顺序并不重要。

# 未知的枚举值

让我们看一个简单的例子:

这里，我们使用`iota`创建了一个 enum，它产生了以下状态:

```
StatusOpen = 0
StatusClosed = 1
StatusUnknown = 2
```

现在，让我们假设这个`Status`类型是 JSON 请求的一部分，并且将被编组/解组。我们可以设计以下结构:

然后，收到这样的请求:

在这里，没什么特别的，地位会被取消安排到`StatusOpen`吧？

然而，让我们来看另一个请求，其中没有设置状态值(无论出于什么原因):

在这种情况下，`Request`结构的`Status`字段将被初始化为其**零值**(对于`uint32`类型:0)。所以，`StatusOpen`而不是`StatusUnknown`。

最佳实践是将枚举的未知值设置为 0:

这里，如果状态不是 JSON 请求的一部分，它将被初始化为`StatusUnknown`，正如我们所期望的。

# 标杆管理

正确的基准很难。有很多因素会影响给定的结果。

一个常见的错误就是被一些编译器优化忽悠了。让我们从[*tei vah/bit vector*](https://github.com/teivah/bitvector/)库中取一个具体的例子:

该函数清除给定范围内的位。为了使它成为长凳，我们可能要这样做:

在这个基准测试中，编译器会注意到`clear`是一个叶函数(没有调用任何其他函数)，所以它会**内联**它。一旦它被内联，它也会注意到没有副作用。因此`clear`调用将被简单地删除，导致不准确的结果。

一种方法是将结果设置为全局变量，如下所示:

这里，编译器不知道调用是否会产生副作用。因此，基准将是准确的。

## 进一步阅读

[](https://dave.cheney.net/high-performance-go-workshop/dotgo-paris.html#watch_out_for_compiler_optimisations) [## 高性能围棋研讨会

### 注意编译器优化

dave.cheney.net](https://dave.cheney.net/high-performance-go-workshop/dotgo-paris.html#watch_out_for_compiler_optimisations) 

# 指针！到处都是指针！

通过值传递变量将创建该变量的副本。而通过指针传递只会复制内存地址。

因此，传递指针总是比**快**，不是吗？

如果你相信这一点，请看看[这个例子](https://gist.github.com/teivah/a32a8e9039314a48f03538f3f9535537)。这是一个基于 0.3 KB 数据结构的基准测试，我们通过指针然后通过值来传递和接收。0.3 KB 并不算大，但应该与我们每天看到的数据结构类型相差不远(对我们大多数人来说)。

当我在本地环境中执行这些基准测试时，通过值传递比通过指针传递快 4 倍多。这可能有点违反直觉，对吗？

对这一结果的解释与围棋如何管理内存有关。我不能像威廉·肯尼迪那样出色地解释它，但是让我们试着总结一下。

可以在**堆**或**栈**上分配变量。作为草稿:

*   堆栈包含给定 **goroutine** 的**正在进行的**变量。一旦函数返回，变量就会从堆栈中弹出。
*   堆中包含了**共享的**变量(全局变量等。).

让我们看一个简单的例子，在这里我们返回一个值:

这里，当前的 goroutine 创建了一个`result`变量。这个变量被压入当前堆栈。一旦函数返回，客户端将收到该变量的副本。变量本身从堆栈中弹出。它仍然存在于内存中，直到被另一个变量擦除，但是它**不能再被访问**。

现在，同样的例子，但是有一个指针:

`result`变量仍然由当前的 goroutine 创建，但是客户端将收到一个指针(变量地址的副本)。如果`result`变量从堆栈中弹出，这个函数**的客户端就不能再访问它了。**

在这种情况下，Go 编译器会将**变量`result`转义**到一个变量可以共享的地方:**堆**。

然而，传递指针是另一种情况。例如:

因为我们在同一个 goroutine 中调用`f`，所以不需要对`p`变量进行转义。它被简单地推到堆栈中，子函数可以访问它。

例如，这是在`io.Reader`的`Read`方法中接收切片而不是返回切片的直接结果。返回一个片(这是一个指针)会将它转义到堆中。

那为什么栈这么**快**呢？有两个主要原因:

*   堆栈不需要有一个**垃圾收集器**。正如我们所说的，变量一旦创建就被简单地推入，然后一旦函数返回就从堆栈中弹出。不需要一个复杂的过程来回收未使用的变量等等。
*   一个堆栈属于一个 goroutine，因此与在堆上存储变量相比，存储变量不需要**同步**。这也导致了性能的提高。

作为结论，当我们创建一个函数时，我们的默认行为应该是使用**值而不是指针**。只有当我们想要**共享**一个变量时，才应该使用指针。

然后，如果我们遇到性能问题，一个可能的优化是检查指针在某些特定情况下是否有用。使用下面的命令:`go build -gcflags "-m -m"`可以知道编译器何时将变量转义到堆中。

但是，对于我们的大多数日常用例来说，值是最合适的。

## **延伸阅读**

[](https://www.ardanlabs.com/blog/2017/05/language-mechanics-on-stacks-and-pointers.html) [## 栈和指针的语言机制

### 前奏这是一个四部分系列的第一篇文章，将提供对机械和设计的理解…

www.ardanlabs.com](https://www.ardanlabs.com/blog/2017/05/language-mechanics-on-stacks-and-pointers.html) 

# 中断 for/switch 或 for/select

如果`f`返回 true，下面的例子会发生什么？

我们将调用`break`语句。然而，这将中断`switch`语句，**而不是 for 循环**。

相同的问题:

`break`与`select`语句有关，与 for 循环无关。

断开 for/switch 或 for/select 的一个可能的解决方案是使用标记为 break 的**，如下所示:**

# 错误管理

围棋处理错误的方式还有点年轻。如果这是 Go 2 最值得期待的特性之一，那就不是巧合了。

当前的标准库(在 Go 1.13 之前)只提供了构造错误的函数，所以你可能想看看 [*pkg/errors*](https://github.com/pkg/errors) (如果还没有这样做的话)。

这个库很好地遵循了下面的经验法则，而这条法则并不总是被遵守:

*一个错误应该只处理* ***一次*** *。记录错误* ***就是*** *处理错误。因此，错误应该***被记录或传播。**

*对于当前的标准库，很难做到这一点，因为我们可能想给错误添加一些上下文，并有某种形式的层次结构。*

*让我们看一个例子，看看 REST 调用会导致什么样的 DB 问题:*

```
*unable to serve HTTP POST request for customer 1234
 |_ unable to insert customer contract abcd
     |_ unable to commit transaction*
```

*如果我们使用 *pkg/errors* ，我们可以这样做:*

*初始错误(如果不是外部库返回的)可以用`errors.New`创建。中间层`insert`通过添加更多的上下文来包装这个错误。然后，父进程通过记录错误来处理它。每个级别要么返回错误，要么处理错误。*

*例如，我们可能还想检查错误原因本身，以实现重试。假设我们有一个来自外部库的`db`包来处理数据库访问。这个库可能会返回一个名为`db.DBError`的暂时错误。为了确定我们是否需要重试，我们必须检查错误原因:*

*这是使用`errors.Cause`完成的，它也来自 *pkg/errors* :*

*我见过的一个常见错误是部分使用 *pkg/errors* 。例如，检查错误是这样完成的:*

*在本例中，如果`db.DBError`被包装，它将永远不会触发重试。*

 *[## 不要只是检查错误，要优雅地处理它们

### 这篇文章摘自我最近在日本东京举行的 GoCon 春季会议上的演讲。我花了很多…

dave.cheney.net](https://dave.cheney.net/2016/04/27/dont-just-check-errors-handle-them-gracefully)* 

# *切片初始化*

*有时，我们知道切片的最终长度。例如，假设我们想要将一个切片`Foo`转换为一个切片`Bar`，这意味着两个切片将具有相同的长度。*

*我经常看到切片这样初始化:*

*切片不是一个神奇的结构。在引擎盖下，如果没有更多可用空间，它会实施一个**增长**战略。在这种情况下，会自动创建一个新数组(具有更大的容量[)并复制所有项目。](https://www.darkcoding.net/software/go-how-slices-grow/)*

*现在，假设我们需要多次重复这个增长操作，因为我们的`[]Foo`包含数千个元素？插入的分摊时间复杂度(平均值)将保持为 O(1)，但实际上，它将对性能产生**影响**。*

*因此，如果我们知道最终长度，我们可以:*

*   *用预定义的长度初始化它:*

*   *或者用 0 长度和预定义的容量初始化它:*

*最好的选择是什么？第一个稍微快一点。然而，您可能更喜欢第二种方法，因为它可以使事情更加一致:不管我们是否知道初始大小，在片的末尾添加元素都是使用`append`来完成的。*

# *上下文管理*

*`context.Context`经常被开发者误解。根据官方文件:*

> *上下文携带截止日期、取消信号和其他跨 API 边界的值。*

*这种描述太普通了，以至于有些人不知道为什么以及如何使用它。*

*让我们试着详述一下。上下文可以承载:*

*   *一个**期限**。它表示持续时间(例如 250 ms)或日期时间(例如`2019-01-08 01:00:00`)，如果达到该时间，我们认为必须取消正在进行的活动(I/O 请求、等待通道输入等)。).*
*   *一个**取消信号**(基本上是一个`<-chan struct{}`)。在这里，行为是相似的。一旦我们收到信号，我们必须停止正在进行的活动。例如，假设我们收到两个请求。一个用来插入一些数据，另一个用来取消第一个请求(因为它不再相关了)。这可以通过在第一个调用中使用可取消的上下文来实现，一旦我们得到第二个请求，这个上下文就会被取消。*
*   *键/值的列表(都基于一个`interface{}`类型)。*

*补充两点。首先，上下文是**可组合的**。因此，我们可以有一个上下文来承载一个截止日期和一个键/值列表。此外，多个 goroutines 可以**共享**相同的上下文，因此取消信号可以潜在地停止**多个活动**。*

*回到我们的话题，这里有一个我看到的具体错误。*

*一个 Go 应用程序是基于 [*urfave/cli*](https://github.com/urfave/cli) (如果你不知道，这是一个在 Go 中创建命令行应用程序的很好的库)。一旦开始，开发人员*从某种应用程序上下文继承*。这意味着当应用程序停止时，库将使用这个上下文来发送一个取消信号。*

*我所经历的是，这个上下文是在调用 gRPC 端点时直接传递的。这是**而不是**我们想要做的。*

*相反，我们希望向 gRPC 库指示:*请在应用程序停止时或 100 毫秒后取消请求。**

*为了实现这一点，我们可以简单地创建一个组合上下文。如果`parent`是应用上下文的名称(由*ur save/CLI*创建)，那么我们可以简单地这样做:*

*上下文理解起来并不复杂，在我看来，这是这种语言最好的特点之一。*

## *进一步阅读*

*[](http://p.agnihotry.com/post/understanding_the_context_package_in_golang/) [## 了解 golang 中的上下文包

### go 中的上下文包在与 API 和慢流程交互时可以派上用场，特别是在…

p.agnihotry.com](http://p.agnihotry.com/post/understanding_the_context_package_in_golang/)  [## gRPC 和截止日期

### 当您使用 gRPC 时，gRPC 库负责通信、编组、解组和截止日期强制…

grpc.io](https://grpc.io/blog/deadlines/) 

# 不使用-race 选项

我经常看到的一个错误是在没有`-race`选项的情况下测试 Go 应用程序。

正如这份[报告](https://blog.acolyer.org/2019/05/17/understanding-real-world-concurrency-bugs-in-go/)中所描述的，尽管 Go 是*“为了让并发编程更容易、更不容易出错而设计的*”，但我们仍然饱受并发问题的困扰。

显然，Go race 检测器不能解决所有的并发问题。然而，它是**有价值的**工具，我们应该在测试我们的应用程序时总是启用它。

## 进一步阅读

[](https://medium.com/@val_deleplace/does-the-race-detector-catch-all-data-races-1afed51d57fb) [## Go race 检测器能捕获所有的数据竞争错误吗？

### TL；DR:当数据竞争情况发生时，它会检测到这些情况。

medium.com](https://medium.com/@val_deleplace/does-the-race-detector-catch-all-data-races-1afed51d57fb) 

# 使用文件名作为输入

另一个常见的错误是将文件名传递给函数。

假设我们必须实现一个函数来计算文件中空行的数量。最自然的实现应该是这样的:

`filename`作为输入给出，所以我们打开它，然后实现我们的逻辑，对吗？

现在，假设我们想在这个函数的基础上实现**单元测试**来测试一个普通文件、一个空文件、一个不同编码类型的文件等等。它很容易变得非常难以管理。

此外，如果我们想要实现相同的逻辑，但是对于一个 HTTP 主体，例如，我们将不得不为此创建另一个函数。

Go 附带了两个伟大的抽象:`io.Reader`和`io.Writer`。我们可以简单地传递一个`io.Reader`来代替传递文件名，这个`io.Reader`将**抽象出**数据源。

是文件吗？一个 HTTP 主体？字节缓冲区？这并不重要，因为我们仍将使用相同的`Read`方法。

在我们的例子中，我们甚至可以缓冲输入来逐行读取它。所以，我们可以用`bufio.Reader`和它的`ReadLine`方法:

打开文件本身的责任现在委托给了`count`客户:

在第二种实现中，不管实际的数据源是什么，都可以调用函数**。同时，这将**促进**我们的单元测试，因为我们可以简单地从`string`创建一个`bufio.Reader`:**

# Goroutines 和循环变量

我见过的最后一个常见错误是在循环变量中使用 goroutines。

以下示例的输出是什么？

`1 2 3`不分先后？没有。

在这个例子中，每个 goroutine **共享**同一个变量实例，所以它将产生`3 3 3`(最有可能)。

有两种方法可以解决这个问题。第一个是将变量`i`的值传递给闭包(内部函数):

第二个是在 for 循环范围内创建另一个变量:

调用`i := i`可能看起来有点奇怪，但它完全有效。在循环中意味着在另一个作用域中。所以`i := i`创建了另一个名为`i`的变量实例。当然，为了可读性，我们可能想用不同的名称来调用它。

[![](img/8fa32b4483c3ffb458830e0701b481e9.png)](https://twitter.com/teivah)

❤️喜欢我的工作吗？你可以考虑成为 GitHub 赞助商:[https://github.com/sponsors/teivah](https://github.com/sponsors/teivah)。

## 进一步阅读

[](https://www.manning.com/books/100-go-mistakes-how-to-avoid-them) [## 100 个 Go 错误:如何避免它们

### 在你的 Go 代码中发现你自己都不知道的错误，并通过避免常见的…

www.manning.com](https://www.manning.com/books/100-go-mistakes-how-to-avoid-them) [](https://github.com/golang/go/wiki/CommonMistakes) [## golang/go

### 当新的程序员开始使用 Go 或者老的 Go 程序员开始使用一个新概念时，有一些共同的…

github.com](https://github.com/golang/go/wiki/CommonMistakes) 

## 翻译

*   [中文](https://tomotoes.com/blog/the-top-10-most-common-mistakes-ive-seen-in-go-projects/)
*   [俄语](https://37yonub.ru/articles/common-mistakes)
*   [波斯语](https://blog.faradars.org/top-10-most-common-mistakes-in-go-projects/)
*   [日语](https://techblog.szksh.cloud/the-top-10-most-common-mistakes-ive-seen-in-go-projects/)(仅限子部分)
*   [法语](https://blog.otso.fr/2019-12-26-top-10-erreurs-communes-projets-golang.html)

你还有其他想分享的翻译吗？请随时联系我，以便我可以在这里包括它。

![](img/1be1009eeb21be8dacfbf3afe8e56b10.png)

你还想提到其他常见的错误吗？欢迎分享，继续讨论；)*