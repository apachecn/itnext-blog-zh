# 如何优雅地处理空引用

> 原文：<https://itnext.io/how-to-elegantly-deal-with-null-references-86a94494428?source=collection_archive---------3----------------------->

![](img/48cfacf5754fa29a76bdfa6b36e3a258.png)

事情是这样的:在过去的几年里，在更优雅地处理空引用方面有了很多改进。最突出的是，C#8 增加了可空引用类型特性，这允许以一种很好的方式对空引用进行编译时检查。它是所有以前试图做类似事情的库的“官方”替代，基于来自 Resharper 或其他人的类似[NotNull]的属性。

这些变化是非常受欢迎的，它们确实有助于编写更安全的代码，尤其是当涉及到我从未想过可能返回空引用的方法时。

不过，对我来说，这只是正确处理空值的一半。尽管这些变化有所帮助，但它们也导致了大量的无效支票。现在您对什么可能为空或不为空有了更好的了解，您的代码可能会有更多这样的部分:

```
public interface IMyInterface
{
 void Run(string message);
}internal class MyImplementer : IMyInterface
{
 public void Run(string message) => Console.WriteLine(message);
}void Main()
{
 // Our null reference
 IMyInterface? potentialNullInterface = null;
 if (potentialNullInterface is null)
 {
     potentialNullInterface = new MyImplementer();
 } ...

 if (potentialNullInterface is not null)
 {
  potentialNullInterface.Run("Hello World");
 } ...

 IMyInterface? otherInterface = potentialNullInterface ?? new MyImplementer(); ...

 otherInterface?.Run("Hello World"); 
}
```

这些只是关于如何检查可空性的一些例子。尤其是空合并检查非常简洁和方便，但有时甚至这些检查都感觉太臃肿或太复杂而难以考虑。这只是你在阅读代码部分时必须记住的另一点上下文。

# 空对象模式

除了在代码中散布空检查，还有另一种叫做空对象(有时也称为 void 对象)的方法，它绝对不需要空检查，但仍然具有相同的行为。让我们来看一个可能熟悉的场景。

假设您编写了一些代码来为一个任意的操作提供度量标准，并且您通过下面的简单实现做到了这一点:

```
public interface IMetricsWriter
{
 Task IncreaseCounter(string counterName);
}internal class MetricsWriter : IMetricsWriter
{
 private readonly HttpClient _metricsSinkClient;public MetricsWriter(HttpClient metricsSinkClient) => _metricsSinkClient = metricsSinkClient;public async Task IncreaseCounter(string counterName)
 {
  // Post our new metrics to a server
  await _metricsSinkClient.PostAsync(...);
 }
}
```

目前看来还算合理。您实现了一个接口和一个将指标写入远程端点的实现。

在使用它的时候，你的新接口可能会通过 DI 注入到另一个类中，例如:

```
public class LoginService
{
 private readonly IMetricsWriter _metricsWriter;public LoginService(IMetricsWriter metricsWriter) => _metricsWriter = metricsWriter;

 public async Task Login(string userName, string password)
 {
  // Do some login actions here
  ...

  await _metricsWriter.IncreaseCounter("Login");
 }
}
```

太好了！现在，我们有了一个登录服务，一旦它收到一个登录，就会自动增加一个指标。

然而，在某个时间点，您决定要构建另一个服务，它与我们的初始服务共享完全相同的 LoginService，但是对于这个服务，我们**不想**跟踪任何指标。现在该怎么办？使指标编写器可为空？这肯定是可行的，但是您最终会得到这样的结果:

```
public async Task Login(string userName, string password)
 {
  // Do some login actions here

  if (_metricsWriter is not null)
  {
   await _metricsWriter.IncreaseCounter("Login");
  }
 }
```

这看起来还不错，但是如果服务增长，可能会出现更多这样的场景，通过您必须实现的所有检查和保护，它将变得越来越复杂。让我们采取另一种方法。让我们坚持最初的非空服务，改为实现一个**空对象**！

实现实际上非常简单:

```
public interface IMetricsWriter
{
 Task IncreaseCounter(string counterName);
}internal class NullMetricsWriter : IMetricsWriter
{
 public Task IncreaseCounter(string counterName) => Task.CompletedTask;
}
```

就是这样！我们已经实现了一个 100%兼容的接口实现，只是这个实现完全没有任何变化。我们所做的就是返回一个完成的任务，这样我们就符合方法签名了。

因为我们最初说过，通过依赖注入将**度量写入器**注入到服务中，所以我们现在要做的就是将这个新的 null 实现绑定到我们新服务中的接口。

然后，当有人登录时，我们新的 null 实现被触发来增加计数器，完全不做任何事情，但仍然返回其完成的任务，调用服务很高兴。不需要更多的空检查，但是我们实现了与常规空引用相同的流程。

概括一下:虽然通常您会将 null 视为与接口的实际实现完全不同的实体，但是 null 对象模式将可空性行为提升到了我们的对象的第一类实现。它不再需要空检查，因为它**是我们接口的有效实现。**

# 何时不使用它

空对象模式是我喜欢使用的东西，但它也不是适合所有用例的解决方案。在您开始用这个解决方案替换所有的空支票之前，有一些事情需要记住。

1.  当使用这种模式时，**调用者绝对看不出**在一个接口后面可能有一个空对象。如果你的同事查看了 LoginService 并运行了它，**如果不知道你的 DI 注册，他就无法知道那里有什么。他甚至可能认为这正确地编写了度量标准，并将其部署到生产中，并且可能在一周之后感到惊讶，当他查看您的度量标准时，什么也没有看到。通过适当的空值检查，这一点很明显。**
2.  使用该模式作为应用程序启动和到事件总线的异步连接之间的桥梁可能很有吸引力，例如，通过一个 **NullEvent** ，直到您可以用一个适当的事件替换它。但是同样，null 事件只会吞噬每个调用，有时为了不丢失数据，您最好先实现一个缓冲。
3.  如果你做得太多，你可能会分配很多不必要的对象。您可能想考虑创建这些对象的单例实现，这很容易做到，因为这些对象中没有状态，因此没有并发问题。
4.  当您有返回值的方法时，使用这种模式尤其棘手。对于引用类型，您通常可以很容易地通过，因为您可以退回到 null。但是，如果一个空对象必须实现的方法返回一个值类型，该怎么办呢？对于返回 int 的签名，你会返回 0 吗？这里可能会出现许多定义问题。

但是即使有提到的缺点，它仍然是一个很好的模式，特别是当涉及到您可能想要避开的非关键接口时，可能是为了测试目的，或者是为了解决依赖注入设置，如果这些依赖的消费者对处理特定的接口不感兴趣的话。这方面的主要例子是日志记录、指标甚至事件。

总而言之:你必须评估这是否适合你的应用环境。缺点是否值得，或者您的代码部分是否重要？无论结果如何，拥有另一个工具总是有助于您获得尽可能干净的代码。