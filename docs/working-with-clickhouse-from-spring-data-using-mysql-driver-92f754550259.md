# 使用 MySql 驱动程序从 Spring 数据使用 ClickHouse

> 原文：<https://itnext.io/working-with-clickhouse-from-spring-data-using-mysql-driver-92f754550259?source=collection_archive---------6----------------------->

![](img/5f4e0fabd4c5d9fe5a38389ce40327d4.png)

由[万花筒](https://unsplash.com/@kaleidico)在 [Unsplash](https://unsplash.com) 上拍摄的照片

前一段时间我接到一个任务，要写一个将数据插入 ClickHouse 的服务。我的团队使用 Kotlin 和 Spring 框架，所以我决定尝试 Spring Data JDBC 作为 ClickHouse 的 ORM 框架。经过一番研究，我发现 ClickHouse 有一个 [MySql 接口](https://clickhouse.tech/docs/en/interfaces/mysql/)。因此，Spring Data JDBC 很可能能够使用 MySql 驱动程序与 ClickHouse 进行通信。

不幸的是，Spring Data JDBC 并不能通过 MySql 驱动程序与 ClickHouse 一起使用。在这种方法行得通之前，必须解决几个问题。

这篇文章是关于克服我在执行任务时遇到的问题和解决方法的。

**更新:**写完这篇文章后，我发现了关于 [ClickHouse 方言的春天数据 JDBC](https://github.com/pelenthium/clickhouse-dialect-spring-boot-starter) 。从 Spring 数据中使用 ClickHouse 似乎是更正确的方式。在实现我的方法之前检查一下这个库。

**免责声明**:我写的是 ClickHouse 的一个罕见用例。ClickHouse 是 OLAP 数据库，不能用作 OLTP 数据库，所以除非你确信这种方法是正确的，否则不要重复它。ClickHouse 不支持更新/删除和许多其他 SQL 语句，在为您的项目选择 ClickHouse 时要小心。

# 演示项目

在这篇文章中，我使用了科特林、Spring Boot 和 ClickHouse 的官方 docker 图片。你可以在我的 GitHub 上的[这个 repo](https://github.com/Jaitl/clickhouse-spring-data-demo) 里找到源代码。在`README.MD`文件中有一个指南，向您解释如何运行和测试演示项目。

## 演示项目的 API 和 DB 模式

为了演示基本原理，我在演示项目中创建了一个简单的 API 和简单的 DB 模式。我的 API 只有两个方法:**创建一个项目**和**获取所有项目**因为 ClickHouse 不支持更新和删除操作，所以我不能实现所有的 CRUD 方法。

`DataItem`是我的`items`表的数据模型。`items`表中的项目将被映射到`DataItem`。

以下是我的 API 的端点:

```
POST **v1/demo/item
GET v1/demo/item**
```

`POST`方法将创建一个带有`DataItem`模型的新条目，而`GET`方法将从`items`表中返回所有条目。

让我们来看看我遇到的问题以及如何解决它们。

# 问题 1。交易

Spring Data 在每次查询后执行提交事务，但是 ClickHouse 不支持事务，因此在执行对 ClickHouse 的查询时会抛出异常。

为了解决这个问题，我通过用空的`commit`和`rollback`方法覆盖`PlatformTransactionManager`类来禁止提交事务。

# 问题二。调试日志记录级别

当`DEBUG`级别为 Spring 数据 JDBC 启用时，MySql 驱动程序将崩溃，并出现异常。发生这种情况是因为驱动程序试图执行`WARNINGS`命令来从 DB 获取用于调试的附加信息，但是 ClickHouse 不支持这种声明。

为了解决这个问题，我将 Spring 数据 JDBC 记录器的记录器级别设置为`INFO`。

```
<**logger name="org.springframework.jdbc" level="INFO"** />
```

# 问题三。数组数据类型

MySql 不支持数组数据类型，因此当我将一个列表或数组保存到数据库时，MySql 驱动程序会抛出一个异常。

为了解决这个问题，我围绕列表数据类型创建了一个自定义包装器，并为自定义包装器创建了自定义转换器。

不要忘记注册自定义转换。

如果您遇到不支持的数据类型的问题，您必须为每个类型创建一个自定义的转换器。

# 问题 4。主要 ID 生成

ClickHouse 没有为 ID 自动增加数据类型。ClickHouse 假定 ID 字段是由用户分配的。如果您在代码中填写项目的`ID`字段，那么当 Spring Data 试图更新一个项目而不是创建一个新项目时，问题就出现了。

为了解决这个问题，我从`Persistable`接口继承了我的模型，并覆盖了`isNew`方法，这样它总是返回`true`。

# 结论

正如你所看到的，当你使用 JDBC Spring Data 的 ClickHouse 时会有一些困难。幸运的是，所有的困难都可以解决，您可以享受使用强大的 ORM 库向 ClickHouse 插入数据，而无需手动将实体映射到 SQL 语句。

感谢阅读。我希望这有所帮助。如果你有任何问题，请随时回复。

# 依赖

1.  [演示项目的源代码](https://github.com/Jaitl/clickhouse-spring-data-demo)
2.  [ClickHouse MySql 接口](https://clickhouse.tech/docs/en/interfaces/mysql/)