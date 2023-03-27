# Vertx 中异步编程的三种范式

> 原文：<https://itnext.io/three-paradigms-of-asynchronous-programming-in-vertx-b01ab24d0927?source=collection_archive---------1----------------------->

我想通过一个使用 Vertx 框架和 Kotlin 编程语言的简单 web 应用程序的例子来展示异步编程的三个范例——回调、未来和协同程序。

假设我们正在编写一个应用程序，它接收 HTTP 请求中的一个字符串，通过这个字符串在 DB 中搜索一个 URL，获取这个 URL 内容并将其发送回客户端。

Vertx 是为高负载应用创建的异步框架，它使用 Netty、新 I/O、事件总线。

在 Vertx 中很常见，一个 Verticle(类似于 Actor，如果你知道 Akka)接收一个请求，将接收到的字符串通过事件总线发送到另一个 Verticle——business Verticle，它自己负责获取。

```
object Main {
    [@JvmStatic](http://twitter.com/JvmStatic)
    fun main(args: Array<String>) {
        val vertx =  Vertx.vertx()
        vertx.deployVerticle(HttpVerticle())
        vertx.deployVerticle(BusinessVerticleCoroutines())
    }
}class HttpVerticle : AbstractVerticle() { [@Throws](http://twitter.com/Throws)(Exception::class)
    override fun start(startFuture: Future<Void>) {
        val router = createRouter()vertx.createHttpServer()
            .requestHandler(router)
            .listen(8080) { result ->
                if (result.succeeded()) {
                    startFuture.complete()
                } else {
                    startFuture.fail(result.cause())
                }
            }
    }private fun createRouter(): Router = Router.router(vertx).apply {
        get("/").handler(handlerRoot)
    }private val handlerRoot = Handler<RoutingContext> { rc ->
        vertx.eventBus().send("my.addr", rc.request().getParam("id") ?: "") { resp: AsyncResult<Message<String>> ->
            if (resp.succeeded()) {
                rc.response().end(resp.result().body())
            } else {
                rc.fail(500)
            }
        }
    }}
```

在标准 Vertx API 中，所有异步流都是通过回调来完成的，因此 BusinessVerticle 的初始实现如下所示:

```
class BusinessVerticle : AbstractVerticle() {private lateinit var dbclient: JDBCClient
    private lateinit var webclient: WebClientoverride fun start() {
        vertx.eventBus().consumer<String>("my.addr") { message ->
            handleMessage(message)
        }
        dbclient = JDBCClient.createShared(
            vertx, JsonObject()
                .put("url", "jdbc:postgresql://localhost:5432/payroll")
                .put("driver_class", "org.postgresql.Driver")
                .put("user", "vala")
                .put("password", "vala")
                .put("max_pool_size", 30)
        )val options = WebClientOptions()
            .setUserAgent("My-App/1.2.3")options.isKeepAlive = false
        webclient = WebClient.create(vertx, options)
    }private fun handleMessage(message: Message<String>) {
        dbclient.getConnection { res ->
            if (res.succeeded()) {val connection = res.result()connection.query("SELECT url FROM payee_company where name='${message.body()}'") { res2 ->
                    if (res2.succeeded()) {
                        try {
                            val url = res2.result().rows[0].getString("url").removePrefix("http://")
                            webclient
                                .get(url,"/")
                                .send { ar ->
                                    if (ar.succeeded()) {
                                        // Obtain response
                                        val response = ar.result()
                                        message.reply(response.bodyAsString())
                                    } else {
                                        message.fail(500, ar.cause().message)
                                    }
                                }} catch (e: Exception) {
                            message.fail(500, e.message)
                        }
                    } else {
                        message.fail(500, res2.cause().message)
                    }
                }
            } else {
                message.fail(500, res.cause().message)
            }
        }
    }
}
```

这看起来很糟糕。回调和错误处理在几个地方完成。

让我们通过将每个回调提取到一个单独的方法来改善这种情况:

