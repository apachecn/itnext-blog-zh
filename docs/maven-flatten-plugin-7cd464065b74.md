# 扁平化 Maven 插件

> 原文：<https://itnext.io/maven-flatten-plugin-7cd464065b74?source=collection_archive---------6----------------------->

![](img/2c697a2a5012bd68a241085d980add32.png)

一位 Apache Maven 提交者最近写了关于 Maven 5 的计划。我认为以下是最重要的变化之一:

> *总之，我们需要区分两种 POM 类型:*
> 
> **存储在项目源代码控制中的*构建 *POM，它在构建时使用 v5 模式，要求新的 Maven 版本能够使用与新模式相关联的新特性，*
> 
> * *消费者*POM，它在良好的旧 v4 模式中发布到 Maven Central，因此每个过去或未来的构建工具都可以像往常一样使用预构建的工件作为它们的依赖项。

这是一个重要的二分法，我忽略了很长时间:

*   POM 的消费者需要一些数据，*例如*，在他们的依赖列表中有项目的用户
*   二进制(ies)生成器需要其他数据

还有其他的担忧。例如，变量对于项目开发人员利用 DRY 是有意义的。对于消费者来说，这是一个额外的间接层，使得理解 POM 更加困难。

在 Reddit 上，用户 pmarschall 提到他们已经在 [Maven Flatten 插件](https://www.mojohaus.org/flatten-maven-plugin/)的帮助下分离了 Maven 当前版本中的关注点。它引起了我的兴趣，我想尝试一下。为此，我使用了 Spring 宠物诊所项目— commit [a7439c7](https://github.com/spring-projects/spring-petclinic) 。

用法非常简单。只需在`plugins`部分添加以下代码片段:

```
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>flatten-maven-plugin</artifactId>
    <version>1.2.5</version>
    <configuration>
    </configuration>
    <executions>
        <execution>
            <id>flatten</id>
            <phase>process-resources</phase>
            <goals>
                <goal>flatten</goal>
            </goals>
        </execution>
        <execution>
            <id>flatten.clean</id>
            <phase>clean</phase>
            <goals>
                <goal>clean</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

现在，如果您执行 Maven `process-resources`阶段，插件将创建一个 POM 的缩短版本，命名为`.flattened-pom.xml`。与初始的 POM 相比，除了坐标之外，展平的 POM 只有三个部分:`licenses`、`dependencies`和`repositories`。此外，Maven 已经解析了所有变量。如果您运行`install`阶段，然后检查您的本地 Maven 库，您会注意到 POM 与扁平化的 POM 匹配，而不是主 POM。如果想生成展平的 POM，但不想替换主 POM，使用`-DupdatePomFile=false`。

默认情况下，插件只保留了`licenses`、`dependencies`和`repositories`部分。您可以通过 POM 配置保留和不保留哪些部分。比如插件去掉了`name`，但是需要的话可以轻松保留。只需添加相关配置:

```
<configuration>
    <pomElements>
        <name>keep</name>
    </pomElements>
</configuration>
```

上面的方法给了你最大的灵活性。然而，插件的开发者已经考虑了哪些配置包是有意义的，并提供了开箱即用的配置包。以下是描述它们的文档摘录:

*   `oss`:对于希望保留除`repositories`和`pluginRepositories`之外的所有`FlattenDescriptor`可选 POM 元素的开源软件项目。
*   `ossrh`:保留 OSS 存储库托管所需的所有`FlattenDescriptor`可选 POM 元素。
*   `bom`:与`ossrh`相似，但增加了`dependencyManagement`和`properties`。特别是，它将保持 dependencyManagement 不变，而不解决父影响和导入范围的依赖关系。如果您的 POM 代表一个 BOM，并且您不想按原样部署它(删除父对象和解析版本变量等),这将非常有用。).
*   `defaults`:默认模式，删除除`repositories`外的所有`FlattenDescriptor`可选 POM 元素。
*   `clean`:删除所有`FlattenDescriptor`可选的 POM 元素。
*   `fatjar`:删除所有`FlattenDescriptor`可选 POM 元素和所有`dependencies`。
*   `resolveCiFriendliesOnly`:仅解析变量 revision、sha1 和 changelist。保留其他所有东西。请参阅 Maven CI Friendly 了解更多详细信息。

# 结论

Maven Flatten 插件将构建 POM 和消费者 POM 分开。不需要等到 Maven 5 发布。这是免费的，所以如果你是一个图书馆提供商，你可能应该考虑使用它。

**更进一步:**

*   [从 Maven 3 到 Maven 5](https://www.javaadvent.com/2021/12/from-maven-3-to-maven-5.html)
*   [展平 Maven 插件](https://www.mojohaus.org/flatten-maven-plugin/)
*   [扁平化 Mojo 描述和配置](https://www.mojohaus.org/flatten-maven-plugin/flatten-mojo.html)

*原载于* [*一个 Java 怪胎*](https://blog.frankel.ch/maven-flatten-plugin/)*2022 年 1 月 30 日*