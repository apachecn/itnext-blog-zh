# 集成 Hibernate Reactive 和 Spring

> 原文：<https://itnext.io/integrating-hibernate-reactive-with-spring-5427440607fe?source=collection_archive---------0----------------------->

Hibernate 启动了一个子项目——Hibernate Reactive 支持反应流，但是在我写这篇文章的时候，Spring 仍然没有支持 Hibernate Reactive。好消息是集成工作并不复杂。在这篇文章中，我们将尝试集成最新的 Hibernate Reactive 和 Spring 框架。

![](img/f9351fb548f3e465146a30fb41d9fca2.png)

照片由 [Vivek Kumar](https://unsplash.com/@qriusv?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 在 [Unsplash](https://unsplash.com/s/photos/china-landscape?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄

在前一篇[将 Vertx 与 Spring 框架集成](/integrating-vertx-application-with-spring-framework-fb8fca81a357)和[后一篇](/building-a-vertx-application-with-smallrye-mutiny-bindings-spring-and-hibernate-reactive-5cf10b57983a)中，我们已经将 Hibernate Reactive 与 Spring IOC 容器集成，但是在那些帖子中，web 处理是由 Vertx Web 完成的。在这篇文章中，我们将使用现有的 Spring WebFlux。

打开浏览器，导航到[https://start . spring . io](https://start.spring.io)，生成一个 Spring 项目框架，包含以下依赖关系:

*   *WebFlux*
*   *龙目岛*

将下载的文件解压到光盘中，并将项目导入到 IDE 中。

打开项目 *pom.xml* 文件，添加以下依赖项。

```
<dependency>
    <groupId>io.vertx</groupId>
    <artifactId>vertx-pg-client</artifactId>
    <version>${vertx-pg-client.version}</version>
</dependency><dependency>
    <groupId>org.hibernate.reactive</groupId>
    <artifactId>hibernate-reactive-core</artifactId>
    <version>${hibernate-reactive.version}</version>
</dependency><dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-jpamodelgen</artifactId>
    <optional>true</optional>
</dependency>
```

在上面的代码中:

*   `vertx-pg-client`是休眠反应所需的 Postgres 反应驱动程序。
*   `hibernate-reactive-core`是 Hibernate Reactive 的核心依赖。
*   类似于一般的 Hibernate/JPA 支持，`hibernate-jpamodelgen`用于从`@Entity`类生成实体元数据类。

将 *persistence.xml* 添加到*src/main/resources/META-INF*文件夹中。

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

注意`provider`必须使用新 Hibernate Reactive 中提供的`ReactivePersistenceProvider`类。您必须将所有的实体类添加到这个 *persistence.xml* 文件中。

然后声明一个`Mutiny.SessionFactory` bean。`blogPU`是在 *persistence.xml* 文件中配置的持久性单元名称。

```
@Bean
public Mutiny.SessionFactory sessionFactory() {
    return Persistence.createEntityManagerFactory("blogPU")
        .unwrap(Mutiny.SessionFactory.class);
}
```

创建一个示例实体类。

```
@Data
@NoArgsConstructor
@AllArgsConstructor(staticName = "of")
@Builder
@Entity
@Table(name = "posts")
public class Post { @Id
    @GeneratedValue(generator = "uuid")
    @GenericGenerator(name = "uuid", strategy = "uuid2")
    UUID id;
    String title;
    String content; @Builder.Default
    @Column(name = "created_at")
    @CreationTimestamp
    LocalDateTime createdAt = LocalDateTime.now();
}
```

然后为它创建一个`Repository`类。

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

到目前为止，我们已经将 Hibernate Reactive 与 Spring IOC 容器集成在一起，接下来我们将使用`PostRepositoy`与后端数据库握手。让我们开始构建 web 处理部分。

Hibernate Reactive 中有两种不同类型的异步 API，一种基于 Java 8 `CompletionStage`，另一种基于 [Smallrye 社区项目](https://smallrye.io/smallrye-mutiny)。后者完全实现了 Reactive Streams 规范，我们在本文中使用 SmallRye 哗变。

但遗憾的是，Spring 并没有像 RxJava 2/3 等那样内置 Smallrye 兵变支持。

我们可以使用一些可能的解决方案来克服这一障碍。

*   将 SmallRye APIs 转换为 Reactor API，然后在`RouterFunction`或`Controller`类中直接使用 Reactor API。
*   类似于 Spring WebFlux 中现有的 RxJava 1/2/3、JDK 9+流支持，我们可以将 Smallry Munity 注册为另一个 ReactiveStreams 正式反应器的替代方案。

让我们一个一个的去探索。

首先，让我们尝试将社区 API 转换为反应器 API。假设我们将使用`RouterFunction`来处理 web 请求。

将以下依赖项添加到项目 *pom.xml* 文件中。

```
<dependency>
    <groupId>io.smallrye.reactive</groupId>
    <artifactId>mutiny-reactor</artifactId>
    <version>${mutiny-reactor.version}</version>
</dependency>
```

`mutiny-reactor`提供了一些可以用来在 SmallRye 哗变和 Reactor 之间转换 API 的工具。

下面是一个`PostsHandler`的例子，我们将所有的 web 处理程序集中在一个类中。在这个类中，我们将所有的兵变 API 转换为反应堆 API。

```
@Component
@RequiredArgsConstructor
class PostsHandler { private final PostRepository posts; public Mono<ServerResponse> all(ServerRequest req) {
        return ServerResponse.ok().body(this.posts.findAll().convert().with(toMono()), Post.class);
    } public Mono<ServerResponse> create(ServerRequest req) {
        return req.bodyToMono(CreatePostCommand.class)
            .flatMap(post -> this.posts.save(
                        Post.builder()
                            .title(post.getTitle())
                            .content(post.getContent())
                            .build()
                    )
                    .convert().with(toMono())
            )
            .flatMap(p -> ServerResponse.created(URI.create("/posts/" + p.getId())).build());
    } public Mono<ServerResponse> get(ServerRequest req) {
        var id = UUID.fromString(req.pathVariable("id"));
        return this.posts.findById(id).convert().with(toMono())
            .flatMap(post -> ServerResponse.ok().body(Mono.just(post), Post.class))
            .switchIfEmpty(ServerResponse.notFound().build());
    } public Mono<ServerResponse> update(ServerRequest req) { var id = UUID.fromString(req.pathVariable("id"));
        return Mono.zip((data) -> {
                    Post p = (Post) data[0];
                    UpdatePostCommand p2 = (UpdatePostCommand) data[1];
                    p.setTitle(p2.getTitle());
                    p.setContent(p2.getContent());
                    return p;
                },
                this.posts.findById(id).convert().with(toMono()),
                req.bodyToMono(UpdatePostCommand.class)
            )
            //.cast(Post.class)
            .flatMap(post -> this.posts.save(post).convert().with(toMono()))
            .flatMap(post -> ServerResponse.noContent().build());
    } public Mono<ServerResponse> delete(ServerRequest req) {
        var id = UUID.fromString(req.pathVariable("id"));
        return this.posts.deleteById(id).convert().with(toMono())
            .flatMap(d -> ServerResponse.noContent().build());
    }
}
```

然后在一个`RouterFunction` bean 中组装 web 处理程序。

```
@Bean
public RouterFunction<ServerResponse> routes(PostsHandler handler) {
    return route(GET("/posts"), handler::all)
        .andRoute(POST("/posts"), handler::create)
        .andRoute(GET("/posts/{id}"), handler::get)
        .andRoute(PUT("/posts/{id}"), handler::update)
        .andRoute(DELETE("/posts/{id}"), handler::delete);
}
```

在启动应用程序时，添加一个`DataInitializer` bean 来初始化一些示例数据。

```
@Component
@RequiredArgsConstructor
public class DataInitializer implements ApplicationRunner { private final static Logger LOGGER = Logger.getLogger(DataInitializer.class.getName()); private final Mutiny.SessionFactory sessionFactory; @Override
    public void run(ApplicationArguments args) throws Exception {
        LOGGER.info("Data initialization is starting..."); Post first = Post.of(null, "Hello Spring", "My first post of Spring", null);
        Post second = Post.of(null, "Hello Hibernate Reactive", "My second Hibernate Reactive", null); sessionFactory
            .withTransaction(
                (conn, tx) -> conn.createQuery("DELETE FROM Post").executeUpdate()
                    .flatMap(r -> conn.persistAll(first, second))
                    .chain(conn::flush)
                    .flatMap(r -> conn.createQuery("SELECT p from Post p", Post.class).getResultList())
            )
            .subscribe()
            .with(
                data -> LOGGER.log(Level.INFO, "saved data:{0}", data),
                throwable -> LOGGER.warning("Data initialization is failed:" + throwable.getMessage())
            );
    }
}
```

启动 Postgres 数据库。有一个[*docker-compose . yml*](https://github.com/hantsy/spring-puzzles/blob/master/hibernate-reactive/docker-compose.yml)文件可用于在 Docker 容器中启动 Postgres 实例。

然后通过 Spring Boot Maven 插件运行应用程序。

```
// start postgres database
docker compose up // run the application
mvn clean spring-root:run
```

当应用程序成功运行时，打开您的终端，尝试使用`curl`命令测试[http://localhost:8080/posts](http://localhost:8080/posts)端点。

```
# curl http://localhost:8080/posts
[{"id":"0998578e-0553-480b-bbb7-e96fd402455f","title":"Hello Spring","content":"My first post of Spring","createdAt":"2021-08-26T22:37:02.076284"},{"id":"e09ffa71-905f-4241-9449-0860977de666","title":"Hello Hibernate Reactive","content":"My second Hibernate Reactive","createdAt":"2021-08-26T22:37:02.116677"}]# curl http://localhost:8080/posts/0998578e-0553-480b-bbb7-e96fd402455f
{"id":"0998578e-0553-480b-bbb7-e96fd402455f","title":"Hello Spring","content":"My first post of Spring","createdAt":"2021-08-26T22:37:02.076284"}
```

那么我们来讨论第二种解决方案。

Spring 内部使用一个`ReactiveAdapterRegistry`来注册所有的 reactive streams 实现，比如 RxJava 2/3、JDK 9+ Flow 等。当序列化实现者的特定 API 时，它将查找注册表并将其转换为可由 Spring 框架处理的标准 RectiveStreams APIs。

我们将创建一个新的适配器来注册预期的哗变 API。

```
@Component
@RequiredArgsConstructor
@Slf4j
public class MutinyAdapter {
    private final ReactiveAdapterRegistry registry; @PostConstruct
    public void registerAdapters(){
        log.debug("registering MutinyAdapter");
        registry.registerReactiveType(
            ReactiveTypeDescriptor.singleOptionalValue(Uni.class, ()-> Uni.createFrom().nothing()),
            uni ->((Uni<?>)uni).convert().toPublisher(),
            publisher ->  Uni.createFrom().publisher(publisher)
        ); registry.registerReactiveType(
            ReactiveTypeDescriptor.multiValue(Multi.class, ()-> Multi.createFrom().empty()),
            multi -> (Multi<?>) multi,
            publisher-> Multi.createFrom().publisher(publisher));
    }
}
```

然后创建一个直接调用`PostRepository`的`@RestController` bean。如你所见，所有方法都直接返回一个`ResponseEntity`类型或一个`Uni<ResponseEntity>`类型，不需要显式的转换工作。

```
@RestController
@RequestMapping("/posts")
@RequiredArgsConstructor
class PostController { private final PostRepository posts; @GetMapping(value = "", produces = MediaType.APPLICATION_JSON_VALUE)
    public ResponseEntity<?> all() {
        return ok().body(this.posts.findAll());
    } @PostMapping(value = "", consumes = MediaType.APPLICATION_JSON_VALUE)
    public Uni<ResponseEntity<?>> create(@RequestBody CreatePostCommand data) {
        return this.posts.save(
                Post.builder()
                    .title(data.getTitle())
                    .content(data.getContent())
                    .build()
            )
            .map(p -> created(URI.create("/posts/" + p.getId())).build());
    } @GetMapping(value = "{id}", produces = MediaType.APPLICATION_JSON_VALUE)
    public Uni<ResponseEntity<Post>> get(@PathVariable UUID id) {
        return this.posts.findById(id)
            .map(post -> ok().body(post));
    } @PutMapping(value = "{id}", consumes = MediaType.APPLICATION_JSON_VALUE)
    public Uni<ResponseEntity<?>> update(@PathVariable UUID id, @RequestBody UpdatePostCommand data) { return Uni.combine().all()
            .unis(
                this.posts.findById(id),
                Uni.createFrom().item(data)
            )
            .combinedWith((p, d) -> {
                p.setTitle(d.getTitle());
                p.setContent(d.getContent());
                return p;
            })
            .flatMap(this.posts::save)
            .map(post -> noContent().build());
    } @DeleteMapping("{id}")
    public Uni<ResponseEntity<?>> delete(@PathVariable UUID id) {
        return this.posts.deleteById(id).map(d -> noContent().build());
    }
}
```

再次运行此应用程序，您将获得与前一个解决方案相同的结果。

## 从我的 GitHub 获得这篇文章的源代码，它们在两个独立的项目中可用， [hibernate-reactive](https://github.com/hantsy/spring-puzzles/tree/master/hibernate-reactive) 和[hibernate-reactive-哗变](https://github.com/hantsy/spring-puzzles/tree/master/hibernate-reactive-mutiny)。