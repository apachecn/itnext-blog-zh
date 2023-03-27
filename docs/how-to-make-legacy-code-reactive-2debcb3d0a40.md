# 如何使遗留代码具有反应性

> 原文：<https://itnext.io/how-to-make-legacy-code-reactive-2debcb3d0a40?source=collection_archive---------0----------------------->

![](img/5b5d0cdd6fa0e9e7670b627f3231fa7c.png)

你喜欢反应流。太好了。您正在使用的应用程序是非反应性的，并且充满了阻塞调用。不太好。在本文中，我们将探索连接反应式和非反应式代码的选项，并潜在地使应用程序更具性能。

注意:这些例子以 Java 和 Spring Reactor 为特色，但是所有反应库的原理和缺陷都是相似的。如果您正在使用 RxJS，文章[中关于在 RxJS](https://blog.angularindepth.com/the-extensive-guide-to-creating-streams-in-rxjs-aaa02baaff9a) 中创建流的广泛指南可能会有所帮助。如果您不使用 Spring Reactor:

*   在反应库中通量是可观察到的
*   Mono 在一些 ReactiveX 库中是单一的。这是一个只发出一个值(或者没有值)的可观测值。有点像承诺或未来。

# **调度程序**

在我们开始反应之前，让我们来谈谈调度程序。它们是反应流的重要组成部分，即使你可能从未接触过它们。调度器决定何时在哪个线程(注意:不在 RxJS 中)上执行操作。

所有的反应库都允许你使用`publishOn`或`subscribeOn`操作符来改变默认行为。我最近写了关于这个的[。我强烈推荐 Spring Reactor 投稿人 Simon Baslé的一篇演讲，他以简洁的可视化展示了这些操作符是如何工作的(从 16:49 开始)。](/reactive-streams-and-multithreading-efd63b67de9a)

可用的排定程序类型因库而异。您可以在 ReactiveX 文档的[调度器部分或您的图书馆文档中找到您喜欢的口味列表。](http://reactivex.io/documentation/scheduler.html)

通常，您可以选择使用共享调度程序或创建自己的调度程序。在 Spring Reactor 中创建的三个最重要的组件是:

*   单线程:`Schedulers.single(...)`创建一个由所有操作共享的线程
*   有限并行性:`Schedulers.parallel(...)`创建一个固定的线程池，允许您并行运行操作
*   无限并行:`Schedulers.newElastic(...)`根据需要创建新线程，重用它们并丢弃未使用的线程。

创建自己的调度程序有什么意义？您可以完全控制线程行为。如果应用程序中使用共享调度程序的任何流包含阻塞或昂贵的操作，那么使用同一调度程序的所有其他流都会受到影响—它们会被阻塞。然而，如果一个流使用一个定制的调度器(你所需要的只是一行代码)，它就不会受到影响。

# 取消阻止呼叫

以下面的代码为例:

```
class MyService {

  public String blockingMethod() {
    ...
    return this.blockingWebService();
  }}
```

当我们调用`blockingMethod`时，调用者不得不等待，线程被阻塞。引入反应流的一种简单方法是将`blockingWebService`包装成这样的流:

```
public Mono<String> blockingMethod() {
    ...
    return Mono.just(this.blockingWebService());
}
```

这毫无意义。`blockingWebService`在我们返回流之前被称为**。我们至少可以做的是**推迟**调用`blockingWebService`直到订阅。最简单的方法是使用`fromSupplier`:**

```
public Mono<String> blockingMethod() {
    ...
    return Mono.fromSupplier(() -> this.blockingWebService());
}
```

现在`blockingWebService`在有人订阅返回的`Mono`时执行。这也意味着调用者可以让它在不同的线程上运行:

```
myService.blockingMethod()
  .subscribeOn(Schedulers.single())
  .subscribe();
```

调用者仍然需要知道该方法包含阻塞代码。让我们关心自己的线程。

```
class MyService { private Scheduler scheduler = Schedulers.newElastic("myThreads");

  public String blockingMethod() {
    ...
    return Mono.fromSupplier(() -> this.blockingWebService())
               .subscribeOn(this.scheduler);
  }}
```

你可能想知道为什么我使用了弹性调度器。如果我使用一个固定的池或者一个单独的线程，当足够多的调用者调用这个方法时，这个方法将会阻塞。

这里有一个陷阱:链中每一个后面的`subscribeOn`都会覆盖我们的`subscribeOn`。不过，这是打电话者的选择。

一切都好吗？哦，那取决于订户的数量。下一节还有一件事。

# 转换异步结构

也许你的代码已经使用了异步结构，比如期货或承诺。它们通常可以很容易地转换成流:

```
class MyService {

  public CompletableFuture<String> nonBlockingMethod() {
    ...
    return this.blockingWebService();
  }}Mono.fromFuture(myService.nonBlockingMethod())
  .subscribe(...);
```

如果代码已经在另一个线程中运行，我们就没事了。还是我们？调用 web 服务不会阻塞，但是会在任何人订阅之前立即调用该服务。让我们解决这个问题，并返回流:

```
public Mono<String> nonBlockingMethod() {
  ...
  return Mono.defer(
           () -> Mono.fromFuture(this.blockingWebService())
         );
}
```

完美。差不多了。像我们这样推迟执行有一个副作用:为每个订户执行代码。你可能想要也可能不想要。如果您选择这种解决方案，那么记录行为，以便客户可以在必要时调整他们的行为。

# 转换回调

如果你使用一个支持回调的 API，那已经是异步的了。但是我们怎么能把它变成一条小溪呢？大多数库使用`create`函数/方法，或者`Observable`构造函数。

下面是一个没有流的简单示例:

```
callWebService("http://foo", response -> {
  //process the response
});
```

现在有了流:

```
Mono.create(sink -> {
  callWebService("http://foo", response -> {
   sink.success(response);
  });
})
.map(...)
.subscribe(...);
```

我们所要做的就是创建一个只向流发出的回调。然后，您可以使用流操作符处理结果。一些图书馆有专门的功能。例如 RxJS 有`bindCallback`和`fromEventPattern`。

# 并行运行代码

## 异步运行昂贵的代码

想象这样的代码:

```
class MyService {

  public String expensiveMethod() {
    //expensive calculations
    return result;
  }}value = myService.expensiveMethod();
```

这没有什么问题，但是线程可能会被阻塞很长时间。这也没有错。但是，如果这会阻塞您的应用程序或服务，使其不可用，那么您应该让它异步运行。

对于反应流，一种方法是将计算提取到另一种方法中，并使用前面介绍的技术:

```
class MyService { private Scheduler scheduler = Schedulers.newElastic("myThreads");

  public Mono<String> expensiveMethod() {
    return Mono.fromSupplier(() -> this.expensiveCalculations())
               .subscribeOn(this.scheduler);
  } private String expensiveCalculations() {
    ...
  }}// Or public Mono<String> expensiveMethod() {
    return Mono.create(sink -> {
                  //expensive calculations
                  sink.success(result);
                })
               .subscribeOn(this.scheduler);
  }
```

## 并行处理数据

Java 8 streams 引入的一个好处是，只需调用`parallel()`，就可以让数据被多个线程并行处理。我们可以对反应流做同样的事情，并且我们对线程有更多的控制。注意:不是所有的库都支持这个。

```
Scheduler scheduler = Schedulers.newParallel("foo");myStream
  .parallel()
  .runOn(scheduler)
  .map(...)
  .subscribe(...);
```

请注意，`parallel()`仅**准备**并行处理的流。实际的线程行为由`runOn`定义。没有任何争论`parallel`将工作分成和 CPU 核心一样多的部分。如果我们有四个，那么四个元素**可以被并行处理。我们的调度程序提供了与 CPU 内核一样多的线程。完全匹配。**

另一个场景是并行运行几个**独立的**操作，并在所有操作完成后做一些事情。一些库提供了一个`forkJoin`操作符，但是更通用的解决方案是`zip`。

```
Flux.zip(stream1, stream2, stream3)
  .subscribe(combinedResult -> ...);// Produces
// [ value stream1, value stream2, value stream3] |
```

在 Spring Reactor 中，组合的结果是一个`Tuple`。在其他库中，它可以是数组或列表。如果你的流发出一个以上的值，第一个，第二个，等等。所有流的合并。

有几点需要考虑。如果流运行在不同的线程上，它们只能并行运行。根据流的不同，可能需要为一个、一些或所有流引入调度程序:

```
Scheduler scheduler = Schedulers.newElastic("foo");Flux.zip(
  stream1.subscribeOn(scheduler), 
  stream2.subscribeOn(scheduler),
  stream3.subscribeOn(scheduler)
  )
  .subscribe(combinedResult -> ...);
```

另一个问题是，如果一个流失败，合并的流也会失败。如果你需要所有的操作都成功才能继续，那很好。否则，我们可以用回退值或回退流来解决问题:

```
Flux.zip(
  stream1.onErrorReturn("),
  stream2.onErrorReturn("),
  stream3.onErrorReturn(")
  )
  .subscribe(combineResult -> ...);
```

请注意，Spring Reactor 不允许`null`值。订户必须能够处理回退值。另一个选项是`merge`结合`onErrorResume`:

```
Flux.merge(
    stream1.onErrorResume(e -> Flux.*empty*()),
    stream2.onErrorResume(e -> Flux.*empty*()),
    stream3.onErrorResume(e -> Flux.*empty*())
  )
  .buffer(3)
  .subscribe(combineResult -> ...);
```

如果发生错误，我们返回一个空流。你可能会奇怪我为什么用`buffer`。如果没有它`merge`，我们将有一个发出 3 个元素的流，每个元素对应一个源流(假设每个源流只发出一个元素)。Buffer 将元素组合成一个类似于`zip`的列表(在一些库中是数组)。如果一个流失败了，我们需要知道哪个流成功了，因为我们需要处理结果，那么我们需要映射源流的结果:

```
Flux.merge(
    stream1.onErrorResume(e -> Flux.*empty*())
      .map(v -> Tuples.*of*(1, v)),
    stream2.onErrorResume(e -> Flux.*empty*())
      .map(v -> Tuples.*of*(2, v)),
    stream3.onErrorResume(e -> Flux.*empty*())
      .map(v -> Tuples.*of*(3, v))
  )
  .buffer(3)
  .subscribe(combineResult -> ...);// Produces
// [ [1, value stream1], [2, value stream2], [3, value stream3]] |
```

# 结论

反应式编程的最佳解决方案是使用完全反应式堆栈。这并不总是可能的。使用我介绍的技术，您仍然可以实现高度的“反应性”。调度程序和并行处理可以帮助您轻松提高性能。

如果你知道一些其他有用的技术，请在评论中分享它们或链接到现有的资源。