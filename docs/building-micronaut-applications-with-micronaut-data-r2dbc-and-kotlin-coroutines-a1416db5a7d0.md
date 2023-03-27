# 使用 Micronaut 数据 R2dbc 和 Kotlin 协程构建 Micronaut 应用程序

> 原文：<https://itnext.io/building-micronaut-applications-with-micronaut-data-r2dbc-and-kotlin-coroutines-a1416db5a7d0?source=collection_archive---------1----------------------->

在本文中，我们将继续探索 Micronaut 数据 R2dbc，并用数据 R2dbc 和 Kotlin 协同例程重写前面的数据 Jdbc/Kotlin 示例。

![](img/d38920140561b805e62148f808dfd013.png)

照片由 [Gigi](https://unsplash.com/@ling_gigi?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 在 [Unsplash](https://unsplash.com/s/photos/china-snow?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄

与 Jdbc 不同，R2dbc 是另一种 RDBMS 数据库连接规范，但它为用户提供了异步无阻塞 API。R2dbc API 完全兼容反应流规范。Kotlin 协同程序是一个官方的 Kotlin 扩展，提供了一个基于事件循环的异步编程模型。

# 入门指南

打开浏览器，导航到 [Micronaut Launch](https://micronaut.io/launch) 为这篇文章生成一个新的项目框架。在此页面上选择以下项目。

*   Java 版本: **17**
*   语言:**科特林**
*   构建工具:**格雷尔·科特林**
*   测试框架: **Kotest**
*   包含的特性: **data-r2dbc** ， **postgres** ，**kot Lin-extension-functions**等。

点击**生成项目**按钮，生成一个项目档案，下载后解压到磁盘，导入到你的 IDE，比如 IDEA。

打开 *pom.xml* 文件，将 Kotlin 协程添加到项目依赖项中。

```
//kotlin coroutines
implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core")
implementation("org.jetbrains.kotlinx:kotlinx-coroutines-reactor")
```

`kotlinx-coroutines-reactor`提供了反应器 API 和 Kotlin 协同程序 API 之间的交换。

创建映射到数据库中表的实体。

```
@MappedEntity(value = "posts", namingStrategy = NamingStrategies.UnderScoreSeparatedLowerCase::class)
data class Post(
    @AutoPopulated//generated value UUID does not work here.
    @field:Id var id: UUID? = null,
    var title: String,
    var content: String,
    var status: Status? = Status.DRAFT,
    @field:DateCreated var createdAt: LocalDateTime? = LocalDateTime.now()
)
```

Micronaut Data R2dbc 不包含一个`UUID` ID 生成器策略，这里我们使用`@AutoPopulated`在持久化到数据库之前生成一个随机 UUID。

为`Post`实体创建一个存储库接口。

```
@R2dbcRepository(dialect = Dialect.POSTGRES)
interface PostRepository : CoroutineCrudRepository<Post, UUID>, CoroutineJpaSpecificationExecutor<Post>
```

Micronaut Data 为 ReactiveStreams API 提供了几个知识库接口，对于 Reactor 用户来说，有`ReactorCrudRepository`。`CoroutineCrudRepository`是一个 Kotlin 协同程序兼容的存储库接口，它在函数中返回一个*暂停*结果。

> *`*@R2dbcRepository*`*这里需要一个* `*dialect*` *，否则应用启动时会失败。**

*类似地，`JpaSpecificationExecutor`有一些反应流的变体，`CoroutineJpaSpecificationExecutor`为 Kotlin 协程做好了准备。我们已经创建了一个`Specificaitons`来设置查询、更新和删除操作的几个标准，我们将在本帖中重用它们。*

*让我们转到控制器，创建一个名为`PostController`的新控制器类。*

```
*@Controller("/posts")
class PostController(private val posts: PostRepository) { @Get(uri = "/", produces = [MediaType.APPLICATION_JSON])
    fun all(): HttpResponse<Flow<Post>> = ok(posts.findAll()) @Get(uri = "/{id}", produces = [MediaType.APPLICATION_JSON])
    suspend fun byId(@PathVariable id: UUID): HttpResponse<Any> {
        val post = posts.findById(id) ?: return notFound()
        return ok(post)
    } @io.micronaut.http.annotation.Post(consumes = [MediaType.APPLICATION_JSON])
    suspend fun create(@Body body: Post): HttpResponse<Any> {
        val saved = posts.save(body)
        return created(URI.create("/posts/" + saved.id))
    }
}*
```

*它看起来非常类似于我们在上一篇文章中所做的 Jdbc 版本，但是这里我们返回一个特定于`Flow`类型的 Kotlin 协同程序或者使用一个`suspend`函数。区别在于所有这些方法都是在协程上下文中执行的。*

*现在让我们试着通过一个`DataInitializer` bean 添加一些样本数据，这个 bean 监听一个`ServerStartUpEvent`。*

```
*@Singleton
class DataInitializer(private val posts: PostRepository) { @EventListener//does not support `suspend`
    fun onStartUp(e: ServerStartupEvent) {
        log.info("starting data initialization at StartUpEvent: $e") runBlocking {
            val deleteAll = posts.deleteAll()
            log.info("deleted posts: $deleteAll") val data = listOf(
                Post(title = "Building Restful APIs with Micronaut and Kotlin Coroutine", content = "test"),
                Post(title = "Building Restful APIs with Micronaut and Kotlin Coroutine: part 2", content = "test")
            )
            data.forEach { log.debug("saving: $it") }
            posts.saveAll(data)
                .onEach { log.debug("saved post: $it") }
                .onCompletion { log.debug("completed.") }
                .flowOn(Dispatchers.IO)
                .launchIn(this);
        } log.info("data initialization is done...")
    } companion object DataInitializer {
        private val log = LoggerFactory.getLogger(DataInitializer::class.java)
    }}*
```

*`EventListener`不支持`suspend`功能，使用`runBlocking`阻塞当前线程，依次运行*暂停*功能。*

# *JPA 标准 API*

*Micronaut Data 为 Data Jdbc 和 Data R2dbc 都提供了 JPA criteria API 支持，还为 Reactive Streams API 添加了一些`JpaSpecificationExecutor`变体，如前几节所述，Kotlin 协程有一个`CoroutineJpaSpecificationExecutor`。*

*将`jakarta-persistence-api`添加到依赖关系中，以提供 JPA 标准 API。*

```
*implementation("jakarta.persistence:jakarta.persistence-api:3.0.0")*
```

*让我们重复使用我们在上一篇文章中创建的`Specifications`。*

*创建一个测试来验证`Specifications`中定义的标准。*

```
*@MicronautTest(environments = [Environment.TEST], startApplication = false)
class PostRepositoryTest(
    private val posts: PostRepository,
    private val template: R2dbcOperations
) : StringSpec({ "save and find posts" {
        val sql = "insert into posts(title, content, status) values ($1, $2, $3)";
        Mono
            .from(template.withTransaction { status: ReactiveTransactionStatus<Connection> ->
                Mono.from(
                    status.connection.createStatement(sql)
                        .bind(0, "test title")
                        .bind(1, "test content")
                        .bind(2, "DRAFT")
                        .execute()
                ).flatMap { Mono.from(it.rowsUpdated) }
            })
            .log()
            .`as` { StepVerifier.create(it) }
            .consumeNextWith { it shouldBeEqualComparingTo 1 }
            .verifyComplete() runBlocking {
            val all = posts.findAll().toList()
            all shouldHaveSize 1
            log.debug("all posts: $all")
            all.map { it.title }.forAny { it shouldContain "test" }
        } } "find by title" {
        val sql = "insert into posts(title, content, status) values ($1, $2, $3)";
        Mono
            .from(template.withTransaction { status: ReactiveTransactionStatus<Connection> ->
                Mono.from(
                    status.connection.createStatement(sql)
                        .bind(0, "test title")
                        .bind(1, "test content")
                        .bind(2, "DRAFT")
                        .execute()
                ).flatMap { Mono.from(it.rowsUpdated) }
            })
            .`as` { StepVerifier.create(it) }
            .consumeNextWith { it shouldBeEqualComparingTo 1 }
            .verifyComplete() runBlocking {
            val all = posts.findAll(Specifications.titleLike("test")).toList()
            log.debug("all posts size:{}", all.size)
            all shouldHaveSize 1 val all2 = posts.findAll(Specifications.titleLike("test2")).toList()
            log.debug("all2 posts size:{}", all2.size)
            all2 shouldHaveSize 0
        } } "find by keyword" {
        val sql = "insert into posts(title, content, status) values ($1, $2, $3)";
        Flux
            .from(template.withTransaction { status: ReactiveTransactionStatus<Connection> ->
                val statement = status.connection.createStatement(sql)
                statement
                    .bind(0, "test title")
                    .bind(1, "test content")
                    .bind(2, "DRAFT")
                    .add()
                statement.bind(0, "test2 title")
                    .bind(1, "test2 content")
                    .bind(2, "DRAFT")
                    .add() Flux.from(statement.execute()).flatMap { Flux.from(it.rowsUpdated) }
            })
            .`as` { StepVerifier.create(it) }
            .consumeNextWith { it shouldBeEqualComparingTo 1 }
            .consumeNextWith { it shouldBeEqualComparingTo 1 }
            .verifyComplete() runBlocking {
            val all = posts.findAll(Specifications.byKeyword("test")).toList()
            log.debug("all posts size:{}", all.size)
            all shouldHaveSize 2 val all2 = posts.findAll(Specifications.byKeyword("test2")).toList()
            log.debug("all2 posts size:{}", all2.size)
            all2 shouldHaveSize 1
        }
    } "update posts" {
        val sql = "insert into posts(title, content, status) values ($1, $2, $3)";
        Flux
            .from(template.withTransaction { status: ReactiveTransactionStatus<Connection> ->
                val statement = status.connection.createStatement(sql)
                statement
                    .bind(0, "test title")
                    .bind(1, "test content")
                    .bind(2, "PENDING_MODERATED")
                    .add() statement
                    .bind(0, "test2 title")
                    .bind(1, "test2 content")
                    .bind(2, "PENDING_MODERATED")
                    .add() Flux.from(statement.execute()).flatMap { Flux.from(it.rowsUpdated) }
            })
            .`as` { StepVerifier.create(it) }
            .consumeNextWith { it shouldBeEqualComparingTo 1 }
            .consumeNextWith { it shouldBeEqualComparingTo 1 }
            .verifyComplete() runBlocking {
            val updated = posts.updateAll(Specifications.rejectAllPendingModerated())
            log.debug("updated posts size:{}", updated)
            updated shouldBe 2 val all = posts.findAll().toList()
            all shouldHaveSize 2
            all.map { it.status }.forAny { it shouldBe Status.REJECTED }
        }
    } "remove posts" {
        val sql = "insert into posts(title, content, status) values ($1, $2, $3)";
        Flux
            .from(template.withTransaction { status: ReactiveTransactionStatus<Connection> ->
                val statement = status.connection.createStatement(sql)
                statement
                    .bind(0, "test title")
                    .bind(1, "test content")
                    .bind(2, "REJECTED")
                    .add()
                statement
                    .bind(0, "test2 title")
                    .bind(1, "test2 content")
                    .bind(2, "DRAFT")
                    .add() Flux.from(statement.execute()).flatMap { Flux.from(it.rowsUpdated) }
            })
            .`as` { StepVerifier.create(it) }
            .consumeNextWith { it shouldBeEqualComparingTo 1 }
            .consumeNextWith { it shouldBeEqualComparingTo 1 }
            .verifyComplete() runBlocking {
            val deleted = posts.deleteAll(Specifications.removeAllRejected())
            log.debug("deleted posts size:{}", deleted)
            deleted shouldBe 1 val all = posts.findAll().toList()
            all shouldHaveSize 1
            all.map { it.status }.forAny { it shouldBe Status.DRAFT }
        }
    }}) {
    companion object {
        private val log: Logger = LoggerFactory.getLogger(PostRepositoryTest::class.java)
    } override fun beforeEach(testCase: TestCase) {
        val sql = "delete from posts"; val latch = CountDownLatch(1)
        Mono
            .from(
                this.template.withConnection { conn: Connection ->
                    Mono.from(conn.beginTransaction())
                        .then(Mono.from(conn.createStatement(sql).execute())
                            .flatMap { Mono.from(it.rowsUpdated) }
                            .doOnNext { log.debug("deleted rows: $it ") }
                        )
                        .then(Mono.from(conn.commitTransaction()))
                        .doOnError { Mono.from(conn.rollbackTransaction()).then() }
                }
            )
            .log()
            .doOnTerminate { latch.countDown() }
            .subscribe(
                { data -> log.debug("deleted posts: $data ") },
                { error -> log.error("error of cleaning posts: $error") },
                { log.info("done") }
            ) latch.await(5000, TimeUnit.MILLISECONDS)
    }
}*
```

*我们将现有的 Jdbc 版本转换为 R2dbc，主要有一些不同。*

*   *与阻塞`TransactionOperations`类似，`R2dbcOperations`提供了`withConnection`和`withTransaction`来将数据操作包装在连接或事务边界内。*
*   *R2dbc `Connection`基于 ReactiveStreams API。*
*   *将参数绑定到 SQL 语句时，参数索引从 **0** 开始。*
*   *SQL 参数占位符依赖于数据库本身，例如，Postgres 使用`$1`、`$2`...*

# *测试控制器*

*在这篇文章中，我们仍然使用 Kotest 作为测试框架，正如你在上面的`PostRepositoryTest`中看到的，我们使用一个`runBlocking`将协程的执行封装在一个阻塞的上下文中。*

*`kotlinx-coroutines-test`提供了一些助手来简化 Kotlin 协程的测试，例如`runBlockingTest`等。将`kotlinx-coroutines-test`添加到测试依赖项中。*

```
*//gradle.properties
kotlinCoVersion=1.6.0-RC//build.gradle.kt
val kotlinCoVersion=project.properties.get("kotlinCoVersoin")//update versions of kotlin coroutines
implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:${kotlinCoVersion}")
implementation("org.jetbrains.kotlinx:kotlinx-coroutines-reactor:${kotlinCoVersion}")testImplementation("org.jetbrains.kotlinx:kotlinx-coroutines-test:${kotlinCoVersion}")*
```

> **还有一个* [*的问题*](https://stackoverflow.com/questions/70243380/test-kotlin-coroutines-with-runblockingtest-failed) *要在一次测试中使用* `*runBlockingTest*` *，确保你使用的是最新的 1.6.0-RC，而使用* `*runTest*` *代替。**

*类似于`runBlocking`，您可以使用`runTest`来包装测试功能。*

```
*@Test
fun `test GET all posts endpoint with runTest`() = runTest {
    val response = client.exchange("/posts", Array<Post>::class.java).awaitSingle()
    response.status shouldBe HttpStatus.OK
    response.body()!!.map { it.title }.forAny {
        it shouldContain "Micronaut"
    }
}*
```

> **`*runBlockingTest*`*在 Kotlin 协程的最新 1.6.0 版本中已被弃用。***

**我们还可以在测试控制器时模拟存储库，就像我们在上一篇文章中所做的那样。Mockk 为 Kotlin 协程提供了一些变种，比如`coEvery`、`coVerify`等。**

```
**@MicronautTest(environments = ["mock"])
class PostControllerTest(
    private val postRepository: PostRepository,
    @Client("/") private var client: HttpClient
) : FunSpec({ test("test get posts endpoint") {
        val posts = getMock(postRepository)
        coEvery { posts.findAll() }
            .returns(
                flowOf(
                    Post(
                        id = UUID.randomUUID(),
                        title = "test title",
                        content = "test content",
                        status = Status.DRAFT,
                        createdAt = LocalDateTime.now()
                    )
                )
            )
        val response = client.toBlocking().exchange("/posts", Array<Post>::class.java) response.status shouldBe HttpStatus.OK
        response.body()!![0].title shouldBe "test title" coVerify(exactly = 1) { posts.findAll() }
    }
}) {
    @MockBean(PostRepository::class)
    fun mockedPostRepository() = mockk<PostRepository>()
}**
```

**首先，为`PostRepository`创建一个模拟 bean，然后用`coEvery`进行存根化，并用`coVerify`子句验证模拟中的调用。**

## **从我的 Github 获取完整的[源代码](https://github.com/hantsy/micronaut-sandbox/tree/master/r2dbc-kotlin-co)。**