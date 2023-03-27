# 收集科特林流方法的差异

> 原文：<https://itnext.io/differences-in-methods-of-collecting-kotlin-flows-3d1d4efd1c2?source=collection_archive---------0----------------------->

你们中的一些人可能最近开始使用 Kotlin Flow，JetBrains 的新框架来处理可观察的流。

你可能已经从网上的许多文章中知道，为了收集流量，你有两个主要的选择。你可以使用`.collect()`，或者`.launchIn()`。(出于本文的目的，我们避免其他终端运营商，如`.toList()`等)

这两种方法有什么区别？

有一些显而易见的问题，其中一个非常微妙，不是每个人都能立即发现的，如果你想让你的代码像预期的那样运行，你应该注意这一点。

## 明显的差异

首先，如果我们看一下这两个方法的方法签名，我们会知道`.collect()`是暂停方法，而`.launchIn()`不是:

```
public suspend fun collect(collector: FlowCollector<T>)public fun <T> Flow<T>.launchIn(scope: CoroutineScope): Job
```

实际的区别是，你只能从另一个挂起函数或协程调用`collect()`方法。比如像这样:

```
coroutineScope.launch {
    flowOf(1, 2, 3)
        .collect { println(it) }
}
```

而`.launchIn()`可以在任何常规函数中这样调用:

```
flowOf(1, 2, 3)
    .onEach { println(it) }
    .launchIn(coroutineScope)
```

使用`.launchIn()`方法，您还会得到一个`Job`作为返回值，因此您可以通过取消作业来取消流程:

```
val job = *flowOf*(1, 2, 3)
    .*onEach* **{** *println*(**it**) **}** .*launchIn*(coroutineScope)

job.cancel()
```

这两种方法之间还有一个区别，可能不太明显，但还是回到了这个事实上，即`.collect()`是一个暂停函数。

## 更微妙的区别

让我们在这里做一个小测验。你认为这段代码的输出会是什么？

```
runBlocking {
    flowOf(1, 2, 3)
        .onEach { delay(100) }
        .collect { println(it) } flowOf("a", "b", "c")
        .collect { println(it) }
}
```

由于第一个`Flow`和第二个`Flow`中的每次发射都有延迟，因此会立即发射和打印，可能会出现以下输出:

```
a
b
c
1
2
3
```

但是，最终的输出实际上是这样的:

```
1
2
3
a
b
c
```

为什么？

## 收集是一个暂停功能

这种行为的原因是，`.collect()`是一个暂停功能。它挂起协程，直到完成自己的工作。因此，在我们的代码被截取的情况下，协程在每次出现延迟时都被挂起，并且协程不会继续执行任何其他代码，直到第一个流被收集。这意味着，在第一个流完成之前，根本不会收集第二个流。

好的，所以在实际场景中，这很可能不是你想要的。您可能希望在输出中首先看到字母 a、b、c，然后才是 1、2、3。

那么如何才能做到这一点呢？我们可以使用`.launchIn()`

## 但是为什么会起作用呢？

让我们看看`.launchIn()`方法的实现:

```
public fun <T> Flow<T>.launchIn(scope: CoroutineScope): Job =     scope.*launch* **{** collect() *// tail-call* **}**
```

从上面的代码可以看出，`.launchIn()`确实在内部调用了`.collect()`方法。然而，它收集流量在一个`scope.launch {}`块。这段代码意味着在参数中指定的协程范围内启动了一个新的子协程。

所以在这段代码中:

```
runBlocking {
    flowOf(1, 2, 3)
        .onEach { delay(100) }
        .onEach { println(it) }
        .launchIn(this) flowOf("a", "b", "c")
        .onEach { println(it) }
        .launchIn(this)
}
```

当第一个流被收集时，由`runBlocking {}`启动的协程不被挂起。相反，在阻塞协程的范围内启动了一个新的子协程(通过`runBlocking {}`启动)，这个新的子协程将被挂起。

这意味着第二个流没有被阻止收集。这意味着，输出是你所期望的:

```
a
b
c
1
2
3
```

## 结论

所以总之，我建议在收集你的流量时更喜欢用`.launchIn()`而不是`.collect()`，以避免意外的错误。只有当你绝对确定，你不介意并且将来也不会介意它挂起你的协程时，才使用`.collect()`(例如，因为协程被启动来收集这一个特定的流，并且不会在其中执行任何其他的东西，甚至将来你也不想在协程中添加任何其他的东西)。