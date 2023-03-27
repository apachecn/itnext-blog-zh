# 减少 REST APIs 上的带宽和请求数量

> 原文：<https://itnext.io/reducing-bandwidth-and-number-of-requests-on-rest-apis-ec418b5fd82a?source=collection_archive---------0----------------------->

![](img/a2e0e54c7c11ca44f8a5f927e8906b58.png)

REST APIs 已经成为软件行业的标准，任何参与其中的人都为之开发或与之交互。虽然出现了其他选项，如 GraphQL，但它们通常更复杂，需要不同的工具和不同的方法，不适合许多用例。因此，REST 是事实上的标准。

尽管如此，REST APIs 也有缺点。当我们请求一个特定的资源时，我们通常不能选择我们想要检索的字段，相反，我们得到所有可用的字段，尽管我们只需要几个。

```
GET /users/10
---
{
    "id": 10,
    "name": "John Doe",
    "dob": "1990-01-23",
    "phoneNumber": "55000000000",
    "email": "[johndoe@email.com](mailto:johndoe@email.com)",
    "address": {
        "street": "fake road 121",
        "city": "London",
        "country": "United Kingdom"
    },
    "preferences": {
        ...
    }
}
```

另一个问题是，如果您想获得与第一个请求相关的额外数据，您需要做一个新的请求来收集它，这可以快速累积我们的应用程序发出的请求的数量。

```
GET /users/10/teams
---
[{
    "id": 13,
    "name": "Team A"
},
{
    "id": 18,
    "name": "Team B"
}]
```

为了解决这个问题，编写了 [REST 模式](https://github.com/goncalo-oliveira/rest-schema-spec)规范。它描述了一组扩展，允许我们准确地告诉 API 我们希望从请求中得到什么。其思想是，它的实现是通用的、一致的，跨越整个 API，而不是特定于某个特定的资源。

规范本身是语言不可知的，因为它描述了一个可以在多种编程语言或框架中实现的概念。在编写[时，REST 模式](https://github.com/goncalo-oliveira/rest-schema-spec)规范在版本`0.1`中。

# 模式映射

假设我们想要检索一个用户列表，但是我们只对用户的`id`和`name`感兴趣，其他的都不感兴趣。如果不使用`Schema-Mapping`，请求将类似于

```
GET /users
```

这种反应可能会给我们提供比我们需要的更多的信息

```
[{
    "id": 10, 
    "name": "John Doe",
    "dob": "1990-01-23",
    "phoneNumber": "55000000000",
    "email": "[johndoe@email.com](mailto:johndoe@email.com)"
},
{
    "id": 10, 
    "name": "John Doe",
    "dob": "1990-01-23",
    "phoneNumber": "55000000000",
    "email": "[johndoe@email.com](mailto:johndoe@email.com)"
}]
```

为了定义我们想要接收的数据，我们需要用 JSON 或 YAML 编写一个模式规范。我将使用 JSON，因为产生的 base64 更小(如果先缩小，甚至更小)。

```
{
    "spec": {
        "_": ["id", "name"]
    }
}
```

然后，这个模式被编码为 base64 或 base64Url 字符串，并在`X-Schema-Map`头中传递。我将再次使用 base64Url，因为结果字符串更小(不需要在末尾填充`=`)。

```
GET /users
X-Schema-Map: eyJzcGVjIjp7Il8iOlsiaWQiLCJuYW1lIl19fQ
```

这将给我们以下小得多的响应

```
[{
    "id": 10, 
    "name": "John Doe"
},
{
    "id": 10, 
   "name": "John Doe"
}]
```

现在，假设原始示例没有带来那么多数据，但它减少了 70%,任何使用过真实场景的人都知道，这可以大幅减少带宽，提高请求速度，甚至提高客户端的数据处理速度。

# 架构-包含

现在，假设我们想要检索特定用户的数据，但是我们还想要检索他所属的团队。正如我们前面看到的，这有两种不同的资源。首先我们会得到用户

```
GET /users/10
---
{
    "id": 10, 
    "name": "John Doe",
    "dob": "1990-01-23",
    "phoneNumber": "55000000000",
    "email": "[johndoe@email.com](mailto:johndoe@email.com)"
}
```

然后是用户的团队

```
GET /users/10/teams
---
[{
    "id": 13,
    "name": "Team A"
},
{
    "id": 18,
    "name": "Team B"
}]
```

我们可以通过使用`Schema-Mapping`来实现这一点，并定义我们想要从用户那里检索的所有属性，包括团队。然而，因为我们想从用户那里获得所有的默认数据，加上团队，我们可以定义额外的属性来检索默认属性之外的内容。

```
{
    "spec": {
        "_": ["teams"]
    }
}
```

和以前一样，我们将模式编码为 base64Url 字符串，但是这次我们在`X-Schema-Include`头中传递它。

```
GET /users/10
X-Schema-Include: eyJzcGVjIjp7Il8iOlsidGVhbXMiXX19
```

只需一个请求，我们就能获得用户的所有详细信息

```
{
    "id": 10, 
    "name": "John Doe",
    "dob": "1990-01-23",
    "phoneNumber": "55000000000",
    "email": "[johndoe@email.com](mailto:johndoe@email.com)",
    "teams": [{
        "id": 13,
        "name": "Team A"
    },
    {
        "id": 18,
        "name": "Team B"
    }]
}
```

# 铆钉铆合

使用相同的标准 REST APIs，有可能提出减少带宽和应用程序需要处理的请求数量的策略。 [REST 模式](https://github.com/goncalo-oliveira/rest-schema-spec)规范试图描述这一点，我相信这是一个有趣的想法，尽管获得您的反馈会很好，所以请随意留下您的评论。

对于那些在. NET 环境中工作的人来说，ASP.NET 已经有了一个[c#实现，而且功能齐全，尽管你应该记住它仍然是一项正在进行的工作。如果你是喜欢冒险的人，请随意看看或者贡献自己的一份力量。](https://github.com/goncalo-oliveira/rest-schema-aspnet)