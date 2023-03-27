# 通过 Vertx HttpClient/WebClient 使用 RESTful APIs

> 原文：<https://itnext.io/consuming-with-restful-apis-with-vertx-httpclient-webclient-6a2f61d94029?source=collection_archive---------3----------------------->

我们已经讨论了如何用基本的 Eclipse Vertx Web 特性构建 RESTful APIs。在这篇文章中，我们将介绍如何创建一个 *Http 客户端*并与 RESTful APIs 交互。

![](img/2827a7fde31bda3478e8b3b1d8862fbe.png)

约书亚·厄尔在 [Unsplash](https://unsplash.com/s/photos/china-landscape?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄的照片

与创建 HttpServer 类似，您可以从`Vertx`实例创建一个 HttpClient。

```
var options = new HttpClientOptions()
    .setDefaultPort(8888);
var client = vertx.createHttpClient(options);
```

然后，您可以像下面这样向服务器发送请求。

```
client.request(HttpMethod.GET, "/hello")
    .compose(req -> req.send().compose(HttpClientResponse::body))
    .onSuccess(...)
    .onFailure(...)
```

`HttpClient`是一个低级 API，提供对请求和响应信息的细粒度控制。Vertx 提供了一个更高级的 API 来与服务器端握手，它被称为`WebClient`。

类似于`HttpClient`的创建，像这样创建一个`WebClient`实例。

```
var options = new WebClientOptions()
            .setDefaultHost("localhost")
            .setDefaultPort(8888);
var webclient = WebClient.create(vertx, options);
```

或者从现有的`HttpClient`实例创建一个`WebClient`。

```
var webclient = WebClient.wrap(httpClient, options);
```

一个使用 WebClient 发送请求的例子。

```
client.get("/posts")
    .send()
    .onSuccess(...)
    .onFailure(...)
```

与`HttpClient`相比，它提供了更全面的发送请求体的方法，例如 JSON 对象、多部分形式等。

接下来让我们来测试 Vertx，并使用 HttpClient/WebClient 来测试 RESTful API 端点。

下面的例子是一个简单的 JUnit 5 测试。

```
@ExtendWith(VertxExtension.class)
public class TestMainVerticle {
    private final static Logger LOGGER = Logger.getLogger(TestMainVerticle.class.getName());
    HttpClient client; @BeforeEach
    void setup(Vertx vertx, VertxTestContext testContext) {
        vertx.deployVerticle(new MainVerticle(), testContext.succeeding(id -> testContext.completeNow()));
        var options = new HttpClientOptions()
            .setDefaultPort(8888);
        this.client = vertx.createHttpClient(options);
    } @DisplayName("Check the HTTP response...")
    void testHello(Vertx vertx, VertxTestContext testContext) {
        client.request(HttpMethod.GET, "/hello")
            .compose(req -> req.send().compose(HttpClientResponse::body))
            .onComplete(
                testContext.succeeding(
                    buffer -> testContext.verify(
                        () -> {
                            assertThat(buffer.toString()).isEqualTo("Hello from my route");
                            testContext.completeNow();
                        }
                    )
                )
            );
    }}
```

使用`@ExtendWith(VertxExtension.class)`，它允许您在测试方法中注入`Vertx`和`VertxTestContext`参数。`VertxTestContext`包装一个`CountDownLatch`表示异步执行完成，你必须调用`completeNow`或`failNow`来结束执行，否则测试执行将被阻塞直到超时。

让我们看一个测试 */posts* 端点以获取所有帖子的例子。

```
@Test
void testGetAll(Vertx vertx, VertxTestContext testContext) {
    client.request(HttpMethod.GET, "/posts")
        .flatMap(HttpClientRequest::send)
        .flatMap(HttpClientResponse::body)
        .onComplete(
        testContext.succeeding(
            buffer -> testContext.verify(
                () -> {
                    assertThat(buffer.toJsonArray().size()).isEqualTo(2);
                    testContext.completeNow();
                }
            )
        )
    );
}
```

下面的例子测试了返回 404 HTTP 状态码的`PostNotFoundException`。

```
@Test
void testGetByNoneExistingId(Vertx vertx, VertxTestContext testContext) {
    var postByIdUrl = "/posts/" + UUID.randomUUID();
    client.request(HttpMethod.GET, postByIdUrl)
        .flatMap(HttpClientRequest::send)
        .onComplete(
        testContext.succeeding(
            response -> testContext.verify(
                () -> {
                    assertThat(response.statusCode()).isEqualTo(404);
                    testContext.completeNow();
                }
            )
        )
    );
}
```

客户端操作可以被链接。下面是一个测试 CRUD 流的例子。

```
@Test
void testCreatePost(Vertx vertx, VertxTestContext testContext) {
    client.request(HttpMethod.POST, "/posts")
        .flatMap(req -> req.putHeader("Content-Type", "application/json")
                 .send(Json.encode(CreatePostCommand.of("test title", "test content of my post")))
                 .onSuccess(
                     response -> assertThat(response.statusCode()).isEqualTo(201)
                 )
                )
        .flatMap(response -> {
            String location = response.getHeader("Location");
            LOGGER.log(Level.INFO, "location header: {0}", location);
            assertThat(location).isNotNull(); return Future.succeededFuture(location);
        })
        .flatMap(url -> client.request(HttpMethod.GET, url)
                 .flatMap(HttpClientRequest::send)
                 .onSuccess(response -> {
                     LOGGER.log(Level.INFO, "http status: {0}", response.statusCode());
                     assertThat(response.statusCode()).isEqualTo(200);
                 })
                 .flatMap(res -> client.request(HttpMethod.PUT, url)
                          .flatMap(req -> req.putHeader("Content-Type", "application/json")
                                   .send(Json.encode(CreatePostCommand.of("updated test title", "updated test content of my post")))
                                  )
                          .onSuccess(response -> {
                              LOGGER.log(Level.INFO, "http status: {0}", response.statusCode());
                              assertThat(response.statusCode()).isEqualTo(204);
                          })
                         )
                 .flatMap(res -> client.request(HttpMethod.GET, url)
                          .flatMap(HttpClientRequest::send)
                          .onSuccess(response -> {
                              LOGGER.log(Level.INFO, "http status: {0}", response.statusCode());
                              assertThat(response.statusCode()).isEqualTo(200); })
                          .flatMap(HttpClientResponse::body)
                          .onSuccess(body -> assertThat(body.toJsonObject().getString("title")).isEqualTo("updated test title"))
                         )
                 .flatMap(res -> client.request(HttpMethod.DELETE, url)
                          .flatMap(HttpClientRequest::send)
                          .onSuccess(response -> {
                              LOGGER.log(Level.INFO, "http status: {0}", response.statusCode());
                              assertThat(response.statusCode()).isEqualTo(204);
                          })
                         )
                 .flatMap(res -> client.request(HttpMethod.GET, url)
                          .flatMap(HttpClientRequest::send)
                          .onSuccess(response -> {
                              LOGGER.log(Level.INFO, "http status: {0}", response.statusCode());
                              assertThat(response.statusCode()).isEqualTo(404);
                          })
                         )
                )
        .onComplete(
        testContext.succeeding(id -> testContext.completeNow())
    );
}
```

同样，您可以在测试中使用`WebClient`。

下面是一个使用 RxJava 3 绑定的`WebClient`的例子。

```
@Test
void testGetAll(Vertx vertx, VertxTestContext testContext) {
    client.get("/posts")
        .rxSend()
        .subscribe(
        response -> {
            assertThat(response.statusCode()).isEqualTo(200);
            assertThat(response.bodyAsJsonArray().size()).isEqualTo(2); testContext.completeNow();
        }
    );
}
```

`WebClient`提供了更流畅的 API。在大多数情况下，发送请求要比原始的 HttpClient 容易，尤其是。处理多部分表单。我们将在今后探索它。