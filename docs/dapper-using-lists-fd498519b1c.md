# Dapper 使用列表

> 原文：<https://itnext.io/dapper-using-lists-fd498519b1c?source=collection_archive---------7----------------------->

上周在[ASP.NET Core with Dapper](https://elanderson.net/2019/02/asp-net-core-with-dapper/)上的帖子涵盖了在 ASP.NET Core 项目中使用 Dapper 的基本知识。本周我们将修改上周的[样本代码，展示使用 Dapper 处理列表，而不是一次只处理一个对象。](https://github.com/elanderson/ASP.NET-Core-Dapper/tree/bbdd6da0eb88b6186035223de0b11724d8c29a5a)

![](img/e07fcff7a89c029339b5760db138c972.png)

这篇文章会很短，但是我认为有必要指出 Dapper 可以通过传递 IEnumerable 来多次执行一个查询。

## 样本

以下示例连接到数据库，删除所有现有的联系人，在 contact 表中植入 3 个联系人(这是我们的列表)，然后选择所有联系人(这作为 IEnumerable 出现)。

```
using (var connection = 
           new SqlConnection(_configuration
                             .GetConnectionString("DefaultConnection")))
{
    connection.Open();

    connection.Execute("DELETE Contacts");

    var seedContacts = new List<Contact>
                       {
                           new Contact
                           {
                               Name = "Charlie Plumber",
                               Address = "123 Main St",
                               City = "Nashville",
                               Subregion = "TN",
                               Email = "cplumber@fake.com"
                           },
                           new Contact
                           {
                               Name = "Teddy Pierce",
                               Address = "6708 1st St",
                               City = "Nashville",
                               Subregion = "TN",
                               Email = "tpierce@fake.com"
                           },
                           new Contact
                           {
                               Name = "Kate Pierce",
                               Address = "6708 1st St",
                               City = "Nashville",
                               Subregion = "TN",
                               Email = "kpierce@fake.com"
                           }
                       };

    connection.Execute(@"INSERT INTO Contacts (Name, Address, City, 
                                               Subregion, Email) 
                         VALUES (@Name, @Address, @City, 
                                 @Subregion, @Email)",
                       seedContacts);

    var allContacts = connection
                      .Query<Contact>(@"SELECT Id, Name, Address, City, 
                                               Subregion, Email 
                                        FROM Contacts");
}
```

正如所料，最终结果包含三个联系人记录。

## 包扎

希望这篇简短的帖子有助于填补上周帖子的一个空白。确保并查看 [Dapper GitHub 页面](https://github.com/StackExchange/Dapper)除了我在这两篇文章中提到的特性，它还支持许多其他功能，如存储过程、多结果集等。最终状态的代码可以在这里找到。

*最初发表于* [*埃里克·安德森*](https://elanderson.net/2019/02/dapper-using-lists/) *。*