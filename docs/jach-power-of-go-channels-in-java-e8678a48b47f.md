# JaCh——Java 中 Go 通道的力量

> 原文：<https://itnext.io/jach-power-of-go-channels-in-java-e8678a48b47f?source=collection_archive---------3----------------------->

![](img/8392133f80a3573015c224611eeab413.png)

马克西米利安·魏斯贝克尔在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上的照片

Google 的 Go 从一开始就被设计成将软件和硬件的最新发展直接融入到语言中。为了实现多个运行线程之间共享数据访问的同步，传统语言依赖锁和监视器，go 从[通信顺序进程](http://www.usingcsp.com/)中获得灵感，并围绕通道和 Go 例程构建其同步和数据访问模式。

Go 对共享数据结构的方法通常被浓缩成以下语句:

> 不通过共享内存进行交流；相反，通过交流来分享记忆。

这意味着传统语言使用锁和互斥锁来控制对多线程间共享数据的访问，而 Go 鼓励进程通过通道交流状态。这意味着共享内存只能由通道访问，并且每个并行例程可以独立地处理共享数据，而不用担心竞争情况。

## 在 Java 中调整通道和例程

JaCh(Java Channels 的缩写)是一个库，它试图将 Go 通道的功能及其特性引入 Java 语言，并在简单易用的 API 中鼓励 CSP 的 Java 设计模式。

**通道**作为一种数据结构，一端接受数据写入，另一端将数据输出到数据读取器。尽管没有关于数据顺序的明确保证，但它仍然可以被建模为一个**队列。** Java 内置了对[**blocking queue**](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/BlockingQueue.html)**的支持，这也保证了通道读线程和写线程可以分别在空线程和满线程的情况下被阻塞。**

****例程**非常类似于 Java 中的一个线程。将例程简单地实现为一个 [**Runnable**](https://docs.oracle.com/javase/8/docs/api/java/lang/Runnable.html) 对象的包装器是有意义的，该对象可以在线程池中执行。**

## **通过交流分享记忆**

**尽管没有强制执行，Go 的理念鼓励通道使用具体类型而不是指针类型。这确保了内存不会被 go-routines 共享。副作用是在通道中传递的对象需要是轻量级的，以便于更快的内存复制。**

**在 Java 中，每个对象实际上都是一个对象引用(类似于指针)。每当一个对象按原样写入 Java 通道时，它的引用将实际驻留在通道中，并且实际的对象将在线程间共享——这将违反通过通信共享内存的原则。为了确保每个对象在被写入通道之前都被复制到一个新对象，使用了一个`Copier`实例。JaCh 使用 [Kryo](https://github.com/EsotericSoftware/kryo) 作为默认的复印机实现，但是用户可以自由实现他们自己的复印机。**

# **JaCh 与 Go 频道**

**在这一节中，我们来看看 JaCh 和 Go 的通道之间的一些特性对等性。下表简要介绍了 JaCh 提供的模拟 Go 通道的特性**

## **创建频道**

**JaCh 在`JachChannels`类中定义了静态的`make`方法来轻松创建通道。**

## **在通道上迭代**

**JaCh 使用`Channel`接口来建模通道，通道又从`Iterable`扩展而来。这意味着 JaCh 中的通道可以使用增强的 for 循环(for-each 循环)或者使用`forEach`方法并传递一个`Consumer`来迭代。因为 JaCh 通道是在不同的线程中异步关闭的，所以迭代器不可能干净地退出，只能通过抛出异常来退出循环。因此，for-each 循环需要包装在 try-catch 块中。`forEach`方法很好地处理了这种情况，不需要任何 try-catch 包装。**

## **选择，用于-选择和选择-默认**

**由于通道是 Go 中的本地构造，它提供了直接的语言绑定来等待多个通道，并在数据到达时选择一个通道。此外，这样的 select 语句可以在 for 循环中连续轮询不同的通道，直到所有通道都关闭或者循环通过中断退出。**

**JaCh 试图使用`select`和`untilDone`方法提供一个类似的构造。因为这些方法接受一个`Consumer`类型，所以与匿名内部类或 lambda 相关的限制适用于这些情况。其中最值得注意的是，消费者内部使用的任何变量都必须是 final 或[有效的 final](https://www.baeldung.com/java-effectively-final) 。为了提前打破循环，在`Selector`类中提供了一个特殊的消费者`BREAK_ACTION`。**

## **计时器和计时器**

**JaCh 还提供了创建`Timer`或`Ticker`的能力，它们在某个时间间隔后通过信道发送信息。尽管 Java 已经有了自己的`Timer`和`TimerTask`版本，但是这些类允许用户围绕基于定时器接收数据的通道来设计他们的代码。**

## **JaCh 中的例程**

**由于函数是 Go 中的一级对象，所以要创建 go-routine，只需调用以`go`关键字为前缀的函数即可。JaCh 提供了一种类似的方法，通过多个`go`方法创建一个例程。`go`方法接受一个 lambda 函数，该函数可以接受 0 到 9 个参数，后跟参数值。然后，它在一个单独的线程中运行这个 lambda，给人一种并行 go-routine 的错觉。**

# **得到 JaCh**

**JaCh 在 central 中作为 maven 库提供。下面的 pom 依赖项将把它添加到您的项目中使用**

```
<dependency>
    <groupId>io.github.daichi-m</groupId>
    <artifactId>jach</artifactId>
    <version>LATEST_VERSION</version>
</dependency>
```

**它是在 MIT 许可下开源的，代码可以在 [Github](https://github.com/daichi-m/jach) 中获得。repo 还有一个 [samples 项目](https://github.com/daichi-m/jach/tree/develop/samples)，其中包含 JaCh 各种特性的示例代码。**

# **脚注**

1.  **[有效的围棋](https://golang.org/doc/effective_go#concurrency)是了解更多围棋通道和套路的绝佳去处。**
2.  **选择 Kryo 而不是原生 Java 序列化的原因是，它比原生 Java 版本快得多，相当轻量级，并且足够稳定，可以跨多个流行的开源项目使用。**