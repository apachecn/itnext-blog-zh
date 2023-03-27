# 我如何将 Django ORM 与 SQLAlchemy 合并，以便更容易地进行数据分析

> 原文：<https://itnext.io/how-i-merged-django-orm-with-sqlalchemy-for-easier-data-analysis-6ccf64484cab?source=collection_archive---------0----------------------->

![](img/4e05628df58f210f3ce0445017474d10.png)

> 注意:由于降价，代码示例可能显示不正确。我建议[继续阅读我们博客](https://djangostars.com/blog/merging-django-orm-with-sqlalchemy-for-easier-data-analysis/?utm_source=medium&utm_medium=itnext.io&utm_campaign=djangoorm&utm_content=continue%20reading)上的原始文章，以确保所有示例都能正确显示。

用 Django 框架开发产品通常简单明了；优秀的文档，许多现成的工具，大量的开源库和大型社区。Django ORM 完全控制 SQL 层，防止您出错，并控制查询的底层细节，因此您可以花更多时间用 Python 代码设计和构建应用程序结构。然而，有时这种行为可能会造成伤害——例如，当您正在构建一个与数据分析相关的项目时。用 Django 构建高级查询并不容易；如果不在某处记录或打印生成的 SQL 查询，就很难阅读(用 Python)和理解 SQL 级别的内容。此外，这样的查询可能不够高效，所以当您将更多的数据加载到 DB 中进行处理时，这将对您产生不利影响。在某个时刻，您会发现自己通过 Django cursor 做了太多的原始 SQL，这时您应该休息一下，看看另一个有趣的工具，它位于 ORM 层和原始 SQL 查询层之间。

正如你在文章标题中看到的，我们成功地将 Django ORM 和 SQLAlchemy Core 混合在一起，我们对结果非常满意。我们构建了一个应用程序，通过将数据汇总到图表和表格中，根据吞吐量/效率/员工成本进行评分，并突出显示异常值来帮助分析 EMR 系统产生的数据，从而优化诊所的业务流程并节省资金。

# 把 Django ORM 和 SQLAlchemy 混在一起有什么意义？

我们从 Django ORM 中走出来完成这个任务有几个原因:

*   对于 ORM world，一个对象就是数据库中的一条记录，但是这里我们只处理聚合数据。
*   一些聚合非常棘手，Django ORM 功能不足以满足需求。老实说，有时在一些简单的情况下，很难(甚至不可能)让 ORM 完全按照您想要的方式产生 SQL 查询，当您处理一个**大数据**时，它会对性能产生很大影响。
*   如果您正在通过 Django ORM 构建高级查询，那么很难阅读和理解 Python 中的这类查询，也很难预测哪个 SQL 查询将被生成并处理给数据库。

值得一提的是，我们还建立了第二个数据库，由 Django ORM 处理，以涵盖其他 web 应用程序相关的任务和业务逻辑需求，它很好地做到了这一点。Django ORM 正从一个版本发展到另一个版本，提供越来越多的功能。例如，在最近的版本中，添加了许多简洁的功能，如支持[子查询表达式](https://docs.djangoproject.com/en/1.11/ref/models/expressions/#subquery-expressions)或[窗口函数](https://docs.djangoproject.com/en/2.1/ref/models/database-functions/#window-functions)以及许多其他功能，如果您的问题比修复几个查询更复杂，那么您一定要在使用原始 SQL 或查看 SQLAlchemy 等工具之前尝试一下这些功能。

这就是为什么我们决定看看 SQLAlchemy。它由两部分组成— ORM 和 Core。SQLAlchemy ORM 类似于 Django ORM，但同时，它们又有所不同。与 Django 的活动记录方法相比，SQLAlchemy ORM 使用了不同的概念，即数据映射器。就你在 Django 上构建项目而言，你绝对不应该切换 ORM(如果你没有非常特殊的原因要这么做)，因为你想使用 Django REST 框架、Django-admin 和其他与 Django 模型相关的好东西。

SQLAlchemy 的第二部分称为核心。它位于高级 ORM 和低级 SQL 之间。核心非常强大灵活；它让您能够构建任何您想要的 SQL 查询，当您在 Python 中看到这样的查询时，很容易理解发生了什么。例如，请看文档中的一个示例查询:

```
q = session.query(User).filter(User.name.like('e%')).\  
    limit(5).from_self().\
    join(User.addresses).filter(Address.email.like('q%')).\
    order_by(User.name)
```

这将导致

```
SELECT anon_1.user_id AS anon_1_user_id,  
       anon_1.user_name AS anon_1_user_name
FROM (SELECT "user".id AS user_id, "user".name AS user_name  
FROM "user"  
WHERE "user".name LIKE :name_1  
 LIMIT :param_1) AS anon_1
JOIN address ON anon_1.user_id = address.user_id  
WHERE address.email LIKE :email_1 ORDER BY anon_1.user_name
```

***注意*** :有了这样的技巧，我们就不会陷入`N+1 problem` : `from_select`对查询做了一个额外的`SELECT`包装，所以我们先减少行数(通过`LIKE`和`LIMIT`)，然后才加入地址信息。

# 如何混合 Django 应用程序和 SQLALchemy

因此，如果您感兴趣并想尝试将 SQLAlchemy 与 Django 应用程序混合使用，这里有一些提示可以帮助您。

首先，你需要[用 Engine 创建](http://docs.sqlalchemy.org/en/latest/core/engines.html#engine-configuration)一个全局变量，但是与 DB 的实际连接将在第一次`connect`或`execute`调用时建立。

```
sa_engine = create_engine(settings.DB_CONNECTION_URL, pool_recycle=settings.POOL_RECYCLE)
```

创建*引擎接受连接的附加配置。MySQL/MariaDB/AWS Aurora(MySQL 兼容)有一个交互式*超时设置，默认为 8h，所以如果没有 pool_recycle 额外参数，你会变得很烦人`SQLError: (OperationalError) (2006, ‘MySQL server has gone away’)`。所以`POOL_RECYCLE`应该比`interactive_timeout`小。例如它的一半:`POOL_RECYCLE = 4 * 60 * 60`

下一步是构建您的查询。根据您的应用程序架构，您可以使用 `Table`和`Column`类来声明表和字段[(这些类也可以用于 ORM)，或者如果您的应用程序已经以另一种方式存储了表和列名，您可以通过 table ( `table_name`)和 column ( `col_name`)函数(如此处的](http://docs.sqlalchemy.org/en/latest/core/metadata.html)[所示](http://docs.sqlalchemy.org/en/latest/core/selectable.html#sqlalchemy.sql.expression.table))就地完成。

在我们的应用程序中，我们选择了第二个选项，因为我们在自己的声明性语法中存储了关于聚合、公式和格式的信息。然后，我们构建了一个层，它读取这样的结构并根据提供的指令执行查询。

当您的查询准备好时，只需调用`sa_engine.execute(query)`。游标将一直打开，直到您读取所有数据或者显式关闭它。

有一件很烦的事情值得一提。正如[文档所说的](http://docs.sqlalchemy.org/en/latest/faq/sqlexpressions.html#how-do-i-render-sql-expressions-as-strings-possibly-with-bound-parameters-inlined)，SQLAlchemy 进行查询字符串化的能力有限，所以不容易得到最终要执行的查询。您可以单独打印查询:

```
print(query)SELECT  
role_group_id, role_group_name, nr_patients  
FROM "StaffSummary"  
WHERE day >= :day_1 AND day <= :day_2 AND location_id = :location_id_1 AND service_id = :service_id_1
```

(这个看起来没有那么可怕，但是对于更复杂的查询，它可能有大约 20 多个占位符，手动填充这些占位符非常烦人而且非常耗时，以便稍后在 SQL 控制台中播放。)

如果您只有字符串和数字要插入到查询中，这将为您工作

```
print(s.compile(compile_kwargs={"literal_binds": True}))
```

对于约会来说，这样的伎俩是行不通的。StackOverflow 上有一个关于如何实现预期结果的讨论，但是解决方案看起来没有吸引力。

另一种选择是通过数据库配置将查询记录到文件中，但是在这种情况下，您可能会面临另一个问题；如果 Django ORM 也连接到这个数据库，就很难找到想要调试的查询。

## 你可能也喜欢

[](https://djangostars.com/blog/debugging-python-applications-with-pdb/?utm_source=medium&utm_medium=itnext.io&utm_campaign=djangoorm&utm_content=you%20may%20also%20like1) [## 用 pdb 调试 Python 应用程序

### 调试并不是一个新技巧——大多数开发人员都在工作中积极地使用它。当然，每个人都有自己的方法…

djangostars.com](https://djangostars.com/blog/debugging-python-applications-with-pdb/?utm_source=medium&utm_medium=itnext.io&utm_campaign=djangoorm&utm_content=you%20may%20also%20like1) [](https://djangostars.com/blog/django-performance-optimization-tips/?utm_source=medium&utm_medium=itnext.io&utm_campaign=djangoorm&utm_content=you%20may%20also%20like2) [## Django 性能优化技巧

### 当开发人员接到一个在 Django 上进行性能优化的任务时，我经常会遇到这种情况…

djangostars.com](https://djangostars.com/blog/django-performance-optimization-tips/?utm_source=medium&utm_medium=itnext.io&utm_campaign=djangoorm&utm_content=you%20may%20also%20like2) 

# 测试

注意:Pytest [multidb 注意](http://pytest-django.readthedocs.io/en/latest/database.html#tests-requiring-multiple-databases)说“目前 pytest-django 没有明确支持 django 的多数据库支持。不过，您可以使用普通的 Django TestCase 实例来使用它的 multi_db 支持。”

那么这意味着什么——不支持？默认情况下，Django 将为`DATABASES`定义中列出的每个 db 创建并删除(在所有测试结束时)一个测试数据库。这个特性也能与`pytests`完美配合。

Django `TestCase`和`TransactionTestCase`与`multi_db=True`能够在测试之间清除多个数据库中的数据。也可以通过`django-fixtures`将数据加载到第二个数据库中，但最好使用[型号*妈咪*](https://model-mommy.readthedocs.io/en/latest/basic_usage.html) *或* [*工厂*男孩](https://model-mommy.readthedocs.io/en/latest/basic_usage.html)来代替，它们不受该属性的影响。

在 [pytest-django 讨论](https://github.com/pytest-dev/pytest-django/issues/76)中有一些建议如何解决这个问题并使`multi_db`能够继续 pytesting。

有一个重要的建议——对于有 Django 模型的表，您应该通过 Django-ORM 将数据保存到 DB。否则，您将在编写测试时面临问题。`TestCase`将无法回滚在 Django DB 连接之外发生的其他事务。如果您遇到这种情况，您可以将`TransactionalTestCase`与`multi_db=True`一起用于触发功能的测试，这将通过 SQLAlchemy 连接产生 DB 写入，但是请记住，这种测试比常规的`TestCase`要慢。

另外，另一个场景也是可能的——您只有一个数据库中的 Django-models，并且您通过 SQLAlchemy 使用第二个数据库。这种情况下，`multi_db`根本不影响你。在这种情况下，您需要编写一个 pytest-fixture(或者在`setUp`中作为 mixin 和触发器逻辑，如果您使用 unittests 的话),它将从 SQL 文件创建 DB 结构。这样的文件应该在`CREATE TABLE`之前包含`DROP TABLE IF EXISTS`语句。这个 fixture 应该被应用到每个操作这个数据库的测试用例中。其他设备可以将数据加载到创建的表中。

注意:这样的测试会比较慢，因为每次测试都要重新创建表。理想情况下，应该创建一次表(声明为`@pytest.fixture(scope='session', autouse=True)`)，并且每个事务应该回滚每个测试的数据。这并不容易实现，因为不同的连接:Django & SQLAlchemy 或 SQLAlchemy 连接池的不同连接，例如，在您的测试中，您启动事务，用测试数据填充 DB，然后运行测试并回滚事务(它没有被提交)。但是在测试过程中，您的应用程序代码可能会像 connection.execute(query)一样对数据库进行查询，这些查询是在创建测试数据的事务之外执行的。因此，使用默认的事务隔离级别，应用程序将看不到任何数据，只能看到空表。对于 SQLAlchemy 连接，可以将事务隔离级别更改为`READ UNCOMMITTED`，一切都将按预期工作，但这绝对不是一个解决方案。

# 结论

综上所述，SQLAlchemy Core 是一个很棒的工具，它让您更接近 SQL，并让您理解和完全控制查询。如果您正在构建需要高级聚合的应用程序(或其中的一部分),那么作为 Django ORM 工具的替代，值得一试 SQLAlchemy 核心功能。

请继续阅读，了解如何更轻松地为数据分析项目构建高级查询。了解我们如何设法混合 Django ORM 和 SQLAlchemy Core，以及我们从中获得了什么。

[![](img/45a1ce60def7b50ffe2311c7a897d600.png)](https://djangostars.com/services/python-django-development/?utm_source=medium&utm_medium=itnext.io&utm_campaign=djangoorm&utm_content=banner_end)

> 这篇关于 [**合并 Django ORM**](https://djangostars.com/blog/merging-django-orm-with-sqlalchemy-for-easier-data-analysis/?utm_source=medium&utm_medium=itnext.io&utm_campaign=djangoorm&utm_content=Merging%20Django%20ORM) 的文章最初发布在 **Django Stars 博客**上。

> 如果你觉得这篇文章有用，请点击👏下面的按钮:)