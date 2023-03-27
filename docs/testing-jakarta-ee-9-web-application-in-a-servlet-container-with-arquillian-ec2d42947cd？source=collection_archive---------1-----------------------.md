# 用 Arquillian 在 Servlet 容器中测试 Jakarta EE 9 Web 应用程序

> 原文：<https://itnext.io/testing-jakarta-ee-9-web-application-in-a-servlet-container-with-arquillian-ec2d42947cd?source=collection_archive---------1----------------------->

![](img/bfc78e68662297020d23994c842a06a0.png)

Photo by <a href=”[https://unsplash.com/@whiterainforest?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText](https://unsplash.com/@whiterainforest?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)">White.Rainforest ∙ 易雨白林.</a> on <a href=”[https://unsplash.com/s/photos/china-mountain?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText](https://unsplash.com/s/photos/china-mountain?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)">Unsplash</a>

在上一篇文章[使用 Servlet 容器构建 Jakarta EE 9 web 应用](https://github.com/hantsy/jakartaee9-servlet-starter-boilerplate/blob/master/docs/build.md)中，我们描述了如何使用核心 JakartaEE 组件启动 JakartaEE 9 Web 应用，包括 CDI、Jakarta Faces、Jakarta Servlet、Jakarta Pages、Jakarta REST 等，并在兼容 JakartaEE 9 的 Servlet 容器中运行它。在本帖中，我们将讨论如何使用 Arquillian 测试框架在 Servlet 容器中测试这些组件。

Arquillian 项目为 Apache Tomcat 和 Eclipse Jetty 提供官方支持，更多信息请访问[Arquillian Container Tomcat](https://github.com/arquillian/arquillian-container-tomcat)和 [Arquillian Container Jetty](https://github.com/arquillian/arquillian-container-jetty) 。目前，这两个项目都提供了一个支持最新 Apache Tomcat 10 和 Eclipse Jetty 11 的*嵌入式*容器适配器，但是没有可用的托管和远程适配器。

如果你是 Arquillian 新手，请先阅读官方的[入门指南](https://arquillian.org/guides/)，或者探索我之前关于[测试 Jakarta EE 8 应用](https://hantsy.github.io/jakartaee8-starter-boilerplate/)和 [Jakarta EE 9 应用](https://hantsy.github.io/jakartaee9-starter-boilerplate/)的 Arquillian 文章，了解 Arquillian 的基础知识。

# 配置阿奎利亚语

首先将 Arquillian Core 和 JUnit *BOM* 添加到项目 *pom.xml* 文件的 *dependencyManagement* 部分。

```
<dependencyManagement>
    // ... 
    <dependencies>
        <dependency>
            <groupId>org.jboss.arquillian</groupId>
            <artifactId>arquillian-bom</artifactId>
            <version>${arquillian-bom.version}</version>
            <scope>import</scope>
            <type>pom</type>
        </dependency>
        <dependency>
            <groupId>org.junit</groupId>
            <artifactId>junit-bom</artifactId>
            <version>${junit-jupiter.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

在项目*依赖关系*部分添加以下依赖关系。

```
<dependencies>
    //...
    <dependency>
        <groupId>org.jboss.arquillian.protocol</groupId>
        <artifactId>arquillian-protocol-servlet-jakarta</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.jboss.shrinkwrap.resolver</groupId>
        <artifactId>shrinkwrap-resolver-impl-maven</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

您还可以添加以下测试实用程序库来改进您的测试代码。

```
<dependencies>
    //....
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter-params</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.mockito</groupId>
        <artifactId>mockito-core</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.hamcrest</groupId>
        <artifactId>hamcrest</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.assertj</groupId>
        <artifactId>assertj-core</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

接下来，我们将配置 Arquillian Tomcat 嵌入式适配器，以针对嵌入式 Apache Tomcat 容器运行测试代码。

# 配置 Arquillian Tomcat 嵌入式适配器

创建一个新的 Maven 概要文件来集中 Arquillian tomcat 嵌入式适配器的所有配置。

```
<profile>
    <id>arq-tomcat-embedded</id>
    <properties>
        <skipTests>false</skipTests>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.jboss.arquillian.container</groupId>
            <artifactId>arquillian-tomcat-embedded-10</artifactId>
            <version>${arquillian-tomcat.version}</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.apache.tomcat.embed</groupId>
            <artifactId>tomcat-embed-core</artifactId>
            <version>${tomcat.version}</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.apache.tomcat.embed</groupId>
            <artifactId>tomcat-embed-jasper</artifactId>
            <version>${tomcat.version}</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.apache.tomcat.embed</groupId>
            <artifactId>tomcat-embed-websocket</artifactId>
            <version>${tomcat.version}</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
</profile>
```

现在我们将编写一些测试代码。

# 测试雅加达组件

对于简单的 POJOs，您可以编写一个简单的 JUnit 测试来验证功能。例如，`GreetingMessage`是一个简单的 POJO，用于组装可读的问候消息。我们可以编写一个简单的 JUnit 测试来检查它是否按预期工作。

```
public class GreetingMessageTest { @Test
    public void testGreetingMessage() {
        var message = GreetingMessage.of("Say Hello to JatartaEE");
        assertThat(message.getMessage()).isEqualTo("Say Hello to JatartaEE");
    }
}
```

`GreetingService` bean 本身只是实现了一个简单的功能，该功能用于使用`buildGreetingMessage`方法构建问候消息，该方法接受一个参数来设置问候的目标。就像前面的测试示例一样，创建一个简单的 JUnit 测试来验证它是否如预期的那样工作。

```
public class GreetingServiceUnitTest { GreetingService service; @BeforeEach
    public void setup() {
        service = new GreetingService();
    } @Test
    public void testGreeting() {
        var message = service.buildGreetingMessage("JakartaEE");
        assertThat(message.getMessage()).startsWith("Say Hello to JakartaEE");
    }
}
```

`Hello`豆依赖于`GreetingService`豆。为了在单元测试中测试`Hello`的功能，我们可以使用 Mockito 来隔离依赖- `GreetingService`。在下面的`HelloTest`中，我们在测试中创建了一个`GreetingService`的模拟对象。

```
public class HelloTest { @ParameterizedTest
    @MethodSource("provideQueryCriteria")
    public void testCreateMessage(String name, String result) {
        var service = mock(GreetingService.class);
        given(service.buildGreetingMessage(name)).willReturn(GreetingMessage.of("Say Hello to " + name));

        var hello = new Hello(service);

        hello.setName(name);
        hello.createMessage();
        assertThat(hello.getName()).isEqualTo(name);
        assertThat(hello.getMessage().getMessage()).isEqualTo(result);
        verify(service, times(1)).buildGreetingMessage(anyString());
        verifyNoMoreInteractions(service);
    }

    static Stream<Arguments> provideQueryCriteria() {
        return Stream.of(
                Arguments.of("Tomcat", "Say Hello to Tomcat"),
                Arguments.of("JakartaEE", "Say Hello to JakartaEE")
        );
    }
}
```

我们已经在单元测试中测试了简单的 POJOs，对于其他 Jakarta EE 组件，如 Servlet、Jakarta Pages 等，我们必须验证 Servlet 容器中的功能，我们将使用 Arquillian 为这个场景编写集成测试。

为了在不同的阶段运行单元测试和集成测试，我们可以如下配置`maven-surefire-plugin`和`maven-failsafe-plugin`，并确保集成测试在`integration-test`阶段运行。

```
<plugins>
    //...
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-surefire-plugin</artifactId>
        <version>${maven-surefire-plugin.version}</version>
        <configuration>
            <skipTests>${skipTests}</skipTests>
        </configuration>
        <executions>
            <execution>
                <id>default-test</id>
                <phase>test</phase>
                <goals>
                    <goal>test</goal>
                </goals>
                <configuration>
                    <excludes>
                        <exclude>**/it/**</exclude>
                    </excludes>
                </configuration>
            </execution>
        </executions>
    </plugin>
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-failsafe-plugin</artifactId>
        <version>${maven-failsafe-plugin.version}</version>
        <configuration>
            <skipITs>${skipTests}</skipITs>
        </configuration>
        <executions>
            <execution>
                <id>integration-test</id>
                <phase>integration-test</phase>
                <goals>
                    <goal>integration-test</goal>
                    <goal>verify</goal>
                </goals>
                <configuration>
                    <includes>
                        <include>**/it/**</include>
                    </includes>
                </configuration>
            </execution>
        </executions>
    </plugin>
</plugins>
```

首先，让我们看看用于测试简单 CDI bean `GreetingService`的`GreetingServiceTest`。

```
@ExtendWith(ArquillianExtension.class)
public class GreetingServiceTest {

    private final static Logger LOGGER = Logger.getLogger(GreetingServiceTest.class.getName());

    @Deployment
    public static WebArchive createDeployment() {
        var war = ShrinkWrap.create(WebArchive.class)
                .addClass(GreetingMessage.class)
                .addClass(GreetingService.class)
                .addAsWebInfResource(EmptyAsset.INSTANCE, "beans.xml")
                .addAsWebInfResource("test-web.xml", "web.xml")
                .addAsWebInfResource(new File("src/main/webapp/WEB-INF/jetty-env.xml"), "jetty-env.xml");

        Deployments.addExtraJars(war);

        LOGGER.log(Level.INFO, "war deployment: {0}", war.toString(true));
        return war;
    }

    @Inject
    GreetingService service;

    @Test
    @DisplayName("testing buildGreetingMessage")
    public void should_create_greeting() {
        LOGGER.log(Level.INFO, " Running test:: GreetingServiceTest#should_create_greeting ... ");
        var message = service.buildGreetingMessage("Jakarta EE");
        assertTrue(message.getMessage().startsWith("Say Hello to Jakarta EE at "),
                "message should start with \"Say Hello to Jakarta EE at \"");
    }
}
```

如您所见，Arquillian 集成测试用`@ExtendWith(ArquillianExtension.class)`进行了注释，这是标准的 JUnit 5 扩展。

在 Arquillian 测试中，您必须通过一个静态的`@Deployment`注释方法创建一个最小的部署档案。在`@Deployment`方法中，您可以在运行测试用例之前准备将要打包并部署到目标运行时的资源。

在测试类中，您可以像 CDI bean 一样注入可用的 bean，例如，我们在这里注入`GreetingService`，然后在测试方法中，使用`GreetingService` bean 来验证功能。

打开您的终端，执行以下命令运行`GreetingServiceTest`。

```
mvn clean verify -Parq-tomcat-embeded -Dit.test=GreetingServiceTest
```

当运行 Arquillian 测试时，它会将部署资源打包到一个可部署的档案中，并将其部署到目标容器中，然后在容器中运行测试，JUnit 客户端代理将通过与容器中的测试交互的代理收集运行结果。

让我们来测试一下`GreetingResource`。

```
@ExtendWith(ArquillianExtension.class)
public class GreetingResourceTest {

    private final static Logger LOGGER = Logger.getLogger(GreetingResourceTest.class.getName());

    @Deployment(testable = false)
    public static WebArchive createDeployment() {
        var war = ShrinkWrap.create(WebArchive.class)
                .addClass(GreetingMessage.class)
                .addClass(GreetingService.class)
                .addClasses(GreetingResource.class)
                .addClasses(RestActivator.class)
                // Enable CDI (Optional since Java EE 7.0)
                .addAsWebInfResource(EmptyAsset.INSTANCE, "beans.xml")
                .addAsWebInfResource("test-web.xml", "web.xml")
                .addAsWebInfResource(new File("src/main/webapp/WEB-INF/jetty-env.xml"), "jetty-env.xml");

        Deployments.addExtraJars(war);

        LOGGER.log(Level.INFO, "war deployment: {0}", war.toString(true));
        return war;
    }

    @ArquillianResource
    private URL base;

    private Client client;

    @BeforeEach
    public void setup() {
        LOGGER.info("call BeforeEach");
        this.client = ClientBuilder.newClient();
    }

    @AfterEach
    public void teardown() {
        LOGGER.info("call AfterEach");
        if (this.client != null) {
            this.client.close();
        }
    }

    @Test
    @DisplayName("Given a name:`JakartaEE` should return `Say Hello to JakartaEE`")
    public void should_create_greeting() throws MalformedURLException {
        LOGGER.log(Level.INFO, " client: {0}, baseURL: {1}", new Object[]{client, base});
        final var greetingTarget = this.client.target(new URL(this.base, "api/greeting/JakartaEE").toExternalForm());
        try (final Response greetingGetResponse = greetingTarget.request()
                .accept(MediaType.APPLICATION_JSON)
                .get()) {
            assertThat(greetingGetResponse.getStatus()).isEqualTo(200);
            assertThat(greetingGetResponse.readEntity(GreetingMessage.class).getMessage())
                    .startsWith("Say Hello to JakartaEE");
        }
    }
}
```

与`GreetingServiceTest`不同，为了测试“GreetingResource”的功能，我们使用 Jakarta REST 客户端 API 与客户端视图中的 HTTP APIs 进行交互。

在`@Deployment`注释中添加一个`testable=false`属性意味着所有的测试都将在客户端模式下运行。

或者，您也可以在测试方法上添加一个`@RunAsClient`来本地运行它。

在部署之后，`@ArquillianResource`将在容器中注入部署档案的基本 URL。

执行以下命令运行`GreetingServiceTest`。

```
mvn clean verify -Parq-tomcat-embeded -Dit.test=GreetingResourceTest
```

> *如果在部署方法上应用了* `*@Deployment(testable=true)*` *，那么所有的测试都以客户端模式运行，我们不能像前面的例子那样在测试类中注入 beans。*

类似地，我们可以创建*客户端模式*测试来验证简单的 Jakarta Servlet、Jakarta Faces、Jakarta Pages 等的功能。完整的代码可以在[这里](https://github.com/hantsy/jakartaee9-servlet-starter-boilerplate)找到。

> *要验证渲染后的 HTML 页面中的 HTML 元素和 Ajax 交互，请查看* [*Arquillian 扩展无人机*](https://github.com/arquillian/arquillian-extension-drone) *和* [*Arquillian 石墨烯*](https://github.com/arquillian/arquillian-graphene) *。*

# 配置 Jetty 嵌入式适配器

添加新的 Maven 配置文件来配置 Arquillian Jetty 嵌入式适配器。

```
<profile>
    <id>arq-jetty-embedded</id>
    <properties>
        <skipTests>false</skipTests>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.jboss.arquillian.container</groupId>
            <artifactId>arquillian-jetty-embedded-11</artifactId>
            <version>${arquillian-jetty.version}</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.eclipse.jetty</groupId>
            <artifactId>jetty-annotations</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.eclipse.jetty</groupId>
            <artifactId>jetty-plus</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.eclipse.jetty</groupId>
            <artifactId>jetty-deploy</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.eclipse.jetty</groupId>
            <artifactId>jetty-servlet</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.eclipse.jetty</groupId>
            <artifactId>jetty-webapp</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.eclipse.jetty</groupId>
            <artifactId>jetty-cdi</artifactId>
            <scope>test</scope>
            <!-- to remove when 11.0.10 released -->
            <!-- see [https://github.com/eclipse/jetty.project/pull/7991](https://github.com/eclipse/jetty.project/pull/7991) -->
            <version>${jetty.version}</version>
        </dependency>
        <dependency>
            <groupId>org.eclipse.jetty.websocket</groupId>
            <artifactId>websocket-jakarta-server</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.eclipse.jetty</groupId>
            <artifactId>apache-jsp</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-simple</artifactId>
            <version>1.7.36</version>
            <scope>test</scope>
        </dependency>
        <!-- see: [https://github.com/arquillian/arquillian-container-jetty/pull/108](https://github.com/arquillian/arquillian-container-jetty/pull/108) -->
        <!--<dependency>
            <groupId>org.jboss.arquillian.testenricher</groupId>
            <artifactId>arquillian-testenricher-cdi-jakarta</artifactId>
        </dependency>-->
    </dependencies>
</profile>
```

例如，针对`arq-jetty-embedded`概要文件运行之前的测试。

```
mvn clean verify -Parq-jetty-embeded -Dit.test=GreetingResourceTest
```

# 配置嵌入式阿奎利亚焊缝

Arquillian 项目提供了一个官方扩展，用于在嵌入式焊接容器中测试 CDI beans。

创建一个新的 Maven 概要文件来配置 Arquillian Weld 嵌入式适配器，并使用`maven-failsafe-plugin`过滤掉 Jakarta Servlet、Jakarta Faces 等的测试。

```
<profile>
    <id>arq-weld</id>
    <properties>
        <skipTests>false</skipTests>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.jboss.arquillian.container</groupId>
            <artifactId>arquillian-weld-embedded</artifactId>
            <version>${arquillian-weld-embedded.version}</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.jboss.weld</groupId>
            <artifactId>weld-core-impl</artifactId>
            <version>${weld.version}</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-failsafe-plugin</artifactId>
                <version>${maven-failsafe-plugin.version}</version>
                <configuration>
                    <systemPropertyVariables>
                        <arquillian.launch>arq-weld</arquillian.launch>
                    </systemPropertyVariables>
                    <excludes>
                        <exclude>**/it/GreetingResourceTest*</exclude>
                        <exclude>**/it/GreetingServletTest*</exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>
</profile>
```

执行以下命令运行`GreetingServiceTest`。

```
mvn clean verify -Parq-weld -Dit.test=GreetingServiceTest
```

> *Jakarta Servlet、Jakarta Pages 和 Jakarta Faces 的测试代码需要一个 Servlet 容器。*

从我的 Github 帐户获取[完整的源代码](https://github.com/hantsy/jakartaee9-servlet-starter-boilerplate)。