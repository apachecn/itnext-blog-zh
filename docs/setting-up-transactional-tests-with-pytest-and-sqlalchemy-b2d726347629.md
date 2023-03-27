# 使用 Pytest 和 SQLAlchemy 设置事务性测试

> 原文：<https://itnext.io/setting-up-transactional-tests-with-pytest-and-sqlalchemy-b2d726347629?source=collection_archive---------1----------------------->

![](img/8965f6170adc861cad6d91b079dc8159.png)

照片由[克里斯里德](https://unsplash.com/@cdr6934?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 拍摄

来自 Ruby on Rails 的背景，我非常欣赏那些成为解决问题的社区标准的解决方案。其中一个就是 [DatabaseCleaner gem](https://github.com/DatabaseCleaner/database_cleaner) ，它可以确保你的测试是分开运行的，并且它们之间没有数据泄漏。

最近我在寻找一个类似的 Python 解决方案，但是，令我惊讶的是，我没有找到任何解决方案。然而，很容易利用 SQLAlchemy 和 Pytest 提供的功能将测试包装在单独的数据库事务中。让我向你展示一个简洁的解决问题的方法，希望你会觉得用起来很方便。

# 设置数据库连接

在我们能够在测试中使用包含事务的数据库之前，我们需要专门为测试设置一个单独的 DB 实例。然后，从我们的测试套件中，我们需要连接到数据库。以下是如何与 MySQL 数据库建立连接的示例:

如果您使用另一个数据库引擎，请前往 [SQLAlchemy 文档](https://docs.sqlalchemy.org/en/13/core/engines.html)获取有关如何构建不同连接字符串的信息。

# 表创建和数据库播种

现在我们需要重新创建我们的数据库结构。让我们假设您的应用程序中的所有模型都在`models.py`文件中声明。

SQLAlchemy 提供了一些方法来轻松地创建和删除模式中声明的表:`create_all`和`drop_all`。我们将在测试套件执行的开始使用它们，以确保所有的表都在适当的位置。在完整的测试运行之后，我们将删除所有的表，以便下一次执行可以从头开始。

如果您需要用一些数据预先配置数据库，您可以运行一个方法来播种数据库。一个简单的例子如下:

# 交易中的包装测试

作为最后一步，我们需要建立一种在测试套件中使用事务的方法。因此，我们将构建一个夹具，为每个测试创建一个新的事务。

然后，您可以将 fixture 注入到您的测试用例中。在每个测试执行的最后，所有创建的数据都将被清除，确保测试用例的分离。

# 摘要

一旦您知道如何做，用 Pytest 和 SQLAlchemy 设置工作事务并不需要太多时间。我希望你会发现这个解决方案简洁易用。当然，如果你有任何想法要分享或提出改进，请在评论中告诉我——我总是很乐意学习新的东西。

作为将来的参考，下面是一个完整的代码，您可以在您的项目中重用。祝你好运，并享受用事务编写测试的乐趣！