# 深入了解配置等待

> 原文：<https://itnext.io/a-deep-dive-into-configureawait-65f52b9605c2?source=collection_archive---------0----------------------->

![](img/0d09d90f5868bba6371166b3d8600142.png)

里卡多四世塔马约在 [Unsplash](https://unsplash.com/s/photos/airport-worker?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上的照片

好吧好吧好吧！(是的，它是马修·麦康纳[的参考文献](https://www.youtube.com/watch?v=EuER2Puym4I))

您可能已经注意到了，我非常喜欢异步及其内在特性。我喜欢艰难的挑战，深刻理解异步是一个挑战。

异步的美妙之处在于它可以是甜蜜的，也可以是苦涩的，这取决于你如何实现它。但是，如果你理解它是如何在引擎盖下工作的，以及它的句法糖，你就能掌握它，并享受它的全部潜力。

在我的上一篇博文中，[。NET 异步编程简而言之](https://medium.com/@nelsonparente/net-async-programming-in-a-nutshell-dc01c2e71a20)，我们在。NET 和实现最佳实践。当我们在。NET 中有一个扩展方法几乎总是出现在讨论中，那就是*configurewait(false)。*

你写*。ConfigureAwait(false)* 在每次*等待*使用后？如果是，你能解释为什么吗？或者这是你机械地做的事情，因为它是“模式”？

在更好地理解*configurewait(false)*之前，我们必须后退几步，看看 *SynchronizationContext* 。

## 同步上下文

微软[文档声明](https://docs.microsoft.com/en-us/dotnet/api/system.threading.synchronizationcontext?view=net-5.0)*SynchronizationContext*类提供了在各种同步模型中传播同步上下文的基本功能。你从描述中了解它是做什么的吗？我没有，所以我必须深入调查。

每个正在运行的线程都有它的“当前”*同步上下文* 并且在这个类中有一个特殊的方法，即 [Post](https://docs.microsoft.com/en-us/dotnet/api/system.threading.synchronizationcontext.post?view=net-5.0#System_Threading_SynchronizationContext_Post_System_Threading_SendOrPostCallback_System_Object_) :

```
public virtual void Post (System.Threading.SendOrPostCallback d, object? state)
```

在 *SynchronizationContext* 默认实现中的这个方法接收一个委托并调用*线程池。将被异步调用的 QueueUserWorkItem* 。然后*线程池*包含一个有工作要做的队列，将在任务执行完成时回调结果。

注意到*帖*是虚法？虚拟方法可以被不同地覆盖和实现，这就是一些框架实际做的，它们扩展 *SynchronizationContext* 并覆盖虚拟方法的实现。例如，。NET 框架使用[**AspNetSynchronizationContext**](https://referencesource.microsoft.com/#system.web/AspNetSynchronizationContext.cs)**，WPF 使用[**DispatcherSynchronizationContext**](https://github.com/dotnet/wpf/blob/ac9d1b7a6b0ee7c44fd2875a1174b820b3940619/src/Microsoft.DotNet.Wpf/src/WindowsBase/System/Windows/Threading/DispatcherSynchronizationContext.cs)，每个框架都以更适合自己的方式实现这个类。**

**更简单地说，当我们在 C#中*等待*任何东西时，编译器会转换代码，向“可等待的”( *Task* )请求“等待者”( *TaskAwaiter < T >* )。然后 awaiter 负责抓取当前的 *SynchronizationContext* ，然后在等待的*任务*完成时返回回调。**

## **configurewait(false)**

**大概此时此刻，你正在疑惑*configurewait*到底有什么特别之处。除了一个小细节外，*配置并没有什么特别的。***

***configurewait(continueOnCapturedContext:false)*避免了回调在原来的*SynchronizationContext***中执行，这有很多好处。****

****第一个是性能，使用*configurewait(false)*将不会有任何队列，或者等待原始上下文可用所涉及的额外工作，*任务*将在可用的上下文中恢复。第二个好处是避免死锁，想象一下，由于某种原因，所需的*SynchronizationContext***无限期地处于阻塞状态，必须等待它被释放才能恢复执行，这将导致应用程序中的死锁。******

******很少有使用*configurewait(true)*的用例，它实际上没有什么意义。在 99%的情况下，您应该使用*configurewait(false)*。******

******英寸 NET Framework 默认情况下，*任务*将在捕获的上下文上继续执行，这是*configurewait(true)*。正如我们上面看到的，使用*configurewait(false)*有几个好处。******

******当使用 async/await 时，这种默认行为导致*configurewait(false)*遍布我们的代码库，并成为异步编程中的一种 [Cargo Cult 反模式](https://exceptionnotfound.net/cargo-cult-programming-the-daily-software-anti-pattern/)。******

> ******当一个程序员向他的项目中添加代码时，尽管他并不完全理解这些代码是做什么的，但还是会出现 Cargo-Cult 编程。******

******我们大多数人在使用*configurewait(false)*时，从来没有问过自己它的真正目的是什么，以及这些东西在幕后是如何工作的。******

## ******。网络核心拯救世界******

*******中的大新闻之一。NET Core* 框架缺少任何一种 *SynchronizationContext* ，这是移除了旧*中使用的[**AspNetSynchronizationContext**](https://referencesource.microsoft.com/#system.web/AspNetSynchronizationContext.cs)**。NET 框架*****。********

******所以在这个*的无语境方法中。NET Core，*默认的异步行为与我们使用*configurewait(false)*时的行为相同，因此当*异步*处理程序恢复执行时，一个线程从线程池中取出并开始工作。但是请注意，await 关键字不能捕获上下文并不意味着您可以避免死锁，异步实现中的错误仍然会导致线程阻塞。******

******这时，有人可能会想，在*里。NET Core* 你不需要将*configurewait(false)*散布在你的代码中。差不多！******

******![](img/f8dfb35af23b9ad45c844673cd455b10.png)******

******这几乎是真的，如果在遗留框架中使用库，仍然建议使用库的*configurewait(false)*作为后备。但是大多数情况下是的，在*中。NET Core* 可以去掉*configurewait(false)*用法。******

******结束了！******

******作为一名工程师，这些事情让我越来越想继续研究我日常使用的技术的核心，这引发了我的好奇心，并且理解所有这些复杂概念如何协同工作是非常有益的。******

******在这篇博文中，我们稍微深入了一下*中异步编程的核心。NET* ，*SynchronizationContext**和，我们明白了*configurewait(false)*传播的原因。我希望，看完这篇博客后，你不是这个货物邪教的成员之一，现在你知道为什么使用 *ConfigureAwait(false)* 以及它如何成为[货物邪教反模式](https://exceptionnotfound.net/cargo-cult-programming-the-daily-software-anti-pattern/)的最大例子之一。*******

*****尼尔森出局。*****

*****[![](img/69716627feab2505c60838bbd29241a9.png)](https://www.buymeacoffee.com/nelsonparente)*****