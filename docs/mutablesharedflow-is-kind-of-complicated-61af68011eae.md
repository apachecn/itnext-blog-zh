# 可变共享流有点复杂

> 原文：<https://itnext.io/mutablesharedflow-is-kind-of-complicated-61af68011eae?source=collection_archive---------0----------------------->

![](img/75e0648f9a871d227da17e3c28882c1a.png)

[乌斯曼·优素福](https://unsplash.com/@usmanyousaf?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

从 Kotlin 协同程序版本`1.5.0`、`BroadcastChannel`和`ConflatedBroadcastChannel`开始被标记为`ObsoleteCoroutinesApi`，开发者现在应该使用`SharedFlow`和`StateFlow`来代替。

Kotlin 文档甚至给出了如何从这些通道迁移到各自的流 API 的便捷指南:

> 要将 [BroadcastChannel](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/-broadcast-channel/index.html) 用法迁移到 [SharedFlow](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-shared-flow/index.html) ，首先用`MutableSharedFlow(0, extraBufferCapacity=capacity)`替换`BroadcastChannel(capacity)`构造函数的用法(广播频道不会向新订户重播值)。将 [send](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/-send-channel/send.html) 和 [trySend](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/-send-channel/try-send.html) 呼叫替换为 [emit](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-mutable-shared-flow/emit.html) 和 [tryEmit](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-mutable-shared-flow/try-emit.html) ，并将用户代码转换为流量运营商。

好的，但是也许你会对这个参数`extraBufferCapacity`感到疑惑。它实际上是做什么的？而且`MutableSharedFlow`不是已经有`replay`参数了吗？有什么区别？

## 证明文件

在这种情况下，查看文档对这两个参数的描述通常是有用的(唉，并不总是有用的):

> `replay` -重放给新订户的值的数量(不能为负数，默认为零)。
> 
> `extraBufferCapacity` -除了`replay`之外缓冲的值的数量。[发射](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-mutable-shared-flow/emit.html)在有缓冲空间剩余时不暂停(可选，不能为负，默认为零)。

我不知道你怎么想，但对我来说，这并不十分清楚上面提出的那些问题。我知道`replay`是做什么的，但是在阅读了文档之后，`extraBufferCapacity`仍然和我研究它之前一样难以捉摸。而这个参数**的某些**后果**在第一次通读后根本不明显**。

是时候深入挖掘和研究这些概念了。

## 重播

Replay 参数很容易解释，熟悉 RxJava 的人会非常了解这个参数。本质上，用`replay=x`创建一个`MutableSharedFlow`和在 RxJava 中创建`ReplaySubject`是一回事。

让我们来看看这段代码:

```
*runBlocking* **{** val testFlow = *MutableSharedFlow*<String>()
    testFlow.tryEmit("a")
    val job = testFlow
        .*onEach* **{** *println*(**it**) **}** .*launchIn*(this)
    delay(100) *// to give enough time for println to be executed before cancelling job* job.cancel()
**}**
```

这段代码的预期输出是什么？

….

嗯，实际上**没有输出**。为什么？

这是因为首先向`testFlow`发出一个值，而`testFlow`仅在发出一个值后**才被收集。在执行这一行代码时，没有人在监听事件:`testFlow.tryEmit(“test”)`。**

因此，如果您想要缓存事件直到流被实际收集，您可以使用参数`replay`。该值是一个整数，指定应该缓存多少个最后事件。对于这个例子，让我们用`replay=1`来解决。现在让我们检查这段代码:

```
*runBlocking* **{** val testFlow = *MutableSharedFlow*<String>(replay = 1)
    testFlow.tryEmit("a")
    testFlow.tryEmit("b")
    val job = testFlow
        .*onEach* **{** *println*(**it**) **}** .*launchIn*(this)
    delay(100) *// to give enough time for println to be executed before cancelling job* job.cancel()
**}**
```

产量是多少？

…

是`b`。为什么？

我们已经指定了`replay=1`，这意味着最后一个`1`事件将被缓存，以供所有将来的收集器使用。有两个事件发送到`testFlow`，分别是`a`和`b`(按此顺序)。最后发出的事件是`b`,因此这是将要发出的事件。

值得注意的是，如果我们稍后再添加一个或多个收集器，它们仍然会收到这个`b`值。按照这个例子:

```
*runBlocking* **{** val testFlow = *MutableSharedFlow*<String>(replay = 1)
    testFlow.tryEmit("a")
    testFlow.tryEmit("b")
    val job = testFlow
        .*onEach* **{** *println*(**it**) **}** .*launchIn*(this)
    delay(100) *// to give enough time for println to be executed before cancelling job* val job2 = testFlow
        .*onEach* **{** *println*(**it**) **}** .*launchIn*(this)

    job.cancel()
    delay(100)
    job2.cancel()
**}**
```

这一次的输出是:

```
b
b
```

这是因为使用`replay`缓存是为所有未来的收集者保留的。

好的，很好，但是有时你不想要这种行为。有时，如果没有人在听，没关系，你想丢弃那些事件，不缓存它们。例如，让我们说你的`MutableSharedFlow`发送地理位置数据给收藏家。您可能不希望任何下一个收集器接收一些缓存的值，您可能宁愿等待下一个(更新的)值。在这种情况下，指定`reply=0`(或者不指定任何值，因为缺省值是`0`)是可行的方法。(对于 Rx 人来说，这大概相当于使用`PublishSubject`。

那么这种方法可能会出什么问题呢？

事实上，有一个问题你需要小心。

## tryEmit 和非阻塞协同作用域

在前面的例子中，我们在阻塞`CoroutineScope`(由`runBlocking {}`创建)中收集流。

在应用程序的实际情况中，您可能会使用不同的协程作用域，或者是您的平台提供的，或者是您自己使用`CoroutineScope()`方法创建的。

代码可以分布在多个文件中，假设您想要使用`tryEmit`，而不是`emit`，因为您想要从一个普通函数中发出事件，而不是从一个挂起的函数中发出事件(在一个协程之外)。

这段代码模拟了这些条件:

```
fun main() {
    val testSharedFlow = MutableSharedFlow<String>() runBlocking {
        CoroutineScope(Job()).launch {
            testSharedFlow
                .onStart { println("start") }
                .collect { println(it) }
        } delay(100) // to give enough time for println to be executed before execution finishes
    } testSharedFlow.tryEmit("a")
    testSharedFlow.tryEmit("b")
}
```

那么这段代码会输出什么呢？事件是在一个收集器开始收集之后发出的，所以我们应该看到`a`和`b`都发出了，*对吗*？

## 不

令人惊讶的答案是，我们只会收到上面这段代码的输出`start`。怎么回事？

回答起来会有点复杂，但是请继续听我说。

我们必须考虑这个过程中涉及的两方。 ***发射极*** 和 ***集电极*** 。*发射器*是将事件推给`MutableSharedFlow`的发射器。*收藏者*正在从流中阅读。

*发射极*和*集电极*相互独立。 ***发射器*** 试图**向`MutableSharedFlow`发射**一个事件，它们**不一定要等待**等待`Collectors`来收集它们。

为了在收集到事件之前不阻塞线程，`tryEmit`方法所做的是向`MutableSharedFlow`的缓存发送一个值。如果您使用`replay=1`或更多，那么您可以指定一些缓冲区大小(`1`或更多)，事件将被保存在缓存中，直到被收集器收集。否则，**缓存大小为 0** ，因此事件只是**被丢弃**。

但是正如我们上面讨论的，有些情况下，使用`replay`是不可接受的。如何解决这个问题？

## 选项 1:用 emit 代替 tryEmit

一种方法是使用`emit`而不是`tryEmit`:

```
fun main() {
    val testSharedFlow = MutableSharedFlow<String>() runBlocking {
        CoroutineScope(Job()).launch {
            testSharedFlow
                .onStart { println("start") }
                .collect { println(it) }
        } delay(100) // to give enough time for println to be executed before execution finishes
    }

    runBlocking { 
        testSharedFlow.emit("a")
        testSharedFlow.emit("b")
    }
}
```

这一次输出将如下所示:

```
start
a
b
```

但是，为什么这种方法一开始就应该有效呢？

那是因为`emit`是一个悬浮函数。所以`emit`实际上可以等待收集器处理事件，并在此之后继续执行剩余的代码。

好吧，但这有点作弊。还记得我们上面说过的，我们想要特别使用`tryEmit`以便我们也可以在协程之外发出事件吗？作为一个暂停函数，我们不得不在协程中调用它。如何才能让`tryEmit`发挥作用？

## 选项 2:额外缓冲能力

如果你还记得这篇文章是如何开始的，我们对这个参数`extraBufferCapacity`很好奇。

先在实践中看到再讨论，到底是怎么回事。按照迁移指南，我们可能会得到类似这样的结果:

```
fun main() {
    val testSharedFlow = MutableSharedFlow<String>(extraBufferCapacity = 1) runBlocking {
        CoroutineScope(Job()).launch {
            testSharedFlow
                .onStart { println("start") }
                .collect { println(it) }
        } delay(1000) // to give enough time for println to be executed before execution finishes
    } testSharedFlow.tryEmit("a")
    testSharedFlow.tryEmit("b")
    runBlocking { delay(100) } // to give enough time for println to be executed before execution finishes
}
```

这段代码的输出将是:

```
start
a
```

好的，那么为什么我们现在只看到了`a`？`b`去哪了？为什么我们会看到`a`？

让我们回到我们说过的`tryEmit`方法:

> 为了在收集到事件之前不阻塞线程，方法`tryEmit`所做的是向`MutableSharedFlow`的缓存发送一个值。

没错。而当你创建一个像这样的可变共享流`MutableSharedFlow<String>()`时，内部缓存大小为 0。

如前所述，由于缓存大小为 0，事件会被立即丢弃。现在你可能开始明白了。`extraBufferCapacity`参数指定由`tryEmit`(或者在某些情况下甚至是`emit`)发送到`MutableSharedFlow`的事件的缓存大小，而不会导致新收集器的任何事件重放。

因为在上面的例子中`extraBufferCapacity`被设置为`1`，所以只有一个事件被缓存，那就是`a`。

## 但是为什么是 a 呢？为什么不是 b？

这听起来很可疑，不是吗？在一个类似的例子中指定`replay=1`输出`b`，`extraBufferCapacity=1`输出`a`。太奇怪了。为什么？

原因是，重放高速缓存试图重放传递给它的最后的**事件。所以当一个新的事件进来时，重放缓存只是`1`，它将覆盖那个事件。有点道理。**

有了`extraBufferCapacity`，那一个就只剩下给[提供缓冲的**快** *发射极*和**慢**集电极](https://elizarov.medium.com/shared-flows-broadcast-channels-899b675e805c) 了。也就是说，如果*发射器*产生事件的速度快于*收集器*收集事件的速度。默认情况下，第一个事件被缓存，当缓存满后，任何其他事件都会被丢弃。类似于如果`extraBufferCapacity`是`0`——这实际上是默认值。

## 但是我不想放弃新的事件:-(

正如我上面写的，这是默认行为。然而，这种默认行为可能不符合您的需求。幸运的是，有一种方法可以改变默认行为，通过指定参数`onBufferOverflow`。

像往常一样，让我们看看文档对这个参数有什么说明:

> `onBufferOverflow` -配置缓冲区溢出时的[发射](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-mutable-shared-flow/emit.html)动作。可选，默认为[暂停](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/-buffer-overflow/-s-u-s-p-e-n-d.html)尝试发出一个值。除了 [BufferOverflow 以外的值。只有在`replay > 0`或`extraBufferCapacity > 0`时才支持暂停](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/-buffer-overflow/-s-u-s-p-e-n-d.html)。**仅当至少有一个订户未准备好接受新值时，缓冲区溢出才会发生。**在没有用户的情况下，仅存储最近的[重放](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-mutable-shared-flow.html#kotlinx.coroutines.flow$MutableSharedFlow(kotlin.Int,%20kotlin.Int,%20kotlinx.coroutines.channels.BufferOverflow)/replay)值，缓冲区溢出行为不会被触发，也不会产生影响。

以及`BufferOverflow`的哪些值可用？是`SUSPEND`、`DROP_OLDEST`和`DROP_LATEST`。`SUSPEND`策略在上面的文档中有解释，与我上面写的关于`emit`方法的内容一致，增加了一些细节，关于它什么时候真正挂起，什么时候可以立即执行。

你可能已经猜到，为了保留新事件，丢弃旧事件，应该使用`BufferOverflow.DROP_OLDEST`策略。事实上这段代码:

```
fun main() {
    val testSharedFlow = MutableSharedFlow<String>(extraBufferCapacity = 1, onBufferOverflow = BufferOverflow.DROP_OLDEST) runBlocking {
        CoroutineScope(Job()).launch {
            testSharedFlow
                .onStart { println("start") }
                .collect { println(it) }
        } delay(100) // to give enough time for println to be executed before cancelling job
    } testSharedFlow.tryEmit("a")
    testSharedFlow.tryEmit("b")
    runBlocking { delay(100) }
}
```

将输出:

```
start
b
```

这可能是你通常想要的。

当然，如果您不想错过任何事件，甚至将`extraBufferCapacity`增加到更大的值也是明智的。

## 最后的想法

希望这篇文章对正确使用`MutableSharedFlow`有所启发。正如您所见，使用开箱即用的`MutableSharedFlow`并不像听起来那么简单。此外，文档描述的迁移模式虽然等同于以前的`BroadcastChannel`API，但不一定是您可能**需要的**(您可能需要在您的应用程序中指定`onBufferOverflow = BufferOverflow.DROP_OLDEST`，这取决于您的用例)。

*注意:根据这篇文章，我已经建议去掉* `*replay*` *和* `*extraBufferCapacity*` *参数的默认值，因为我认为默认值会导致意想不到的行为，并且可能会在许多应用程序中引入未被注意到的错误。可以在这里支持这个提议:* [*https://github . com/kot Lin/kot linx . coroutines/issues/2387 # issue comment-850774203*](https://github.com/Kotlin/kotlinx.coroutines/issues/2387#issuecomment-850774203)