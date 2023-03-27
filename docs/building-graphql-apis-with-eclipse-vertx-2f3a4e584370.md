# 用 Eclipse Vertx 构建 GraphQL APIs

> 原文：<https://itnext.io/building-graphql-apis-with-eclipse-vertx-2f3a4e584370?source=collection_archive---------1----------------------->

我们在之前的 [Quarkus GraphQL 帖子](/building-graphql-apis-with-quarkus-dbbf23f897df)中讨论过 GraphQL。在本文中，我们将探索 Eclipse Vertx 中的 GraphQL 支持。

![](img/f12666d091748b31d5d349aa9c710e4b.png)

[杨烁](https://unsplash.com/@yangshuo?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)在 [Unsplash](https://unsplash.com/s/photos/china-mountain?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上的照片

> Quarkus 还包括一个替代的 GraphQL 扩展，它使用 Eclipse Vertx GraphQL 特性。

按照[用 Eclipse Vertx](/building-restful-apis-with-eclipse-vertx-4ce89d8eeb74) 构建 RESTful APIs 中的步骤，创建一个新的 Eclipse Vertx 项目，不要忘记将 *GraphQL* 添加到**依赖项**中。

或者直接在现有的 *pom.xml* 文件中添加以下依赖关系。

```
<dependency>
     <groupId>io.vertx</groupId>
     <artifactId>vertx-web-graphql</artifactId>
</dependency>
```

## 从我的 Github 中查看[完整的示例代码。](https://github.com/hantsy/vertx-sandbox/tree/master/graphql)

Vertx 提供了一个特定的`GraphQLHandler`来处理来自客户端的 GraphQL 请求。

用以下内容填充`MainVerticle`。

```
@Slf4j
public class MainVerticle extends AbstractVerticle { static {
        log.info("Customizing the built-in jackson ObjectMapper...");
        var objectMapper = DatabindCodec.mapper();
        objectMapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
        objectMapper.disable(SerializationFeature.WRITE_DATE_TIMESTAMPS_AS_NANOSECONDS);
        objectMapper.disable(DeserializationFeature.READ_DATE_TIMESTAMPS_AS_NANOSECONDS); JavaTimeModule module = new JavaTimeModule();
        objectMapper.registerModule(module);
    } @Override
    public void start(Promise<Void> startPromise) throws Exception {
        log.info("Starting HTTP server...");
        //setupLogging(); //Create a PgPool instance
        var pgPool = pgPool(); // instantiate repos
        var postRepository = new PostRepository(pgPool);
        var commentRepository = new CommentRepository(pgPool);
        var authorRepository = new AuthorRepository(pgPool); // Initializing the sample data
        var initializer = new DataInitializer(postRepository, commentRepository, authorRepository);
        initializer.run(); //assemble PostService
        var postService = new PostService(postRepository, commentRepository, authorRepository);
        var authorService = new AuthorService(authorRepository); // assemble DataLoaders
        var dataLoaders = new DataLoaders(authorService, postService); //assemble DataFetcher
        var dataFetchers = new DataFetchers(postService); // setup GraphQL
        GraphQL graphQL = setupGraphQLJava(dataFetchers); // Configure routes
        var router = setupRoutes(graphQL, dataLoaders); // enable GraphQL Websocket Protocol
        HttpServerOptions httpServerOptions = new HttpServerOptions()
            .addWebSocketSubProtocol("graphql-ws");
        // Create the HTTP server
        vertx.createHttpServer(httpServerOptions)
            // Handle every request using the router
            .requestHandler(router)
            // Start listening
            .listen(8080)
            // Print the port
            .onSuccess(server -> {
                startPromise.complete();
                log.info("HTTP server started on port " + server.actualPort());
            })
            .onFailure(event -> {
                startPromise.fail(event);
                log.info("Failed to start HTTP server:" + event.getMessage());
            })
        ;
    } //create routes
    private Router setupRoutes(GraphQL graphQL, DataLoaders dataLoaders) { // Create a Router
        Router router = Router.router(vertx); // register BodyHandler globally.
        router.route().handler(BodyHandler.create()); // register GraphQL Subscription websocket handler.
        ApolloWSOptions apolloWSOptions = new ApolloWSOptions();
        router.route("/graphql").handler(
            ApolloWSHandler.create(graphQL, apolloWSOptions)
                .connectionInitHandler(connectionInitEvent -> {
                    JsonObject payload = connectionInitEvent.message().content().getJsonObject("payload");
                    log.info("connection init event: {}", payload);
                    if (payload != null && payload.containsKey("rejectMessage")) {
                        connectionInitEvent.fail(payload.getString("rejectMessage"));
                        return;
                    }
                    connectionInitEvent.complete(payload);
                })
                //.connectionHandler(event -> log.info("connection event: {}", event))
                //.messageHandler(msg -> log.info("websocket message: {}", msg.content().toString()))
                //.endHandler(event -> log.info("end event: {}", event))
        ); GraphQLHandlerOptions options = new GraphQLHandlerOptions()
            // enable multipart for file upload.
            .setRequestMultipartEnabled(true)
            .setRequestBatchingEnabled(true);
        // register `/graphql` for GraphQL handler
        // alternatively, use `router.route()` to enable GET and POST http methods
        router.post("/graphql")
            .handler(
                GraphQLHandler.create(graphQL, options)
                    .dataLoaderRegistry(buildDataLoaderRegistry(dataLoaders))
                //.locale()
                //.queryContext()
            ); // register `/graphiql` endpoint for the GraphiQL UI
        GraphiQLHandlerOptions graphiqlOptions = new GraphiQLHandlerOptions()
            .setEnabled(true);
        router.get("/graphiql/*").handler(GraphiQLHandler.create(graphiqlOptions)); router.get("/hello").handler(rc -> rc.response().end("Hello from my route")); return router;
    } private Function<RoutingContext, DataLoaderRegistry> buildDataLoaderRegistry(DataLoaders dataLoaders) {
        DataLoaderRegistry registry = new DataLoaderRegistry();
        registry.register("commentsLoader", dataLoaders.commentsLoader());
        registry.register("authorsLoader", dataLoaders.authorsLoader());
        return rc -> registry;
    } @SneakyThrows
    private GraphQL setupGraphQLJava(DataFetchers dataFetchers) {
        TypeDefinitionRegistry typeDefinitionRegistry = buildTypeDefinitionRegistry();
        RuntimeWiring runtimeWiring = buildRuntimeWiring(dataFetchers);
        GraphQLSchema graphQLSchema = buildGraphQLSchema(typeDefinitionRegistry, runtimeWiring);
        return buildGraphQL(graphQLSchema);
    } private GraphQL buildGraphQL(GraphQLSchema graphQLSchema) {
        return GraphQL.newGraphQL(graphQLSchema)
            .defaultDataFetcherExceptionHandler(new CustomDataFetchingExceptionHandler())
            //.queryExecutionStrategy()
            //.mutationExecutionStrategy()
            //.subscriptionExecutionStrategy()
            //.instrumentation()
            .build();
    } private GraphQLSchema buildGraphQLSchema(TypeDefinitionRegistry typeDefinitionRegistry, RuntimeWiring runtimeWiring) {
        SchemaGenerator schemaGenerator = new SchemaGenerator();
        GraphQLSchema graphQLSchema = schemaGenerator.makeExecutableSchema(typeDefinitionRegistry, runtimeWiring);
        return graphQLSchema;
    } private TypeDefinitionRegistry buildTypeDefinitionRegistry() throws IOException, URISyntaxException {
        var schema = Files.readString(Paths.get(getClass().getResource("/schema/schema.graphql").toURI()));
        //String schema = vertx.fileSystem().readFileBlocking("/schema/schema.graphql").toString(); SchemaParser schemaParser = new SchemaParser();
        TypeDefinitionRegistry typeDefinitionRegistry = schemaParser.parse(schema);
        return typeDefinitionRegistry;
    } private RuntimeWiring buildRuntimeWiring(DataFetchers dataFetchers) {
        return newRuntimeWiring()
            // the following codes are moved to CodeRegistry, the central place to configure the execution codes.
            /*
            .wiringFactory(new WiringFactory() {
                @Override
                public DataFetcher<Object> getDefaultDataFetcher(FieldWiringEnvironment environment) {
                    return VertxPropertyDataFetcher.create(environment.getFieldDefinition().getName());
                }
            })
            .type("Query", builder -> builder
                .dataFetcher("postById", dataFetchers.getPostById())
                .dataFetcher("allPosts", dataFetchers.getAllPosts())
            )
            .type("Mutation", builder -> builder.dataFetcher("createPost", dataFetchers.createPost()))
            .type("Post", builder -> builder
                .dataFetcher("author", dataFetchers.authorOfPost())
                .dataFetcher("comments", dataFetchers.commentsOfPost())
            )
            */
            .codeRegistry(buildCodeRegistry(dataFetchers))
            .scalar(Scalars.localDateTimeType())
            .scalar(Scalars.uuidType())
            .scalar(UploadScalar.build())// handling `Upload` scalar
            .directive("uppercase", new UpperCaseDirectiveWiring())
            .build();
    } private GraphQLCodeRegistry buildCodeRegistry(DataFetchers dataFetchers) {
        return GraphQLCodeRegistry.newCodeRegistry()
            .dataFetchers("Query", Map.of(
                "postById", dataFetchers.getPostById(),
                "allPosts", dataFetchers.getAllPosts()
            ))
            .dataFetchers("Mutation", Map.of(
                "createPost", dataFetchers.createPost(),
                "upload", dataFetchers.upload(),
                "addComment", dataFetchers.addComment()
            ))
            .dataFetchers("Subscription", Map.of(
                "commentAdded", dataFetchers.commentAdded()
            ))
            .dataFetchers("Post", Map.of(
                "author", dataFetchers.authorOfPost(),
                "comments", dataFetchers.commentsOfPost()
            ))
            //.typeResolver()
            //.fieldVisibility()
            .defaultDataFetcher(environment -> VertxPropertyDataFetcher.create(environment.getFieldDefinition().getName()))
            .build();
    } private PgPool pgPool() {
        PgConnectOptions connectOptions = new PgConnectOptions()
            .setPort(5432)
            .setHost("localhost")
            .setDatabase("blogdb")
            .setUser("user")
            .setPassword("password"); // Pool Options
        PoolOptions poolOptions = new PoolOptions().setMaxSize(5); // Create the pool from the data object
        PgPool pool = PgPool.pool(vertx, connectOptions, poolOptions); return pool;
    }}
```

`start`方法类似于之前帖子中的方法，但是这里它启用了 *graphql-ws* WebSocket 子协议来激活 GraphQL *订阅*支持。

在 *setupRoutes* 方法中，添加了 route */graphql* 来使用`GraphQLHanlder`处理 HTTP 请求，并通过 */graphql* 端点启用 WebSocket 支持，还添加了 route*/graphql*来激活 graph QL 交互式 Web UI。

如您所见，下面的代码用于创建一个`GraphQLHandler`实例。

```
GraphQLHandler.create(graphQL, options)
                    .dataLoaderRegistry(buildDataLoaderRegistry(dataLoaders))
```

它需要一个`GraphQL`实例和可选的`options`来初始化一个`GraphQL`实例。

要构建一个`GraphQL`实例，需要一个`GraphQLSchema`，它依赖于以下两个基本对象:

*   一个`TypeDefinitionRegistry`从文件中解析 graphql 方案定义，参见上面的`buildTypeDefinitionRegistry`方法。
*   一个`RuntimeWiring`实例组装运行时处理程序来服务 GraphQL 请求，参见上面的`buildRuntimeWiring`方法。

在 GraphQLHandler 中，注册将在每个请求中实例化的全局数据加载器。它用于降低 N+1 查询中执行查询的频率，并提高应用程序性能。

让我们来看看*main/resources/schema/schema . graphql*文件夹下的 graph QL 模式文件。

```
directive @uppercase on FIELD_DEFINITIONscalar LocalDateTime
scalar UUID
scalar Uploadtype Post {
    id: ID!
    title: String! @uppercase
    content: String
    comments: [Comment]
    status: PostStatus
    createdAt: LocalDateTime
    authorId: String
    author: Author
}type Author {
    id: ID!
    name: String!
    email: String!
    createdAt: LocalDateTime
    posts: [Post]
}
type Comment {
    id: ID!
    content: String!
    createdAt: LocalDateTime
    postId: String!
}input CreatePostInput {
    title: String!
    content: String!
}input CommentInput {
    postId: String!
    content: String!
}type Query {
    allPosts: [Post!]!
    postById(postId: String!): Post
}type Mutation {
    createPost(createPostInput: CreatePostInput!): UUID!
    upload(file: Upload!): Boolean
    addComment(commentInput: CommentInput!): UUID!
}type Subscription{
    commentAdded: Comment!
}enum PostStatus {
    DRAFT
    PENDING_MODERATION
    PUBLISHED
}
```

在这个模式文件中，我们声明了 3 个顶级类型:*查询*、*突变*和*订阅*，它们提供了在我们的应用程序中定义的基本操作。`Post`和`Comment`是通用类型，表示返回的响应中使用的类型。`CreatePostInput`和`CommentInput`是输入参数，表示 graphql 请求的有效负载。*标量*关键字定义自定义标量类型。*指令*定义了应用于字段的自定义指令*@大写*。

在`MainVerticle.buildCodeRegistry`方法中，它根据模式定义中定义的类型的坐标来组装数据提取器。

例如，当执行一个*查询* : `postById`时，它将执行`buildCodeRegistry`方法中定义的`dataFetchers.postById`。

```
.dataFetchers("Query", Map.of(
                "postById", dataFetchers.getPostById(),
    ...
```

查看`buildRuntimeWiring`，还会显示`Scalar`、`Directive`等。

以下是自定义标量类型的示例。

```
// LocalDateTimeScalar
public class LocalDateTimeScalar implements Coercing<LocalDateTime, String> {
    @Override
    public String serialize(Object dataFetcherResult) throws CoercingSerializeException {
        if (dataFetcherResult instanceof LocalDateTime) {
            return ((LocalDateTime) dataFetcherResult).format(DateTimeFormatter.ISO_DATE_TIME);
        } else {
            throw new CoercingSerializeException("Not a valid DateTime");
        }
    } @Override
    public LocalDateTime parseValue(Object input) throws CoercingParseValueException {
        return LocalDateTime.parse(input.toString(), DateTimeFormatter.ISO_DATE_TIME);
    } @Override
    public LocalDateTime parseLiteral(Object input) throws CoercingParseLiteralException {
        if (input instanceof StringValue) {
            return LocalDateTime.parse(((StringValue) input).getValue(), DateTimeFormatter.ISO_DATE_TIME);
        } throw new CoercingParseLiteralException("Value is not a valid ISO date time");
    }
}
//Scalars
public class Scalars { public static GraphQLScalarType localDateTimeType() {
        return GraphQLScalarType.newScalar()
                .name("LocalDateTime")
                .description("LocalDateTime type")
                .coercing(new LocalDateTimeScalar())
                .build();
    }
}//register custom scalar type in the MainVertcle buildRuntimeWiring
newRuntimeWiring()
    ...
    .scalar(Scalars.localDateTimeType())
```

Vertx GraphQL 提供了一个`UploadScalar`用于上传文件。查看源代码并亲自探索`UUIDScalar`实现。

类似地，在`buildRuntimeWiring`中注册一个自定义*指令*，例如`@uppercase`指令。

```
//UpperCaseDirectiveWiring
public class UpperCaseDirectiveWiring implements SchemaDirectiveWiring {
    @Override
    public GraphQLFieldDefinition onField(SchemaDirectiveWiringEnvironment<GraphQLFieldDefinition> env) { var field = env.getElement();
        var parentType = env.getFieldsContainer(); var originalDataFetcher = env.getCodeRegistry().getDataFetcher(parentType, field);
        var dataFetcher = DataFetcherFactories.wrapDataFetcher(originalDataFetcher,
                (dataFetchingEnvironment, value) -> {
                    if (value instanceof String s) {
                        return s.toUpperCase();
                    }
                    return value;
                }
        ); env.getCodeRegistry().dataFetcher(parentType, field, dataFetcher);
        return field;
    }
}
//register custom scalar directive in the MainVertcle buildRuntimeWiring
newRuntimeWiring()
    ...
    .directive("uppercase", new UpperCaseDirectiveWiring())
```

让我们继续讨论负责在运行时解析类型值的数据获取器。

例如，在 GraphiQL UI 页面中，尝试发送一个像这样的预定义查询。

```
query {
    allPosts{
        id
        title
        content
        author{ name }
        comments{ content }
    }
}
```

也就是说，它将执行一个`allPosts` *查询*，并返回一个 Post 数组，每一项包括字段`id`、`title`和`content`以及一个带有精确`name`字段的`author`对象，一个`comments`数组每一项包括一个单独的`content`字段。

当 GraphQL 请求被发送时，`GraphQLHandler`将处理它。

*   验证 GraphQL 请求，并确保它遵循模式类型定义。
*   通过类型坐标、*查询*和*all post*定位数据获取器。
*   根据请求格式组合返回值。
*   当解析`author`字段时，它将尝试定位 *Post* 和 *author* 以找到现有的数据获取器(`dataFetchers.authorOfPost()`)来处理它。类似地，它通过`dataFetchers.commentsOfPost()`处理*注释*。
*   其他通用字段，使用默认数据提取器直接返回本帖对应字段的值。

`Mutation`处理与`Query`类似，但它是为执行某些突变而设计的。例如，使用变异`createPost`创建一个新的 post，它接受一个`CreatePostInput`输入参数，然后委托给`dataFecthers.createPost`来处理它。

文件上传由 [GraphQL 多部分请求规范](https://github.com/jaydenseric/graphql-multipart-request-spec)定义，不是标准 GraphQL 规范的一部分。

下面是处理文件上传的数据获取器。

```
public DataFetcher<Boolean> upload() {
    return (DataFetchingEnvironment dfe) -> { FileUpload upload = dfe.getArgument("file");
        log.info("name: {}", upload.name());
        log.info("file name: {}", upload.fileName());
        log.info("uploaded file name: {}", upload.uploadedFileName());
        log.info("content type: {}", upload.contentType());
        log.info("charset: {}", upload.charSet());
        log.info("size: {}", upload.size());
        //            String fileContent = Vertx.vertx().fileSystem().readFileBlocking(upload.uploadedFileName()).toString();
        //            log.info("file content: {}", fileContent);
        return true;
    };
}
```

Vertx 为上传的文件创建一个临时文件，很容易从本地文件系统读取文件。

对于`Subscription`，用于跟踪来自后端的更新，如股票交易消息、通知等。GraphQL Java 要求它必须返回 react vestreams`Publisher`类型。

以下是添加注释时发送通知的示例。

```
public VertxDataFetcher<UUID> addComment() {
    return VertxDataFetcher.create((DataFetchingEnvironment dfe) -> {
        var commentInputArg = dfe.getArgument("commentInput");
        var jacksonMapper = DatabindCodec.mapper();
        var input = jacksonMapper.convertValue(commentInputArg, CommentInput.class);
        return this.posts.addComment(input)
            .onSuccess(id -> this.posts.getCommentById(id.toString())
                       .onSuccess(c -> subject.onNext(c)));
    });
}private ReplaySubject<Comment> subject = ReplaySubject.create(1);public DataFetcher<Publisher<Comment>> commentAdded() {
    return (DataFetchingEnvironment dfe) -> {
        ApolloWSMessage message = dfe.getContext();
        log.info("msg: {}, connectionParams: {}", message.content(), message.connectionParams());
        ConnectableObservable<Comment> connectableObservable = subject.share().publish();
        connectableObservable.connect();
        log.info("connect to `commentAdded`");
        return connectableObservable.toFlowable(BackpressureStrategy.BUFFER);
    };
}
```

上面的例子使用 RxJava 3 的`ReplaySubject`作为处理器向连接的客户端发送消息。我们已经在`MainVerticle`中配置了使用 WebSocket 协议来处理订阅。在下一篇文章中，我们将创建一个 WebSocket 客户端来使用这个消息端点。

在这里，我们跳过其他类似于前 Quarkus GraphQL 帖子的代码，查看来自我的 Github 的[完整示例代码，自己探索一下。](https://github.com/hantsy/vertx-sandbox/tree/master/graphql)

与 Quarkus 不同，Eclipse Vertx 不提供特定的 GraphQL 客户端来简化 Java 中的 GraphQL 请求。

要为 GraphQL APIs 编写测试，您必须切换到使用通用 Vertx Http 客户端。而且你必须很好地了解 HTTP 规范上的 T2 graph QL。

下面是一个使用 Vertx HttpClient 测试 *allPosts* 查询和 *createPost* 变异的例子。

```
@ExtendWith(VertxExtension.class)
@Slf4j
public class TestMainVerticle { HttpClient client; @BeforeEach
    void setup(Vertx vertx, VertxTestContext testContext) {
        vertx.deployVerticle(new MainVerticle(), testContext.succeeding(id -> testContext.completeNow()));
        var options = new HttpClientOptions()
            .setDefaultHost("localhost")
            .setDefaultPort(8080);
        client = vertx.createHttpClient(options);
    } @Test
    void getAllPosts(Vertx vertx, VertxTestContext testContext) throws Throwable {
        var query = """
            query {
                allPosts{
                    id
                    title
                    content
                    author{ name }
                    comments{ content }
                }
            }""";
        client.request(HttpMethod.POST, "/graphql")
            .flatMap(req -> req.putHeader("Content-Type", "application/json")
                .putHeader("Accept", "application/json")
                .send(Json.encode(Map.of("query", query)))//have to use Json.encode to convert objects to json string.
                .flatMap(HttpClientResponse::body)
            )
            .onComplete(
                testContext.succeeding(
                    buffer -> testContext.verify(
                        () -> {
                            log.info("buf: {}", buffer.toString());
                            JsonArray array = buffer.toJsonObject()
                                .getJsonObject("data")
                                .getJsonArray("allPosts");
                            assertThat(array.size()).isGreaterThan(0); var titles = array.getList().stream().map(o -> ((Map<String, Object>) o).get("title")).toList();
                            assertThat(titles).allMatch(s -> ((String) s).startsWith("DGS POST"));
                            testContext.completeNow();
                        }
                    )
                )
            );
    } @Test
    void createPost(Vertx vertx, VertxTestContext testContext) throws Throwable {
        String TITLE = "My post created by Vertx HttpClient";
        //var creatPostQuery = "mutation newPost($input:CreatePostInput!){ createPost(createPostInput:$input)}";
        var creatPostQuery = """
            mutation newPost($input:CreatePostInput!){
                createPost(createPostInput:$input)
            }""";
        client.request(HttpMethod.POST, "/graphql")
            .flatMap(req -> {
                    String encodedJson = Json.encode(Map.of(
                        "query", creatPostQuery,
                        "variables", Map.of(
                            "input", Map.of(
                                "title", TITLE,
                                "content", "content of my post"
                            )
                        )
                    ));
                    log.info("sending encoded json: {}", encodedJson);
                    return req.putHeader("Content-Type", "application/json")
                        .putHeader("Accept", "application/json")
                        .send(encodedJson)//have to use Json.encode to convert objects to json string.
                        .flatMap(HttpClientResponse::body);
                }
            )
            .flatMap(buf -> {
                Object id = buf.toJsonObject().getJsonObject("data").getValue("createPost"); log.info("created post: {}", id);
                assertThat(id).isNotNull(); var postById = """
                    query post($id:String!) {
                        postById(postId:$id){id title content}
                    }"""; return client.request(HttpMethod.POST, "/graphql")
                    .flatMap(req -> req.putHeader("Content-Type", "application/json")
                        .putHeader("Accept", "application/json")
                        .send(Json.encode(Map.of(
                            "query", postById,
                            "variables", Map.of("id", id.toString())
                        )))//have to use Json.encode to convert objects to json string.
                        .flatMap(HttpClientResponse::body)
                    );
            })
            .onComplete(
                testContext.succeeding(
                    buffer -> testContext.verify(
                        () -> {
                            log.info("buf: {}", buffer.toString());
                            String title = buffer.toJsonObject()
                                .getJsonObject("data")
                                .getJsonObject("postById")
                                .getString("title");
                            assertThat(title).isEqualTo(TITLE.toUpperCase());
                            testContext.completeNow();
                        }
                    )
                )
            );
    }
}
```

从我的 Github 中获取[示例代码。](https://github.com/hantsy/vertx-sandbox/tree/master/graphql)