```
class BusinessVerticle : AbstractVerticle() {private lateinit var dbclient: JDBCClient
    private lateinit var webclient: WebClientoverride fun start() {
        vertx.eventBus().consumer<String>("my.addr") { message ->
            handleMessage(message)
        }
        dbclient = JDBCClient.createShared(
            vertx, JsonObject()
                .put("url", "jdbc:postgresql://localhost:5432/payroll")
                .put("driver_class", "org.postgresql.Driver")
                .put("user", "vala")
                .put("password", "vala")
                .put("max_pool_size", 30)
        )val options = WebClientOptions()
            .setUserAgent("My-App/1.2.3")options.isKeepAlive = false
        webclient = WebClient.create(vertx, options)
    }private fun handleMessage(message: Message<String>) {
        dbclient.getConnection { res ->
            handleConnectionCallback(res, message)
        }
    }private fun handleConnectionCallback(
        res: AsyncResult<SQLConnection>,
        message: Message<String>
    ) {
        if (res.succeeded()) {val connection = res.result()connection.query("SELECT url FROM payee_company where name='${message.body()}'") { res2 ->
                handleQueryCallBack(res2, message)
            }
        } else {
            message.fail(500, res.cause().message)
        }
    }private fun handleQueryCallBack(
        res2: AsyncResult<ResultSet>,
        message: Message<String>
    ) {
        if (res2.succeeded()) {
            try {
                val url = res2.result().rows[0].getString("url").removePrefix("http://")
                webclient
                    .get(url, "/")
                    .send { ar ->
                        handleHttpCallback(ar, message)
                    }} catch (e: Exception) {
                message.fail(500, e.message)
            }
        } else {
            message.fail(500, res2.cause().message)
        }
    }private fun handleHttpCallback(
        ar: AsyncResult<HttpResponse<Buffer>>,
        message: Message<String>
    ) {
        if (ar.succeeded()) {
            // Obtain response
            val response = ar.result()
            message.reply(response.bodyAsString())
        } else {
            message.fail(500, ar.cause().message)
        }
    }
}
```

好点了，但还是不好。

不可读的代码。我们应该传递给所有方法的消息对象，在每个回调中单独处理错误。

让我们用 *Future* s 来重写吧， *Future* s 的一大优势就是能够将它们链起来，用 *Future.compose()*

首先，让我们翻译标准的 Vertx 方法，这些方法接收对返回 Future 的方法的回调。我们将使用“扩展方法”——kot Lin 特性，它允许向现有类添加方法。

```
fun JDBCClient.getConnectionF(): Future<SQLConnection> {
    val f = Future.future<SQLConnection>()
    getConnection { res ->
        if (res.succeeded()) {
            val connection = res.result()
            f.complete(connection)
        } else {
            f.fail(res.cause())
        }
    }
    return f
}fun SQLConnection.queryF(query:String): Future<ResultSet> {
    val f = Future.future<ResultSet>()
    query(query) { res ->
        if (res.succeeded()) {
            val resultSet = res.result()
            f.complete(resultSet)
        } else {
            f.fail(res.cause())
        }
    }
    return f
}fun  HttpRequest<Buffer>.sendF(): Future<HttpResponse<Buffer>> {
    val f = Future.future<HttpResponse<Buffer>>()
    send() { res ->
        if (res.succeeded()) {
            val response = res.result()
            f.complete(response)
        } else {
            f.fail(res.cause())
        }
    }
    return f
}
```

然后*business verticle . handle message*方法将被转换为:

```
private fun handleMessage(message: Message<String>) {
        val content = getContent(message)content.setHandler{res->
            if (res.succeeded()) {
                // Obtain response
                val response = res.result()
                message.reply(response)
            } else {
                message.fail(500, res.cause().message)
            }
        }
    }private fun getContent(message: Message<String>): Future<String> {
        val connection = dbclient.getConnectionF()
        val resultSet = connection.compose { it.queryF("SELECT url FROM payee_company where name='${message.body()}'") }
        val url = resultSet.map { it.rows[0].getString("url").removePrefix("http://") }
        val httpResponse = url.compose { webclient.get(it, "/").sendF() }
        val content = httpResponse.map { it.bodyAsString() }
        return content
    }
```

看起来不错。
简单、可读的代码。在一个地方处理错误。如果需要，我们可以为不同的异常添加不同的错误处理和/或将其提取到单独的方法中。

但是如果我们需要，在某种情况下，停止 *Future.compose* ()？
例如，如果 DB 中没有记录，我们想返回一个 HTTP 代码为 200 的响应“no records ”,而不是错误和代码 500？

唯一的方法是抛出一个特殊的异常，并对此类异常进行特殊处理:

