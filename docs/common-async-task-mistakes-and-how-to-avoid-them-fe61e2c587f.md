# 常见的异步/任务错误，以及如何避免它们

> 原文：<https://itnext.io/common-async-task-mistakes-and-how-to-avoid-them-fe61e2c587f?source=collection_archive---------1----------------------->

![](img/0a13b31d876795c911a7d0f585072634.png)

的。Net async / await 机制是使异步代码可访问的天赐之物，但是尽管它是一个极好的抽象，许多开发人员仍然会陷入许多微妙的陷阱。

因此，对于我的异步主题故事系列中的这篇文章，我真的想引入一些有趣的(也是重要的)陷阱，许多开发人员(包括我)经常会陷入这些陷阱，因为它们往往很难介绍，但又很难发现。

# 异步任务方法中的未观察任务异常

开发人员在使用异步代码时要学的第一件事就是避免使用*异步 void* 方法，因为它们在抛出异常时可能会产生灾难性的影响。对于*异步任务*方法，异常被放在任务上(正如我在[https://Stefan sch . medium . com/what-the-async-keyword-actually-does-bb 10d 54 ce 31 c](https://stefansch.medium.com/what-the-async-keyword-actually-does-bb10d54ce31c)中解释的)。

所以，这是安全的使用，对不对？答案是:大部分情况下是这样，但根据使用情况，也可能不是这样。

想象一下这段代码:

```
async Task Main()
{
    await DoStuff();
}async Task DoStuff()
{
    await Task.Delay(100);
    throw new Exception();
}
```

当按原样运行代码时，一切正常。该异常被正确抛出，因为我们*等待 DoStuff()* ，这反过来意味着我们正在消耗放置在*任务*上的*异常*。

现在，看看这段代码

```
async Task Main()
{
    DoStuff();
}async Task DoStuff()
{
    await Task.Delay(100);
    throw new Exception();
}
```

它看起来很温顺，但问题在于缺少了等待的*。一个*任务*，就像任何其他引用类型一样，驻留在堆上，并且在某个时候也会被垃圾收集。*

当 GC 收集到一个*任务*，并且*任务*仍然有一个*异常*附加在它上面，因为它从未被等待过而从未被重新抛出，那么 CLR 将抛出一个*UnobservedTaskException*。相信我，你不会想在你的生产日志中看到这些，因为调试它们通常是非常痛苦的。

实际上，我们可以通过以下调整来展示这种行为:

```
async Task Main()
{
    TaskScheduler.UnobservedTaskException += (sender, args) =>
    {
        Console.WriteLine("Whoopsie");
    }; DoStuff();
    GC.Collect();
}async Task DoStuff()
{
    await Task.Delay(100);
    throw new Exception();
}// Output:
Whoopsie
```

我们手动触发 GC 收集，并为事件处理程序附加一个监听器，该事件处理程序在发生*UnobservedTaskException*时引发。

所以，确保你正确地等待*你的*任务*，或者正确地在内部捕获*异常*，这样它们就不会向外泄露！*

# 正确等待并发任务

下面是利用并发异步执行的一个很好的例子。想象一下这段代码:

```
async Task Main()
{
    await DoStuff(1);
    await DoStuff(2);
}async Task DoStuff(int number)
{
    // Imagine a Network call here.
    await Task.Delay(500);
    Console.WriteLine($"{number} done!");
}
```

这看起来很好，但是整个 *Main()* 方法在这里将花费大约 1 秒钟，因为执行仍然是顺序发生的。我们可以做得更好！

```
async Task Main()
{
    var t1 = DoStuff(1);
    var t2 = DoStuff(2);
    await t1;
    await t2;
}async Task DoStuff(int number)
{
    // Imagine a Network call here.
    await Task.Delay(500);
    Console.WriteLine($"{number} done!");
}
```

太好了！我们在这里成功地利用了并发性。我们同时启动了两个单元的工作，尽管做了同样多的工作，方法只执行了 0.5 秒！我们*等待 t1* ，但是等到 *t1* 完成，我们*等待 t2* ， *t2* 也已经完成！

但是，如果您注意了上一节，就很容易想到这里的问题。如果 *DoStuff()* 在两次调用中都抛出异常怎么办？我们会正确地*等待 T1* ，t1 会引发它的*异常*，它会冒泡，因为我们没有在 *Main()* 中捕捉到*异常*。

但是 *t2* 呢？我们从来没有达到等待 t2 的地步！这最终会再次被发现为*未观察到的任务异常*。

相反，应该使用以下内容:

```
async Task Main()
{
    var t1 = DoStuff(1);
    var t2 = DoStuff(2);
    await Task.WhenAll(t1, t2);
}async Task DoStuff(int number)
{
    // Imagine a Network call here.
    await Task.Delay(500);
    Console.WriteLine($"{number} done!");
}
```

*任务。WhenAll* 是任务并行库的一个实用函数，本质上是将一组 awaitables 捆绑成一个单独的。这就是我们正确修正上面的例子所需要的。如果没有修改，这将只抛出第一个*异常*，但是其他*异常*仍然可以被正确观察到。

# 异步方法同步运行，直到第一个 await

大概我最喜欢的异步方法的缺陷是它们在方法开始时用同步代码表现出来的行为。请参见以下示例:

```
async Task Main()
{
    var t1 = DoStuff();
    var t2 = DoStuff();
    await Task.WhenAll(t1, t2);
}async Task DoStuff()
{
    Thread.Sleep(500);
    await Task.Delay(500);
}
```

看这段代码，*do stuf()*方法整体等待 1 秒。因为我们没有直接等待它，而是同时开始两个任务*和*，所以总处理时间需要 1 秒，因为两个方法都是独立运行的，对吗？

实际上，执行需要 1.5 秒——这是怎么发生的？

*异步*方法的行为在逻辑上类似于非*异步*方法，至少直到第一个*等待*，因此第一个上下文切换发生。到目前为止，一切都将同步执行。所以在我们的例子中:

1.  对 *DoStuff()* 的第一次调用开始，休眠 500 毫秒。
2.  睡眠结束后，执行*任务。Delay(500)* 开始运行，进一步的执行被推迟，我们返回到 *Main()* 方法。
3.  现在，第二个 *DoStuff()* 启动，再次休眠 500ms。
4.  现在，第二个延迟也被触发，我们到达 *Main()* 方法中的点，在那里我们等待两个*任务*完成。 *t1* 此时将已经完成，因为延迟同时发生在第二个*线程上。Sleep()* 跑了，所以现在我们实质上是在等待第二次延迟剩余的 500ms。
5.  1.5 秒后，一切完成。

只需稍加修改，我们就可以将执行时间缩短到 1 秒钟:

```
async Task Main()
{
    var t1 = DoStuff();
    var t2 = DoStuff();
    await Task.WhenAll(t1, t2);
}async Task DoStuff()
{
    await Task.Delay(500);
    Thread.Sleep(500);
}
```

现在， *DoStuff()* 调用将延迟并立即返回，而延续将发生在随机的*线程池*线程上，这次不会阻塞主线程。

然而，解决这个问题的更常见的方法是使用*任务。Run()* 来(在我们的例子中)启动 *DoStuff()* 方法，作为*任务。Run()* 确保我们传递给它的委托直接在 Threadpool 上启动，同时保持主线程空闲。

记住这一点很重要，因为结构以及 *async* 方法的工作单元各自的 CPU 或 IO 性质可能意味着整个执行过程中的宝贵时间。

暂时就这样吧！如果这篇文章很受欢迎，我将继续讨论另一组常见的陷阱。