# 使用 Arquillian 和 Glassfish v6 测试 Jakarta EE 9 应用程序

> 原文：<https://itnext.io/testing-jakarta-ee-9-applications-with-arquillian-and-glassfish-v6-9981e454e6eb?source=collection_archive---------2----------------------->

[Arquillian](http://www.arquillian.org) 增加了一个新模块[Arquillian Container glassfish 6](https://github.com/arquillian/arquillian-container-glassfish6)以配合 Jakarta EE 9 的变化和 Arquillian Core 1.7.0 中引入的特性。

![](img/bbe525b66addc9069a161aebd3e74370.png)

Arquillian 容器 Glassfish6 被指定在 Glassfish v6 上运行测试，Glassfish V6 是一个全功能的 Jakarta EE 9 兼容应用服务器，因此您可以使用这个新的 Arquillian Glassfish 容器测试所有 Jakarta EE 9 组件。

在本文中，我们将尝试使用托管和远程适配器在 Glassfish 容器上运行我们的测试。

*   使用托管适配器时，Arquillian 能够管理 Glassfish 服务器的生命周期，例如，在测试执行期间启动和停止容器。
*   当使用远程适配器时，Arquillian 将尝试对远程容器运行测试，并通过代理收集测试报告并发送回客户端(IDE、Maven 命令控制台等)。).

