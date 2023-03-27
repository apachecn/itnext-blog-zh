# 使用 Vertx HttpClient/WebClient 使用 GraphQL APIs

> 原文：<https://itnext.io/consuming-graphql-apis-with-vertx-httpclient-webclient-db410c410aa2?source=collection_archive---------0----------------------->

在[之前的一篇文章](https://github.com/hantsy/vertx-sandbox/blob/master/docs/client)中，我们已经介绍了使用 Vertx HttpClient 和 WebClient 消费 RESTful APIs。在这篇文章中，我们将使用我们在上一篇文章的[中创建的 GraphQL APIs。](https://github.com/hantsy/vertx-sandbox/blob/master/docs/graphql.md)

![](img/8166aa811d3c05626ec35cb176650ce5.png)

萨姆·巴耶在 [Unsplash](https://unsplash.com/s/photos/china-mountain?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上的照片

首先，让我们回顾一下 HttpClient 和 WebClient 之间的区别。

*   HttpClient 是一个低级 API，它提供了与 Http 服务器交互的细粒度控制。
*   WebClient 是一个高级 API，它提供了更方便的方法来简化处理 web 请求和响应。在大多数情况下，WebClient 是首选，但它缺乏 WebSocket 支持，所以当启动 WebSocket 连接时，我们必须切换到 HttpClient。

## 从我的 Github 查看[完整的示例代码。](https://github.com/hantsy/vertx-sandbox/tree/master/graphql-webclient)

假设您已经阅读了 [GraphQL over HTTP 规范](https://graphql.org/learn/serving-over-http/)和 [GraphQL 多部分请求规范](https://github.com/jaydenseric/graphql-multipart-request-spec)。

通过 [Eclipse Vertx Starter](https://start.vertx.io) 创建一个 Eclipse Vertx 项目。

在`MainVerticle`类的 start 方法中，首先创建一个`WebClient`对象。

```
var options = new WebClientOptions()
    .setUserAgent(WebClientOptions.loadUserAgent())
    .setDefaultHost("localhost")
    .setDefaultPort(8080);var client = WebClient.create(vertx, options);
```

下面是一个发送 GraphQL 请求来检索所有帖子的示例。

```
client.post("/graphql")
     .sendJson(Map.of(
         "query", "query posts{ allPosts{ id title content author{ name } comments{ content createdAt} createdAt}}",
         "variables", Map.of()
     ))
     .onSuccess(
         data -> log.info("data of allPosts: {}", data.bodyAsString())
     )
     .onFailure(e -> log.error("error: {}", e));
```

请求正文格式如下。

```
{
  "query": "...",
  "operationName": "...",
  "variables": { "myVariable": "someValue", ... }
}
```

`query`正在接受字符串形式的*模式定义语言*。*操作名称*和*变量*是可选的。

响应体是这样的。

```
{
  "data": "...",
  "errors": "..."
}
```

当一个错误发生时，数据是空的，而 errors 将包含描述错误或异常的错误细节。

> 与 RESTful APIs 不同，在大多数情况下，如果出现错误，并且错误来自我们的应用程序本身，HTTP 响应状态代码总是 200。

下面的例子演示了如何创建一个帖子，然后通过 id 检索新创建的帖子。

```
client.post("/graphql")
     .sendJson(Map.of(
         "query", "mutation newPost($input:CreatePostInput!){ createPost(createPostInput:$input)}",
         "variables", Map.of(
             "input", Map.of("title", "create post from WebClient", "content", "content of the new post")
         )
     ))
     .onSuccess(
         data -> {
             log.info("data of createPost: {}", data.bodyAsString());
             var createdId = data.bodyAsJsonObject().getJsonObject("data").getString("createPost");
             // get the created post.
             getPostById(client, createdId);
             // add comment.
             addComment(client, createdId);
         }
     )
     .onFailure(e -> log.error("error: {}", e));//getPostById
private void getPostById(WebClient client, String id) {
        client.post("/graphql")
            .sendJson(Map.of(
                "query", "query post($id:String!){ postById(postId:$id){ id title content author{ name } comments{ content createdAt} createdAt}}",
                "variables", Map.of(
                    "id", id
                )
            ))
            .onSuccess(
                data -> log.info("data of postByID: {}", data.bodyAsString())
            )
            .onFailure(e -> log.error("error: {}", e));
    }
```

添加一个`addComment`方法来演示为文章创建评论的过程，同时订阅`commentAdded`事件(*订阅*)。

```
private void addComment(WebClient client, String id) {
 // switch to HttpClient to handle WebSocket
    var options = new HttpClientOptions()
        .setWebSocketClosingTimeout(7200)
        .setDefaultHost("localhost")
        .setDefaultPort(8080); // see: [https://github.com/vert-x3/vertx-web/blob/master/vertx-web-graphql/src/test/java/io/vertx/ext/web/handler/graphql/ApolloWSHandlerTest.java](https://github.com/vert-x3/vertx-web/blob/master/vertx-web-graphql/src/test/java/io/vertx/ext/web/handler/graphql/ApolloWSHandlerTest.java)
    var httpClient = vertx.createHttpClient(options);
    httpClient.webSocket("/graphql")
        .onSuccess(ws -> {
            ws.closeHandler(v -> log.info("websocket is being closed"));
            ws.endHandler(v -> log.info("websocket is being ended"));
            ws.exceptionHandler(e -> log.info("catching websocket exception: {}", e.getMessage())); ws.textMessageHandler(text -> {
                //log.info("websocket message handler:{}", text);
                JsonObject obj = new JsonObject(text);
                ApolloWSMessageType type = ApolloWSMessageType.from(obj.getString("type"));
                if (type.equals(CONNECTION_KEEP_ALIVE)) {
                    return;// do nothing when ka.
                } else if (type.equals(DATA)) {
                    // handle the subscription `commentAdded` data.
                    log.info("subscription commentAdded data: {}", obj.getJsonObject("payload").getJsonObject("data").getJsonObject("commentAdded"));
                }
            }); JsonObject messageInit = new JsonObject()
                .put("type", "connection_init")//this is required to initialize a connection.
                .put("id", "1"); JsonObject message = new JsonObject()
                .put("payload", new JsonObject()
                     .put("query", "subscription onCommentAdded { commentAdded { id content } }"))
                .put("type", "start")
                .put("id", "1"); ws.write(messageInit.toBuffer());
            ws.write(message.toBuffer());
        })
        .onFailure(e -> log.error("error: {}", e)); addCommentToPost(client, id);
    addCommentToPost(client, id);
    addCommentToPost(client, id);
}
```

在上面的`addComment`方法中，我们切换到使用`HttpClient`来处理 WebSocket 请求。

首先，它打开一个到 */graphql* WebSocket 端点的 WebSocket 连接，然后将 graphql 请求作为消息负载写入，并使用一个`textMessageHandler`回调钩子从服务器端获取 WebSocket 响应。

让我们看看上传的文件。

```
private void uploadFile(WebClient client) {
    Buffer fileBuf = vertx.fileSystem().readFileBlocking("test.txt");
    MultipartForm form = MultipartForm.create();
    String query = """
        mutation upload($file:Upload!){
        upload(file:$file)
    }
    """;
        var variables = new HashMap<String, Object>();
    variables.put("file", null);
    form.attribute("operations", Json.encode(Map.of("query", query, "variables", variables)));
    form.attribute("map", Json.encode(Map.of("file0", List.of("variables.file"))));
    form.textFileUpload("file0", "test.txt", fileBuf, "text/plain"); client.post("/graphql")
        .sendMultipartForm(form)
        .onSuccess(
        data -> log.info("data of upload: {}", data.bodyAsString())
    )
    .onFailure(e -> log.error("error: {}", e));
}
```

如您所见，使用高级 WebClient API 创建多部分表单并通过`sendMultipartForm`直接发送是非常容易的。

从我的 Github 中获取[示例代码。](https://github.com/hantsy/vertx-sandbox/tree/master/graphql-webclient)