# 用 SmallRye 兵变绑定、Spring 和 Hibernate 反应式构建 Vertx 应用程序

> 原文：<https://itnext.io/building-a-vertx-application-with-smallrye-mutiny-bindings-spring-and-hibernate-reactive-5cf10b57983a?source=collection_archive---------2----------------------->

在 [Spring integration post](https://github.com/hantsy/vertx-sandbox/blob/master/docs/spring.md) 中，我们使用 Spring 组装资源并启动应用程序。

![](img/27b970583e1ef6af449cc380f62fe99a.png)

Photo by [昊蓝 毛](https://unsplash.com/@maohaolan?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/china-landscape?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)

在这篇文章中，我们将重用 Spring 基础代码，但是:

*   使用 Hibernate Reactive 替换原始的 Postgres 客户端
*   并使用 SmallRye 哗变 Vertx 绑定来替换原来的 Vertx API，如 Future 等..

将 Hibernate 相关的依赖项添加到项目 *pom.xml* 文件中。

```
<dependency>
    <groupId>org.hibernate.reactive</groupId>
    <artifactId>hibernate-reactive-core</artifactId>
    <version>${hibernate-reactive.version}</version>
</dependency>
<!-- [https://mvnrepository.com/artifact/org.hibernate/hibernate-jpamodelgen](https://mvnrepository.com/artifact/org.hibernate/hibernate-jpamodelgen) -->
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-jpamodelgen</artifactId>
    <version>${hibernate.version}</version>
    <scope>provided</scope>
</dependency>
```

在上述代码中，

*   `hibernate-reactive-core`提供 Hibernate Reactive，使用 Vertx Postgres Client 等实现 reactive APIs，目前支持 Java 8 `CompletionStage`和 SmallRye 哗变。在本帖中，我们只使用 SmallRye 兵变 API。
*   在编译项目时，`hibernate-jpamodelgen`将生成 JPA `Entity`元数据类。

在*main/resources/META-INF*文件夹中添加一个 *persistence.xml* 配置。

```
<persistence 
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence http://xmlns.jcp.org/xml/ns/persistence/persistence_2_2.xsd"
             version="2.2"> <persistence-unit name="blogPU">
        <provider>org.hibernate.reactive.provider.ReactivePersistenceProvider</provider> <class>com.example.demo.Post</class> <properties> <!-- PostgreSQL -->
            <property name="javax.persistence.jdbc.url"
                      value="jdbc:postgresql://localhost:5432/blogdb"/> <!-- Credentials -->
            <property name="javax.persistence.jdbc.user"
                      value="user"/>
            <property name="javax.persistence.jdbc.password"
                      value="password"/> <!-- The Vert.x SQL Client connection pool size -->
            <property name="hibernate.connection.pool_size"
                      value="10"/> <!-- Automatic schema export -->
            <property name="javax.persistence.schema-generation.database.action"
                      value="drop-and-create"/> <!-- SQL statement logging -->
            <property name="hibernate.show_sql" value="true"/>
            <property name="hibernate.format_sql" value="true"/>
            <property name="hibernate.highlight_sql" value="true"/> </properties> </persistence-unit></persistence>
```

这是一个标准的 JPA 配置，但是这里我们使用特定的`org.hibernate.reactive.provider.ReactivePersistenceProvider`作为提供者来提供 ReactiveStreams 支持。

接下来，添加 SmallRye 相关的依赖项。

```
<dependency>
    <groupId>io.smallrye.reactive</groupId>
    <artifactId>smallrye-mutiny-vertx-core</artifactId>
    <version>${mutiny-vertx.version}</version>
</dependency>
<dependency>
    <groupId>io.smallrye.reactive</groupId>
    <artifactId>smallrye-mutiny-vertx-web</artifactId>
    <version>${mutiny-vertx.version}</version>
</dependency>
<dependency>
    <groupId>io.smallrye.reactive</groupId>
    <artifactId>smallrye-mutiny-vertx-pg-client</artifactId>
    <version>${mutiny-vertx.version}</version>
</dependency>
```

在`DemoApplication`中，曝光一个`Mutiny.SessionFactory` bean。

```
@Bean
public Mutiny.SessionFactory sessionFactory() {
    return Persistence.createEntityManagerFactory("blogPU")
        .unwrap(Mutiny.SessionFactory.class);
}
```

> *注意:您必须更新所有导入才能使用* `*io.vertx.mutiny*` *包中的物品，包括* `*Vertx*` *等。*

用以下零件替换`PostRepository`。

```
@Component
@RequiredArgsConstructor
public class PostRepository {
    private static final Logger LOGGER = Logger.getLogger(PostRepository.class.getName()); private final Mutiny.SessionFactory sessionFactory; public Uni<List<Post>> findAll() {
        CriteriaBuilder cb = this.sessionFactory.getCriteriaBuilder();
        // create query
        CriteriaQuery<Post> query = cb.createQuery(Post.class);
        // set the root class
        Root<Post> root = query.from(Post.class);
        return this.sessionFactory.withSession(session -> session.createQuery(query).getResultList());
    } public Uni<List<Post>> findByKeyword(String q, int offset, int limit) { CriteriaBuilder cb = this.sessionFactory.getCriteriaBuilder();
        // create query
        CriteriaQuery<Post> query = cb.createQuery(Post.class);
        // set the root class
        Root<Post> root = query.from(Post.class); // if keyword is provided
        if (q != null && !q.trim().isEmpty()) {
            query.where(
                cb.or(
                    cb.like(root.get(Post_.title), "%" + q + "%"),
                    cb.like(root.get(Post_.content), "%" + q + "%")
                )
            );
        }
        //perform query
        return this.sessionFactory.withSession(session -> session.createQuery(query)
            .setFirstResult(offset)
            .setMaxResults(limit)
            .getResultList());
    } public Uni<Post> findById(UUID id) {
        Objects.requireNonNull(id, "id can not be null");
        return this.sessionFactory.withSession(session -> session.find(Post.class, id))
            .onItem().ifNull().failWith(() -> new PostNotFoundException(id));
    } public Uni<Post> save(Post post) {
        if (post.getId() == null) {
            return this.sessionFactory.withSession(session ->
                session.persist(post)
                    .chain(session::flush)
                    .replaceWith(post)
            );
        } else {
            return this.sessionFactory.withSession(session -> session.merge(post).onItem().call(session::flush));
        }
    } public Uni<Post[]> saveAll(List<Post> data) {
        Post[] array = data.toArray(new Post[0]);
        return this.sessionFactory.withSession(session -> {
            session.persistAll(array);
            session.flush();
            return Uni.createFrom().item(array);
        });
    } public Uni<Integer> deleteById(UUID id) {
        CriteriaBuilder cb = this.sessionFactory.getCriteriaBuilder();
        // create delete
        CriteriaDelete<Post> delete = cb.createCriteriaDelete(Post.class);
        // set the root class
        Root<Post> root = delete.from(Post.class);
        // set where clause
        delete.where(cb.equal(root.get(Post_.id), id));
        // perform update
        return this.sessionFactory.withTransaction((session, tx) ->
            session.createQuery(delete).executeUpdate()
        );
    } public Uni<Integer> deleteAll() {
        CriteriaBuilder cb = this.sessionFactory.getCriteriaBuilder();
        // create delete
        CriteriaDelete<Post> delete = cb.createCriteriaDelete(Post.class);
        // set the root class
        Root<Post> root = delete.from(Post.class);
        // perform update
        return this.sessionFactory.withTransaction((session, tx) ->
            session.createQuery(delete).executeUpdate()
        );
    }}
```

它非常类似于标准的 JPA 代码，但是当使用`Mutiny.SessionFactory`执行查询时，它将返回 SmallRye 哗变特定的`Uni`类型。

更新`PostsHandler`的内容。

```
@Component
@RequiredArgsConstructor
class PostsHandler {
    private static final Logger LOGGER = Logger.getLogger(PostsHandler.class.getSimpleName()); private final PostRepository posts; public void all(RoutingContext rc) {
//        var params = rc.queryParams();
//        var q = params.get("q");
//        var limit = params.get("limit") == null ? 10 : Integer.parseInt(params.get("q"));
//        var offset = params.get("offset") == null ? 0 : Integer.parseInt(params.get("offset"));
//        LOGGER.log(Level.INFO, " find by keyword: q={0}, limit={1}, offset={2}", new Object[]{q, limit, offset});
        this.posts.findAll()
            .subscribe()
            .with(
                data -> {
                    LOGGER.log(Level.INFO, "posts data: {0}", data);
                    rc.response().endAndAwait(Json.encode(data));
                },
                rc::fail
            );
    } public void get(RoutingContext rc) {
        var params = rc.pathParams();
        var id = params.get("id");
        this.posts.findById(UUID.fromString(id))
            .subscribe()
            .with(
                post -> rc.response().endAndAwait(Json.encode(post)),
                throwable -> rc.fail(404, throwable)
            );
    } public void save(RoutingContext rc) {
        //rc.getBodyAsJson().mapTo(PostForm.class)
        var body = rc.getBodyAsJson();
        LOGGER.log(Level.INFO, "request body: {0}", body);
        var form = body.mapTo(CreatePostCommand.class);
        this.posts
            .save(Post.builder()
                .title(form.getTitle())
                .content(form.getContent())
                .build()
            )
            .subscribe()
            .with(
                savedId -> rc.response()
                    .putHeader("Location", "/posts/" + savedId)
                    .setStatusCode(201)
                    .endAndAwait(),
                throwable -> rc.fail(404, throwable)
            );
    } public void update(RoutingContext rc) {
        var params = rc.pathParams();
        var id = params.get("id");
        var body = rc.getBodyAsJson();
        LOGGER.log(Level.INFO, "\npath param id: {0}\nrequest body: {1}", new Object[]{id, body});
        var form = body.mapTo(CreatePostCommand.class);
        this.posts.findById(UUID.fromString(id))
            .flatMap(
                post -> {
                    post.setTitle(form.getTitle());
                    post.setContent(form.getContent()); return this.posts.save(post);
                }
            )
            .subscribe()
            .with(
                data -> rc.response().setStatusCode(204).endAndAwait(),
                throwable -> rc.fail(404, throwable)
            );
    } public void delete(RoutingContext rc) {
        var params = rc.pathParams();
        var id = params.get("id"); var uuid = UUID.fromString(id);
        this.posts.findById(uuid)
            .flatMap(
                post -> this.posts.deleteById(uuid)
            )
            .subscribe()
            .with(
                data -> rc.response().setStatusCode(204).endAndAwait(),
                throwable -> rc.fail(404, throwable)
            );
    }
}
```

我们来看看`MainVerticle`。

```
//...other imports.
import io.smallrye.mutiny.Uni;
import io.smallrye.mutiny.vertx.core.AbstractVerticle;
import io.vertx.core.json.jackson.DatabindCodec;
import io.vertx.mutiny.ext.web.Router;
import io.vertx.mutiny.ext.web.handler.BodyHandler;@Component
@RequiredArgsConstructor
public class MainVerticle extends AbstractVerticle {
    final PostsHandler postHandlers; private final static Logger LOGGER = Logger.getLogger(MainVerticle.class.getName()); static {
        LOGGER.info("Customizing the built-in jackson ObjectMapper...");
        var objectMapper = DatabindCodec.mapper();
        objectMapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
        objectMapper.disable(SerializationFeature.WRITE_DATE_TIMESTAMPS_AS_NANOSECONDS);
        objectMapper.disable(DeserializationFeature.READ_DATE_TIMESTAMPS_AS_NANOSECONDS); JavaTimeModule module = new JavaTimeModule();
        objectMapper.registerModule(module);
    } @Override
    public Uni<Void> asyncStart() {
        LOGGER.log(Level.INFO, "Starting HTTP server..."); // Configure routes
        var router = routes(postHandlers); // Create the HTTP server
        return vertx.createHttpServer()
            // Handle every request using the router
            .requestHandler(router)
            // Start listening
            .listen(8888)
            // Print the port
            .onItem().invoke(() -> LOGGER.info("Http server is listening on http://127.0.0.1:8888"))
            .onFailure().invoke(Throwable::printStackTrace)
            .replaceWithVoid();
    } //create routes
    private Router routes(PostsHandler handlers) { // Create a Router
        Router router = Router.router(vertx);
        // register BodyHandler globally.
        //router.route().handler(BodyHandler.create());
        router.get("/posts").produces("application/json")
            .handler(handlers::all);
        router.post("/posts").consumes("application/json")
            .handler(BodyHandler.create())
            .handler(handlers::save);
        router.get("/posts/:id").produces("application/json")
            .handler(handlers::get)
            .failureHandler(frc -> frc.response().setStatusCode(404).endAndAwait());
        router.put("/posts/:id").consumes("application/json")
            .handler(BodyHandler.create())
            .handler(handlers::update);
        router.delete("/posts/:id")
            .handler(handlers::delete); router.get("/hello").handler(rc -> rc.response().end("Hello from my route")); return router;
    }}
```

上面的代码和以前的 Spring 版本很像。

SmallRye 哗变版本`Router`提供了一些变体方法来接受一个简单的函数，而不是原来的`RequestHandler`，例如，有一个使用`respond`方法的例子。

```
//MainVerticle.java
router.get("/posts").produces("application/json")
    .respond(handlers::all);//PostHandler.java
public Uni<List<Post>> all(RoutingContext rc) {
    return this.posts.findAll();
}
```

为了测试应用程序，将以下依赖项添加到*测试*范围中。

```
<dependency>
    <groupId>io.smallrye.reactive</groupId>
    <artifactId>smallrye-mutiny-vertx-junit5</artifactId>
    <version>${mutiny-vertx.version}</version>
    <scope>test</scope>
</dependency>
```

它为测试方法中的`io.vertx.mutiny.core.Vertx`和`TestContext`提供参数注入。

```
import io.vertx.mutiny.core.Vertx;
import io.vertx.mutiny.core.http.HttpClientRequest;
import io.vertx.mutiny.core.http.HttpClientResponse;
//other imports@SpringJUnitConfig(classes = DemoApplication.class)
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
@ExtendWith(VertxExtension.class)
public class TestMainVerticle {
    private final static Logger LOGGER = Logger.getLogger(TestMainVerticle.class.getName()); @Autowired
    ApplicationContext context; Vertx vertx; @BeforeAll
    public void setupAll(VertxTestContext testContext) {
        vertx = context.getBean(Vertx.class);
        var factory = context.getBean(VerticleFactory.class);
        vertx.deployVerticle(factory.prefix() + ":" + MainVerticle.class.getName())
            .subscribe()
            .with(id -> {
                    LOGGER.info("deployed:" + id);
                    testContext.completeNow();
                },
                testContext::failNow
            );
    } @Test
    public void testVertx(VertxTestContext testContext) {
        assertThat(vertx).isNotNull();
        testContext.completeNow();
    } @Test
    void testGetAll(VertxTestContext testContext) {
        LOGGER.log(Level.INFO, "running test: {0}", "testGetAll");
        var options = new HttpClientOptions()
            .setDefaultPort(8888);
        var client = vertx.createHttpClient(options); client.request(HttpMethod.GET, "/posts")
            .flatMap(HttpClientRequest::send)
            .flatMap(HttpClientResponse::body)
            .subscribe()
            .with(buffer ->
                    testContext.verify(
                        () -> {
                            LOGGER.log(Level.INFO, "response buffer: {0}", new Object[]{buffer.toString()});
                            assertThat(buffer.toJsonArray().size()).isGreaterThan(0);
                            testContext.completeNow();
                        }
                    ),
                e -> {
                    LOGGER.log(Level.ALL, "error: {0}", e.getMessage());
                    testContext.failNow(e);
                }
            );
    } }
```

## [从我的 github 获取示例代码。](https://github.com/hantsy/vertx-sandbox/tree/master/mutiny-spring-hibernate)