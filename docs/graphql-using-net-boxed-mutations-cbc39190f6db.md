# 使用 GraphQL。NET Boxed:突变

> 原文：<https://itnext.io/graphql-using-net-boxed-mutations-cbc39190f6db?source=collection_archive---------2----------------------->

![](img/ec0b04437a7edcd3e26a43b7f02b0450.png)

这篇文章将继续我使用[对](https://github.com/Dotnet-Boxed/Templates) [GraphQL](https://graphql.org/) 的探索。NET Boxed 模板作为跳开点。我开始写的代码可以在[这里](https://github.com/elanderson/ASP.NET-Core-GraphQl/tree/f8a353800ab5006c78b5445cc0204cb66f367147)找到。使用查看 [GraphQL。NET Boxed: Queries](https://elanderson.net/2018/07/graphql-using-net-boxed-queries/) 从上周开始探索查询。

突变是 GraphQL 允许客户端请求更改服务器端数据的方式。

## 出发点

正如我们上周发现的，MainSchema 是发现 GraphQL 如何在这个模板中设置的中心点。仅供参考，这里是完整的类。

```
public class MainSchema : Schema
{
    public MainSchema(
        QueryObject query,
        MutationObject mutation,
        SubscriptionObject subscription,
        IDependencyResolver resolver)
        : base(resolver)
    {
        this.Query = resolver.Resolve<QueryObject>();
        this.Mutation = mutation;
        this.Subscription = subscription;
    }
}
```

我们感兴趣的是被赋予了变异对象的变异属性。

## 突变对象

下面是完整的 MutationObject 类，它定义了该服务器上的哪些对象允许哪些变化，以及收到变化请求时会发生什么。

```
public class MutationObject : ObjectGraphType<object>
{
    public MutationObject(IHumanRepository humanRepository)
    {
        this.Name = "Mutation";
        this.Description = "The mutation type for updates to our data.";
        this.FieldAsync<HumanObject, Human>(
            "createHuman",
            "Create a new human.",
            arguments: new QueryArguments(
                new QueryArgument<NonNullGraphType<HumanInputObject>>()
                {
                    Name = "human",
                    Description = "The human you want to create.",
                }),
            resolve: context =>
            {
                var human = context.GetArgument<Human>("human");
                return humanRepository.AddHuman(human,
                                                context.CancellationToken);
            });
    }
}
```

这与上周设置的 QueryObject 非常相似。第一个大的不同是在 QueryArguments 中。变异采用 HumanInputObject 类而不是 ID。如果你看一下查询参数，你会发现这个参数不允许为空。

```
new QueryArgument<NonNullGraphType<HumanInputObject>>()
```

什么是 HumanInputObject 类？它是一个 InputObjectGraphType，定义了突变查询参数的形状。正如您在下面看到的，它提供了名称、描述和字段列表。

```
public class HumanInputObject : InputObjectGraphType
{
    public HumanInputObject()
    {
        this.Name = "HumanInput";
        this.Description = "A humanoid creature from Star Wars.";
        this.Field<NonNullGraphType<StringGraphType>>(nameof(Human.Name));
        this.Field<StringGraphType>(nameof(Human.HomePlanet));
        this.Field<ListGraphType<EpisodeEnumeration>>(nameof(Human.AppearsIn), "Which movie they appear in.");
    }
}
```

另外，请注意，这些字段在 human 类的属性上使用 nameof，以确保名称匹配，这将防止该项目正在处理的 3 个不同的 Human 类之间的映射出现任何问题。下面是从上述示例中提取的字段定义的示例。

```
this.Field<NonNullGraphType<StringGraphType>>(nameof(Human.Name));
```

另一件要注意的事情是，即使在字段级别，除了设置字段类型之外，您还可以设置字段是否允许为空。

回到变异对象，让我们看看 FieldAsync 调用内部的解析。

```
resolve: context =>
{
    var human = context.GetArgument<Human>("human");
    return humanRepository.AddHuman(human, context.CancellationToken);
});
```

这是提取人工查询参数，并将其转换为人类类的实例，然后发送到存储库进行保存。

## 包扎

这涵盖了突变的基本探索。我在考虑订阅。

相关的示例代码可以在[这里](https://github.com/elanderson/ASP.NET-Core-GraphQl/tree/59792e870382dba7c6c40d444cfe573577b2569b)找到。

*原载于*[](https://elanderson.net/2018/07/graphql-using-net-boxed-mutations/)**。**