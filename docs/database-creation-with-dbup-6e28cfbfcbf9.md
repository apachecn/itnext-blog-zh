# 使用 DbUp 创建数据库

> 原文：<https://itnext.io/database-creation-with-dbup-6e28cfbfcbf9?source=collection_archive---------2----------------------->

这将是一篇简短的帖子，展示如何配置 DbUp，以便在我们过去讨论过的迁移功能的基础上创建一个新的数据库。使用下面的帖子来了解我们过去讨论过的 DbUp 主题。

[使用 DbUp 进行数据库迁移](https://elanderson.net/2020/08/database-migrations-with-dbup/)
[使用 DbUp 进行基于代码的数据库迁移](https://elanderson.net/2020/08/code-based-database-migrations-with-dbup/)
[始终使用 DbUp 运行迁移](https://elanderson.net/2020/08/always-run-migrations-with-dbup/)
[使用 DbUp 输出日志脚本](https://elanderson.net/2020/08/logging-script-output-with-dbup/)
[使用 DbUp 进行事务中的迁移](https://elanderson.net/2020/09/migrations-in-transactions-with-dbup/)

![](img/acc93938a68015d19f1c30925e8e29f1.png)

## 数据库创建

创建数据库的代码是一行程序，只需要一个连接字符串，应该在任何迁移之前运行。以下是我们一直在使用的示例应用程序的 **Main** 函数的一部分，添加并突出显示了 **EnsureDatabase** 行，这是负责数据库创建的代码。

```
static int Main(string[] args)
{
    var connectionString =
        args.FirstOrDefault()
        ?? "Server=localhost; Database=WideWorldImporters; Trusted_connection=true";

    EnsureDatabase.For.SqlDatabase(connectionString);

    var upgrader =
        DeployChanges.To
                     .SqlDatabase(connectionString)
                     .WithScriptsAndCodeEmbeddedInAssembly(Assembly.GetExecutingAssembly(),
                                                           f => !f.Contains(".AlwaysRun."))
                     .LogToConsole()
                     .WithTransaction()
                     .Build();

    var result = upgrader.PerformUpgrade();
```

## 包扎

如您所见，在 DbUp 应用程序中添加创建数据库的功能非常简单。[官方 DbUp 文档中的数据库创建](https://dbup.readthedocs.io/en/latest/)只是一个注释，很容易被忽略，所以我想写一个帖子来指出它，因为当你需要它时，它是一个强大的功能。在我们的示例中，如果数据库不存在，应用程序将总是尝试创建数据库，但是如果这不符合您的用例，那么可能值得将数据库创建包含在一个参数中，以便用户可以决定是否应该创建数据库。

*原载于* [*埃里克·安德森*](https://elanderson.net/2020/10/database-creation-with-dbup/) *。*