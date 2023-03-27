# MikroORM 5:更严格、更安全、更智能

> 原文：<https://itnext.io/mikro-orm-5-stricter-safer-smarter-b8412e84cca4?source=collection_archive---------2----------------------->

MikroORM 的下一个主要版本刚刚发布。题目说:更严格，更安全，更聪明——为什么？

![](img/d296387739508f0d63dc32217a8a6785.png)

*   大大提高了类型安全性(例如，填充和部分加载提示)
*   自动刷新模式(这样我们就不会丢失内存中的更改)
*   自动刷新加载的实体(对`refresh: true`说再见)
*   修改了模式差异，支持自动向下迁移
*   还有更多更多的 …

> 这一次花了将近一年的时间——V5 的初始工作开始于 2021 年 3 月[。](https://github.com/mikro-orm/mikro-orm/issues/1623)

# 以防你不知道…

如果你从未听说过[mikro RM](https://github.com/mikro-orm/mikro-orm)，它是一个带有工作单元和身份映射的类型化数据映射 ORM。它目前支持 MongoDB、MySQL、PostgreSQL 和 SQLite 驱动程序。ORM 的主要特点是:

*   [隐性交易](https://github.com/mikro-orm/mikro-orm#implicit-transactions)
*   [基于变更集的持久性](https://github.com/mikro-orm/mikro-orm#changeset-based-persistence)
*   [身份地图](https://mikro-orm.io/docs/identity-map/)

![](img/e2640baf18330045f5febbf3f6a69b28.png)

你可以在这里阅读完整的[介绍性文章](https://medium.com/dailyjs/introducing-mikro-orm-typescript-data-mapper-orm-with-identity-map-9ba58d049e02)(但是请注意，自那篇文章发表以来，许多事情已经发生了变化)或者浏览文档。

# 4.x 版本的快速摘要

在我们深入了解 v5 之前，让我们回顾一下 4.x 版本中发生了什么:

*   [结果缓存](https://mikro-orm.io/docs/caching/)
*   [自动交易上下文](https://github.com/mikro-orm/mikro-orm/pull/959)
*   [嵌套的可嵌入内容](https://mikro-orm.io/docs/embeddables/#nested-embeddables)以及该领域的许多其他改进
*   [使用环境变量进行配置](https://mikro-orm.io/docs/configuration/#using-environment-variables)

历史课已经讲够了，让我们来谈谈未来吧！

# 改进的类型安全性

让我们直接进入最有趣的特性——几乎到处都是严格的类型！`em.create()`、`toJSON()`、`toObject()`、填充、部分加载和提示排序，所有这些(甚至更多！)现在严格打字。

让我们看看下面的例子:

首先，我们使用`em.create()`在一个步骤中构建整个实体图。它将验证有效负载的类型和可选性。实体上的一些属性可能有通过钩子或数据库函数提供的默认值——虽然我们可能想将它们定义为必需的属性，但它们在`em.create()`的上下文中应该是可选的。为了解决这个问题，我们可以通过`OptionalProps`符号来指定应该被认为是可选的属性:

> 有些属性名总是被认为是可选的:`id`、`_id`、`uuid`。

然后我们加载所有的`Author`实体，填充它们的图书和图书标签。这里所有的`FindOptions`都是严格类型化的，此外，我们甚至可以跳过`populate`提示，因为它可以从`fields`选项中自动推断出来。

![](img/4dffb94c45ea336f6f9cb13423f65d6d.png)

我们可能仍然需要一些 dto 的类型转换。实体的序列化形式可能非常不可预测——有许多变量定义了实体将如何序列化，例如加载的关系与引用、属性序列化程序、惰性属性、自定义实体序列化程序/ `toJSON`方法、急切加载、递归检查……因此，`EntityDTO`类型上的所有关系都被视为已加载，这样做主要是为了更好地进行 DX，就好像我们将所有关系都类型化为`Primary<T> | EntityDTO<T>`(例如`number | EntityDTO<Book>`)一样，这不可能从智能感知/自动建议中受益。想象一下这个场景:

# 验证改进

除了编译时验证之外，我们还在启动插入查询之前进行了运行时验证，以确保所需的属性有它们的值。这主要在 mongo 中很重要，在那里我们没有模式级的可选性检查。

当我们尝试使用 CLI 而不在本地安装它时，我们也会收到警告。如果我们忘记更新一些 ORM 包，并以版本不匹配和安装了多个核心包而告终，该怎么办？我们现在也验证了这一点！

# 返工模式差异

模式差异一直是最薄弱的环节之一。通常，会产生额外的查询，或者甚至不可能达到完全同步的状态。

模式差异已经被完全重写，以解决所有当前已知的问题，并在此基础上添加了更多的*:*

*   不同的外键约束
*   适当的索引差异(在我们只比较名称之前)
*   自定义索引表达式
*   评论差异
*   列长不同(如`numeric(10,2)`或`varchar(100)`
*   更改主键类型
*   模式/名称空间差异(仅限 Postgres)
*   自动向下迁移(尚不支持 SQLite)
*   检查约束支持(仅限 Postgres)

# 更智能的迁移

在生产环境中，我们可能希望使用编译的迁移文件。从 v5 开始，这几乎可以开箱即用，我们需要做的就是相应地配置迁移路径。执行的迁移现在忽略文件扩展名，因此我们可以在同一个数据库上使用 node 和 ts-node。这是以向后兼容的方式完成的。

创建新迁移现在会自动将目标架构快照保存到 migrations 文件夹中。如果我们尝试创建新的迁移，而不是使用当前的数据库模式，将会使用该快照。这意味着，如果我们试图在运行未决迁移之前创建新的迁移，我们仍然会得到正确的模式差异(如果没有进行额外的更改，将不会创建迁移)。

> 快照应该像常规迁移文件一样进行版本控制。

# 自动冲洗模式

![](img/8fd22d2760efdd9e833eb39f3685d441.png)

到目前为止，刷新一直是一个显式的操作。使用 v5，我们可以配置刷新策略，类似于 JPA/hibernate 的工作方式。我们有 3 种冲洗模式:

*   `FlushMode.COMMIT`-`EntityManager`尝试延迟刷新，直到提交当前事务，尽管它也可能过早刷新。
*   `FlushMode.AUTO` -这是默认模式，仅在必要时刷新`EntityManager`。
*   `FlushMode.ALWAYS` -每次查询前刷新`EntityManager`。

`FlushMode.AUTO`将尝试检测我们正在查询的实体的变化，如果有重叠，则刷新:

文档中关于冲洗模式[的更多信息。](https://mikro-orm.io/docs/unit-of-work/#flush-modes)

# 自动刷新已加载的实体

以前，当一个实体被加载并且我们需要重新加载它时，需要在选项中提供显式的`refresh: true`。实体的刷新也有一个有问题的副作用——实体数据(用于计算变更集)总是根据新加载的实体进行更新，因此会忘记以前的状态(导致可能丢失刷新前对实体所做的更新)。

现在，我们总是将新加载的数据与当前状态合并，当我们看到更新的属性时，我们会保留更改后的值。此外，对于带有主键条件的`em.findOne()`,我们试图通过比较选项和已经加载的属性名来检测重新加载实体是否有意义。在这一步中，考虑了`fields`和`populate`选项，以支持部分加载和惰性属性。

对于`em.findOne()`中的复杂条件和通过`em.find()`的任何查询，我们总是无论如何都要进行查询，但是现在不是忽略数据以防这样的实体被加载，而是以同样的方式将它们合并。

# 播种机包装

MikroORM v5 现在有了一个新的包，可以用初始数据或测试数据植入数据库。它允许像往常一样通过相同的 EntityManager API 创建实体，添加对实体工厂的支持，并通过 faker(最新发布的社区版本)生成假数据。

更多示例参见[播种机文档](https://mikro-orm.io/docs/seeding)。

# 多态嵌入

多态嵌入允许我们为一个嵌入的属性定义多个类，根据鉴别器列使用正确的类，类似于单表继承的工作方式。虽然这目前只适用于嵌入式，但对多态实体的支持可能会在 5.x 版本中增加。

查看[文档](https://mikro-orm.io/docs/embeddables/#polymorphic-embeddables)获取完整示例。

在可嵌入性方面还有许多其他小的改进，并且解决了许多问题。两个例子:

*   支持多对一关系(仅存储主键，并能够像使用常规实体一样填充关系)
*   支持`onCreate`和`onUpdate`属性选项

# 填充惰性标量属性

以前，填充惰性标量属性的唯一方法是在包含实体的初始加载期间。如果这样的实体已经加载到身份映射中(没有这个属性)，我们需要刷新它的状态——可能会丢失一些状态。MikroORM v5 也允许通过`em.populate()`填充这些属性。这样做永远不会覆盖我们可能对实体所做的任何内存中的更改。

# 不使用 EntityManager 创建引用

当我们想要创建一个引用，即一个仅由其主键表示的实体时，我们总是必须能够访问当前的 EntityManager 实例，因为这样的实体总是需要被管理。

多亏了`Reference`类上新的助手方法，我们现在可以在不访问`EntityManager`的情况下创建实体引用。如果您想从内部实体构造函数创建引用，这可能会很方便:

> `Reference`包装器是一个可选的类，允许在关系上有更多的类型安全。或者，我们可以使用`Reference.createNakedFromPK()`。

这将创建一个非托管引用，一旦拥有实体被刷新，该引用将被合并到`EntityManager`中。注意，在我们刷新它之前，像`Reference.init()`或`Reference.load()`这样的方法将不可用，因为它们需要`EntityManager`实例。

# 更聪明的`expr`帮手

`expr()`助手可以用来避开严格的输入。它是一个标识函数，除了返回它的参数之外什么也不做——它所做的只是告诉 TypeScript 该值实际上是一个不同的类型(准确地说是一个泛型`string`)。

我们现在可以用另外两种方式使用助手:

*   使用回调签名来允许表达式的动态别名
*   使用数组参数来比较元组

# 一个合适的 QueryBuilder

`QueryBuilder`现在知道它的类型，并且`getResult()`和`execute()`方法基于它被类型化。我们也可以直接等待`QueryBuilder`实例，它将自动执行 QB 并返回适当的响应。QB 实例现在根据`select/insert/update/delete/truncate`方法的使用类型化为:

*   `SelectQueryBuilder` —等待生成实体数组
*   `CountQueryBuilder` —等待产量编号
*   `InsertQueryBuilder` —等待产量`QueryResult`
*   `UpdateQueryBuilder` —等待收益`QueryResult`
*   `DeleteQueryBuilder` —等待收益`QueryResult`
*   `TruncateQueryBuilder` —等待产量`QueryResult`

![](img/694c3a262cfbbbc54fd246ab92f2e5ff.png)

# 通配符模式实体

到目前为止，我们能够在特定的模式中或者没有模式的情况下定义实体。然后这些实体使用基于 ORM 配置或`FindOptions`的模式。这允许我们从特定的模式中读取实体，但是我们忽略了工作单元的力量。

在 v5 中，实体实例现在拥有模式名(作为`WrappedEntity`的一部分)。受管实体将拥有来自`FindOptions`的模式或元数据。像`em.create()`或`em.getReference()`这样创建新实体实例的方法现在有一个 options 参数来允许设置模式。我们也可以用`wrap(entity).getSchema()`和`wrap(entity).setSchema()`。

实体现在可以通过`@Entity({ schema: '*' })`指定通配符模式。这样，它们将在`SchemaGenerator`中被忽略，除非指定了`schema`选项。

*   如果我们指定模式，实体只存在于该模式中
*   如果我们定义`*`模式，实体可以存在于任何模式中，总是由参数控制
*   如果我们跳过模式选项，该值将从全局 ORM 配置中获取

关于这个话题的更多信息可以在[这里](https://mikro-orm.io/docs/next/multiple-schemas#wildcard-schema)找到。

# 实体的深度赋值

另一个弱点是给现有实体赋予新的价值。虽然`wrap().assign()`最初被设计用来更新单个实体及其值，但是很多用户想要分配一个实体图，在一个单独的步骤中更新关系。

在 v5 中，`EntityAssigner`检测什么实体应该被更新的方式已经改变。默认情况下，分配一个深度实体图应该是可能的，不需要任何额外的选项。它的工作基于匹配的实体主键，所以如果你想更新一个关系而不是创建一个新的关系，确保你首先加载它并把它的主键传递给`assign`助手:

如果我们希望总是更新实体，即使在`data`中没有实体 PK，我们也可以使用`updateByPrimaryKey: false`:

关于此主题的更多示例可在文档中找到[。](https://mikro-orm.io/docs/entity-helper/#updating-deep-entity-graph)

# ES 模块的实验支持

虽然 MikroORM v5 仍然作为 CommonJS 编译和发布，但我们添加了几个改进，应该也允许它与 ESM 项目一起使用。也就是说，我们使用`gen-esm-wrapper`包来允许使用命名导入，并且我们使用一个讨厌的技巧来保持动态导入，而不是编译它们来要求语句——为此我们需要使用`MIKRO_ORM_DYNAMIC_IMPORTS` env var。这将允许我们对 ES 模块使用基于文件夹的发现，这在以前是不可能的。

# 其他显著变化

*   联合装载策略的部分装载支持(`fields`
*   `AsyncLocalStorage`在`RequestContext`助手中默认使用
*   `onLoad`事件(类似于`onInit`，但是允许异步，只对加载的实体触发，而不是引用)
*   从 CLI 配置中导出异步函数
*   SQL 的可配置别名策略
*   允许提供[自定义](https://mikro-orm.io/docs/logging) `[Logger](https://mikro-orm.io/docs/logging)` [实例](https://mikro-orm.io/docs/logging)
*   `[persist](https://mikro-orm.io/docs/configuration/#persist-created-entities-automatically)` [选项在](https://mikro-orm.io/docs/configuration/#persist-created-entities-automatically) `[em.create()](https://mikro-orm.io/docs/configuration/#persist-created-entities-automatically)` [和](https://mikro-orm.io/docs/configuration/#persist-created-entities-automatically) `[persistOnCreate](https://mikro-orm.io/docs/configuration/#persist-created-entities-automatically)` [全局配置](https://mikro-orm.io/docs/configuration/#persist-created-entities-automatically)
*   实体生成器中的多对多支持
*   支持指定事务隔离级别
*   控制[填充提示的条件](https://mikro-orm.io/docs/loading-strategies#population-where-condition)
*   修改后的 [API 文档](https://mikro-orm.io/api)
*   还有*很多很多*更，看这里的[全改版](https://github.com/mikro-orm/mikro-orm/blob/master/CHANGELOG.md#500-rc2-2022-02-03)

也一定要检查[升级指南](https://mikro-orm.io/docs/upgrading-v4-to-v5)。

# 下一步是什么？

以下是我希望继续关注的一些事情:

*   允许为 M:N 关系指定 pivot 实体(这样我们可以在那里有额外的列，但出于阅读的目的，仍然将其映射为 M:N)
*   支持数据库视图(或者仅仅是代表 SQL 表达式的实体)
*   更多的驱动程序——也就是更好的——sqlite3 和蟑螂听起来像是唾手可得的果实，因为 knex 现在本地支持它们

> *喜欢*[*MikroORM*](https://mikro-orm.io/)*？⭐️* [*在 GitHub 上明星 it*](https://github.com/mikro-orm/mikro-orm) *分享这篇文章给你的朋友。如果你想在经济上支持这个项目，你可以通过 GitHub 赞助商来做。*