```
class NoContentException(message:String):Exception(message)private fun getContent(message: Message<String>): Future<String> {
        val connection = dbclient.getConnectionF()
        val resultSet = connection.compose { it.queryF("SELECT url FROM payee_company where name='${message.body()}'") }
        val url = resultSet.map {
            if (it.numRows<1)
                throw NoContentException("No records")
            it.rows[0].getString("url").removePrefix("http://")
        }
        val httpResponse = url.compose { webclient.get(it, "/").sendF() }
        val content = httpResponse.map { it.bodyAsString() }
        return content
    }private fun handleMessage(message: Message<String>) {
        val content = getContent(message)content.setHandler{res->
            if (res.succeeded()) {
                // Obtain response
                val response = res.result()
                message.reply(response)
            } else {
                if (res.cause() is NoContentException)
                    message.reply(res.cause().message)
                else
                    message.fail(500, res.cause().message)
            }
        }
    }
```

它工作得很好，但看起来不太好——我们正在使用异常进行流控制，如果流中有许多“特殊情况”,代码的可读性会差得多。

所以，让我们试着用 Kotlin 协程做同样的事情。
关于 Kotlin 协程的文章很多，这里就不解释它们是什么，如何工作了。

在 Vertx 的最新版本中，可以包含自动生成的所有回调方法的协程友好版本。

你应该添加库
*‘vertx-lang-kotlin-coroutines’
‘vertx-lang-kotlin’*

并且得到*JDBC client . getconnectionawait()、SQLConnection.queryAwait()* 等。

如果我们使用这些方法，我们的消息处理方法将变得更简单:

```
private suspend fun handleMessage(message: Message<String>) {
        try {
            val content = getContent(message)
            message.reply(content)
        } catch(e:Exception){
            message.fail(500, e.message)
        }}private suspend fun getContent(message: Message<String>): String {
        val connection = dbclient.getConnectionAwait()
        val resultSet = connection.queryAwait("SELECT url FROM payee_company where name='${message.body()}'")
        val url =  resultSet.rows[0].getString("url").removePrefix("http://")
        val httpResponse = webclient.get(url, "/").sendAwait()
        val content = httpResponse.bodyAsString()
        return content
    }
```

此外，您应该更改事件总线消息订阅代码:

```
vertx.eventBus().consumer<String>("my.addr") { message ->
           GlobalScope.launch (vertx.dispatcher()) {  handleMessage(message)}
        }
```

这里发生了什么？

所有这些“await”方法都以异步方式调用代码，然后等待结果，在等待的同时，线程切换到运行另一个协程。

如果我们期待那些“await”方法的实现，我们会看到与我们自制的 *Futures* 的实现非常相似的东西:

```
suspend fun SQLClient.getConnectionAwait(): SQLConnection {
  return awaitResult {
    this.getConnection(it)
  }
}
suspend fun <T> awaitResult(block: (h: Handler<AsyncResult<T>>) -> Unit): T {
  val asyncResult = awaitEvent(block)
  if (asyncResult.succeeded()) return asyncResult.result()
  else throw asyncResult.cause()
}
suspend fun <T> awaitEvent(block: (h: Handler<T>) -> Unit): T {
  return suspendCancellableCoroutine { cont: CancellableContinuation<T> ->
    try {
      block.invoke(Handler { t ->
        cont.resume(t)
      })
    } catch (e: Exception) {
      cont.resumeWithException(e)
    }
  }
}
```

但是这里我们得到了一个普通的代码——String 作为返回类型(而不是未来的<string>),用 try/catch 代替了丑陋的 AsyncResult 回调。</string>

如果我们需要在中间停止一个流程，我们可以用自然的方式来做，没有例外:

```
private suspend fun getContent(message: Message<String>): String {
        val connection = dbclient.getConnectionAwait()
        val resultSet = connection.queryAwait("SELECT url FROM payee_company where name='${message.body()}'")
        if (resultSet.numRows<1)
            return "No records"
        val url =  resultSet.rows[0].getString("url").removePrefix("http://")
        val httpResponse = webclient.get(url, "/").sendAwait()
        val content = httpResponse.bodyAsString()
        return content
    }
```

依姆霍，这真是太美了！

文章的源代码:https://gitlab.com/bernshtam-articles/vertx-async