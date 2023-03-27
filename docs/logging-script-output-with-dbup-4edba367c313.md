# 使用 DbUp 记录脚本输出

> 原文：<https://itnext.io/logging-script-output-with-dbup-4edba367c313?source=collection_archive---------3----------------------->

在今天的帖子中，我们将讨论如何在 DbUp 运行期间记录脚本输出。如果你是第一次接触这个系列的文章或者 DbUp，你可能会发现下面的文章很有帮助。

[使用 DbUp 进行数据库迁移](https://elanderson.net/2020/08/database-migrations-with-dbup/)
[使用 DbUp 进行基于代码的数据库迁移](https://elanderson.net/2020/08/code-based-database-migrations-with-dbup/)
[始终使用 DbUp 进行迁移](https://elanderson.net/2020/08/always-run-migrations-with-dbup/)

![](img/557dcb46b7968b5c3440d106e7cc896d.png)

## 启用脚本输出日志记录

我将使用我们上周末讨论过的始终运行的迁移来简化测试，但是概念对于普通的迁移是相同的。启用脚本输出日志记录的更改是添加到我们的升级程序设置中的一行，您可以在下面的代码中看到突出显示的行。

```
var alwaysRunUpgrader =
    DeployChanges.To
                 .SqlDatabase(connectionString)
                 .WithScriptsAndCodeEmbeddedInAssembly(Assembly.GetExecutingAssembly(), 
                                                       f => f.Contains(".AlwaysRun."))
                 .JournalTo(new NullJournal())
                 .LogToConsole()
                 .LogScriptOutput()
                 .Build();
```

运行该应用程序将会产生如下结果。注意受影响的记录行，这是我们迁移的输出。

```
Beginning database upgrade
Checking whether journal table exists..
Fetching list of already executed scripts.
No new scripts need to be executed - completing.
Beginning database upgrade
Executing Database Server script 'DbupTest.Scripts.AlwaysRun.Everytime.sql'
RecordsAffected: 0

Upgrade successful
Success!
```

## 提高脚本输出

因此，以上对于只有一条语句的简单迁移来说非常好，但是对于更复杂的迁移，使用 **PRINT** 语句来提供更多信息可能会有所帮助。让我们调整我们的脚本来打印执行的开始和结束时间。以下是示例脚本(是的，我知道它实际上永远不会更新任何东西)。

```
PRINT CONVERT(VarChar, GETDATE(), 121)  + ' Start every time';

UPDATE Application.Cities SET CityName = 'Nothing' WHERE 1 = 0;

PRINT CONVERT(VarChar, GETDATE(), 121)  +  + ' End every time';
```

上述迁移会产生以下输出。

```
Beginning database upgrade
Checking whether journal table exists..
Fetching list of already executed scripts.
No new scripts need to be executed - completing.
Beginning database upgrade
Executing Database Server script 'DbupTest.Scripts.AlwaysRun.Everytime.sql'
2020-08-05 06:15:21.877 Start every time

2020-08-05 06:15:21.877 End every time

RecordsAffected: 0

Upgrade successful
Success!
```

打印是有帮助的，但是在这种情况下，受影响的记录是干扰，特别是因为它是受最后一条语句影响的记录，并且确实按照 SQL 语句打印。为了抑制受消息影响的记录，我们在脚本中使用 **SET NOCOUNT ON** ，如下所示。

```
SET NOCOUNT ON;

PRINT CONVERT(VarChar, GETDATE(), 121)  + ' Start every time with NoCount ON';

UPDATE Application.Cities SET CityName = 'Nothing' WHERE 1 = 0;

PRINT CONVERT(VarChar, GETDATE(), 121)  +  + ' End every time with NoCount ON';

SET NOCOUNT OFF;
```

正如你在下面看到的，我们得到了更清晰的输出。

```
Beginning database upgrade
Checking whether journal table exists..
Fetching list of already executed scripts.
No new scripts need to be executed - completing.
Beginning database upgrade
Executing Database Server script 'DbupTest.Scripts.AlwaysRun.Everytime.sql'
2020-08-05 06:38:18.863 Start every time

2020-08-05 06:38:18.863 End every time

Upgrade successful
Success!
```

## 包扎

记录脚本的输出有时真的很有帮助，但是如果你要像我们上面看到的那样使用它，确保和计划输出是什么样子是很重要的。有 500 个受影响的记录实例并不一定有帮助。

*最初发表于* [*埃里克·安德森*](https://elanderson.net/2020/08/logging-script-output-with-dbup/) *。*