# 使用 Micronaut 数据 Jdbc 和 Kotlin 构建 Micronaut 应用程序

> 原文：<https://itnext.io/building-micronaut-applications-with-micronaut-data-jdbc-and-kotlin-81c1b6cf4b10?source=collection_archive---------0----------------------->

Micronaut Data 对 Jdbc 和 R2dbc 也有很大的支持。在本文中，我们将探索 Micronaut 数据 Jdbc 并用 Kotlin 语言编写示例，最后我们将使用 Kotest 测试组件。

![](img/e9b4cbab42514bc38fab6c8d5d507ef3.png)

[董](https://unsplash.com/@dongmingwei?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)在 [Unsplash](https://unsplash.com/s/photos/china-snow?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍照

# 入门指南

打开浏览器，导航到 [Micronaut Launch](https://micronaut.io/launch) 为这篇文章生成一个新的项目框架。在此页面上选择以下项目。

*   Java 版本: **17**
*   语言:**科特林**
*   构建工具: **Gradle**
*   测试框架: **Kotest**
*   包含的特性: **data-jdbc** 、 **postgres** 等。

点击**生成项目**按钮，生成一个项目档案，下载后解压到磁盘，导入到你的 IDE，比如 IDEA。

创建实体类。

```
@MappedEntity(value = "posts", namingStrategy = NamingStrategies.UnderScoreSeparatedLowerCase::class)
data class Post(
    @field:Id @field:GeneratedValue(GeneratedValue.Type.UUID) var id: UUID? = null,
    var title: String,
    var content: String,
    var status: Status? = Status.DRAFT,
    @field:DateCreated var createdAt: LocalDateTime? = LocalDateTime.now()
)
```

这里我们声明一个 Kotlin `data class`来表示映射表中的数据。类似于 JPA 注释，您可以在映射到后端表主键的字段上设置`ID`和`GeneratedValue`。类似于 Spring 数据项目的审计特性，当实体被持久化时，标注有`@DateCreated`的`createdAt`字段将被自动填充。

状态是一个枚举类。

```
enum class Status {
    DRAFT, PENDING_MODERATED, PUBLISHED, REJECTED
}
```

> *注:* `*ID*` *和* `*GeneratedValue*` *来自* `*io.micronaut.data.annotation*` *包。*

为`Post`实体类创建一个`Repository`。

```
@JdbcRepository
interface PostRepository : PageableRepository<Post, UUID>
```

这里我们使用了一个`JdbcRepository`来表示这个存储库是一个**数据——JDBC**`Repository`。

创建一个 bean 来初始化一些示例数据。

```
@Singleton
@Requires(notEnv = ["mock"])
class DataInitializer(private val posts: PostRepository) { @EventListener
    fun onStartUp(e: ServerStartupEvent) {
        log.info("starting data initialization at ServerStartupEvent: $e") posts.deleteAll() val data = listOf(
            Post(title = "Building Restful APIs with Micronaut and Kotlin", content = "test"),
            Post(title = "Building Restful APIs with Micronaut and Kotlin: part 2", content = "test")
        )
        data.forEach { log.debug("saving: $it") }
        posts.saveAll(data).forEach { log.debug("saved post: $it") }
        log.info("data initialization is done...")
    } companion object DataInitializer {
        private val log = LoggerFactory.getLogger(DataInitializer::class.java)
    }}
```

现在创建一个控制器来公开 RESTful APIs。

```
@Controller("/posts")
class PostController(private val posts: PostRepository) { @Get(uri = "/", produces = [MediaType.APPLICATION_JSON])
    fun all(): HttpResponse<List<Post>> = ok(posts.findAll().toList()) @Get(uri = "/{id}", produces = [MediaType.APPLICATION_JSON])
    fun byId(@PathVariable id: UUID): HttpResponse<Any> {
        val post = posts.findById(id) ?: return notFound()
        return ok(post)
    } @io.micronaut.http.annotation.Post(consumes = [MediaType.APPLICATION_JSON])
    fun create(@Body body: Post): HttpResponse<Any> {
        val saved = posts.save(body)
        return created(URI.create("/posts/" + saved.id))
    }
}
```

现在让我们尝试启动应用程序，确保有一个正在运行的 Postgres 数据库，数据库设置应该与 *application.yaml* 中的配置匹配。

简单地说，你可以通过 docker 编写文件来准备数据库。在 docker 中运行下面的命令启动一个 Postgres，数据库细节在 *docker-compose.yaml* 中定义。

```
# docker compose up postgres
```

现在运行应用程序。

```
# gradlew run 
// or 
# gradlew build
# java build/xxx.jar
```

您可以使用`curl`命令来测试 */posts* 端点。

```
# curl [http://localhost:8080/posts](http://localhost:8080/posts)
```

# 按规格查询

如果你有一些 Spring Data JPA 的经验，你会对 JPA 规范印象深刻，但它只适用于 Spring Data JPA。在 Micronaut 数据中， **data-jdbc** 也支持 JPA 规范的查询。

将`jakarta.persistence:jakarta.persistence-api:3.0.0`添加到依赖项中。

改变`PostRepository`，使其延伸`JpaSpecificationExecutor`。

```
@JdbcRepository
interface PostRepository : PageableRepository<Post, UUID>, JpaSpecificationExecutor<Post>
```

创建一系列的`Specfication`，例如，通过标题查找，通过关键字查找，或者拒绝所有状态为`PENDING_MODERATED`的帖子，删除所有`REJECTED`的帖子。在 Micronaut 数据中，有一些`PredicateSpecification`的变体，比如`QuerySpecificaiton`、`UpdateSpecification`和`DeleteSpecification`。

```
object Specifications { fun titleLike(title: String): PredicateSpecification<Post> {
        return PredicateSpecification<Post> { root, criteriaBuilder ->
            criteriaBuilder.like(
                root.get("title"),
                "%$title%"
            )
        }
    } fun byKeyword(q: String): QuerySpecification<Post> {
        return QuerySpecification<Post> { root, query, criteriaBuilder ->
            criteriaBuilder.or(
                criteriaBuilder.like(root.get("title"), "%$q%"),
                criteriaBuilder.like(root.get("content"), "%$q%")
            )
        }
    } fun rejectAllPendingModerated(): UpdateSpecification<Post> {
        return UpdateSpecification<Post> {root, query, criteriaBuilder ->
            query.set(root.get("status"), Status.REJECTED)
            criteriaBuilder.equal(root.get<Status>("status"), Status.PENDING_MODERATED)
        }
    } fun removeAllRejected(): DeleteSpecification<Post> {
        return DeleteSpecification<Post> {root, query, criteriaBuilder ->
            criteriaBuilder.equal(root.get<Status>("status"), Status.REJECTED)
        }
    }}
```

让我们创建一些测试来验证这些规范。

```
@MicronautTest(environments = [Environment.TEST], startApplication = false)
open class PostRepositoryAnnotationSpec() : AnnotationSpec() {
    companion object {
        private val log: Logger = LoggerFactory.getLogger(PostControllerTest::class.java)
    } @Inject
    private lateinit var posts: PostRepository @Inject
    private lateinit var template: JdbcOperations @Inject
    private lateinit var tx: TransactionOperations<Any> @BeforeEach
    fun beforeEach() {
        val callback: TransactionCallback<Any, Int> = TransactionCallback { _: TransactionStatus<Any> ->
            val sql = "delete from posts";
            this.template.prepareStatement(sql) {
                it.executeUpdate()
            }
        } val cnt = tx.executeWrite(callback)
        println("deleted $cnt");
    } @Test
    fun `test save and find posts`() {
        val sql = "insert into posts(title, content, status) values (?, ?, ?)";
        val insertedCnt = template.prepareStatement(sql) {
            it.setString(1, "test title")
            it.setString(2, "test content")
            it.setString(3, "DRAFT")
            it.executeUpdate()
        } insertedCnt shouldBeEqualComparingTo 1
        val all = posts.findAll()
        all shouldHaveSize 1
        log.debug("all posts: $all")
        all.map { it.title }.forAny { it shouldContain "test" }
    } @Test
    fun `find by title`() {
        val sql = "insert into posts(title, content, status) values (?, ?, ?)";
        val insertedCnt = template.prepareStatement(sql) {
            it.setString(1, "test title")
            it.setString(2, "test content")
            it.setString(3, "DRAFT")
            it.executeUpdate()
        } insertedCnt shouldBeEqualComparingTo 1
        val all = posts.findAll(Specifications.titleLike("test"))
        log.debug("all posts size:{}", all.size)
        all shouldHaveSize 1 val all2 = posts.findAll(Specifications.titleLike("test2"))
        log.debug("all2 posts size:{}", all2.size)
        all2 shouldHaveSize 0
    } @Test
    fun `find by keyword`() {
        val sql = "insert into posts(title, content, status) values (?, ?, ?)";
        val insertedCnt = template.prepareStatement(sql) {
            it.setString(1, "test title")
            it.setString(2, "test content")
            it.setString(3, "DRAFT")
            it.addBatch()
            it.setString(1, "test2 title")
            it.setString(2, "test2 content")
            it.setString(3, "DRAFT")
            it.addBatch()
            it.executeBatch()
        } insertedCnt.any { it == 1 }
        val all = posts.findAll(Specifications.byKeyword("test"))
        log.debug("all posts size:{}", all.size)
        all shouldHaveSize 2 val all2 = posts.findAll(Specifications.byKeyword("test2"))
        log.debug("all2 posts size:{}", all2.size)
        all2 shouldHaveSize 1
    } @Test
    fun `update posts`() {
        val sql = "insert into posts(title, content, status) values (?, ?, ?)";
        val insertedCnt = template.prepareStatement(sql) {
            it.setString(1, "test title")
            it.setString(2, "test content")
            it.setString(3, "PENDING_MODERATED")
            it.addBatch()
            it.setString(1, "test2 title")
            it.setString(2, "test2 content")
            it.setString(3, "PENDING_MODERATED")
            it.addBatch()
            it.executeBatch()
        } insertedCnt.any { it == 1 }
        val updated = posts.updateAll(Specifications.rejectAllPendingModerated())
        log.debug("updated posts size:{}", updated)
        updated shouldBe 2 val all = posts.findAll()
        all shouldHaveSize 2
        all.map { it.status }.forAny { it shouldBe Status.REJECTED }
    } @Test
    fun `remove posts`() {
        val sql = "insert into posts(title, content, status) values (?, ?, ?)";
        val insertedCnt = template.prepareStatement(sql) {
            it.setString(1, "test title")
            it.setString(2, "test content")
            it.setString(3, "REJECTED")
            it.addBatch()
            it.setString(1, "test2 title")
            it.setString(2, "test2 content")
            it.setString(3, "DRAFT")
            it.addBatch()
            it.executeBatch()
        } insertedCnt.any { it == 1 }
        val deleted = posts.deleteAll(Specifications.removeAllRejected())
        log.debug("deleted posts size:{}", deleted)
        deleted shouldBe 1 val all = posts.findAll()
        all shouldHaveSize 1
        all.map { it.status }.forAny { it shouldBe Status.DRAFT }
    }
}
```

类似于 Spring Jdbc 和 Spring Data Jdbc，有一个基于模板的`JdbcOperations` bean 可用于编程数据库操作。在上述测试代码中，我们使用`JdbcOperations`来准备和清理每个测试的样本数据。

在这个应用程序中，我们使用 Kotest 作为测试框架。

Kotest 提供了许多测试代码风格，有些是受 NodeJS 生态系统或 ScalaTest 中现有的`describe/it`子句的启发。

`AnnotationSpec`类似于传统的 JUnit 编码风格，对于那些来自 JUnit 的人来说，迁移到 Kotest 测试框架是零学习曲线。

# Kotest

最简单的是`SpringSpec`，用一个*字符串*来描述功能。让我们用`StringSepc`重写上面的测试代码。

```
@MicronautTest(environments = [Environment.TEST], startApplication = false)
class PostRepositoryTest(
    private val posts: PostRepository,
    private val template: JdbcOperations,
    private val tx: TransactionOperations<Any>
) : StringSpec({ "test save and find posts" {
        val sql = "insert into posts(title, content, status) values (?, ?, ?)";
        val insertedCnt = template.prepareStatement(sql) {
            it.setString(1, "test title")
            it.setString(2, "test content")
            it.setString(3, "DRAFT")
            it.executeUpdate()
        } insertedCnt shouldBeEqualComparingTo 1
        val all = posts.findAll()
        all shouldHaveSize 1
        log.debug("all posts: $all")
        all.map { it.title }.forAny { it shouldContain "test" }
    } "find by title" {
        val sql = "insert into posts(title, content, status) values (?, ?, ?)";
        val insertedCnt = template.prepareStatement(sql) {
            it.setString(1, "test title")
            it.setString(2, "test content")
            it.setString(3, "DRAFT")
            it.executeUpdate()
        } insertedCnt shouldBeEqualComparingTo 1
        val all = posts.findAll(Specifications.titleLike("test"))
        log.debug("all posts size:{}", all.size)
        all shouldHaveSize 1 val all2 = posts.findAll(Specifications.titleLike("test2"))
        log.debug("all2 posts size:{}", all2.size)
        all2 shouldHaveSize 0
    } "find by keyword" {
        val sql = "insert into posts(title, content, status) values (?, ?, ?)";
        val insertedCnt = template.prepareStatement(sql) {
            it.setString(1, "test title")
            it.setString(2, "test content")
            it.setString(3, "DRAFT")
            it.addBatch()
            it.setString(1, "test2 title")
            it.setString(2, "test2 content")
            it.setString(3, "DRAFT")
            it.addBatch()
            it.executeBatch()
        } insertedCnt.any { it == 1 }
        val all = posts.findAll(Specifications.byKeyword("test"))
        log.debug("all posts size:{}", all.size)
        all shouldHaveSize 2 val all2 = posts.findAll(Specifications.byKeyword("test2"))
        log.debug("all2 posts size:{}", all2.size)
        all2 shouldHaveSize 1
    } "update posts" {
        val sql = "insert into posts(title, content, status) values (?, ?, ?)";
        val insertedCnt = template.prepareStatement(sql) {
            it.setString(1, "test title")
            it.setString(2, "test content")
            it.setString(3, "PENDING_MODERATED")
            it.addBatch()
            it.setString(1, "test2 title")
            it.setString(2, "test2 content")
            it.setString(3, "PENDING_MODERATED")
            it.addBatch()
            it.executeBatch()
        } insertedCnt.any { it == 1 }
        val updated = posts.updateAll(Specifications.rejectAllPendingModerated())
        log.debug("updated posts size:{}", updated)
        updated shouldBe 2 val all = posts.findAll()
        all shouldHaveSize 2
        all.map { it.status }.forAny { it shouldBe Status.REJECTED }
    } "remove posts" {
        val sql = "insert into posts(title, content, status) values (?, ?, ?)";
        val insertedCnt = template.prepareStatement(sql) {
            it.setString(1, "test title")
            it.setString(2, "test content")
            it.setString(3, "REJECTED")
            it.addBatch()
            it.setString(1, "test2 title")
            it.setString(2, "test2 content")
            it.setString(3, "DRAFT")
            it.addBatch()
            it.executeBatch()
        } insertedCnt.any { it == 1 }
        val deleted = posts.deleteAll(Specifications.removeAllRejected())
        log.debug("deleted posts size:{}", deleted)
        deleted shouldBe 1 val all = posts.findAll()
        all shouldHaveSize 1
        all.map { it.status }.forAny { it shouldBe Status.DRAFT }
    }}) {
    companion object {
        private val log: Logger = LoggerFactory.getLogger(PostControllerTest::class.java)
    } override fun beforeEach(testCase: TestCase) {
        val callback: TransactionCallback<Any, Int> = TransactionCallback { _: TransactionStatus<Any> ->
            val sql = "delete from posts";
            this.template.prepareStatement(sql) {
                it.executeUpdate()
            }
        } val cnt = tx.executeWrite(callback)
        println("deleted $cnt");
    }
}
```

创建一个测试来测试`PostController`，这里我们使用`FunSpec`，它将测试封装在一个测试方法块中。

```
@MicronautTest(environments = ["mock"])
class PostControllerTest(
    private val postsBean: PostRepository,
    @Client("/") private var client: HttpClient
) : FunSpec({ test("test get posts endpoint") {
        val posts = getMock(postsBean)
        every { posts.findAll() }
            .returns(
                listOf(
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
        response.body()!![0].title shouldBe "test title" verify(exactly = 1) { posts.findAll() }
    }
}) {
    @MockBean(PostRepository::class)
    fun posts() = mockk<PostRepository>()
}
```

这里我们使用**模仿**来创建一个被模仿的`PostRepository`并且`MockBean`位于`SpringSpec`的体内。

以下是使用`SpringSpec`的集成示例。

```
@MicronautTest
class IntegrationTests(
    private val application: EmbeddedApplication<*>,
    @Client("/") private val client: HttpClient
) : StringSpec({ "test the server is running" {
        assert(application.isRunning)
    } "test GET /posts endpoint" {
        val response = client.toBlocking().exchange("/posts", Array<Post>::class.java) response.status shouldBe HttpStatus.OK
        response.body()!!.map { it.title }.forAny {
            it shouldContain "Micronaut"
        }
    }
})
```

从我的 Github 获取完整的[源代码](https://github.com/hantsy/micronaut-sandbox/tree/master/jdbc-kotlin)。