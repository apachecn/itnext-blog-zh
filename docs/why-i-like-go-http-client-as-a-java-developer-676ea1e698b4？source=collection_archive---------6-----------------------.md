# 作为一名 Java 开发人员，我为什么喜欢 Go HTTP client

> 原文：<https://itnext.io/why-i-like-go-http-client-as-a-java-developer-676ea1e698b4?source=collection_archive---------6----------------------->

我们生活在一个微服务的时代。那些在他们的生态系统中有很多交流。你可能见过所谓的微服务死星:

![](img/a4c60038c5c17919ec53633124bc5ae1.png)

图片由[康乃尔大学](http://www.csl.cornell.edu/~delimitrou/papers/2019.asplos.microservices.pdf)的研究人员拍摄

现代互联网的每一次点击都会引发大量的网络呼叫，你可能知道，[网络是不可靠的](https://blog.acolyer.org/2014/12/18/the-network-is-reliable/)。这就是为什么在通过网络获取数据时**应该设置请求超时**的原因之一。最好也结合一些其他技术，但这是另一篇文章(甚至是整本书)的主题。

## 为什么要担心超时呢？

设置它们是非常重要的，因为网络可能会失败，你的实例可能会变慢或崩溃。您不希望用户因为 1000 台服务器中的一台突然关闭而无限期等待。人们讨厌等待，这会影响他们的幸福，因此也会影响你的收入。如果网络调用停滞，您应该放弃它，重试，除非您的集群有问题，否则它最有可能成功。

让我们看看如何用 Java 实现它。

## 通用 Java 方法

java 中最流行的 http 客户端是 Apache HttpClient。下面是配置请求超时的样子:

使用 Apache HttpClient 设置超时

这三者都是不同且独立的。我们在这里面临的最大问题是如何将 500 毫秒表示为 3 种不同的超时——应该是 100 + 100 + 300 还是 50 + 50 + 400？更重要的是，我们关心连接超时是否需要 200 毫秒吗？假设服务器将在 50 毫秒内完成请求，那么总响应时间将是 250 毫秒，在大多数情况下— **您不要在意，这完全没问题！**

另一方面，你不能将超时设置得太大，因为这会导致更长的请求。此外，套接字超时只是从套接字读取的任何连续包之间的超时，**而不是发送回您的整个响应**。

尽管如此，这是我们使用 Apache HttpClient 所能做的一切。让我们来看看围棋。

## Go 标准库

但是，Go 支持整个客户端调用超时:

使用标准 Go http 客户端设置呼叫超时

在这里，我们将调用超时设置为 2 秒，并调用 [httpbin](http://httpbin.org) ，它将模拟工作 1 秒钟，之后返回一个响应。启动此功能将导致成功的通话。

如果我们将调用超时设置为 1 秒，它将失败，并显示类似于 given 的消息，这正是我们所期待的！

```
2019/11/25 19:06:09 Get [http://httpbin.org/delay/1](http://httpbin.org/delay/1): net/http: request canceled (Client.Timeout exceeded while awaiting headers)
```

相反，仅配置呼叫超时**并不总是我们能够实现的最佳**。想象一下，有一个又长又重的请求需要 10 秒钟才能完成。等待 10 秒钟的响应，并发现服务器一直在尝试建立连接，而实际上并没有做任何工作，这是非常糟糕的。

对此的解决方案？—连接超时。是的，我们之前责备过的那个。结合呼叫超时，它为您的网络交互提供强大的保护:

使用标准 Go http 客户端设置呼叫和连接超时

超过连接超时将会产生我们想要的行为:

```
2019/11/25 19:12:25 Get [http://httpbin.org/delay/5](http://httpbin.org/delay/5): dial tcp 54.172.95.6:80: i/o timeout
```

Go 的标准 HTTP 客户端给了你很大的灵活性，库也得到供应商的支持——完美的组合，**这就是我非常喜欢它的原因！**现在让我们再一次回到 Java 世界。

## 那么，Java 是不是注定要失败了？

**没有**还有不太流行的替代品，比如 [JDK 11](https://docs.oracle.com/en/java/javase/11/docs/api/java.net.http/java/net/http/HttpClient.html) http 客户端和 [OkHttp](https://square.github.io/okhttp/) ，支持调用超时功能。可悲的是，在谷歌上搜索时，这些并不是最热门的结果，所以人们不太了解它们，或者不愿意开始使用它们。

## JDK 11

这是一个由标准库提供的现代 http 客户端，支持 HTTP/1.1、HTTP/2、通过 CompletableFuture 的异步调用，并提供了方便的 api。让我们将这两种超时结合起来:

使用 JDK 11 http 客户端设置连接和调用超时

这个客户端将分别为连接和调用超时提供漂亮且易于理解的错误消息(特别是与 Go 相比):

```
Exception in thread "main" java.net.http.HttpConnectTimeoutException: HTTP connect timed outException in thread "main" java.net.http.HttpTimeoutException: request timed out
```

## OkHttp

OkHttp 也支持呼叫和连接超时。两者都可以在客户端抽象级别进行配置，这比 JDK11 实现稍微方便一些:

使用 OkHttp 设置超时

![](img/7b392e66f12fe79dbe7631a8e0a72bf2.png)

[阿格巴洛斯](https://unsplash.com/@agebarros?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)在 [Unsplash](https://unsplash.com/s/photos/timer?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄的照片

## 不管怎样，如何选择超时值？

最后我想说的是如何得出这两个数字。呼叫超时更多地是关于你与其他服务的 SLA/SLO，而连接超时是关于来自底层网络的期望。例如，如果您向同一个数据中心发送请求，那么 100ms 就可以了(尽管它应该比 5ms 更快地建立连接)，但是在移动网络(更容易出错)上运行将需要更长的连接超时。

## 总结

在本文中，我们讨论了为什么超时很重要，为什么“经典”超时不符合现代要求，以及使用什么工具来强制超时。希望你能从中找到有用的东西。