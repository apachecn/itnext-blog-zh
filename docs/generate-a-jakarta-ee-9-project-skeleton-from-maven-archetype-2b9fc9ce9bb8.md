# 从 Maven 原型生成一个 Jakarta EE 9 项目框架

> 原文：<https://itnext.io/generate-a-jakarta-ee-9-project-skeleton-from-maven-archetype-2b9fc9ce9bb8?source=collection_archive---------1----------------------->

我已经为开发人员创建了一个[jakartaee9-starter-boilerplate](https://github.com/hantsy/jakartaee9-starter-boilerplate)库来构建 jakartaee 9 项目。你可以阅读我为代码写的[文章](https://hantsy.github.io/jakartaee9-starter-boilerplate/)。

![](img/9dc6a8198c80965b7b48c9639ca6de79.png)

图片来自[https://unsplash.com/photos/RDW22bbsN0Y](https://unsplash.com/photos/RDW22bbsN0Y)

我在 Jakarta EE 社区的官方邮件列表中分享了这个项目，并了解到 [Eclipse EE4J starter](https://github.com/eclipse-ee4j/starter) 项目计划为 Jakarta EE 9 提供一个定制的 starter 模板。有人建议我创建一个 Maven 原型，作为分享我在 starter 项目中的工作的起点。为此，我为 Jakarta EE 9 仓库创建了另一个 Maven 原型[。在我写下这篇文章的时候，这个 maven 原型已经被批准上传到 Maven 中央存储库。](https://github.com/hantsy/maven-archetype-jakartaee9)

> *这个 maven 原型项目的工作正在移植到 Eclipse EE4J Starter 上，参见:*[*Eclipse-EE4J/Starter # 63*](https://github.com/eclipse-ee4j/starter/pull/63)*。*

# 先决条件

确保您已经安装了以下软件。

*   Java 8
*   阿帕奇 Maven 3.6
*   [GlassFish v6.0](https://glassfish.org)

> *Java 11 正在成为主流版本，但 GlassFish v6.0 仍然停留在 Java 8 上，Java 11 支持计划在即将发布的 v6.1 和 Jakarta EE 9.1 中实现。我会在 Jakarta EE 9.1 发布的时候更新到 Java 11。*

# 生成项目框架

打开您的终端，运行以下命令来生成 Jakarta EE 9 项目框架。

```
> mvn -B  archetype:generate \
-DarchetypeGroupId=io.github.hantsy \
-DarchetypeArtifactId=maven-archetype-jakartaee9  \
-DarchetypeVersion=1.0  \
-DgroupId=com.example \
-Dpackage=com.example.demo \
-DartifactId=myapp \
-Dversion=1.0-SNAPSHOT
```

目前我只将发布版本发布到 Maven 中央存储库。

为了体验 Github 包的特性，我还将下一个`1.1-SNAPSHOT`版本部署到了 Github 包注册中心。如果您想要使用最新的版本，那么在您的`~/.m2/settings.xml`中添加下面的存储库配置。

```
<repositories>
  <repository>
    <id>github-public</id>
    <url>https://public:&#48;28d0c8f723084e499eb87a6fe173b3af295d618@maven.pkg.github.com/hantsy/*</url>
  </repository>
</repositories>
```

项目生成后，切换到项目根。

```
cd mypp
```

将源代码导入到您的 ide 中，如 NetBeans、IntelliJ IDEA 等。您将看到下面的项目结构。

```
├── pom.xml
└── src
    ├── main
    │   ├── java
    │   │   └── com
    │   │       └── example
    │   │           └── demo
    │   │               ├── GreetingMessage.java
    │   │               ├── GreetingResource.java
    │   │               ├── GreetingService.java
    │   │               └── JaxrsActivator.java
    │   └── resources
    │       └── META-INF
    │           └── beans.xml
    └── test
        ├── java
        │   └── com
        │       └── example
        │           └── demo
        │               ├── GreetingResourceTest.java
        │               └── GreetingServiceTest.java
        └── resources
            └── arquillian.xml
```

1.  *pom.xml* 是项目 Maven 模型文件。
2.  在 *src/main/java* 中，有 4 个类，`GreetingMessage`是一个简单的 POJO 来呈现消息字符串，`GreetingResource`是一个 Jakarta REST 资源，`GreetingService`是一个简单的 CDI bean 来构建`GreetingMessage`。`JaxrsActivator`用于激活 Jakarta REST 支持。
3.  在 *src/main/resources* 文件夹中，有一个空的`beans.xml`，它是 CDI beans 的标准配置文件，但是从 Java EE 7 开始变成可选的。
4.  在 *src/test/java* 中，有两个测试类，`GreetingServiceTest`是测试`GreetingService`，`GreetingResourceTest`是测试`GreetingResource`。
5.  在 *src/test/resources* 文件夹中，有一个`arquillian.xml`是 Arquillian 测试框架的配置文件。

您可以使用下面的命令简单地运行应用程序。

```
> mvn clean package cargo:run
[INFO] Scanning for projects...
[INFO]
[INFO] -------------------------< com.example:myapp >--------------------------
[INFO] Building com.example 1.0-SNAPSHOT
[INFO] --------------------------------[ war ]---------------------------------
[INFO]
[INFO] --- maven-clean-plugin:2.5:clean (default-clean) @ myapp ---
[INFO] Deleting D:\tmp\myapp\target
[INFO]
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ myapp ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] Copying 1 resource
[INFO]
[INFO] --- maven-compiler-plugin:3.8.1:compile (default-compile) @ myapp ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 4 source files to D:\tmp\myapp\target\classes
[INFO]
[INFO] --- maven-resources-plugin:2.6:testResources (default-testResources) @ myapp ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] Copying 1 resource
[INFO]
[INFO] --- maven-compiler-plugin:3.8.1:testCompile (default-testCompile) @ myapp ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 2 source files to D:\tmp\myapp\target\test-classes
[INFO]
[INFO] --- maven-surefire-plugin:2.12.4:test (default-test) @ myapp ---
[INFO] Tests are skipped.
[INFO]
[INFO] --- maven-war-plugin:3.3.1:war (default-war) @ myapp ---
[INFO] Packaging webapp
[INFO] Assembling webapp [myapp] in [D:\tmp\myapp\target\myapp]
[INFO] Processing war project
[INFO] Building war: D:\tmp\myapp\target\myapp.war
[INFO]
[INFO] --- cargo-maven3-plugin:1.9.0:run (default-cli) @ myapp ---
[INFO] [en3.ContainerRunMojo] Resolved container artifact org.codehaus.cargo:cargo-core-container-glassfish:jar:1.9.0 for container glassfish6x
[INFO] [talledLocalContainer] Parsed GlassFish version = [6.0.0]
[INFO] [talledLocalContainer] GlassFish 6.0.0 starting...
[INFO] [talledLocalContainer] Using port 4848 for Admin.
[INFO] [talledLocalContainer] Using port 8080 for HTTP Instance.
[INFO] [talledLocalContainer] Using port 7676 for JMS.
[INFO] [talledLocalContainer] Using port 3700 for IIOP.
[INFO] [talledLocalContainer] Using port 8181 for HTTP_SSL.
[INFO] [talledLocalContainer] Using port 3820 for IIOP_SSL.
[INFO] [talledLocalContainer] Using port 3920 for IIOP_MUTUALAUTH.
[INFO] [talledLocalContainer] Using port 8686 for JMX_ADMIN.
[INFO] [talledLocalContainer] Using port 6666 for OSGI_SHELL.
[INFO] [talledLocalContainer] Using port 9009 for JAVA_DEBUGGER.
[INFO] [talledLocalContainer] Distinguished Name of the self-signed X.509 Server Certificate is:
[INFO] [talledLocalContainer] [CN=hantsy-t540p,OU=GlassFish,O=Eclipse.org Foundation Inc,L=Ottawa,ST=Ontario,C=CA]
[INFO] [talledLocalContainer] Distinguished Name of the self-signed X.509 Server Certificate is:
[INFO] [talledLocalContainer] [CN=hantsy-t540p-instance,OU=GlassFish,O=Eclipse.org Foundation Inc,L=Ottawa,ST=Ontario,C=CA]
[INFO] [talledLocalContainer] Domain cargo-domain created.
[INFO] [talledLocalContainer] Domain cargo-domain admin port is 4848.
[INFO] [talledLocalContainer] Domain cargo-domain allows admin login as user "admin" with no password.
[INFO] [talledLocalContainer] Command create-domain executed successfully.
[INFO] [talledLocalContainer] Attempting to start cargo-domain.... Please look at the server log for more details.....
[INFO] [talledLocalContainer] Command delete-jdbc-resource failed.
[INFO] [talledLocalContainer] remote failure: The jdbc resource [ jdbc/__default ] cannot be deleted as it is required to be configured in the system.
[INFO] [talledLocalContainer] JDBC Connection pool DerbyPool deleted successfully
[INFO] [talledLocalContainer] Command delete-jdbc-connection-pool executed successfully.
[INFO] [talledLocalContainer] Application deployed with name myapp.
[INFO] [talledLocalContainer] Command deploy executed successfully.
[INFO] [talledLocalContainer] Application deployed with name cargocpc.
[INFO] [talledLocalContainer] Command deploy executed successfully.
[INFO] [talledLocalContainer] GlassFish 6.0.0 started on port [8080]
[INFO] Press Ctrl-C to stop the container...
```

该命令为您执行一系列任务。

1.  首先，它清理并构建项目到一个 war 包中。
2.  然后使用 cargo maven 插件下载 Glassfish v6.0 的副本，并提取项目构建文件夹中的文件。
3.  然后为这个项目创建一个新的`cargo-dommain`并立即启动它。
4.  Glassfish 运行后，将打包的应用程序部署到 Glassfish。

> *您可以查看*myqpp/target/glassfish 6x-home/cargo-domain/logs*中的* server.log *文件，查看服务器启动和应用部署的详细信息。*

打开另一个终端窗口，尝试测试 sample REST API 端点。

```
> curl [http://localhost:8080/myapp/api/greeting/JakartaEE](http://localhost:8080/myapp/api/greeting/JakartaEE)
{"message":"Say Hello to JakartaEE at 2021-03-31T16:32:39.216"}
```

与我的 [jakartaee 9 starter 模板项目](https://github.com/hantsy/jakartaee9-starter-boilerplate)相比，maven 原型提供了一种体验 jakartaee 9 堆栈的更快方法。

要停止正在运行的应用程序，在原始终端中发送一个`CTRL+C`快捷方式。

```
[INFO] Press Ctrl-C to stop the container...
[INFO] [talledLocalContainer] GlassFish 6.0.0 is stopping...
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  18:46 min
[INFO] Finished at: 2021-03-31T16:43:34+08:00
[INFO] ------------------------------------------------------------------------
[INFO] [talledLocalContainer] Waiting for the domain to stop .
[INFO] [talledLocalContainer] Command stop-domain executed successfully.
[INFO] [talledLocalContainer] GlassFish 6.0.0 is stopped
Terminate batch job (Y/N)? y
```

# 运行样本测试

生成的项目包括两个简单的 JUnit/Arquilian 测试类来演示测试 CDI bean 和 REST API 的技术。

运行示例测试代码。

1.  首先确保有一个运行的 Glassfish 服务器，检查[用 GalssFish 6 测试 Jakarta EE 9 应用程序](https://github.com/hantsy/jakartaee9-starter-boilerplate/blob/master/docs/arq-glassfish.md)。
2.  然后运行以下命令。

*   mvn 清除验证-DskipTests=false

接下来，您可以开始添加代码来丰富应用程序。