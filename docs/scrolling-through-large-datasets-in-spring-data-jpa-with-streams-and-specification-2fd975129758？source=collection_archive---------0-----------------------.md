# 使用流和规范在 Spring Data JPA 中滚动大型数据集

> 原文：<https://itnext.io/scrolling-through-large-datasets-in-spring-data-jpa-with-streams-and-specification-2fd975129758?source=collection_archive---------0----------------------->

*如何高效地构建 CSV 文件——以 Spring Data JPA 的流功能和 JPA 规范(或任何其他相关的东西)为特色。*

![](img/4f370d2b0a9da7b2a9fc665b47f8cb17.png)

示例 GitHub(如果你更愿意浏览代码本身而不是阅读本文):[https://GitHub . com/verzac/demo-spring-data-JPA-stream-and-spec](https://github.com/verzac/demo-spring-data-jpa-stream-and-spec)

# 我们为什么需要这个？

*如果你是专门找这篇文章的，不需要解释为什么这很重要，请随意跳过。*

想象以下场景:您的用户希望根据动态标准(例如，他们的年龄超过 30 岁)从您的数据库下载一个填充了客户数据的 CSV 文件。简单吧？任何系统都应该具有这种导出功能！

但是等等，没那么简单。首先，您不能总是在内存中加载数据集和构造 CSV，以免您的应用程序以最怪异的方式崩溃:耗尽内存。

# 我们要用什么？

创建 CSV 是比较容易的部分:您所要做的就是返回一个包含所有数据的文本。CSV 文本中的每一行都代表一行(和/或本例中的一个实体),每一行都包含一组用逗号分隔的数据。

这是困难的部分:因为您的用户希望按照某个标准动态过滤，所以我们必须使用一种允许您动态构建 WHERE 子句的机制。如果你以前使用过 Spring，你可能会注意到这不是很容易/优雅地做到，因为 Spring 为你设置的许多功能都是在系统启动时设置的。

这就是为什么我们需要 JPA 规范。

JPA 规范允许您在 Spring 数据中动态地创建 WHERE 子句，而无需实际实现自己的存储库类(即，您自己处理 EntityManager 并手动创建查询，这对于许多样板代码来说是一个滑坡)。

JPA 规范有几个优点:

*   默认情况下，它与 Spring 数据一起提供。存在 JPA 规范的第三方替代方案，如 QueryDSL。这些第三方替代品*可能*为一些开发者提供更好的 API。然而，JPA 规范位于 Spring 现有的 Criteria API(自己编写 SQL 字符串的替代方案)之上，这意味着您编写的逻辑可以在 JPA 规范和 Criteria API 之间互换(某种程度上)。
*   您可以使用简单但可行的布尔逻辑来混合和匹配规范。因为这是基于代码的，而不是 Spring 自动为你生成的，所以你可以根据需要动态地混合和匹配。例如，您可以使用 AND 操作符组合以下两个规范:`CustomerSpecification.hasName(“Ben”).and(CustomerSpecification.hasJob(“Software Developer”))`，它相当于`SELECT … WHERE name = ‘Ben’ AND job = ‘Software Developer’`。

现在，这篇博文不会深入讨论如何创建自己的规范；已经有大量的资源可以帮助你做到这一点:

*   上面的 GitHub repo 是一个示例项目，它实现了我们将要在这里讨论的所有内容。
*   [https://spring . io/blog/2011/04/26/advanced-spring-data-JPA-specifications-and-query DSL/](https://spring.io/blog/2011/04/26/advanced-spring-data-jpa-specifications-and-querydsl/)

# 将一切联系在一起

那么，我们如何实现它来构造我们可爱的 CSV 文件呢？

## 将所有内容加载到内存中，然后将它们全部整理到一个 CSV 中。

坦白地说，对于某些系统来说，这已经足够了。例如，如果您知道您的数据永远不会超过 100 行，那么您可能不希望过度设计您的解决方案，而只是快速运行一行程序来实现这一点。类似于…

然而，问题是这种解决方案不太具有可伸缩性，而且坦率地说，很危险，因为将所有内容都加载到内存中意味着您可以加载太多的数据，以至于应用服务器的内存无法容纳。例如，如果您的数据每天都在增长，有一天，您不得不将 1 亿条记录加载到应用程序的内存中，该怎么办？我告诉你:你的应用要崩溃了。

这使我们……

## 逐页加载所有内容，然后将每一页作为 CSV 文件顺序写入输出

这是第二种选择，也是最常见的一种，真的。这种技术背后的基本概念是，您将数据分割成多个一口大小的“页面”，然后逐页处理它们。这可以防止应用服务器崩溃，因为您试图将整个数据库加载到应用程序中。

请记住，我们使用`PrintWrite`来发送数据，而不是在内存中整理完整的列表然后返回。这允许您在处理下一个页面之前将页面内容刷新到您的响应中，有效地防止您的应用程序耗尽内存。

这种方法没有什么大的错误。这是广泛应用中使用的标准方法。如果这种方法对你有效，请继续使用它。

然而，这种方法有一些奇怪的地方。

首先，您必须编写自己的逻辑来“迭代”页面，这意味着需要测试和维护更多的代码。现在，这还不是世界末日，但这样做确实很烦人。记住，你写的代码越少，你需要维护的就越少。

另一个更重要的原因是，据报道，这种分页方法不如下一种方法有效，后者是…

# 使用 Spring Data JPA 的流功能

这就是我们今天要讲的内容。

简而言之，调用您的存储库方法将产生一个`Stream<YourObject>`。您的持久性提供者(例如，通过 ScrollableResultSet 的 Hibernate，通过 ScrollableCursor 的 EclipseLink)将决定如何有效地将您想要的数据传输到您的应用程序中，尽管可以放心的是，他们可以*通常*比您自己实现它们更好地处理分页逻辑。

这种方法有几个优点:

*   更好的代码组织，因为你不必处理数据集的迭代逻辑。
*   性能，因为您不必向数据库发出多个庞大的页面请求。
*   记忆友好！[至少对于 PostgreSQL 和 MySQL 您可以将加载到内存中的条目数量限制为一次只能加载一个条目，其余的都可以进行 GC 处理！

[1][https://www.baeldung.com/spring-data-java-8](https://www.baeldung.com/spring-data-java-8)

Spring 数据已经支持流。有关实现这一点的各种方法，请参见以下代码片段:

## **但是等等，这两个特性还不一定兼容呢！**

有人谈到在 JpaSpecificationExecutor 中引入一个基于流的结果(如果你想使用 JPA 规范，这是你通常会扩展的)，但我不认为它背后有任何真正的驱动(我丢失了票号；埋在 Spring Data 的 JIRA 里，见 [https://jira.spring.io](https://jira.spring.io) 。

事实上，实现它(为你自己)是相当容易的；你所需要做的就是使用 EntityManager 的`getResultStream` 而不是通常的`getResultList` 返回一个列表(`CustomerRepository.findAll`在幕后使用它)。

那么，当 Spring 数据中存在功能缺口时，你会怎么做呢(除了自己实现整个存储库类，只是为了使用一个新的、时髦的功能)？

# 仓库碎片来拯救！

 [## Spring 数据共享——参考文档

### 由于各个 Spring 数据模块的开始日期不同，大多数都有不同的主要和次要版本…

docs.spring.io](https://docs.spring.io/spring-data/data-commons/docs/current/reference/html/#repositories.single-repository-behavior) 

“片段”是某种程度上可重用的、定制的拼图，您的存储库类可以对其进行扩展，以在任何现有的通用查询方法(例如 findAll、findById)之上添加定制的方法/功能。以下是 Spring Data 的文档对碎片的描述:

> 当查询方法需要不同的行为或者无法通过查询派生来实现时，就有必要提供自定义实现。Spring 数据存储库允许您提供定制的存储库代码，并将其与通用的 CRUD 抽象和查询方法功能集成在一起。

你觉得这听起来可怕吗？最初对我来说是这样，但实际上这是我发现的最好的东西，可以用来“聚合填充”Spring 数据中缺失的功能。

你可以这样使用它:

首先，您必须定义一个片段接口:

然后是片段接口的实现(注意，您可以`@Autowire`任何您想要的东西，因为 Spring 在运行时将它实例化为 bean，即使您没有将它作为 Spring 组件):

最后，为了在您的存储库中使用定制方法，您需要做的就是扩展我们制作的片段接口:

瞧啊。现在 CustomerRepository 将有一个接受您的规范的 stream()方法！你可以这样使用它:

请记住，这不仅限于 JPA 规范；使用存储库片段，您实际上可以让您的存储库按照您想要的方式运行，因为您可以完全控制方法是如何执行的。

# 重要花絮

## 流只能在事务中打开

这是因为当我们在流中返回数据时，我们必须保持数据库状态足够稳定，这样我们才能安全地滚动它的数据。除此之外，默认情况下，当您在 Spring 数据中做了一些事情之后，您的 DB 会话是“关闭”的，但是流不一定以这种方式工作。当流仍然打开且尚未关闭时，需要保证连接/会话是打开的，因为客户端仍可能检索数据。

## EntityManager::分离

现在，取决于你问谁，这可能是可选的。我还没有仔细研究过流式对象是如何被处理的，但是我想这样做不会有什么坏处(而且我看过的所有资料都推荐这样做)。

据我所知，如果对象没有被首先分离(我认为这是有意义的)，它不会被自动 GC'd 并从内存中删除。请记住，所有这些都发生在一个事务中，因此返回的“对象”与持久性上下文相关联，这意味着对对象的任何更改都会同步到您的数据库中。

## 获取大小？那是什么？

这设置了 JDBC 驱动程序(即从系统中实际连接到数据库的设备)在给定时间应该获取的行数。不同的 DB 驱动程序需要不同的值来执行逐行提取(这就是我们在这里要做的)。我记得 MySQL 要求 HINT_FETCH_SIZE 为`"" + Integer.MIN_VALUE`，PostgreSQL 要求值设置为 1。

差不多就是这样了！现在，您可以在任何需要这个规范+流能力的存储库中扩展`StreamableJpaSpecificationRepository`。

## 我必须使用 JPA 规范吗？

不可以。存储库允许您使用任何东西来定义您的“标准”。只要它与 Criteria API(或 EntityManager)兼容，就可以使用这个方法来插入任何缺失的功能。

同样，如果您想玩我设置的示例应用程序，请查看我的 GitHub repo，它附带了一个 SQL 脚本，可以用 500，000 条虚拟数据记录播种您的数据库:

[](https://github.com/verzac/demo-spring-data-jpa-stream-and-spec) [## verzac/demo-spring-data-JPA-stream-and-spec

### 或者，您可以自己设置 Postgres 实例并编辑 src/main/resources/application-local . yml docker run…

github.com](https://github.com/verzac/demo-spring-data-jpa-stream-and-spec) 

# 后续步骤

您可以采取各种后续步骤来优化这一功能:

*   您可以将您的 CSV 流功能分离到一个单独的服务中，该服务处理所有这些逻辑，这样您就可以让其他组件重用它。我为以前的项目这样做，它允许任何控制器利用这一功能；控制器所要做的就是将响应的 PrintWriter 传递给服务。
*   将它与 StreamingResponseBody 结合起来，以便将繁重的工作卸载到另一个线程，这样就可以释放 HTTP servlet 线程，从而防止阻塞应用程序。这对于大型数据集非常有用。我在这篇中型文章中发现了这个绝妙的技巧:

[](https://medium.com/swlh/streaming-data-with-spring-boot-restful-web-service-87522511c071) [## 使用 Spring Boot RESTful Web 服务流式传输数据

### 流式数据是一种全新的向网络浏览器发送数据的方式，它提供了显著更快的页面速度…

medium.com](https://medium.com/swlh/streaming-data-with-spring-boot-restful-web-service-87522511c071) 

*   告诉你的用户不要再要求这个功能了，这样你就不用构建了；)

感谢阅读！希望你喜欢这篇文章，并随时向我提供任何反馈意见:)