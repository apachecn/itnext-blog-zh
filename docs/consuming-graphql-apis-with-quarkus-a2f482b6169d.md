# 使用 Quarkus 使用 GraphQL APIs

> 原文：<https://itnext.io/consuming-graphql-apis-with-quarkus-a2f482b6169d?source=collection_archive---------2----------------------->

在[上一篇文章](https://hantsy.medium.com/building-graphql-apis-with-quarkus-dbbf23f897df)中，我们已经构建了一个简单的 GraphQL API 示例，现在让我们讨论如何使用 GraphQL 客户端与后端 graph QL API 进行交互。

![](img/6357465393cb0b31ffb7a85f29b4d364.png)

图片来自[https://unsplash.com/photos/6NzHYsK_-Ow](https://unsplash.com/photos/6NzHYsK_-Ow)

# 生成项目框架

就像我们在过去的帖子中所做的那样，你应该首先准备一个项目框架。

使用 [Quarkus 代码生成器](https://code.quarkus.io)创建一个 Quarkus 项目，将源代码导入你的 IDE。

打开 *pom.xml* 文件，添加以下依赖项。

```
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-smallrye-graphql-client</artifactId>
</dependency>
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-rest-client</artifactId>
</dependency>
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-rest-client-jsonb</artifactId>
</dependency>
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.20</version>
</dependency>
```

Lombok 用于擦除 setters、getters、hashCode、equals、toString 等。让它看起来干净点。

# 声明 GraphQL 客户端 API

与 MicroProfile RestClient 规范类似，SmallRye GraphQL 提供了从接口声明 GraphQL 客户端的相同方法。

> *注意，这个客户端 API 不是 MicroProfile GraphQL 规范的一部分。*

```
@GraphQLClientApi
public interface PostGraphQLClient {
    @Query()
    public List<Post> getAllPosts() ; @Query
    @Description("Get a specific post by providing an id")
    public Post getPostById(@Name("postId") String id); @Mutation
    @Description("Create a new post")
    public Post createPost(@Valid CreatePost createPostInput);
}
```

上述代码中使用的 POJO 类，如`Post`、`Coment`和`CreatePost`都是从我们在上一篇文章中创建的[后台 GraphQL API 项目](https://github.com/hantsy/quarkus-sandbox/blob/master/graphql)中复制过来的。

要定位将要连接的远程 GraphQL API，类似于 MP RestClient API，请配置远程 GraphQL API 的基本 URI。

```
com.example.demo.PostGraphQLClient/mp-graphql/url=http://localhost:8080/graphql
```

现在让我们试着调用`PostGraphQLClient`的`getAllPosts`并打印出所有帖子。

创建一个实现`QuarkusApplication`的类并用`@QuarkusMain`对其进行注释，它将像普通 Java 应用程序中的主类一样工作。

```
@QuarkusMain
public class Main implements QuarkusApplication {
    public static final Logger LOGGER = Logger.getLogger(Main.class.getName()); @Inject
    PostGraphQLClient clientApi; @Override
    public int run(String... args) throws Exception {
        this.clientApi.getAllPosts().forEach(
            p -> LOGGER.log(Level.INFO, "post: {0}", p)
        );
        return 0;
    }
}
```

打开您的终端并切换到项目根文件夹。运行`mvn quarkus:dev`以开发模式启动应用程序。启动后，您将在控制台日志中看到以下信息。

```
2021-06-03 20:21:00,505 INFO  [io.sma.gra.cli.typ.jax.JaxRsTypesafeGraphQLClientProxy] (Quarkus Main Thread) request graphql: query allPosts { allPosts {id title content countOfComment
s comments {id content}} }
2021-06-03 20:21:00,517 INFO  [com.exa.dem.Main] (Quarkus Main Thread) post: Post(id=9ef6d49f-50bf-4973-a122-3ac56a1b8d41, title=title #1, content=test content of #1, countOfComments=2
, comments=[Comment(id=e261e9cb-f06c-4989-bbb5-00319087496d, content=comment #1), Comment(id=7664a80f-0963-46c1-8955-c096d7609c1b, content=comment #2)])
2021-06-03 20:21:00,517 INFO  [com.exa.dem.Main] (Quarkus Main Thread) post: Post(id=0c5501eb-cb7e-42e2-9078-040af89e6310, title=title #2, content=test content of #2, countOfComments=2
, comments=[Comment(id=bf4e2553-cb59-4e87-bc8f-36360732a919, content=comment #1), Comment(id=165d6743-fd74-4ad5-8997-3853fb076403, content=comment #2)])
2021-06-03 20:21:00,517 INFO  [com.exa.dem.Main] (Quarkus Main Thread) post: Post(id=c61c6b62-584b-455f-a2a8-1a4253f832b7, title=title #3, content=test content of #3, countOfComments=0
, comments=[])
2021-06-03 20:21:00,517 INFO  [com.exa.dem.Main] (Quarkus Main Thread) post: Post(id=37e28b7d-11fc-4587-920f-9415da1d93a3, title=title #4, content=test content of #4, countOfComments=2
, comments=[Comment(id=5963588f-fbe0-4c82-88f6-1011bb7538fe, content=comment #1), Comment(id=feaf4d21-e78b-4701-91bb-ea665a9a034c, content=comment #2)])
```

与 REST APIs 相比，GraphQL 最吸引人的特性是它只返回客户端请求的必需字段。在上面的代码中，它返回一个`Post`的所有字段。

假设您只想在执行`allPosts`查询时检索结果中的 *title* 字段，尝试创建一个新的 POJO 类，该类只包含一个 *title* 属性。

```
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@ToString
public class PostSummary {
    String title;
}
```

在上面的`PostGraphQLClient`类中添加一个新方法。

```
@Query("allPosts")
public List<PostSummary> getAllPostSummaries() ;
```

让我们调用它并打印出结果。

```
this.clientApi.getAllPostSummaries().forEach(
    p -> LOGGER.log(Level.INFO, "post summary: {0}", p)
);
```

您可以从应用程序日志中看到以下信息。

```
2021-06-03 20:21:00,533 INFO  [io.sma.gra.cli.typ.jax.JaxRsTypesafeGraphQLClientProxy] (Quarkus Main Thread) request graphql: query allPosts { allPosts {title} }
2021-06-03 20:21:00,533 INFO  [com.exa.dem.Main] (Quarkus Main Thread) post summary: PostSummary(title=title #1)
2021-06-03 20:21:00,533 INFO  [com.exa.dem.Main] (Quarkus Main Thread) post summary: PostSummary(title=title #2)
2021-06-03 20:21:00,533 INFO  [com.exa.dem.Main] (Quarkus Main Thread) post summary: PostSummary(title=title #3)
2021-06-03 20:21:00,533 INFO  [com.exa.dem.Main] (Quarkus Main Thread) post summary: PostSummary(title=title #4)
```

如您所料，它只请求 GraphQL 查询中的 *title* 字段，并在响应中返回确切的字段。

# 处理客户端异常

目前，在调用客户端 API 时，Quarkus 没有像 MP RestClient API 那样提供处理异常的注册表，但是有一些可能的方法来实现这个目的。

困难的方法是使用 try/catch 来处理`GraphQLClientException`，例如。

```
String id = UUID.randomUUID().toString();
// catch a GraphQLClientException.
try {
    var p = this.clientApi.getPostById(id);
    LOGGER.log(Level.INFO, "post: {0}", p);
} catch (GraphQLClientException e) {
    if (e.getErrors().stream().anyMatch(error -> error.getErrorCode().equals("POST_NOT_FOUND"))) {
        throw new PostNotFoundException(id);
    }
}
```

在上面的代码中，如果 postId 不存在，就会抛出一个`GraphQLClientException`。

除此之外，用`ErrorOr`类包装您的响应是一个替代解决方案。

```
@Query
@Description("Get a specific post by providing an id")
public ErrorOr<Post> getPostById(@Name("postId") String id);
```

`ErrorOr`非常类似于 Java 的`Optional`，你可以导航两部分中的一部分，**数据**和**错误**。当错误存在时，`getErrors`方法将以静默的方式从后端 GraphQL API 收集所有错误(与 try/catch 方法相比)。

```
String id = UUID.randomUUID().toString();// return a `ErrorOr` instead.
var post = this.clientApi.getPostById(id);
if (post.isPresent()) {
    LOGGER.log(Level.INFO, "found: {0}", post.get());
}
if (post.isError()) {
    post.getErrors().forEach(
        error -> LOGGER.log(Level.INFO, "error: code={0}, message={1}", new Object[]{error.getErrorCode(), error.getMessage()})
    );
}
```

运行应用程序时，您可以看到以下日志。

```
2021-06-03 20:21:00,423 INFO  [io.sma.gra.cli.typ.jax.JaxRsTypesafeGraphQLClientProxy] (Quarkus Main Thread) request graphql: query postById($arg0: String) { postById(postId: $arg0) {i
d title content countOfComments comments {id content}} }
2021-06-03 20:21:00,505 INFO  [com.exa.dem.Main] (Quarkus Main Thread) error: code=POST_NOT_FOUND, message=Post: 8c46ca01-530e-47c1-a257-e86333bcb69b was not found.
```

# 动态客户端

我们已经讨论了客户端使用`@GraphQLClientApi`，Quarkus 也提供了一个**动态客户端**。它更像是 GraphQL 请求表单的 Java 翻译。

例如，在 [GraphQL UI](http://localhost:8080/q/graphql-ui/) 中执行以下查询来检索所有帖子。

```
query {
  allPosts {
    id
    title
    content
    comments {
      id
      content
    }
  }
}
```

创建一个新的 bean 来做同样的工作。

```
@ApplicationScoped
public class PostDynamicClient { @Inject
    @NamedClient("post-dynamic-client")
    DynamicGraphQLClient dynamicClient; public List<Post> getAllPosts() throws ExecutionException, InterruptedException {
        Document query = document(
                operation(
                        field("allPosts",
                                field("id"),
                                field("title"),
                                field("content"),
                                field("comments",
                                        field("id"),
                                        field("content")
                                )
                        )
                )
        );
        Response response = dynamicClient.executeSync(query);
        return response.getList(Post.class, "allPosts");
    }
}
```

通过`document`、`operation`和`field`静态方法，我们用 Java *方言*构建了相同的查询结构。

用于标识不同客户端的`NamedClient`的值，也用作 *application.properties* 中的配置键。

```
post-dynamic-client/mp-graphql/url=http://localhost:8080/graphql
```

现在尝试调用这个客户端的`getAllPosts`并打印出所有帖子。

```
@Inject
PostDynamicClient dynamicClient;this.dynamicClient.getAllPosts().forEach(
    p -> LOGGER.log(Level.INFO, "post from dynamic client: {0}", p)
);
```

运行应用程序，您可以看到以下信息。

```
1, countOfComments=0, comments=[Comment(id=e261e9cb-f06c-4989-bbb5-00319087496d, content=comment #1), Comment(id=7664a80f-0963-46c1-8955-c096d7609c1b, content=comment #2)])
2021-06-03 20:21:01,507 INFO  [com.exa.dem.Main] (Quarkus Main Thread) post from dynamic client: Post(id=0c5501eb-cb7e-42e2-9078-040af89e6310, title=title #2, content=test content of #
2, countOfComments=0, comments=[Comment(id=bf4e2553-cb59-4e87-bc8f-36360732a919, content=comment #1), Comment(id=165d6743-fd74-4ad5-8997-3853fb076403, content=comment #2)])
2021-06-03 20:21:01,507 INFO  [com.exa.dem.Main] (Quarkus Main Thread) post from dynamic client: Post(id=c61c6b62-584b-455f-a2a8-1a4253f832b7, title=title #3, content=test content of #
3, countOfComments=0, comments=[])
2021-06-03 20:21:01,507 INFO  [com.exa.dem.Main] (Quarkus Main Thread) post from dynamic client: Post(id=37e28b7d-11fc-4587-920f-9415da1d93a3, title=title #4, content=test content of #
4, countOfComments=0, comments=[Comment(id=5963588f-fbe0-4c82-88f6-1011bb7538fe, content=comment #1), Comment(id=feaf4d21-e78b-4701-91bb-ea665a9a034c, content=comment #2)])
```

# HttpClient

在 Quarkus 中，GraphQL 客户端通过 HTTP 协议与后端 GraphQL API 握手。理想情况下，如果您熟悉 GraphQL interexchange format(JSON)，您可以使用任何 HttpClient 发送一个 *POST* 请求来执行 graph QL 查询，包括 cURL、Resteasy Client/JAXRS Client 或简单的 Java 11 HttpClient。

以下是使用`cUrl`命令的例子。

```
curl [http://localhost:8080/graphql](http://localhost:8080/graphql) -H "Accept: application/json" -H "Content-Type: application/json" -d "{\"query\": \"query {  allPosts {  id title content comments { id  content } } }\" }"
{"data":{"allPosts":[{"id":"5049edc2-58a0-4a0a-9204-753fc4541fa0","title":"title #1","content":"test content of #1","comments":[{"id":"1124e65f-bf4b-41a1-9f34-4daab83b3a31","content":"comment #1"},{"id":"cc68bc61-3769-4a9c-a572-fdd66f6ebffe","content":"comment #2"},{"id":"3d9164d1-9da9-4483-abcb-6d29ffc3b938","content":"comment #3"},{"id":"fbba10b4-9309-442b-8b25-c2ef6e25023c","content":"comment #4"}]},{"id":"d3c130ea-f537-4b77-8125-70b620469c9f","title":"title #2","content":"test content of #2","comments":[{"id":"a28d8b31-65fe-4058-a9bc-739731f5b7d6","content":"comment #1"},{"id":"855b6773-305d-488a-932e-5cc12eb51c93","content":"comment #2"},{"id":"74b1e329-2b85-4e3e-82e8-2b49cc3d387b","content":"comment #3"}]},{"id":"2325744b-46ef-4061-bc03-8f0ca5d7c955","title":"title #3","content":"test content of #3","comments":[{"id":"54217aae-9db1-4403-bbd5-f341e4a53d84","content":"comment #1"}]},{"id":"407dd60c-966c-4d82-b260-f58edba7db14","title":"title #4","content":"test content of #4","comments":[{"id":"f46c8c3e-ddbc-48eb-8a0f-952081bf6d35","content":"comment #1"},{"id":"394a8bec-5e0c-4a69-8b82-a71a4f856778","content":"comment #2"}]}]}}
```

在 Java 11 中，增加了一个新的`HttpClient`，下面是一个使用这个 HttpClient 的例子。

```
@ApplicationScoped
public class JvmClient {
    private final ExecutorService executorService = Executors.newFixedThreadPool(5); private final HttpClient httpClient = HttpClient.newBuilder()
            .executor(executorService)
            .version(HttpClient.Version.HTTP_2)
            .build(); @Inject
    Jsonb jsonb; CompletionStage<List<Post>> getAllPosts() { // Have to erase the new line chars in the GraphQL schema to avoid the parsing exception.
        // see: [https://github.com/quarkusio/quarkus/issues/17667](https://github.com/quarkusio/quarkus/issues/17667)
        var queryString = """
                {"query": "query {
                            allPosts {
                              id
                              title
                              content
                              comments {
                                id
                                content
                              }
                            }
                          }"
                }
                """.replaceAll("\\n", " ");
        var request = HttpRequest.newBuilder()
                .POST(HttpRequest.BodyPublishers.ofString(queryString))
                .uri(URI.create("http://localhost:8080/graphql"))
                .header("Accept", "application/json")
                .header("Content-Type", "application/json")
                .build();
        HttpResponse.BodyHandler<String> handler = HttpResponse.BodyHandlers.ofString();
        return this.httpClient
                .sendAsync(request, handler)
                .thenApply(HttpResponse::body)
                .thenApply(this::extractPosts); } /**
     * @param s the GraphQL response, eg.
     * @formatter:off
     *          {
     *              data:{
     *                  allPosts: [
     *                      {
     *                      id:"xxxx",
     *                      title:"test title",
     *                      content:"content"
     *                      }
     *                  ]
     *              }
     *          }
     * @formatter:on
     * @return The parsed the list of post data.
     */
    List<Post> extractPosts(String s) {
        var reader = new StringReader(s);
        var json = Json.createReader(reader).read();
        var pointer = Json.createPointer("/data/allPosts");
        var jsonArray = (JsonArray) pointer.getValue(json);
        //@formatter:off
        return jsonb.fromJson(jsonArray.toString(), new TypeLiteral<List<Post>>() {}.getRawType());
        //@formatter:on
    }
}
```

为了从 GraphQL 客户端响应中提取 posts 数据，我使用 JSONP 指针来定位 *posts* JSON 数组，并通过 JsonB 将其转换为`List<Post>`。将`jsonp`扩展名添加到项目 deps。

```
mvn quarkus:add-extension -Dextensions="jsonp"
```

> *文本块*很适合构成多行字符串，但是有一个* [*问题*](https://github.com/quarkusio/quarkus/issues/17667) *导致 GraphQL 模式解析失败。最后我添加了一个* `*replaceAll*` *的方法来删除换行符来暂时克服这个问题。**

*尝试调用该客户端的`getAllPosts`。*

```
*@Inject
JvmClient jvmClient;this.jvmClient.getAllPosts()
    .thenAccept(
    	p -> LOGGER.log(Level.INFO, "post from jvm client: {0}", p)
	)
    .whenComplete((d, e) -> LOGGER.info("The request is done in the jvm client."))
    .toCompletableFuture()
    .join();*
```

*运行应用程序，您将在应用程序日志中看到以下内容。*

```
*2021-06-03 20:21:01,662 INFO  [com.exa.dem.Main] (ForkJoinPool.commonPool-worker-7) post from jvm client: [{comments=[{id=e261e9cb-f06c-4989-bbb5-00319087496d, content=comment #1}, {id
=7664a80f-0963-46c1-8955-c096d7609c1b, content=comment #2}], id=9ef6d49f-50bf-4973-a122-3ac56a1b8d41, title=title #1, content=test content of #1}, {comments=[{id=bf4e2553-cb59-4e87-bc8
f-36360732a919, content=comment #1}, {id=165d6743-fd74-4ad5-8997-3853fb076403, content=comment #2}], id=0c5501eb-cb7e-42e2-9078-040af89e6310, title=title #2, content=test content of #2
}, {comments=[], id=c61c6b62-584b-455f-a2a8-1a4253f832b7, title=title #3, content=test content of #3}, {comments=[{id=5963588f-fbe0-4c82-88f6-1011bb7538fe, content=comment #1}, {id=fea
f4d21-e78b-4701-91bb-ea665a9a034c, content=comment #2}], id=37e28b7d-11fc-4587-920f-9415da1d93a3, title=title #4, content=test content of #4}]
2021-06-03 20:21:01,662 INFO  [com.exa.dem.Main] (ForkJoinPool.commonPool-worker-7) The request is done in the jvm client.*
```

*源代码中包含了由 Jaxrs 客户端实现的另一个版本。如果您对此感兴趣，可以亲自探索一下 [JaxrsClient 示例](https://github.com/hantsy/quarkus-sandbox/blob/master/graphql-client/src/main/java/com/example/demo/JaxrsClient.java)。*

## *从我的 Github 获得完整的源代码。*