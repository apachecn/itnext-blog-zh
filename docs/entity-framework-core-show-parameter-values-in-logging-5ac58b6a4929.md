# 实体框架核心:在日志中显示参数值

> 原文：<https://itnext.io/entity-framework-core-show-parameter-values-in-logging-5ac58b6a4929?source=collection_archive---------1----------------------->

早在 10 月，我发表了一篇关于[实体框架核心:日志记录](https://elanderson.net/2018/10/entity-framework-core-logging/)的文章，其中涵盖了为实体框架核心启用日志记录。这篇文章将对上一篇文章进行扩展，向您展示除了正在使用的 SQL 语句之外，如何获取查询中使用的参数值。

![](img/f1157a16bc9d550aec1db62888ea13c0.png)

# 回顾

这篇文章不会涉及如何设置一个记录器，详情请见[这篇](https://elanderson.net/2018/10/entity-framework-core-logging/)前一篇文章。以下是基于前一篇文章在日志中获得的信息。

```
Microsoft.EntityFrameworkCore.Database.Command[20101] Executed DbCommand (41ms) [Parameters=[@__id_0='?' (DbType = Int32)], CommandType='Text', CommandTimeout='30'] SELECT TOP(2) [m].[Id], [m].[Address], [m].[City], [m].[Email], [m].[Name], [m].[Phone], [m].[PostalCode], [m].[State] FROM [Contact] AS [m] WHERE [m].[Id] = @__id_0
```

当您需要验证实体框架产生的查询时，这是很有帮助的，但是如果您试图跟踪一个与数据相关的问题，不知道使用了什么参数值可能会很痛苦。

# 启用日志记录中的参数值

实体框架核心提供了一个选项[启用敏感数据记录](https://docs.microsoft.com/en-us/dotnet/api/microsoft.entityframeworkcore.dbcontextoptionsbuilder.enablesensitivedatalogging)。要启用此选项，请打开 **Startup** 类，并在 **ConfigureServices** 函数中，对您想要打开选项的 **DbContext** 的 **AddDbContext** 调用进行以下更改。

```
Before: services .AddDbContext<ContactsContext>(options => options.UseSqlServer(Configuration["Data:ContactsContext:ConnectionString"])); After: services .AddDbContext<ContactsContext>(options => { options.UseSqlServer(Configuration["Data:ContactsContext:ConnectionString"]); options.EnableSensitiveDataLogging(); });
```

现在运行与上面相同的查询，您将看到下面的输出。

```
Microsoft.EntityFrameworkCore.Database.Command[20100] Executing DbCommand [Parameters=[@__id_0='1' (Nullable = true)], CommandType='Text', CommandTimeout='30'] SELECT TOP(2) [m].[Id], [m].[Address], [m].[City], [m].[Email], [m].[Name], [m].[Phone], [m].[PostalCode], [m].[State] FROM [Contact] AS [m] WHERE [m].[Id] = @__id_0
```

如您所见，使用启用敏感数据记录，我们获得了 ID 参数的值，而不是一个问号。

```
Before: [Parameters=[@__id_0='?' (DbType = Int32)] After: [Parameters=[@__id_0='1' (DbType = Int32)]
```

# 包扎

当你需要时，这是一个方便的选择。请小心使用它，因为它可能会暴露应该保密的数据。

*原载于 2019 年 3 月 10 日*[*elanderson.net*](https://elanderson.net/2019/03/entity-framework-core-show-parameter-values-in-logging/)*。*