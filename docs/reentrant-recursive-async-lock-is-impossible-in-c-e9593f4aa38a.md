# 可重入(递归)异步锁在 C#中是不可能的

> 原文：<https://itnext.io/reentrant-recursive-async-lock-is-impossible-in-c-e9593f4aa38a?source=collection_archive---------0----------------------->

*   实现锁重入的标准方法(即线程相似性)不适用于异步锁。
*   一个`ExecutionContext`似乎是线程相似性的有效替代，但实际上不能保证互斥。
*   如果你需要一个可重入的异步锁——你就不走运了，你必须在你的代码库中去掉锁重入。
*   如果你正在使用一些第三方的可重入异步锁，建议你也去掉它。

# 锁定再入

即使没有混合异步，可重入锁也是有争议的，但是有了同步代码，它们就能正确地工作(正如人们所预料的)。通过将一个锁绑定到第一次获取它的线程，然后允许该线程随意地重新获取该锁(例如，[互斥](https://docs.microsoft.com/en-us/dotnet/standard/threading/mutexes)、[监控](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/lock-statement))，我们仍然保留了任何锁的基本保证—互斥(没有并行访问)。

# 异步锁

一旦我们试图锁定一个`await`，事情就会变得更加复杂。

由于`await`前后的代码可以在不同的线程上执行[(一般情况下)，线程仿射锁就不能再用了。试图在除获取锁的线程之外的任何线程上释放这样的锁都会导致异常。](https://devblogs.microsoft.com/dotnet/configureawait-faq/)

事实上，如果你试图在一个`lock`语句中使用`await`来避免麻烦，C#编译器甚至会产生一个编译错误。当然，`lock`语句只是一种语法糖，没有什么能阻止你直接使用同步原语，搬起石头砸自己的脚。

避免在`async`方法中使用传统同步原语的另一个原因是，同步阻塞(尤其是在调用线程上)可能会违背使用异步的初衷。

输入[异步锁](https://devblogs.microsoft.com/pfxteam/building-async-coordination-primitives-part-6-asynclock/)(及其[变体](https://github.com/StephenCleary/AsyncEx/blob/master/doc/AsyncLock.md)):

这些锁通过以下方式与`async`代码一起工作:

1.  允许您异步等待(`await`)
2.  不将锁绑定到任何线程
3.  不允许重入以确保互斥

最后一点对本文特别重要。当然，如果一个锁不是线程仿射的，那么它怎么会允许从哪个线程重入呢？如果我们不能将锁绑定到任何特定的线程，我们还有什么其他的选择呢？

# 执行上下文

[ExecutionContext](https://devblogs.microsoft.com/pfxteam/executioncontext-vs-synchronizationcontext/) 似乎是一个很好的替代线程亲和并允许异步锁重入的候选对象。我们可以使用 [AsyncLocal](https://docs.microsoft.com/en-us/dotnet/api/system.threading.asynclocal-1?view=net-5.0) (或者之前使用 [CallContext)在一个`ExecutionContext`上存储我们自己的数据。LogicalSetData](https://docs.microsoft.com/en-us/dotnet/api/system.runtime.remoting.messaging.callcontext.logicalsetdata?view=netframework-4.8) )，它将在整个异步调用链中的`await`之后流向继续执行的线程(除非手动[抑制](https://docs.microsoft.com/en-us/dotnet/api/system.threading.executioncontext.suppressflow?view=net-5.0#System_Threading_ExecutionContext_SuppressFlow))。在`await`之后，它也可以从我们已经开始执行`async`方法的线程中删除:

问题是默认情况下`ExecutionContext`流向*所有*的线程，你的工作可能会被移动到调用链的下游。当您手动将工作移动到另一个线程时，例如使用`Task.Run`、`Thread.Start`、`ThreadPool.QueueUserWorkItem`等，情况就是如此。也许不太明显，即使您有多个并行执行的异步延续也是如此:

所以我们这里有多个线程并行地“继承”`ExecutionContext`。这意味着`ExecutionContext`不能替代线程亲和性，也不能帮助我们构建锁，而锁首先应该是关于独占访问的。如果多个线程对锁有相同的要求，并被允许并行地重新进入锁，那么这根本就不是锁。

# AsyncLocal ValueChangedHandler

我们已经确定，即使在最简单的情况下，用一个`await`我们也可以在一个异步锁中执行多个线程，因此没有一个线程可以声明锁的独占所有权。但是，如果我们有办法在线程间跟踪`ExecutionContext`流，并将锁所有权作为接力棒传递，会怎么样呢？

![](img/b94151d4a953b8a1cb91d84004359f33.png)

“请将所有权转让”

`AsyncLocal`有一个[构造函数](https://docs.microsoft.com/en-us/dotnet/api/system.threading.asynclocal-1.-ctor?view=net-5.0#System_Threading_AsyncLocal_1__ctor_System_Action_System_Threading_AsyncLocalValueChangedArgs__0___)，当`AsyncLocal`的值在任何线程上改变时，包括当它因为`ExecutionContext`流而改变时，它接受一个被调用的委托:

因此，我们可以跟踪`ExecutionContext`流，从而跟踪哪个线程当前正在一个锁中执行。让我们看看，如果我们实现一个异步锁，试图以这种方式跟踪线程所有权，并且只允许从当前拥有该锁的线程重新进入，会发生什么:

这个例子有点复杂，所以让我们按时间顺序来分析一下可能发生的情况:

```
1\. MainAsync starting on thread 1
2\. ChildAsync starting on thread 1
3\. ChildAsync continuing on thread 2
4\. MainAsync continuing on thread 3
5\. ChildAsync finished NonThreadSafe
6\. MainAsync finished NonThreadSafe
```

在步骤 2 之后，有 100 毫秒的延迟，此时锁内没有线程正在执行，因此没有线程可以拥有它。

在步骤 3 `ChildAsync`中，方法继续在线程池线程`2`上执行。由于这是锁中唯一的线程，所以它被赋予锁的所有权，并被允许重新进入锁以执行`NonThreadSafe`操作。

1 ms 后，在步骤 4 `MainAsync`中，方法继续在线程池线程`3`上执行。它还继续执行`NonThreadSafe`操作，因为它已经在锁内。

此时(在步骤 4 和 5 之间),我们有两个线程并行执行`NonThreadSafe`操作，尽管在这两种情况下操作都受到锁的保护。在最好的情况下，这将以异常结束，在最坏的情况下，以无声的数据损坏结束。

尽管我们设法跟踪了哪些线程正在锁内执行，但它不允许我们将锁的所有权交给任何线程。如果没有锁所有权，我们就不能允许锁重入，如果我们想保证互斥和锁保护操作的线程安全。

# 结论

最后，可重入异步锁在 C#中是不可能的。我集中研究了作为异步锁的线程亲和性替代方法的`ExecutionContext`,主要是因为这是我见过的每一个 [单个](https://github.com/mysteryjeans/Flettu/) [尝试](https://asherman.io/projects/csharp-async-reentrant-locks.html) [在](https://github.com/jasonkuo41/CellWars.Threading.AsyncLock) [实现](https://github.com/StephenCleary/AsyncEx/blob/v4/Source/Unit%20Tests/AdvancedExamples/RecursiveAsyncLockExample.cs)可重入异步锁时尝试的方法。它就是不起作用，并且引入了微妙的、难以诊断的错误。

这里的问题不在于`ExecutionContext`，而在于`async/await`本身，它经常导致代码在多线程上执行。在这种情况下，根本没有办法(即使在理论上)将锁所有权分配给任何线程，并且没有办法以线程安全(即互斥)的方式实现锁重入。

如果你发现自己在寻找一个可重入的异步锁，那么不要在这上面浪费时间。您的选择是:

1.  通过重构代码来消除锁重入(甚至是锁)。
2.  考虑您试图用锁保护的代码是否真的需要异步。尽管普遍认为，`async/await`不会自动让一切变得更好，在某些情况下可能是不必要的。

# 更新

## 1

在发表这篇文章之后，另一个[可重入异步锁](https://github.com/dotnet/wcf/blob/ce7428c2962e4ea4fce9c9c3e5999758a52bc4b9/src/System.Private.ServiceModel/src/Internals/System/Runtime/AsyncLock.cs)的实现引起了我的注意。它似乎成功地消除了我在这里谈到的一些问题(即`WaitAllAsync`的例子)。文章中的最后一个例子/问题仍然存在。

## 2

马修·a·托马斯对我的观点提出了另一个挑战。这个挑战足够严重，让我考虑把它放在这篇文章的顶部，并撤回我的声明。最后我决定反对，主要原因是`ConfigureAwait(false)`。

Matthew 采用的方法依赖于 [SynchronizationContext](https://devblogs.microsoft.com/pfxteam/executioncontext-vs-synchronizationcontext/) 来控制异步延续的调度。问题是`ConfigureAwait(false)`将选择不使用这个上下文，实际上是在逃避被提议的锁。并且*大多数*代码库[应该](https://devblogs.microsoft.com/dotnet/configureawait-faq/) [实际上](https://blog.stephencleary.com/2012/02/async-and-await.html#avoiding-context)为每个等待的任务使用`ConfigureAwait(false)`(顺便提一下，我上面的例子已经在重要的地方使用了它)。

建议的实现仅在以下情况下有效:

1.  您可以控制想要锁定的代码(即，您不想对第三方库中的异步代码施加锁定)。
2.  您愿意打破您将在其中使用锁的类的封装边界，并强制在您需要锁定的代码中的任何地方都不使用`ConfigureAwait(false)`。当然，如果锁应该跨多个类应用互斥保证，这只是一个问题。

所以我的声明现在变成了:可重入异步锁在一般情况下是不可能的，但是在一些特殊情况下它可能会工作。(这种方法还有其他限制，但都没有`ConfigureAwait(false)`那么严重，所以我就不说了)。

## 脚注:

1.  `ExecutionContext`还会沿着*同步*调用链向上流动(也就是说，如果你的方法不是`async`)，从那里它会传播到更意想不到的地方。
2.  在这里，我不想给斯蒂芬·克利里蒙上任何阴影——他从未声称这种特殊的锁可以被任何人使用，它也从未真正出现过。

![](img/53c93d55908b9f50821a5a802acd84bf.png)