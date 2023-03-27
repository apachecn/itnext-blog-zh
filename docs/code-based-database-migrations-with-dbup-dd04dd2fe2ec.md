# 使用 DbUp 进行基于代码的数据库迁移

> 原文：<https://itnext.io/code-based-database-migrations-with-dbup-dd04dd2fe2ec?source=collection_archive---------0----------------------->

在今天的帖子中，我们将了解 DbUp 基于代码的迁移特性，该特性允许代码作为迁移过程的一部分运行，而不仅仅是基于 SQL 的脚本。作为迁移的一部分运行代码的能力为迁移过程提供了极大的灵活性。这是基于上周的文章[使用 DbUp 进行数据库迁移](https://elanderson.net/2020/08/database-migrations-with-dbup/)，如果您是 DbUp 的新手，请务必查看一下。

![](img/d6b2efb9fb5cd3edf98cf9fde205f97b.png)

## 脚本提供程序更改

在我们上周在**程序**类中发布的示例应用程序中，当设置我们的**升级程序**时，我们使用了以下设置。

```
var upgrader =
    DeployChanges.To
                 .SqlDatabase(connectionString)
                 .WithScriptsEmbeddedInAssembly(Assembly.GetExecutingAssembly())
                 .LogToConsole()
                 .Build();
```

上面只选取了嵌入的 SQL 类型文件。为了能够进行基于代码的迁移，我们必须从**withscriptsbembeddedinassembly**脚本提供程序更改为**WithScriptsAndCodeEmbeddedInAssembly**。下面是突出显示了新脚本提供程序的升级程序。

```
var upgrader =
    DeployChanges.To
                 .SqlDatabase(connectionString)
                 .WithScriptsAndCodeEmbeddedInAssembly(Assembly.GetExecutingAssembly())
                 .LogToConsole()
                 .Build();
```

## 基于代码的迁移

要添加基于代码的迁移，请添加一个新类，并将其设置为实现 DbUp 的 **IScript** 接口。我们在上面更改的脚本提供者将获取任何实现了 **IScript** 以及 SQL 脚本文件的非抽象类。结果按照名称进行组合和排序，以确定执行顺序，因此要确保您选择的任何命名约定都能够正确排序 SQL 脚本和代码。

IScript 接口定义了一个函数 ProvideScript，该函数有一个参数，该参数提供了一个创建数据库命令并返回字符串的函数。根据需要，您可以返回一个包含要执行的 SQL 的字符串，或者您可以创建一个命令并直接在数据库上执行您需要的任何命令，或者根据需要将两者结合使用。

为了测试基于代码的迁移，我在 Scripts 文件夹中添加了以下新类。它实际上不会更改任何数据，而是将一些信息记录到控制台。即使不改变数据，它仍然传达了基于代码的迁移是如何工作的。

```
using System;
using System.Data;
using DbUp.Engine;

namespace DbupTest.Scripts
{
    public class Code0002Test : IScript
    {
        public string ProvideScript(Func<IDbCommand> dbCommandFactory)
        {
            using (var cmd = dbCommandFactory())
            {
                cmd.CommandText = "SELECT DeliveryMethodName FROM Application.DeliveryMethods";

                using (var dr = cmd.ExecuteReader())
                {
                    Console.WriteLine("  Delivery Methods");

                    while (dr.Read())
                    {
                        Console.WriteLine($"    {dr["DeliveryMethodName"]}");
                    }
                }
            }

            return string.Empty;
        }
    }
}
```

以下是运行该应用程序的控制台输出。

```
Beginning database upgrade
Checking whether journal table exists..
Fetching list of already executed scripts.
  Delivery Methods
    Air Freight
    Chilled Van
    Courier
    Customer Collect
    Customer Courier to Collect
    Delivery Van
    Post
    Refrigerated Air Freight
    Refrigerated Road Freight
    Road Freight
Executing Database Server script 'DbupTest.Scripts.Code0002Test.cs'
Checking whether journal table exists..
Upgrade successful
Success!
```

## 包扎

在数据库迁移过程中运行代码的能力是一组非常棒的功能。我建议将基于 SQL 的迁移作为默认设置，只有当 SQL 不能提供完成所需任务的方法或者用代码进行迁移更容易理解时，才使用代码。确保并查看关于此功能的[官方 DbUp 文档](https://dbup.readthedocs.io/en/latest/usage/#code-based-scripts)以了解更多信息。

*原载于*[](https://elanderson.net/2020/08/code-based-database-migrations-with-dbup/)**。**