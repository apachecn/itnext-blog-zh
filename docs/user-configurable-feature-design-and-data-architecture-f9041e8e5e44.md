# 用户可配置的特征设计和数据架构

> 原文：<https://itnext.io/user-configurable-feature-design-and-data-architecture-f9041e8e5e44?source=collection_archive---------4----------------------->

![](img/2d74ea32672cf1bbb8653a9e9025e6c1.png)

giphy.com

随着 2020 年接近尾声，我有时间回顾我曾有机会以某种身份参与的十几个项目。我发现最明显的相似之处是许多项目都有某种形式的数据模式，这些数据模式是围绕用户定义的字段、动态屏幕呈现和/或两者的某种组合而设计的。我将用户可配置的设计描述为驱动逻辑的一组数据，在大多数情况下，可以在以后更改，而不需要更新代码。我参与过的一些项目及其可配置性的例子包括:

*   一个相关的评估系统**，其中管理员定义带有问题和评分例外的自定义评估模板**。管理员实时创建新的评估、新的问题和新的评分例外。前端自动显示这些变化，API 已经知道如何使用新的逻辑。
*   一个最小的电子商务网站**，其中的产品选项由管理员**实时定义，而所有这些仍然是一个主产品。管理员还向这些标签添加标签/键和值。每个标签的不同值的每个组合代表一个具有单独 SKU 的库存项目。我们设计系统的方式允许我们只显示可用的选项组合(例如*红色* XL 或*绿色* SM ),同时验证管理员没有复制现有的选项组合。
*   问卷引擎**，管理员在其中定义问题、可用答案和自定义值计算**。管理员创建问题以及每个问题的答案。答案格式取决于问题的类型。并且，可选地，每个问题类型可能具有用于进一步计算所需的其他属性。
*   一个工作流和状态跟踪系统**，其中根据“项目”(数据库表)**定义和设置自定义状态。管理员定义了“项目”需要遵循的状态列表以及遵循的顺序。API 可以验证流程是否被正确遵循，并且前端相应地显示订单。

还有一些其他示例的复杂性更低，但是，许多示例似乎至少有一个小型企业对实时用户可配置数据的需求。当我们第一次处理这些需求时，我们在白板上开了一个很长的设计会议。我们讨论了各种关于数据库模式复杂性、API 逻辑复杂性或前端复杂性的问题。对于任何用户配置的设计，我们发现如果设计不正确，整个系统至少有一部分(数据库、API 或前端)会比其余部分更复杂。考虑到后者，我们试图平衡这三者，使系统的任何一部分都不会:

1.  太难维持了
2.  对于新开发人员来说，在合理的时间内学习太难了
3.  陷入困境，未来的需求需要重新设计

以下是我们在处理这些新项目时一贯遵循的一些原则和策略。

# 验证设计、验证设计和验证设计

在开始为项目编写代码之前，验证和重新验证数据库设计模式的重要性怎么强调都不为过。我们已经经历过——时间紧迫、需求不明确，或者仅仅是一个疏忽——设计在项目的后期不能工作，需要返工。我的一些建议是:

*   尽可能保持简单。保持表的数量少，代码少。如果您发现自己需要四五个表来支持一个可配置的特性，那么您可能需要重新考虑一下您所做的事情的复杂性。我们的大多数项目只使用一两个表，并通过使用几个架构良好的数据库列获得成功。
*   从你的客户或产品所有者那里引出所有的需求，即使是那些没有说出来的。如果你有一个想法，“我想知道他们是否需要这样做……”，那就去问客户吧。用户配置的设计是最难更改的设计之一。
*   画一个流程图。获取项目的所有需求和描述，创建一个示例场景，并运行它。我喜欢在白板上绘制数据库表，并随着过程的进行在下面画线。你会很快发现(使用这种视觉方法)是否有东西丢失或根本没有使用。

![](img/1fe5e7dbbca8c1a5225d035add56952f.png)

马克·拉贝在 [Unsplash](https://unsplash.com/s/photos/whiteboard?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上的照片

# 降低数据库复杂性的枚举

如果您不熟悉什么是枚举，下面是一个代码示例:

```
// c# / .NET Corepublic enum MyEnum
{
    Text = 0,
    Number = 1,
    Dropdown = 2,
    Radio = 3
}
```

枚举是一组预定义的(在编译期间已知的)值，用于区分某些属性或进程。通常，这些值在数据库中存储为整数。在一些项目中，我们使用 EF Core 将枚举值转换成字符串(varchar ),因为手动查询和查看带有枚举名称的数据库记录更容易。

在问卷引擎中，如上所述，问题类型具有 enum 属性`public QuestionType Type { get; set; }`，API 逻辑和前端将出于不同的原因使用该属性。API 逻辑可能知道验证可选属性，并将这些属性用于某些计算。我们有一个常见的可选属性是`public string Pattern { get; set; }`来验证用户输入的正则表达式模式。在前端，我们将使用`Type`属性来确定显示和验证什么组件/表单控件。

我们首先尝试确定所有可能需要的类型，但是，如果忽略了一个类型，简单地添加另一个类型就是在 enum 文件中添加一行代码，并为附加逻辑添加一个 if/case 语句。

# JSON 字段来降低数据库的复杂性

根据经验，为用户定义的、可配置的字段设计数据库模式通常需要多个小表，这会变得非常复杂、非常快速。JSON 字段是抵消数据库设计复杂性的一个很好的方法，EF Core 使这变得很容易。下面是我如何将一个`IEnumerable`或自定义类转换成 JSON 字符串的例子:

```
// c# / .NET Coreprotected override void OnModelCreating(ModelBuilder modelBuilder)
{ modelBuilder.Entity<MyCustomClass>(x =>
    {
        x.Property(y => y.MyCustomClass)
            .HasConversion(
                y => JsonConvert.SerializeObject(y, new JsonSerializerSettings { NullValueHandling = NullValueHandling.Ignore }), y => JsonConvert.DeserializeObject<DepartmentRecord>(y, new JsonSerializerSettings { NullValueHandling = NullValueHandling.Ignore }));
    });}
```

本文开头描述的电子商务示例使用这种技术来保存、显示和验证产品选项。一个产品的示例表可能是这样的:

```
// c# / .NET Corepublic class Product
{
    public int Id { get; set; } public Options Options { get; set; } ...
}public class Options
{
    public string Label { get; set; } public List<string> Values { get; set; }
}
```

我们将把`Options`属性转换成一个 JSON 字符串。纯关系设计的可比方法是创建两个新表；`ProductOptionKey`和`ProductOptionValue`，其中`ProductOptionValue`有一个返回到`ProductOptionKey`的外键，`ProductOptionKey`有一个返回到产品的外键。如您所见，这将增加两个数据库表、两个 c#服务等..我强烈推荐在有用的地方使用 JSON 字段，同时确保它们不只是“酷”。

# 摘要

我希望这篇文章对任何接近或正在从事需要用户配置设计的项目的人有用。我希望传达的主要观点是简单数据库设计和极端彻底性的重要性。在验证数据库设计的同时这样做将会在现在和将来支持您的项目。我预计 2021 年至少会有四五个以上的项目以这样或那样的方式需要这种实践，我真的很享受为他们设计这些解决方案的过程。

# 关于我

我在南卡罗来纳州格林维尔的软件工程咨询公司 Orange Bees 担任首席工程师。我写得棱角分明。NET 应用，在 Azure 架构项目(Azure Developer Associate 认证)，涉猎 ElasticSearch 和 node . js
你可以在 [LinkedIn](https://www.linkedin.com/in/james-l-gross/) 上找我。