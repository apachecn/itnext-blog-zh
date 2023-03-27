# 。SQL Server 临时表的. NET 参数化查询问题

> 原文：<https://itnext.io/net-parameterized-queries-issues-with-sql-server-temp-tables-f5ee4f304523?source=collection_archive---------3----------------------->

在过去几周的工作中，我遇到了许多人在。NET 中包含一个临时表。经过一番挖掘，我们终于找到了问题所在。这篇文章将会讨论这个问题的原因以及解决的方法。本文中使用的数据库是微软的全球进口商样本数据库。关于设置它的说明，请查看我的[从上周开始获取一个 SQL Server 数据库示例](https://elanderson.net/2018/09/getting-a-sample-sql-server-database/)的帖子。

## 示例项目创建

为了使事情尽可能简单，我使用了一个控制台应用程序。NET CLI 命令。

```
dotnet new console
```

后面跟着这个命令从 Nuget 添加到 SQL 客户端中。

```
dotnet add package System.Data.SqlClient
```

下面是完整的`Program`类和样本代码，它将导致本文所处理的异常。是的，我知道这不是构建这种类型代码的最好方式，所以请不要从这个角度来判断。这意味着演示问题很简单。

```
class Program
{
    static void Main(string[] args)
    {
        Console.WriteLine("Running sample");

        using (var connection = 
                  new SqlConnection(@"Data Source=YourServer;
                                      Initial Catalog=YourDatabase;
                                      Integrated Security=SSPI;"))
        {
            connection.Open();

            using (var command = connection.CreateCommand())
            {
                SqlTest(command);
            }
        }

        Console.ReadLine();
    }

    private static void SqlTest(SqlCommand command)
    {
        command.CommandText = @"SELECT OrderId
                                      ,CustomerId
                                      ,SalespersonPersonID
                                      ,BackorderOrderId
                                      ,OrderDate
                                INTO #backorders
                                FROM Sales.Orders
                                WHERE BackorderOrderID IS NOT NULL
                                  AND OrderDate > @OrderDateFilter";

        command.Parameters.Add("@OrderDateFilter", 
                                SqlDbType.DateTime)
                          .Value = DateTime.Now.AddYears(-1);
        command.ExecuteNonQuery();

        command.CommandText = "SELECT OrderId FROM #backorders";

        using (var reader = command.ExecuteReader())
        {
            while (reader.Read())
            {
                Console.WriteLine(reader["OrderId"]);
            }
        }
    }
}
```

## 错误

运行上面存在的应用程序将导致以下错误。

> 无效的对象名' #backorders '。

奇怪的错误，因为我们刚刚创建了**#缺货订单**临时表。让我们试一试没有过滤器。查询现在看起来如下所示。

```
command.CommandText = @"SELECT OrderId
                              ,CustomerId
                              ,SalespersonPersonID
                              ,BackorderOrderId
                              ,OrderDate
                        INTO #backorders
                        FROM Sales.Orders
                        WHERE BackorderOrderID IS NOT NULL";

command.ExecuteNonQuery();
```

现在应用程序运行没有任何问题。如果我们尝试添加回过滤器，但不使用命令参数会怎么样？

```
command.CommandText = @"SELECT OrderId
                              ,CustomerId
                              ,SalespersonPersonID
                              ,BackorderOrderId
                              ,OrderDate
                        INTO #backorders
                        FROM Sales.Orders
                        WHERE BackorderOrderID IS NOT NULL
                          AND OrderDate > '2018-01-01'";

command.ExecuteNonQuery();
```

同样，应用程序运行没有任何问题。

## 原因

为什么添加命令参数会导致我们的临时表消失？我通过使用 SQL Server Profiler 发现了这个问题(在 SQL Server Management Studio 中，可以在工具> SQL Server Profiler 中找到它)。代码返回到原始版本，命令参数和探查器连接到运行示例应用程序的同一服务器，示例应用程序显示了 SQL Server 收到的以下命令。

```
exec sp_executesql N'SELECT OrderId 
                     FROM #backorders',
                   N'@OrderDateFilter datetime',
                   @OrderDateFilter='2017-08-28 06:41:37.457'
```

事实证明，当您在。NET 中，它在 SQL Server 上使用 [sp_executesql](https://docs.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/sp-executesql-transact-sql) 存储过程来执行。这是我之前遗漏的关键信息。现在我知道参数化查询是在存储过程的范围内执行的，这也意味着我们第一个查询中使用的临时表仅限于在创建它的存储过程中使用。

![](img/b778a82554aa39768cb827f193bee5b7.png)

由[卡斯帕·鲁宾](https://unsplash.com/@casparrubin?utm_source=medium&utm_medium=referral)上 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)

## 要修复的选项

第一个选项是在初始数据拉取时不使用参数。我不推荐这个选项。参数提供了我们不想失去的保护级别。

第二个选项和我们解决这个问题的方法是首先创建临时表。现在，临时表已经在存储过程之外创建了，它的作用域是连接，然后允许我们使用参数插入数据。下面的代码是我们使用这种策略的示例。

```
command.CommandText = @"CREATE TABLE #backorders
                        (
                           OrderId int
                          ,CustomerId int
                          ,SalespersonPersonID int
                          ,BackorderOrderID int
                          ,OrderDate date
                        )";

command.ExecuteNonQuery();

command.CommandText = @"INSERT INTO #backorders
                        SELECT OrderId
                              ,CustomerId
                              ,SalespersonPersonID
                              ,BackorderOrderId
                              ,OrderDate
                        FROM Sales.Orders
                        WHERE BackorderOrderID IS NOT NULL
                          AND OrderDate > @OrderDateFilter";

command.Parameters.Add("@OrderDateFilter", 
                       SqlDbType.DateTime)
                  .Value = DateTime.Now.AddYears(-1);
command.ExecuteNonQuery();
```

## 包扎

我希望这能节省一些时间。一旦我明白了这个问题的意义所在。怎么会。NET 处理带参数的 SQL 命令是一直有效的事情之一，直到现在我才需要深入研究。总是有新的东西要学，这也是我热爱我的工作的原因之一。

*原载于*[](https://elanderson.net/2018/10/net-parameterized-queries-issues-with-sql-server-temp-tables/)**。**