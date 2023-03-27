# 关于程序设计中的化妆品与内在特性

> 原文：<https://itnext.io/on-cosmetics-vs-intrinsics-programming-c9d7f47e9fd6?source=collection_archive---------3----------------------->

![](img/172b017b72e746bd797541502ac3208f.png)

万维网上每天都在发生一场无情的战斗。它的目标是决定哪种编程风格是最好的:OOP 还是 FP？我假设命令式和过程式编程不是竞争者的一部分。

争论的范围从事实到无关紧要到完全愚蠢。几年前，我想听马丁·奥德斯基(Scala fame)的视频。我记不得确切的谈话内容和主题。不过，我记得的是引言:他解释说 FP 比 OOP 更受欢迎……因为致力于前者的会议比后者多得多。

当时，我不认为知名度是帮助我交付项目的相关因素。在写这篇文章的时候，我仍然不知道。此外，这相当于说电力不流行，因为没有专门讨论它的会议。恐怕 M. Odersky 把学术研究中的受欢迎程度错当成了相关性。我在他的“争论”后停止了，直到今天，我再也没有看过他的演讲。

话虽如此，我的观点并不是要抨击 m .奥德斯基，而是要强调一些论点的空洞性。例如，对于 FP 爱好者来说，它的不变性不是一，因为 OOP 也可以很好地利用它。唯一的区别是不变性是 FP 中的一个要求。另一方面，过度使用 OOP 会导致像 Java 这样的语言，其中每个方法都必须属于一个类，即使是静态方法。在这种情况下，类只是方法的一个额外的名称空间——它们没有带来 OOP“价值”。

其范围远远超出了面向对象编程和功能编程。考虑以下片段:

```
// Snippet 1fun router(repo: PersonRepository) = router {
    val handler = Handler(repo)
    GET("/person", handler::getAll)
}

class Handler(private val repo: PersonRepository) {
    fun getAll(r: ServerRequest) =
            ok().body(repo.findAll())
}// Snippet 2fun router(repo: PersonRepository) = router {
    val handler = Handler(repo)
    GET("/person/{id}", handler::getOne)
}

class  Handler(private val repo: PersonRepository) {
    fun getAll(r: ServerRequest) =
            ok().bodyValue(repo.findAll())
}
```

显然，区别在于`ok().body()` vs. `ok().bodyValue()`。如果你不熟悉 Spring 框架，你不太可能正确地将左边的代码片段识别为 WebMVC.fn，将右边的代码片段识别为 Web Flux。如果`repo.findAll()`从阻塞更新为非阻塞，情况会更加混乱，因为你不会发现任何区别。您只能通过查看包导入来区分它们:

*   封锁:`org.springframework.web.servlet.function.*`
*   非屏蔽:`org.springframework.web.reactive.function.server.*`

您可以使用注释而不是处理程序来重写以上两个代码片段。

```
// Snippet 1@RestController
class PersonController(private val repo: PersonRepository) { @GetMapping
  fun getAll() = repo.findAll()
}// Snippet 2@RestController
class PersonController(private val repo: PersonRepository) { @GetMapping
  fun getAll() = repo.findAll()
}
```

这两个片段表面上看起来很相似，但是对于导入来说。化妆品是相同的，而内在特性——阻塞与非阻塞——是根本不同的。

让我们来看看 Kotlin [协程](https://kotlinlang.org/docs/coroutines-basics.html)。下面是摘自科特林[文档](https://kotlinlang.org/docs/composing-suspending-functions.html#async-style-functions)的一个片段:

```
measureTimeMillis {
    val one = somethingUsefulOne()                             // 1
    val two = somethingUsefulTwo()                             // 1
    runBlocking {
        println("The answer is ${one.await() + two.await()}")
    }
}
```

1.  函数指向挂起的计算

协同程序代码在异步时表面上看起来是必要的。
这是*协程的*优势:

*   表面上看起来势在必行；因此很容易理解
*   在后台，库异步运行代码

代码具有表面的和内在的特征。我希望上面的几个例子能让你相信它们是完全正交的。你可以用不同的化妆品达到同样的效果，反之亦然。

我们不断地争论*化妆品*、*例如*、注释与“功能性”，但这本质上是个人品味的问题。为了解决问题，我们需要在**内部函数**上花更多的时间:参与者、异步等等。

*原载于* [*一个 Java 极客*](https://blog.frankel.ch/on-cosmetics-vs-intrinsics-programming/)*2022 年 7 月 31 日*