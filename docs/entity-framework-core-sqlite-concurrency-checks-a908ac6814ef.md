# 实体框架核心:SQLite 并发检查

> 原文：<https://itnext.io/entity-framework-core-sqlite-concurrency-checks-a908ac6814ef?source=collection_archive---------2----------------------->

我使用 SQLite 所做的大部分工作都是在单用户系统上进行的，但是最近我不得不从事一个 SQLite 项目，该项目将有少量并发用户，并且用户活动的子集将需要处理并发问题。过去，在这种情况下，我一直使用 SQL Server 并使用 [rowversion 或 timestamp column type](https://docs.microsoft.com/en-us/sql/t-sql/data-types/rowversion-transact-sql?view=sql-server-2017) ,它在任何更新或插入的行上放置一个唯一值。

![](img/d4260c01ebc3ea36b635d5c327cd29d1.png)

由[凯文·Ku](https://unsplash.com/@ikukevk?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

在官方文档中有一页是关于[并发令牌](https://docs.microsoft.com/en-us/ef/core/modeling/concurrency)的，但是对我来说，这并不是很有帮助。谢天谢地，经过一番搜索后，我在 ASP.Net Core 2 . x 中遇到了 GitHub 问题[，带有实体框架核心，并发控制无法与 SQLite](https://github.com/aspnet/EntityFrameworkCore/issues/12260) 一起工作，其中一个回复中有一个可靠的示例。这篇文章将介绍该示例的一个示例实现。代码的起点可以在这个 GitHub repo 中找到[。](https://github.com/elanderson/ASP.NET-Core-Entity-Framework/tree/e2dc7bcf15f40e35f0f73dc267fadddf4431ae94)

## 样本背景

正在使用的示例项目是一个管理联系人列表的简单 web 应用程序。repo 包含一个使用 Postgres 的实现和一个使用 Sqlite 的实现。这整个职位将只是触摸文件中找到的 Sqlite 文件夹/项目。

## 模型变更和数据迁移

SQLite 没有时间戳列的概念，但是这个解决方案将模拟时间戳列。为此，我们将更改在**模型**文件夹中找到的**联系人**模型。我们将添加一个带有**时间戳**数据注释的**时间戳**属性。下面是底部带有新属性的完整模型类。

```
public class Contact
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Address { get; set; }
    public string City { get; set; }
    public string Subregion { get; set; }
    public string PostalCode { get; set; }
    public string Phone { get; set; }
    public string Email { get; set; }
    [Timestamp] public byte[] Timestamp { get; set; }
}
```

接下来，让我们创建一个新的迁移，并对模型进行更改。我在用。因此，从项目目录中的命令提示符处运行以下命令。

```
dotnet ef migrations add ContactTimestamp --context ContactsDbContext
```

在**迁移**目录中，打开新创建的迁移。应该命名为 ***_ContactTimestamp.cs** 之类的。在 **Up** 函数中，我们将向新的时间戳列添加几个触发器。当插入或更新一行时，这些触发器将为 timestamp 列分配一个随机 blob，这就是我们模拟 SQL Server Timestamp 数据类型功能的方式。以下是添加了触发器的完整 **Up** 功能。

```
protected override void Up(MigrationBuilder migrationBuilder)
{
    migrationBuilder.AddColumn<byte[]>(
        name: "Timestamp",
        table: "Contacts",
        rowVersion: true,
        nullable: true);

    migrationBuilder.Sql(
        @"
        CREATE TRIGGER SetContactTimestampOnUpdate
        AFTER UPDATE ON Contacts
        BEGIN
            UPDATE Contacts
            SET Timestamp = randomblob(8)
            WHERE rowid = NEW.rowid;
        END
    ");

    migrationBuilder.Sql(
        @"
        CREATE TRIGGER SetContactTimestampOnInsert
        AFTER INSERT ON Contacts
        BEGIN
            UPDATE Contacts
            SET Timestamp = randomblob(8)
            WHERE rowid = NEW.rowid;
        END
    ");
}
```

要将迁移应用到数据库，您可以使用以下命令。

```
dotnet ef database update --context ContactsDbContext
```

## 测试它

现在进行一个快速的测试，我们将为现有的 **ContactsController** 添加一个**并发测试**函数。这个函数将确保特定的 contact 存在，然后从两个不同的 DBContexts 中提取 contact，对产生的 contact 对象进行变异，然后尝试保存。第一次保存会成功，第二次应该会失败。请注意，这个函数不是一个应该如何做的例子，只是一个快速和肮脏的方法来证明并发检查正在发生。

```
[Route("ConcurrencyTest")]
public void ConcurrencyTest()
{
    var context1 = new ContactsDbContext(new DbContextOptionsBuilder<ContactsDbContext>()
                                         .UseSqlite("Data Source=Database.db").Options);
    var context2 = new ContactsDbContext(new DbContextOptionsBuilder<ContactsDbContext>()
                                         .UseSqlite("Data Source=Database.db").Options);

    var contactFromContext1 = context1.Contacts.FirstOrDefault(c => c.Name == "Test");

    if (contactFromContext1 == null)
    {
        contactFromContext1 = new Contact
        {
            Name = "Test"
        };

        context1.Add(contactFromContext1);
        context1.SaveChanges();
    }

    var contactFromContext2 = context2.Contacts.FirstOrDefault(c => c.Name == "Test");

    contactFromContext1.Address = DateTime.Now.ToString();
    contactFromContext2.Address = DateTime.UtcNow.ToString();

    context1.SaveChanges();
    context2.SaveChanges();
}
```

运行应用程序并点击**concurrency test**路径，对于我的测试来说是[http://localhost:1842/concurrency test](http://localhost:1842/ConcurrencyTest)。下面是产生的异常。

# 处理请求时出现未处理的异常。

> DbUpdateConcurrencyException:数据库操作预期影响 1 行，但实际影响了 0 行。自实体加载后，数据可能已被修改或删除。参见[http://go.microsoft.com/fwlink/?LinkId=527962](http://go.microsoft.com/fwlink/?LinkId=527962)了解和处理乐观并发异常的信息。

## 包扎

虽然信息并不容易找到，但正如您所见，使用 SQLite 的 Entity Framework Core 对并发控制有很好的支持。以上只是其实现的一种选择。我希望这能节省你们的时间。

最终状态的代码可以在这里找到[。](https://github.com/elanderson/ASP.NET-Core-Entity-Framework/tree/d1415d23f00117e3fb727d428dbdd7664ee17a0d)

*原载于* [*埃里克·安德森*](https://elanderson.net/2018/12/entity-framework-core-sqlite-concurrency-checks/) *。*