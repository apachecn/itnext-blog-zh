# 使用 Clojure 与 Postgres 对话

> 原文：<https://itnext.io/talking-to-postgres-with-clojure-3b2b24ebfb3?source=collection_archive---------2----------------------->

纯 Clojure，没有 JDBC！

![](img/8e5d9cebee81d827225d973bb322d03a.png)

照片由[福蒂斯·福托普洛斯](https://unsplash.com/@ffstop?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄

## 灵感

我看了下面这个视频，视频来自 [Rob Norris](https://twitter.com/tpolecat) 关于构建[Skunk](https://github.com/tpolecat/skunk)——一个 Postgres 的纯 Scala 驱动程序:

…然后被吹走了！我喜欢这种简单的方法和不依赖 JDBC 驱动的纯 Scala。我立刻想尝试在 Clojure 中构建类似的东西。所以让我带你走过我旅程的起点！

对于那些在家学习的人来说，这里有一个有用的 Docker 编写文件。

## bytebufferssocket

从与 Skunk 类似的方法开始，我们创建了一个 SocketChannel 来读写字节缓冲区。我们将使用异步通道，即`in-ch`和`out-ch`，而不是在对象上使用读写方法来发送/接收数据。

缓冲袋唱片

客户机和 Postgres 之间最基本的消息有两种类型；前端和后端。前端消息是发送给 Postgres 的消息，后端消息是从 Postgres 接收的消息。

后端消息都有一个 5 字节的头。第一个字节作为消息类型，接下来的 4 个字节是一个代表整个消息长度的`Int32`值，**包括长度本身**。

![](img/7476e9aef7f021b467f18b5957c48abb.png)

为了将接收到的字节缓冲区(来自 Postgres 的后端消息)发送给下游侦听器，我们需要:

*   阅读标题
*   确定总消息长度，
*   然后读入剩余的消息负载。

一旦我们有了完整的消息，我们就可以通过异步通道将它传递到下游。知道了这一点，我们就可以构建一个`init-buffer-socket`函数，用一对`go-loop`来设置我们的`ByteBufferSocket`，以处理如下消息:

从端口和主机名创建、连接并最终关闭一个`SocketChannel`，然后将该`SocketChannel`包装在我们的`ByteBufferSocket`类中，并构建以下实用函数:

对于具有用户“jimmy”和数据库名称“world”的 Postgres 数据库，我们可以发送以下字节数组`startup-bytes` ( [启动消息](https://www.postgresql.org/docs/current/protocol-flow.html#id-1.10.5.7.3)下面的字节是提前生成的🤓)并打印出响应:

我们从 Postgres 得到以下消息！

```
(82 0 0 0 12 0 0 0 5 -112 46 -47 -14)
```

第一个字节，`82`告诉我们这是一个认证请求消息。消息长度`0 0 0 12`是一个`Int32`，即 12，这意味着剩余的有效载荷是 8 个字节长。规范告诉我们接下来的 4 个字节，`0 0 0 5`是一个`Int32`(也就是数字 5)。这又告诉我们该请求是 [AuthenticationMD5Password 后端消息](https://www.postgresql.org/docs/current/protocol-message-formats.html)，最后 4 个字节`-112 46 -47 -14`是 salt。

所以基本上 Postgres 希望我们发送用户名和密码，用 MD5 散列加密，使用上面的盐！

*注意:上面请求的 PasswordMessage 可以用 SQL 计算如下:*

```
concat('md5', 
  md5(concat(
    md5(concat(password, username)), 
  random-salt)))
```

在大约~70 行的 Clojure 中，我们可以来回发送数据到 Postgres！围绕解析消息和远离字节数组还有很多工作要做，所以请继续关注未来的博客文章。与此同时，如果你想了解更多，请前往我在 Clunk GitHub 页面上的[实验分支！](https://github.com/duanebester/clunk/tree/experimental)