> *注意:原来的*[*Aruqillian Glassfish 嵌入式容器*](https://github.com/arquillian/arquillian-container-glassfish/tree/master/glassfish-embedded-3.1) *现在没有移植到最新的 Glassfish v6 上。*

# 先决条件

*   Java 8 ( [OpenJDK](https://openjdk.java.net/install/) 或 [AdoptOpenJDK](https://adoptopenjdk.net/installation.html)
*   最新[阿帕奇 Maven](http://maven.apache.org/download.cgi)
*   [JUnit 的基础知识 5](https://junit.org/junit5/)
*   了解阿尔奎利亚语的基础

> *注意:确保你使用的是 Java 8，Glassfish v6.0 不支持 Java 11。Glassfish v6.1 将专注于 Java 11 支持。*

# 测试 Restful Web 服务

目前，我们的示例项目中只包含了两个简单的 Arquillian 集成测试，我们在上一篇文章中已经介绍了 CDI 组件。

让我们继续讨论`GreetingResourceTest`。

```
@ExtendWith(ArquillianExtension.class)
public class GreetingResourceTest {
    private final static Logger LOGGER = Logger.getLogger(GreetingResourceTest.class.getName()); @Deployment(testable = false)
    public static WebArchive createDeployment() {
        return ShrinkWrap.create(WebArchive.class)
                .addClass(GreetingMessage.class)
                .addClass(GreetingService.class)
                .addClasses(GreetingResource.class, JaxrsActivator.class)
                // Enable CDI
                .addAsWebInfResource(EmptyAsset.INSTANCE, "beans.xml");
    } @ArquillianResource
    private URL base; private Client client; @BeforeEach
    public void setup() {
        this.client = ClientBuilder.newClient();
        //removed the Jackson json provider registry, due to OpenLiberty 21.0.0.1 switched to use Resteasy.
    } @AfterEach
    public void teardown() {
        if (this.client != null) {
            this.client.close();
        }
    } @Test
    public void should_create_greeting() throws MalformedURLException {
        LOGGER.log(Level.INFO, " Running test:: GreetingResourceTest#should_create_greeting ... ");
        final WebTarget greetingTarget = client.target(new URL(base, "api/greeting/JakartaEE").toExternalForm());
        try (final Response greetingGetResponse = greetingTarget.request()
                .accept(MediaType.APPLICATION_JSON)
                .get()) {
            assertThat(greetingGetResponse.getStatus()).isEqualTo(200);
            assertThat(greetingGetResponse.readEntity(GreetingMessage.class).getMessage()).startsWith("Say Hello to JakartaEE");
        }
    }
}
```

在上面的代码中。

*   使用一个`@ExtendWith(ArquillianExtension.class)`来扩展默认的 JUnit 生命周期，JUnit 5 中新引入了`@ExtendWith`来取代`@RunWith(...)`。
*   用`@Deployment(testable = false)`标注的静态方法用于描述部署单元。这里的`testable = false`表示这个测试是以客户端模式运行的，您不能像以前一样注入 beans。
*   通过一个`@ArquillianResource`注释，您可以在这个应用程序被部署到目标服务器之后获得它的 URL。
*   一个测试方法被标注了`@Test`注释。

在`should_create_greeting`方法中，它使用 Restful WS 客户端 API 与 Restful APIs 握手。您应该添加 Restful WS 客户端的实现。目前只有**泽**完成了变身。

```
<!-- Jersey -->
<dependency>
    <groupId>org.glassfish.jersey.media</groupId>
    <artifactId>jersey-media-sse</artifactId>
    <version>${jersey.version}</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.glassfish.jersey.media</groupId>
    <artifactId>jersey-media-json-binding</artifactId>
    <version>${jersey.version}</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.glassfish.jersey.inject</groupId>
    <artifactId>jersey-hk2</artifactId>
    <version>${jersey.version}</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.glassfish.jersey.core</groupId>
    <artifactId>jersey-client</artifactId>
    <version>${jersey.version}</version>
    <scope>test</scope>
</dependency>
```

在上面的代码中，添加了一些依赖项。

*   `jersey-client`是 Jakarta Restful WS Client 的实现。
*   `jersey-hk2`是**球衣**中的依赖注入引擎，将来会被 CDI 取代。
*   使用 **Jakarta JSON 绑定**将`jersey-media-json-binding`用于 JSON 序列化和反序列化。
*   `jersey-media-sse`负责处理媒体类型`text/event-stream`。

在配置 Arquillian Glassfish 容器支持之前，请确保您已经添加了 [Arquillian Jarkarta EE 9 和 JUnit 5 依赖项](https://github.com/hantsy/jakartaee9-starter-boilerplate/blob/master/docs/docs/arq-weld.md)。

# 配置 Glassfish 托管容器适配器

将`arquillian-glassfish-managed-6`依赖项添加到您的项目中。

```
<dependency>
    <groupId>org.jboss.arquillian.container</groupId>
    <artifactId>arquillian-glassfish-managed-6</artifactId>
    <version>${arquillian-glassfish6.version}</version>
    <scope>test</scope>
</dependency>
```

在 *arquillian.xml* 文件中创建一个容器配置。

```
<container qualifier="arq-glassfish-managed">
    <configuration>
        <property name="allowConnectingToRunningServer">false</property>
        <!--            <property name="glassFishHome">target/${glassfish.home}</property>-->
        <property name="adminHost">localhost</property>
        <property name="adminPort">4848</property>
        <property name="enableDerby">${enableDerby:true}</property>
        <property name="outputToConsole">true</property>
    </configuration>
</container>
```

您可以在 *arquillian.xml* 文件中定义一个`glassFishHome`属性来指定本地系统中现有 Glassfish 服务器的位置。

或者，为了获得一个干净的环境来每次运行您的测试，您可以配置一个**依赖 Maven 插件**来下载一份 Glassfish，并使用它来运行您的测试。这是在您的 CI 服务器中连续运行测试的一个很好的选择。

```
<plugin>
     <groupId>org.apache.maven.plugins</groupId>
     <artifactId>maven-dependency-plugin</artifactId>
     <version>${maven-dependency-plugin.version}</version>
     <executions>
         <execution>
             <id>unpack</id>
             <phase>pre-integration-test</phase>
             <goals>
                 <goal>unpack</goal>
             </goals>
             <configuration>
                 <artifactItems>
                     <artifactItem>
                         <groupId>org.glassfish.main.distributions</groupId>
                         <artifactId>glassfish</artifactId>
                         <version>${glassfish.version}</version>
                         <type>zip</type>
                         <overWrite>false</overWrite>
                         <outputDirectory>${project.build.directory}</outputDirectory>
                     </artifactItem>
                 </artifactItems>
             </configuration>
         </execution>
     </executions>
</plugin>
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-failsafe-plugin</artifactId>
    <version>${maven-failsafe-plugin.version}</version>
    <configuration>
        <environmentVariables>
            <GLASSFISH_HOME>${project.build.directory}/glassfish6</GLASSFISH_HOME>
        </environmentVariables>
    </configuration>
</plugin>
```

执行以下命令，对由 Arquillian 管理的 Glassfish 容器运行测试。

```
mvn clean verify -Parq-glassfish-managed
```

在托管模式下，Arquillian 负责启动和停止 Glassfish 容器。

# 配置 Glassfish 远程容器适配器

使用 Arquillian Glassfish 远程容器，您可以针对正在运行的 Glassfish 服务器运行测试。它运行在不同的服务器上。

改为添加以下`arquillian-glassfish-remote-6`依赖项。

```
<dependency>
    <groupId>org.jboss.arquillian.container</groupId>
    <artifactId>arquillian-glassfish-remote-6</artifactId>
    <version>${arquillian-glassfish6.version}</version>
    <scope>test</scope>
</dependency>
// the jersey client depdenceies are omitted.
```

并在该远程容器的 *arquillian.xml* 文件中添加一个*容器*段，如果与默认值不同，则配置`adminHost` `adminPort`、`adminUser`、`adminPassword`。

```
<container qualifier="glassfish-remote">
        <configuration>
            <!--Supported property names:
            [adminHttps,
            remoteServerHttpPort,
            libraries,
            type,
            remoteServerAddress,
            target,
            retries,
            remoteServerAdminPort,
            remoteServerAdminHttps,
            adminUser,
            authorisation,
            waitTimeMs,
            adminPort,
            properties,
            adminPassword,
            adminHost]-->
            <property name="adminHost">localhost</property>
            <property name="adminPort">4848</property>
            <property name="adminUser">admin</property>
            <!-- if https is enabled via `asadmin enable-secure-admin` on a remote server -->
            <!-- <property name="adminHttps">true</property>-->
            <!-- if admin password is changed via `asadmin change-admin-password` -->
            <!--<property name="adminPassword">adminadmin</property>-->
            <!-- default is empty -->
            <property name="adminPassword"></property>
        </configuration>
    </container>
```

在执行测试之前，确保目标 Glassfish 服务器正在运行。

执行以下命令，针对正在运行的 Glassfish 服务器运行测试。

```
mvn clean verfiy -Parq-glassfish-managed
```

# 保护管理

默认情况下，Glassfish 不会为`admin`用户设置密码，您可以通过在 Glassfish 服务器端执行命令`asadmin change-admin-password`来更改密码，不要忘记更改上面 *arquillian.xml* 文件中`adminPassword`属性的值。

要在远程 Glassfish 服务器上部署应用程序，最好在远程 Glassfish 服务器上启用安全连接。在 Glassfish 端执行`asadmin enable-secure-admin`来启用安全连接。相应地，在 *arquillian.xml* 文件中将`adminHttps`设置为 **true** 。

一旦设置了`adminHttps`，当运行测试时，您可能会得到一个与 SSL 认证相关的异常，原因是客户端 JVM 无法识别 Glassfish 服务器中使用的证书。尝试按照以下步骤来克服这一障碍。

首先，使用 JDK 附带的`keytool`工具从 *[Glassfish 安装目录]/Glassfish/domains/domain 1/config/keystore . jks*中导出证书。

```
keytool -export -alias default -file testwlp.crt -keystore [Glassfish install dir]/glassfish/domains/domain1/config/keystore.jks
```

并将其导入到您用来运行测试的 JVM 中。

```
keytool -import -trustcacerts -keystore ${JAVA_HOME}/jre/lib/security/cacerts -storepass changeit -alias testwlp -file testwlp.crt -noprompt
```

现在再次运行测试，证书异常应该会消失。

从我的 Github 中抓取一份[源代码，然后自己探索它们。](https://github.com/hantsy/jakartaee9-starter-boilerplate)