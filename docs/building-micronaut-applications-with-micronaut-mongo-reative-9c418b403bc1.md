# 用 Micronaut Mongo Reative 构建 Micronaut 应用程序

> 原文：<https://itnext.io/building-micronaut-applications-with-micronaut-mongo-reative-9c418b403bc1?source=collection_archive---------3----------------------->

在本帖中，我们将探讨 Micronaut Mongo 的反应特性。与数据 JPA 和 R2dbc 不同，Mongo Reactive 特性不是 Micronaut 数据项目的一部分。Micronaut Mongo Reactive 是官方 Mongo Java 驱动程序的一个轻量级包装器，提供从应用程序属性自动配置`MongoClient`。

![](img/1448c5af1e79724399f97ac0b484465c.png)

由 [Fabian Mardi](https://unsplash.com/@fabianmardi?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 在 [Unsplash](https://unsplash.com/s/photos/snow?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄的照片

# 入门指南

打开浏览器，导航到 [Micronaut Launch](https://micronaut.io/launch) 为这篇文章生成一个新的项目框架。在此页面上选择以下项目。

*   Java 版本: **17**
*   语言: **Java**
*   构建工具: **Gradle**
*   测试框架:**斯波克**
*   包含特性:**蒙哥反应**等。

点击**生成项目**按钮，生成一个项目档案，下载后解压到磁盘，导入到你的 IDE，比如 IDEA。

在前面的例子中，我们使用 JUnit 和 Kotest 作为测试框架，在这个例子中，我们改用 Spock 和 Groovy 来编写测试。

创建一个 Mongo 文档实体类。

```
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor(staticName = "of")
public class Customer {
    private ObjectId id;
    private String name;
    private int age;
    private Address address; public static Customer of(String name, int age, Address address) {
        return Customer.of(null, name, age, address);
    }
}
```

`Address`是一个嵌入在`Customer`文档中的文档。

```
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor(staticName = "of")
public class Address {
    private String street;
    private String city;
    private String zip;
}
```

创建一个`Repository`类来对`Customer`执行 CRUD 操作。

```
@Singleton
@RequiredArgsConstructor
@Slf4j
public class CustomerRepository {
    private final MongoClient mongoClient;
    private final DefaultMongoConfiguration mongoConfiguration; public Flux<Customer> findAll() {
        return Flux.from(customersCollection().find());
    } public Mono<Customer> findById(ObjectId id) {
        return Mono.from(customersCollection().find(Filters.eq(id)));
    } public Mono<ObjectId> insertOne(Customer data) {
        return Mono.from(customersCollection().insertOne(data, new InsertOneOptions().bypassDocumentValidation(false)))
                .mapNotNull(result -> result.getInsertedId().asObjectId().getValue());
    } public Mono<Map<Integer, BsonValue>> insertMany(List<Customer> data) {
        return Mono.from(customersCollection().insertMany(data, new InsertManyOptions().bypassDocumentValidation(false).ordered(true)))
                .map(InsertManyResult::getInsertedIds);
    } public Mono<Long> deleteById(ObjectId id) {
        return Mono.from(customersCollection().deleteOne(Filters.eq(id), new DeleteOptions()))
                .map(DeleteResult::getDeletedCount);
    } public void init() {
        var people = List.of(
                Customer.of("Charles Babbage", 45, Address.of("5 Devonshire Street", "London", "W11")),
                Customer.of("Alan Turing", 28, Address.of("Bletchley Hall", "Bletchley Park", "MK12")),
                Customer.of("Timothy Berners-Lee", 61, Address.of("Colehill", "Wimborne", null))
        );
        Mono.from(customersCollection().drop())
                .then()
                .thenMany(this.insertMany(people))
                .subscribe(
                        result -> result.forEach((key, value) -> log.debug("saved key: {}, value: {}", key, value)),
                        error -> log.debug("initialization failed: {}", error),
                        () -> log.debug("done")
                );
    } public Mono<Long> deleteAll() {
        return Mono.from(customersCollection().deleteMany(Filters.empty(), new DeleteOptions()))
                .map(DeleteResult::getDeletedCount);
    } private MongoCollection<Customer> customersCollection() {
        return mongoClient
                .getDatabase("userdb")
                .getCollection("customers", Customer.class);
    }}
```

当 *application.yml* 中设置了`mongo.uri`时，有一个**反应** `MongoClient` bean 可用。

在上面的代码中:

*   `customersCollection()`方法定义了一个映射到`Customer`类的 Mongo 集合。正如你所看到的，在`Customer`类中定义了一个`ObjectId` id 字段，当保存一个客户实例时，它会为其生成一个新的 ObjectId，并自动保存到 MongoDB 中的*客户*文档`_id`中。
*   `MongoClient`提供了 CRUD 操作的方法，但是它是基于*反应流*API 的。这里我们在这个项目中使用反应器 API，我们使用`Mono`和`Flux`将操作结果包装成反应器友好的 API。

现在让我们创建一个测试来测试`CustomerRepository`。

```
@MicronautTest(startApplication = false)
@Slf4j
class CustomerRepositorySpec extends Specification { @Inject
    EmbeddedApplication<?> application @Inject
    CustomerRepository customerRepository; def setup() {
        CountDownLatch latch = new CountDownLatch(1)
        customerRepository.deleteAll()
                .doOnTerminate(_ -> latch.countDown())
                .subscribe(it -> log.debug "deleted customers: {}", it)
        latch.await(1000, TimeUnit.MILLISECONDS)
    } void 'application is not running'() {
        expect:
        !application.running
    } void 'test findAll'() {
        given:
        this.customerRepository.insertMany(List.of(Customer.of("Jack", 40, null)))
                .block(Duration.ofMillis(5000L)) when:
        def result = this.customerRepository.findAll() then:
        StepVerifier.create(result)
                .expectNextMatches(it -> it.name == "Jack")
                .expectComplete()
                .verify()
    }
}
```

为了测试持久层，我们不需要一个正在运行的服务器。所以将`startApplication = false`添加到`MicronautTest`注释中。

通常，Spock 测试被称为`Specfication`，你可以在你的测试中覆盖生命周期方法，比如`setup`、`setupSpec`等。每个测试都遵循 BDD 模式，也就是`given` / `when` / `then` 模式。

在上面的代码中，我们覆盖了`setup`方法并清除了数据库中的数据。然后创建一个测试来验证插入和查找操作，在`then`块中，我们使用`StepVerify`来断言反应流中的结果。

如果您想要启动一个 Testcontainers Docker 来服务所需的 Mongo 数据库，请尝试使用`Shared`和`AutoCleanup`注释定义一个 Mongo 容器实例，并覆盖`setupSpec`来启动 Mongo 服务，并确保它可用于本规范中的所有测试。

```
@Shared
@AutoCleanup
GenericContainer mongo = new GenericContainer("mongo")
    .withExposedPorts(27017)def setupSpec() {        
    mongo.start()
}
```

像前面的例子一样，我们可以创建一个 bean 来监听一个`ServerStartupEvent`来初始化一些用于测试的样本数据。

```
@Singleton
@Requires(notEnv = "mock")
@Slf4j
@RequiredArgsConstructor
public class DataInitializer {
    private final CustomerRepository customerRepository; @EventListener
    public void onStart(ServerStartupEvent event) {
        log.debug("starting data initialization...");
        this.customerRepository.init();
    }
}
```

尝试创建一个控制器来公开 RESTful APIs。

```
@Controller("/customers")
@RequiredArgsConstructor
@Slf4j
public class CustomerController {
    private final CustomerRepository customerRepository; @Get(uri = "/", produces = {MediaType.APPLICATION_JSON})
    public Flux<?> all() {
        return this.customerRepository.findAll();
    } @Get(uri = "/{id}", produces = {MediaType.APPLICATION_JSON})
    public Mono<MutableHttpResponse<Customer>> byId(@PathVariable ObjectId id) {
        return this.customerRepository.findById(id)
                .map(HttpResponse::ok)
                .switchIfEmpty(Mono.just(notFound()));
    } @Post(uri = "/", consumes = {MediaType.APPLICATION_JSON})
    public Mono<HttpResponse<?>> create(@Body Customer data) {
        return this.customerRepository.insertOne(data)
                .map(id -> created(URI.create("/customers/" + id.toHexString())));
    } @Delete(uri = "/{id}")
    public Mono<HttpResponse<?>> delete(@PathVariable ObjectId id) {
        return this.customerRepository.deleteById(id)
                .map(deleted -> {
                    if (deleted > 0) {
                        return noContent();
                    } else {
                        return notFound();
                    }
                });
    }
}
```

为了处理请求路径中的`ObjectId`，创建一个`TypeConverter`来将 id 从字符串类型转换为`ObjectId`。

```
@Singleton
public class StringToObjectIdConverter implements TypeConverter<String, ObjectId> { @Override
    public Optional<ObjectId> convert(String object, Class<ObjectId> targetType, ConversionContext context) {
        return Optional.of(new ObjectId(object));
    }
}
```

为了在 HTTP 响应中将`Customer`的 id ( `ObjectId`类型)序列化为字符串，创建一个`JsonSerializer`来定制序列化过程。应用时，id 字段被序列化为十六进制字符串，而不是 JSON 对象。

```
@Singleton
public class ObjectIdJsonSerializer extends JsonSerializer<ObjectId> {

    @Override
    public void serialize(ObjectId value, JsonGenerator gen, SerializerProvider serializers) throws IOException {
        gen.writeString(value.toHexString());
    }
}
```

为`CustomerController`创建一个测试。

```
@MicronautTest(environments = ["mock"])
class CustomerControllerSpec extends Specification { @Inject
    EmbeddedApplication<?> application @Inject
    @Client("/")
    ReactorHttpClient client @Inject
    CustomerRepository customerRepository def 'test it works'() {
        expect:
        application.running
    } void 'get all customers'() {
        given:
        1 * customerRepository.findAll() >> Flux.just(Customer.of(ObjectId.get(), "Jack", 40, null), Customer.of(ObjectId.get(), "Rose", 20, null)) when:
        Flux<HttpResponse<String>> resFlux = client.exchange(HttpRequest.GET("/customers"), String).log() then:
        //1 * customers.findAll() >> Flux.just(Customer.of(ObjectId.get(), "Jack", 40, null), Customer.of(ObjectId.get(), "Rose", 20, null))
        StepVerifier.create(resFlux)
        //.expectNextCount(1)
                .consumeNextWith(s -> {
                    assert s.getStatus() == HttpStatus.OK
                    assert s.body().contains('Jack')
                })
                .expectComplete()
                .verify()
    } void 'create a new customer'() {
        given:
        def objId = ObjectId.get()
        1 * customerRepository.insertOne(_) >> Mono.just(objId) when:
        def body = Customer.of(null, "Jack", 40, null)
        Flux<HttpResponse<String>> resFlux = client.exchange(HttpRequest.POST("/customers", body), String).log() then:
        StepVerifier.create(resFlux)
                .consumeNextWith(s -> {
                    assert s.getStatus() == HttpStatus.CREATED
                    assert s.header("Location") == '/customers/' + objId.toHexString()
                })
                .expectComplete()
                .verify()
    } void 'get customer by id '() {
        given:
        1 * customerRepository.findById(_) >> Mono.just(Customer.of(ObjectId.get(), "Jack", 40, null)) when:
        Flux<HttpResponse<String>> resFlux = client.exchange(HttpRequest.GET("/customers/" + ObjectId.get().toHexString()), String).log() then:
        StepVerifier.create(resFlux)
                .consumeNextWith(s -> {
                    assert s.getStatus() == HttpStatus.OK
                    assert s.body().contains('Jack')
                })
                .expectComplete()
                .verify()
    } void 'get customer by none-existing id '() {
        given:
        1 * customerRepository.findById(_) >> Mono.empty() when:
        Flux<HttpResponse<String>> resFlux = client.exchange(HttpRequest.GET("/customers/" + ObjectId.get().toHexString()), String).log() then:
        StepVerifier.create(resFlux)
                .consumeErrorWith(error -> {
                    assert error instanceof HttpClientResponseException
                    assert (error as HttpClientResponseException).status == HttpStatus.NOT_FOUND
                })
                .verify()
    } void 'delete customer by id '() {
        given:
        1 * customerRepository.deleteById(_) >> Mono.just(1L) when:
        Flux<HttpResponse<String>> resFlux = client.exchange(HttpRequest.DELETE("/customers/" + ObjectId.get().toHexString()), String).log() then:
        StepVerifier.create(resFlux)
                .consumeNextWith(s -> {
                    assert s.getStatus() == HttpStatus.NO_CONTENT
                })
                .expectComplete()
                .verify()
    } void 'delete customer by none-existing id '() {
        given:
        1 * customerRepository.deleteById(_) >> Mono.just(0L) when:
        Flux<HttpResponse<String>> resFlux = client.exchange(HttpRequest.DELETE("/customers/" + ObjectId.get().toHexString()), String).log() then:
        StepVerifier.create(resFlux)
                .consumeErrorWith(error -> {
                    assert error instanceof HttpClientResponseException
                    assert (error as HttpClientResponseException).status == HttpStatus.NOT_FOUND
                })
                .verify()
    } @MockBean(CustomerRepository)
    CustomerRepository mockedCustomerRepository() {// must use explicit type declaration
        Mock(CustomerRepository)
    }
}
```

在这个测试中，我们为`CustomerRepository`创建了一个模拟 bean，注意你必须显式声明类型。注意，在`given`块中，与 Mockito 略有不同，它在一个地方设置了假设和断言。

Mongo 的另一个伟大特性是 Gridfs 支持。对于那些家庭使用的云应用程序，它是 AWS S3 存储服务的简单替代方案。

接下来，我们将创建一个简单的上传和下载端点，将二进制数据存储到 Mongo Gridfs 存储中，并从 Gridfs 存储中检索它。

首先声明一个`GridFSBucket` bean。

```
@Factory
public class GridFSConfig { @Bean
    GridFSBucket gridFSBucket(MongoClient client) {
        return GridFSBuckets.create(client.getDatabase("photos"))
                .withChunkSizeBytes(4096)
                //.withReadConcern(ReadConcern.MAJORITY)
                .withWriteConcern(WriteConcern.MAJORITY);
    }
}
```

现在创建一个控制器来处理文件的上传和下载。

```
@Controller("/photos")
@RequiredArgsConstructor
@Slf4j
public class PhotoController { private final GridFSBucket bucket; @Post(uri = "/", consumes = {MediaType.MULTIPART_FORM_DATA})
    public Mono<HttpResponse<?>> upload(StreamingFileUpload file) {
        var filename = file.getFilename();
        var name = file.getName();
        var contentType = file.getContentType();
        var size = file.getSize();
        log.debug("uploading file...\n filename:{},\n name:{},\n contentType: {},\n size: {} ", filename, name, contentType, size);
        var options = new GridFSUploadOptions();
        contentType.ifPresent(c -> options.metadata(new Document("contentType", c)));
        return Mono.from(this.bucket.uploadFromPublisher(
                                filename,
                                Mono.from(file).mapNotNull(partData -> {
                                    try {
                                        return partData.getByteBuffer();
                                    } catch (IOException e) {
                                        e.printStackTrace();
                                    }
                                    return null;
                                }),
                                options
                        )
                )
                .map(ObjectId::toHexString)
                .map(id -> ok(Map.of("id", id)));
    } @Get(uri = "/{id}", produces = {MediaType.APPLICATION_OCTET_STREAM})
    public Mono<HttpResponse<?>> download(@PathVariable ObjectId id) {
        return Mono.from(this.bucket.downloadToPublisher(id))
                .map(HttpResponse::ok);
    } @Delete(uri = "/{id}")
    public Mono<HttpResponse<?>> delete(@PathVariable ObjectId id) {
        return Mono.from(this.bucket.delete(id))
                .map(v -> noContent());
    }
}
```

要上传文件，使用`bucket.uploadFromPublisher`将上传数据传输到 Gridfs bucket 中。要下载文件，调用`downloadToPublisher`将数据读入`ByteBuffer` 并发送 HTTP 响应。要从存储桶中删除它，只需调用 delete 方法。

## 从我的 Github 获取完整的[源代码](https://github.com/hantsy/micronaut-sandbox/tree/master/album-service)。