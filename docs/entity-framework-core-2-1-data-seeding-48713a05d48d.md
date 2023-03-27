# 实体框架核心 2.1:数据播种

> 原文：<https://itnext.io/entity-framework-core-2-1-data-seeding-48713a05d48d?source=collection_archive---------1----------------------->

![](img/5ae5e8477d3cd6e1d11bb5111d1e116a.png)

我花了一些时间来探索随推出的一些新功能。NET Core 2.1 发布，这篇文章将是这个过程的延续。以下是其他帖子的链接。

[作为 Windows 服务托管 ASP.NET 核心应用](https://elanderson.net/2018/09/host-asp-net-core-application-as-a-windows-service/)
[ASP.NET 核心 2.1:action result<T>](https://elanderson.net/2018/09/asp-net-core-2-1-actionresult/)

今天，我们将了解添加到实体框架中的一个新特性，它允许数据播种。我使用此功能的[官方文档](https://docs.microsoft.com/en-us/ef/core/modeling/data-seeding)作为参考。

## 示例应用程序

我们将使用。NET CLI 创建一个新的应用程序使用 MVC 模板与个人授权，因为它是一个模板来与实体框架已经成立。如果你有一个现有的项目，需要添加实体框架，你可以查看[这篇](https://elanderson.net/2018/04/add-entity-framework-core-to-an-existing-asp-net-core-project/)文章。以下是我用来创建项目的命令。

```
dotnet new mvc --auth Individual
```

## 模型

在我们开始数据播种之前，我们需要创建一个实体来播种。在本例中，我们将创建一个联系人实体(惊喜！).在 **Models** 目录下，我创建了一个 **Contacts.cs** 文件，内容如下。

```
public class Contact
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Address { get; set; }
    public string City { get; set; }
    public string State { get; set; }
    public string Zip { get; set; }
}
```

接下来，在**数据**目录中，我们将打开`ApplicationDbContext`类，并为我们的新联系人实体添加一个`DbSet`。将以下属性添加到类中。

```
public DbSet<Contact> Contacts { get; set; }
```

现在我们已经有了我们的`DbContext`设置，让我们使用下面的代码为这个新的联系实体添加一个迁移。NET CLI 命令。

```
dotnet ef migrations add Contacts -o Data/Migrations
```

最后，运行以下命令为该应用程序创建/更新数据库。

```
dotnet ef database update
```

## 数据播种

现在我们的项目已经设置好了，我们可以继续实际的数据播种了。在实体框架中，核心数据播种是在你的`DbContext`类的`OnModelCreating`函数中完成的。在本例中，这是`ApplicationDbContext`类。下面的例子展示了如何使用新的`HasData`方法为`Contact`实体添加种子数据。

```
protected override void OnModelCreating(ModelBuilder builder)
{
    base.OnModelCreating(builder);

    builder.Entity<Contact>().HasData(
        new Contact 
        {
            Id = 1,
            Name = "Eric",
            Address = "100 Main St",
            City = "Hometown",
            State = "TN",
            Zip = "153789"
        }
    );
}
```

数据播种是通过 Entity Framework Core 中的迁移来处理的，这与以前的版本有很大的不同。为了显示我们的种子数据，我们需要创建一个迁移，可以使用下面的命令来完成。

```
dotnet ef migrations add ContactSeedData -o Data/Migrations
```

然后使用以下命令将迁移应用到您的数据库。

```
dotnet ef database update
```

查看迁移创建的代码，您可以看到它只是插入数据。

```
public partial class ContactSeedData : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.InsertData(
            table: "Contacts",
            columns: new[] { "Id", "Address", "City", "Name", "State", "Zip" },
            values: new object[] { 1, "100 Main St", "Hometown", "Eric", "TN",
                                   "153789" });
    }

    protected override void Down(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.DeleteData(
            table: "Contacts",
            keyColumn: "Id",
            keyValue: 1);
    }
}
```

上面的方法在新数据库或新表中非常有效，但是如果您试图将种子数据添加到现有数据库中，就会出现问题。查看 [Rehan 的](https://twitter.com/RehanSaeedUK)关于[迁移到实体框架核心种子数据](https://rehansaeed.com/migrating-to-entity-framework-core-seed-data/)的帖子，了解如何处理这个问题。

## 包扎

我很高兴看到我们有了一种在实体框架核心中预先填充数据的方法，这将使一些场景(如大部分是静态数据)更容易处理。我认为迁移方法的一个缺点是不能有一些内置的测试数据，因为迁移将总是应用于您的数据库。

*原载于*[](https://elanderson.net/2018/09/entity-framework-core-2-1-data-seeding/)**。**