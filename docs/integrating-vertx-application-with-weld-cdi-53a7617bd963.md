# 将 Vertx 应用程序与 Weld/CDI 集成

> 原文：<https://itnext.io/integrating-vertx-application-with-weld-cdi-53a7617bd963?source=collection_archive---------7----------------------->

在上一篇文章中，我们介绍了一种将 Vertx 应用程序与 Spring 框架集成的简单方法。在这篇文章中，我们将尝试将 Vertx 应用程序与 [CDI](https://www.cdi-spec.org) 集成，以取代 Spring。

![](img/b1d8b503bf19b0363061a113a4601ccd.png)

[Robynne Hu](https://unsplash.com/@robynnexy?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 在 [Unsplash](https://unsplash.com/s/photos/china-landscape?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上的照片

> CDI 是 Java EE 6 中引入的依赖和注入规范，目前 Java EE 被重命名为 Jakarta EE，由 Eclipse Foundation 维护。Weld 是 Jakarta CDI 规范的兼容提供商。

添加下列依赖项。

```
<dependency>
    <groupId>org.jboss.weld.se</groupId>
    <artifactId>weld-se-shaded</artifactId>
    <version>${weld.version}</version>
</dependency>
<dependency>
    <groupId>org.jboss</groupId>
    <artifactId>jandex</artifactId>
    <version>2.2.3.Final</version>
</dependency>
```

在上面的代码中:

*   `weld-se-shaded`为 Java SE 平台提供了 CDI 运行时环境。
*   `jandex`将索引应用程序中的类并加速 bean 搜索。

在*main/resources/META-INF*文件夹中添加一个空的`beans.xml`配置，以便在 Java SE 应用程序中启用 CDI 支持。

```
<?xml version="1.0" encoding="UTF-8"?>
<beans

        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
                      http://xmlns.jcp.org/xml/ns/javaee/beans_2_0.xsd"
        bean-discovery-mode="annotated">
</beans>
```

> *在 Java EE/Jakarta EE 世界中，从 Java EE 7 开始默认启用 CDI，beans.xml 配置文件是可选的。*

类似于 Spring 版本，增加一个`DemoApplication`来启动应用。

```
public class DemoApplication { private final static Logger LOGGER = Logger.getLogger(DemoApplication.class.getName()); public static void main(String[] args) {
        var weld = new Weld();
        var container = weld.initialize();
        var vertx = container.select(Vertx.class).get();
        var factory = container.select(VerticleFactory.class).get(); LOGGER.info("vertx clazz:" + vertx.getClass().getName());//Weld does not create proxy classes at runtime on @Singleton beans.
        LOGGER.info("factory clazz:" + factory.getClass().getName());
        // deploy MainVerticle via verticle identifier name
        //var deployOptions = new DeploymentOptions().setInstances(4);
        vertx.deployVerticle(factory.prefix() + ":" + MainVerticle.class.getName());
    }
}
```

这里我们使用`weld.initialize()`来初始化 CDI 容器。然后取回`Vertx` bean 和`VerticleFactory` bean，开始部署`MainVerticle`。

与`SpringAwareVerticleFactory`类似，创建 CDI aware `VerticleFactory`。

```
@ApplicationScoped
public class CdiAwareVerticleFactory implements VerticleFactory {

    @Inject
    private Instance<Object> instance; @Override
    public String prefix() {
        return "cdi";
    } @Override
    public void createVerticle(String verticleName, ClassLoader classLoader, Promise<Callable<Verticle>> promise) {
        String clazz = VerticleFactory.removePrefix(verticleName);
        promise.complete(() -> (Verticle) instance.select(Class.forName(clazz)).get());
    }
}
```

创建一个简单的`Resources`类并公开`Vertx`和`PgPool`bean。

```
@ApplicationScoped
public class Resources {
    private final static Logger LOGGER = Logger.getLogger(Resources.class.getName()); @Produces
    @Singleton
    public Vertx vertx(VerticleFactory verticleFactory) {
        Vertx vertx = Vertx.vertx();
        vertx.registerVerticleFactory(verticleFactory);
        return vertx;
    } @Produces
    public PgPool pgPool(Vertx vertx) {
        PgConnectOptions connectOptions = new PgConnectOptions()
            .setPort(5432)
            .setHost("localhost")
            .setDatabase("blogdb")
            .setUser("user")
            .setPassword("password"); // Pool Options
        PoolOptions poolOptions = new PoolOptions().setMaxSize(5); // Create the pool from the data object
        PgPool pool = PgPool.pool(vertx, connectOptions, poolOptions); return pool;
    } public void disposesPgPool(@Disposes PgPool pgPool) {
        LOGGER.info("disposing PgPool...");
        pgPool.close().onSuccess(v -> LOGGER.info("PgPool is closed successfully."));
    }
}
```

其他豆和 Spring 版本差不多，只是用 CDI `@ApplicaitonScoped`代替了 Spring `@Component`。

```
@ApplicationScoped
@RequiredArgsConstructor
public class MainVerticle extends AbstractVerticle {
    final PostsHandler postHandlers;

    //...
}@ApplicationScoped
@RequiredArgsConstructor
class PostsHandler {
    private final PostRepository posts;

    //...
}@ApplicationScoped
@RequiredArgsConstructor
public class PostRepository { private final PgPool client;

    //...
}@ApplicationScoped
@RequiredArgsConstructor
public class DataInitializer { private  final PgPool client;

    //...
}
```

请注意，在上面的`Resources`类中，我们给`Vertx`添加了一个`@Singleton`，与`ApplicationScoped`略有不同，Weld 并没有为它创建代理对象。

> *这里我们必须在 Vertx bean 上添加* `*@Singleton*` *，否则在应用程序启动阶段会出现向* `*VertxImpl*` *的转换错误，因为 CDI 不会为* `*VertxImpl*` *创建代理 bean。*

在`DemoApplication`中，我们添加了一些日志来打印`Vertx`和`VerticleFactory`bean 的类名。当启动应用程序时，在控制台中，您将看到如下的类名。

```
//..
INFO: vertx clazz:io.vertx.core.impl.VertxImpl
//...
INFO: factory clazz:com.example.demo.CdiAwareVerticleFactory$Proxy$_$$_WeldClientProxy
```

默认情况下，CDI 将为所有 beans 创建代理类，但是如果用`@Singletone`注释，它将直接使用实例。

为了测试应用程序，将下面的依赖项添加到*测试*范围中。

```
<dependency>
    <groupId>org.jboss.weld</groupId>
    <artifactId>weld-junit5</artifactId>
    <version>2.0.2.Final</version>
    <scope>test</scope>
    <exclusions>
        <exclusion>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-api</artifactId>
        </exclusion>
        <exclusion>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-engine</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

与 Spring 版本类似，在运行测试之前，在 JUnit `@BeforeAll`钩子中手动部署`Verticle`。

```
@EnableAutoWeld
@AddPackages(DemoApplication.class)
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
@ExtendWith(VertxExtension.class)
public class TestMainVerticle {
    private final static Logger LOGGER = Logger.getLogger(TestMainVerticle.class.getName()); @Inject
    Instance<Object> context; Vertx vertx; @BeforeAll
    public void setupAll(VertxTestContext testContext) {
        vertx = context.select(Vertx.class).get();
        var factory = context.select(VerticleFactory.class).get();
        vertx.deployVerticle(factory.prefix() + ":" + MainVerticle.class.getName())
            .onSuccess(id -> {
                LOGGER.info("deployed:" + id);
                testContext.completeNow();
            });
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
            .flatMap(req -> req.send().flatMap(HttpClientResponse::body))
            .onSuccess(
                buffer -> testContext.verify(
                    () -> {
                        LOGGER.log(Level.INFO, "response buffer: {0}", new Object[]{buffer.toString()});
                        assertThat(buffer.toJsonArray().size()).isGreaterThan(0);
                        testContext.completeNow();
                    }
                )
            )
            .onFailure(e -> LOGGER.log(Level.ALL, "error: {0}", e.getMessage()));
    }}
```

从我的 github 中获取[示例代码。](https://github.com/hantsy/vertx-sandbox/tree/master/post-service-cdi)