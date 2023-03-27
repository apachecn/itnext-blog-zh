# 将 Vertx 应用程序与 Spring 框架集成

> 原文：<https://itnext.io/integrating-vertx-application-with-spring-framework-fb8fca81a357?source=collection_archive---------4----------------------->

如前所示，我们在`MainVerticle`类中手工组装了所有东西。我们将使用 Spring IOC 来管理依赖关系。

![](img/d92eb3cc46094db4a7d3eb59a8ac2552.png)

[Hanson Lu](https://unsplash.com/@hansonluu?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 在 [Unsplash](https://unsplash.com/s/photos/china-landscape?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上的照片

例如，它看起来像下面这样。

```
//Create a PgPool instance
var pgPool = pgPool();//Creating PostRepository
var postRepository = PostRepository.create(pgPool);//Creating PostHandler
var postHandlers = PostsHandler.create(postRepository);// Initializing the sample data
var initializer = DataInitializer.create(pgPool);
initializer.run();// Configure routes
var router = routes(postHandlers);// Create the HTTP server
vertx.createHttpServer()...
```

在本帖中，我们将介绍 Spring 框架来管理上述项目的依赖关系。

将 Spring 上下文依赖添加到项目 *pom.xml* 文件中。

```
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
</dependency>
```

`spring-context`提供了基本的 IOC 功能，用于组装依赖项和提供依赖项注入。

创建一个`@Configuration`类来启动 Spring 应用程序上下文。

```
@Configuration
@ComponentScan
public class DemoApplication { public static void main(String[] args) {
        var context = new AnnotationConfigApplicationContext(DemoApplication.class);
        var vertx = context.getBean(Vertx.class);
        var factory  = context.getBean(VerticleFactory.class); // deploy MainVerticle via verticle identifier name
        vertx.deployVerticle(factory.prefix()+":"+MainVerticle.class.getName());
    } @Bean
    public Vertx vertx(VerticleFactory verticleFactory) {
        Vertx vertx = Vertx.vertx();
        vertx.registerVerticleFactory(verticleFactory);
        return vertx;
    } @Bean
    public PgPool pgPool(Vertx vertx) {
        PgConnectOptions connectOptions = new PgConnectOptions()
            .setPort(5432)
            .setHost("localhost")
            .setDatabase("blogdb")
            .setUser("user")
            .setPassword("password"); // Pool Options
        PoolOptions poolOptions = new PoolOptions().setMaxSize(5); // Create the pool from the data object
        PgPool pool = PgPool.pool(vertx, connectOptions, poolOptions); return pool;
    }
}
```

在 main 方法中，我们使用一个`AnnotationConfigApplicationContext`来扫描 Spring 组件并组装依赖项。然后从 Spring 应用上下文中取出`Vertx` bean 和`VerticleFactory` bean，调用`vertx.deployVerticle`。

> *`*VerticleFactory*`*是一个 Vertx 内置钩子，用于实例化一个 Verticle 实例。**

*在这个配置中，我们还声明了 beans:*

*   *`Vertx`接受`VerticleFactory` bean 并在`Vertx` bean 中注册一个`VerticleFactory`。*
*   *`PgPool`用于访问 Postgres 数据库。您可以将数据库配置移动到一个*属性*文件中，并通过 Spring `@PropertySource`注释加载它。*

*让我们来看看`VerticleFactory`bean——一个 Spring 上下文感知的 VerticleFactory 实现。*

```
*@Component
public class SpringAwareVerticleFactory implements VerticleFactory, ApplicationContextAware { private ApplicationContext applicationContext; @Override
    public String prefix() {
        return "spring";
    } @Override
    public void createVerticle(String verticleName, ClassLoader classLoader, Promise<Callable<Verticle>> promise) {
        String clazz = VerticleFactory.removePrefix(verticleName);
        promise.complete(() -> (Verticle) applicationContext.getBean(Class.forName(clazz)));
    } @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }
}*
```

*当按名称实例化一个`Verticle`时，它将调用`createVerticle`方法，该方法在 Spring 应用程序上下文中查找 beans。*

*再来看看其他的类，直接声明为 Spring `@Component`。关于完整的源代码，请查看来自我的 Github 的[vertx-sandbox/post-service-spring](https://github.com/hantsy/vertx-sandbox/tree/master/post-service-spring)。*

```
*@Component
@RequiredArgsConstructor
public class MainVerticle extends AbstractVerticle {
    final PostsHandler postHandlers;

    //...
}@Component
@RequiredArgsConstructor
class PostsHandler {
    private final PostRepository posts;

    //...
}@Component
@RequiredArgsConstructor
public class PostRepository { private final PgPool client;

    //...
}@Component
@RequiredArgsConstructor
public class DataInitializer { private  final PgPool client;

    //...
}*
```

*当`DemoApplicaiton`运行时，可以扫描所有的`@Component`。*

*如您所见，通过 Spring IOC 容器，我们删除了组装依赖项的所有手动步骤。*

*现在让我们尝试启动应用程序。*

*在前一种应用中，应用可以通过 maven exec 插件和 jar 文件运行。但是它使用一个内置的`Launcher`类来部署`Verticle`。我们已经配置了使用 Spring 来完成同样的工作，所以将 maven exec 插件和 maven shade 插件的配置改为如下。*

```
*<plugin>
    <artifactId>maven-shade-plugin</artifactId>
    <version>${maven-shade-plugin.version}</version>
    <executions>
        <execution>
            <phase>package</phase>
            <goals>
                <goal>shade</goal>
            </goals>
            <configuration>
                <transformers>
                    <transformer
                                 implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                        <manifestEntries>
                            <Main-Class>com.example.demo.DemoApplication</Main-Class>
                        </manifestEntries>
                    </transformer>
                    <transformer
                                 implementation="org.apache.maven.plugins.shade.resource.ServicesResourceTransformer"/>
                </transformers>
                <artifactSet>
                </artifactSet>
                <outputFile>${project.build.directory}/${project.artifactId}-${project.version}-fat.jar
                </outputFile>
            </configuration>
        </execution>
    </executions>
</plugin>
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>exec-maven-plugin</artifactId>
    <version>${exec-maven-plugin.version}</version>
    <configuration>
        <mainClass>com.example.demo.DemoApplication</mainClass>
    </configuration>
</plugin>*
```

*现在运行以下命令来启动应用程序。*

```
*mvn clean compile exec:java//or
mvn clean package
java -jar target\xxx-fat.jar*
```

*在`TestMainVerticle`中，让我们做一些修改来使用 Spring IOC 容器，例如，您可以从 Spring 应用程序上下文中获得一个`Vertx`实例。*

```
*@SpringJUnitConfig(classes = DemoApplication.class)
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
@ExtendWith(VertxExtension.class)
public class TestMainVerticle {
    private final static Logger LOGGER = Logger.getLogger(TestMainVerticle.class.getName()); @Autowired
    ApplicationContext context; Vertx vertx; @BeforeAll
    public void setupAll(VertxTestContext testContext) {
        vertx = context.getBean(Vertx.class);
        var factory = context.getBean(VerticleFactory.class);
        vertx.deployVerticle(factory.prefix() + ":" + MainVerticle.class.getName())
            .onSuccess(id -> {
                LOGGER.info("deployed:" + id);
                testContext.completeNow();
            })
            .onFailure(testContext::failNow);
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
            .onFailure(e -> {
                LOGGER.log(Level.ALL, "error: {0}", e.getMessage());
                testContext.failNow(e);
            });
    }
}*
```

*从我的 github 中获取[示例代码。](https://github.com/hantsy/vertx-sandbox/tree/master//post-service-spring)*