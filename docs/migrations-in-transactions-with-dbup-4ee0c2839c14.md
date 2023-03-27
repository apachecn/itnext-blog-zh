# 使用 DbUp 在事务中迁移

> 原文：<https://itnext.io/migrations-in-transactions-with-dbup-4ee0c2839c14?source=collection_archive---------5----------------------->

这篇文章将继续我们对 DbUp 的探索，并在迁移执行中添加事务。如果您是 DbUp 的新手，可以看看下面的帖子来了解这个系列。

[使用 DbUp 进行数据库迁移](https://elanderson.net/2020/08/database-migrations-with-dbup/)
[使用 DbUp 进行基于代码的数据库迁移](https://elanderson.net/2020/08/code-based-database-migrations-with-dbup/)
[始终使用 DbUp 运行迁移](https://elanderson.net/2020/08/always-run-migrations-with-dbup/)
[使用 DbUp 输出日志脚本](https://elanderson.net/2020/08/logging-script-output-with-dbup/)

![](img/890c25520b5057cac99acdd86fc80b95.png)

## 交易类型和限制

DbUp 有三个不同的事务选项。缺省值是 no transaction，这是我们到目前为止一直使用的。其他两个选项是每次迁移一个事务，最后是在单个升级程序中对所有迁移执行一个事务。据我所知，目前还没有办法在多个升级者之间使用相同的事务，这意味着在我们的示例应用程序中，我们可以让正常的迁移正常工作并提交，但在我们始终运行的迁移中会出现问题。要记住的另一件事是，当改变数据库的结构时，一些数据库提供者不支持事务。正如 DbUp 文档所指出的，Postgres 文档很好地总结了不同数据库提供者对 DDL 的事务支持。

## 添加交易

在下面的示例中，我们已经将升级程序更改为使用单个事务。这是升级程序的完整代码，突出显示了更改。

```
var upgrader =
    DeployChanges.To
                 .SqlDatabase(connectionString)
                 .WithScriptsAndCodeEmbeddedInAssembly(Assembly.GetExecutingAssembly(),
                                                       f => !f.Contains(".AlwaysRun."))
                 .LogToConsole()
                 .WithTransaction()
                 .Build();
```

其他选项有**with transaction perscript**用于每次迁移处理一个事务，以及**with not transaction**用于不处理任何事务(或者忽略它，因为这是默认值)。下面是两个不同的升级者进行事务处理时的输出示例，突出显示了 begin 事务处理。

```
Beginning transaction
Beginning database upgrade
Checking whether journal table exists..
Fetching list of already executed scripts.
No new scripts need to be executed - completing.
Beginning transaction
Beginning database upgrade
Executing Database Server script 'DbupTest.Scripts.AlwaysRun.01-EverytimeNoPrints.sql'
Executing Database Server script 'DbupTest.Scripts.AlwaysRun.02-EverytimeNoPrintsNoCountOn.sql'
Executing Database Server script 'DbupTest.Scripts.AlwaysRun.03-EverytimePrints.sql'
Executing Database Server script 'DbupTest.Scripts.AlwaysRun.04-EverytimePrintNoCountOn.sql'
Upgrade successful
Success!
```

## 包扎

DbUp 使得向迁移中添加事务变得非常简单，所以这是一篇非常短的文章。如果有一个跨多个升级者使用事务的选项就好了。关于交易的官方 DbUp 文档可以在[这里](https://dbup.readthedocs.io/en/latest/more-info/transactions/)找到。

*原载于*[](https://elanderson.net/2020/09/migrations-in-transactions-with-dbup/)**。**