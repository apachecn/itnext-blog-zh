# 同一项目中的多个 Spring Boot 应用程序

> 原文：<https://itnext.io/multiple-spring-boot-apps-same-project-f6208d1a97d9?source=collection_archive---------4----------------------->

![](img/fb2616aadf012f900b01723dc46bb152.png)

我经常在我的演示中使用 Spring Boot 框架。最新的也没什么不同。它展示了如何使用两种不同的代码路径实现 CQRS:

*   命令部分通过 [Spring Data JPA](https://spring.io/projects/spring-data-jpa) 实现
*   查询部分通过 [jOOQ](https://www.jooq.org/)

我的用例是一个银行应用程序，它提供了一个 REST 层，允许客户调用任何部分。使用`curl`演示查询部分非常简单，因为 URL 并不复杂:

```
curl localhost:8080/balance/123          // 1
```

1.  查询账户`123`的余额

另一方面，创建一个新的操作，*，例如*，一个信用，需要将数据传递给`curl`。虽然这样做是可行的，但是有效载荷本身的结构*很复杂，因为它是 JSON 的一部分。因此，使用`curl`在现场观众面前演示命令部分是有风险的。我倾向于避免不必要的风险，所以我考虑了一些替代方案。*

我的第一个想法是提前准备命令，在演示过程中进行复制粘贴。我想做几个不同的手术，所以我必须:

*   要么准备一个命令，粘贴它并在运行它之前更改它
*   或者准备所有不同的命令

对我来说，两者都太尴尬了。

另一个选择是在同一个项目中创建另一个`@SpringBootApplication`注释类:

```
@SpringBootApplication
public class GeneratorApplication { @Bean
    public CommandLineRunner run() {
        var template = new RestTemplate();
        return args -> {
            var operation = generateRandomOperation();          // 1
            LongStream.range(0, Long.parseLong(args[0]))        // 2
                .forEach(
                    operation -> template.postForObject(        // 3
                        "http://localhost:8080/operation",
                        operation,
                        Object.class));
        };
    } public static void main(String[] args) {
        new SpringApplicationBuilder(GeneratorApplication.class)
                .run(args);
    }
}
```

1.  不知何故，生成一个随机的`Operation`
2.  从参数中获取调用次数
3.  调用主 web 应用程序的 URL

当我在另一个 web 应用程序之后启动这个应用程序时，它失败了。这有两个原因:

1.  两个应用程序共享同一个 Maven POM。当`spring-boot-starter-web`在类路径上时，生成器应用程序试图启动 Tomcat。它失败是因为第一个应用程序绑定了默认端口。
2.  默认情况下，Spring Boot 依赖于组件扫描。因此，web 应用程序扫描生成器应用程序及其声明的 beans 并创建它们。可以这样重新定义一些 beans。然而，webapp 也创建了上面的`CommandLineRunner` bean。因此，当它的服务器还没有准备好的时候，它就“向自己发布”。

最直接的解决方案是将每个应用程序的类移动到它们自己专用的 Maven 模块中。您需要在每个模块中创建一个 POM，只包含必要的依赖项。另外，我需要在 webapp 的 runner 中使用几个类。虽然我可以在其他模块中复制它们，但这是额外的工作和复杂性。

为了防止类路径扫描，我们将每个应用程序类移动到它的包中。请注意，当包有父子关系时，它不起作用:它们必须是兄弟。

为了创建特定类的 beans，我们需要依赖特定的注释，这取决于它们的性质:

*   对于 JPA 实体，`@EntityScan`，指向要扫描的包
*   对于 JPA 存储库，`@EnableJpaRepositories`，指向要扫描的包
*   对于其他类，`@Import`指向生成 bean 的类

最后一步是阻止生成器应用程序启动 web 服务器。您可以在启动应用程序时对其进行配置。

```
@SpringBootApplication
@EnableJpaRepositories("org.hazelcast.cqrs")                    // 1
@EntityScan("org.hazelcast.cqrs")                               // 2
public class GeneratorApplication { public static void main(String[] args) {
        new SpringApplicationBuilder(GeneratorApplication.class)
                .web(WebApplicationType.NONE)                   // 3
                .run(args);
    } // Command-line runner
}
```

1.  扫描 JPA 存储库
2.  扫描 JPA 实体
3.  阻止 web 服务器启动

它像预期的那样工作，我们最终可以获得设置的好处。

对于现实世界的应用程序，您可能会创建一个自动执行的 JAR。您需要开发一种在构建期间配置主类的方法，每个应用程序一个。我认为最好不要这样做，把它作为一个演示版。

*   [创建一个非网络应用](https://docs.spring.io/spring-boot/docs/2.6.x/reference/htmlsingle/#howto.application.non-web-application)
*   [使用 Spring 数据仓库](https://docs.spring.io/spring-boot/docs/2.6.x/reference/htmlsingle/#howto.data-access.spring-data-repositories)
*   [将@Entity 定义从弹簧配置中分离出来](https://docs.spring.io/spring-boot/docs/2.6.x/reference/htmlsingle/#howto.data-access.separate-entity-definitions-from-spring-configuration)
*   [导入附加配置类](https://docs.spring.io/spring-boot/docs/2.6.x/reference/htmlsingle/#using.configuration-classes.importing-additional-configuration)

*原载于* [*一个 Java 极客*](https://blog.frankel.ch/multiple-spring-boot-apps-same-project/)*2021 年 12 月 5 日*