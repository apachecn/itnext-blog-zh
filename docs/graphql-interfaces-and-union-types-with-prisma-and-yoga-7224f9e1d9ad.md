# GraphQL 与 Prisma 和 Yoga 的接口(和联合类型)

> 原文：<https://itnext.io/graphql-interfaces-and-union-types-with-prisma-and-yoga-7224f9e1d9ad?source=collection_archive---------4----------------------->

![](img/7e5aabe9f1ed29763fafbaa1c66b0d27.png)

克林特·王茂林在 [Unsplash](https://unsplash.com/?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄的照片

> [点击这里在 LinkedIn 上分享这篇文章](https://www.linkedin.com/cws/share?url=https%3A%2F%2Fitnext.io%2Fgraphql-interfaces-and-union-types-with-prisma-and-yoga-7224f9e1d9ad%3Futm_source%3Dmedium_sharelink%26utm_medium%3Dsocial%26utm_campaign%3Dbuffer)

# GraphQL 是什么？

GraphQL 是一种 API 查询语言，来自脸书团队，最近已经接管了互联网。它的优势在于围绕强类型 API 契约构建，该契约详尽地定义了 API 中的数据及其模式、如何请求数据等等。它支持具有受控水合作用的深度嵌套查询，并允许 API 客户机将来自不同来源或模型的数据组合成一个查询。使用 GraphQL，您可以准确地获得您想要的数据，以您想要的方式格式化，并且在一个查询中，解决了传统 REST APIs 的几个问题。此外，API 契约概念支持各种强大的开发工具，下面我将介绍其中的一些。

# 我的 GraphQL 堆栈

*   [**Prisma**](https://www.prisma.io/) ，由 [Graphcool](https://twitter.com/graphcool) 的神奇团队开发，是一种 GraphQL ORM，它获取你在 [SDL(模式定义语言)](https://blog.graph.cool/graphql-sdl-schema-definition-language-6755bcb9ce51)中定义的数据模式，并为其生成数据库和 API。为(嵌套的)CRUD 操作生成的 API 的广泛性是惊人的。您可以在他们的云中部署您的数据库服务，或者在您的基础设施上使用 docker。最重要的是，Prisma 附带了[绑定](https://www.prisma.io/docs/reference/prisma-bindings/overview-oobi0eicho)，为在 Prisma 服务之上构建 GraphQL 服务器提供了一个便利层。
*   [**【graph QL-yoga】**](https://github.com/graphcool/graphql-yoga)，也是由 Graphcool(这些家伙是上🔥)，是构建 GraphQL 服务器最简单的方法。它基于或兼容大多数用于用 Javascript 构建 GraphQL 服务器的事实上的标准库，但它从改善开发人员体验的角度出发，通过使用合理的默认值和更具声明性的配置方法，使一切设置更容易。它或多或少涵盖了整个 GraphQL 规范，甚至包括对订阅的 WebSockets 支持。
*   [**GraphQL 游乐场**](https://github.com/graphcool/graphql-playground) ，也由 graph cool(wuut？😱)，是一个基于 web 的 GraphQL client / IDE，它通过自省 API 契约来加速您的开发工作流，从而为它提供一个自动的交互式文档，以及一个具有自动完成功能的查询接口，并根据您的模式进行验证。它包含了漂亮的小特性，是任何 GraphQL 的首选工具。
*   由 [Apollo](https://twitter.com/apollographql) 的天才们开发的 [**Apollo 客户端**](https://www.apollographql.com/client) 可能是目前最好的 GraphQL 客户端。它与每一个主要的前端平台兼容，专注于在 UI 组件中获取数据，而不需要为获取数据做任何准备。我喜欢 React 的声明式数据获取方法，以及它支持的高级数据加载特性。例如缓存、加载、乐观 UI、分页等。devtools 对您的开发人员体验也是一个很好的补充。

# 现在谈谈界面…

## 一些背景

GraphQL 模式规范支持[接口](https://graphql.org/learn/schema/#interfaces)和[联合类型](https://graphql.org/learn/schema/#union-types)。接口是一种抽象类型，它包含一组特定的字段，类型必须包含这些字段才能实现接口，而联合类型允许在不共享任何结构的情况下对几种类型进行分组。

对于任何重要的数据结构，您很可能需要利用这些结构来建模您的数据。问题是:

1.  Prisma 还不支持接口或联合类型。它们中的每一个都有未解决的问题—参见[接口](https://github.com/graphcool/prisma/issues/83)和[联合类型](https://github.com/graphcool/prisma/issues/165)。
2.  graphql-yoga 支持这两者，但是它们的用法还没有被记录下来，这使得实际上很难实现任何东西。不久前我打开了[的一个问题](https://github.com/graphcool/graphql-yoga/issues/121)想知道更多，这篇文章把我带到了这里。

## 我的方法

由于 Prisma 目前只支持类型和枚举，我们必须找到一种方法来建模我们的数据，而不使用 Prisma 中的接口。然而，我们可以使用 GraphQL 服务器上的接口(graphql-yoga ),以便面向客户端的 API 被适当地构造，并且用户可以使用[内联片段](https://graphql.org/learn/queries/#inline-fragments)跨类型请求数据。

这给我们留下了两个选择:

1.  将所有带有可选类型特定字段的数据存储在 Prisma 中的一个类型(接口)下，然后在应用服务器中的原始类型之间拆分数据。
2.  在 Prisma 上存储每个原始类型的数据，并在应用服务器上拼接查询内容。

选项 2 的问题是您失去了分页的一致性。如何获得界面的最后 20 项？每种基本类型需要多少个？你可以做 20 个，把它们分类，然后拿走 20 个，但这在我看来很不雅。

所以我选了选项 1，我们来看看怎么实现。我将按照文档中使用的模式[给出代码片段。](https://graphql.org/learn/schema/#interfaces)

## Prisma 解决方案

基本上，我们希望将所有基本类型合并为一个单一的“接口”类型。特定于类型的字段必须是可选的，因为它们不会对每个条目都可用，并且它们以基本类型的名称为前缀，以确保它们是唯一的。在文档中，我们有:

GraphQL 文档中的接口示例

我们的解决方案是:

带有模拟界面的 Prisma 的 datamodel.graphql 文件

## 在 graphql-yoga 中映射接口

根据需要，我们在面向客户端 API 的模式中声明了与文档中相同的接口和基本类型。我们还复制了 Prisma 生成的`dbCharacters`查询的模式，作为面向客户端的 API 的`characters`查询。这可能会更好。然而，返回类型被更改为我们的接口，因此返回的项应该被映射到一个基本类型，在这个基本类型上可以使用特定于类型的内联片段。

src/schema.graphql

为了将 Prisma 返回的条目映射到原始类型，我们需要在 resolvers 对象的根处为我们的接口提供一个类型解析器。我已经将接口解析器的声明分离到一个单独的文件中，并通过将[对象析构为](https://developer.mozilla.org/my/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)导入到解析器对象中。参见`interfaces.js`文件中的`__resolveType`示例。这是一个展示如何解析类型的简单示例。您将根据数据的特定业务逻辑来实现您的。

src/resolvers/index.js

src/resolvers/interfaces.js

最后要做的是实现接口的客户端 API。它由 Prisma 的相应 API 支持，但是我们需要在两个模式之间转换 I/o。`characters`查询的解析器在`Query.js`文件中实现，这非常经典。实现细节如下:

1.  我们必须确保在查询中为原始类型选择的所有字段都是从 Prisma 请求的。为此，我在`interfaces.js`中编写了一个名为`makeSelection`的实用函数，它从解析器获取`info`对象并解析查询 AST ( `GraphQLResolveInfo`)以生成发送给 Prisma 的字符串选择。这将修改选择，以确保嵌套在内联片段(如`...on Droid { primaryFunction }`)中的所有字段将作为普通前缀字段从 Prisma 中查询，如`droid_primaryFunction`。在检查`info`对象并将其映射到要发送给 Prisma 的预期选择时，这个方法的代码几乎是反复试验的。*免责声明*:该代码仅涵盖我一直需要的查询，可能需要添加以涵盖所有用例。还要注意，我不是 ASTs 专家，所以可能有更好的方法，如果你知道的话，请在评论中提出建议。
2.  我们必须将从 Prisma 收到的对象格式化回它们在客户端 API 模式中的预期形式。我使用了另一个名为`formatPrimitiveFields`的实用函数，也可以在`interfaces.js`中找到，它接受一个字段，比如`droid_primaryFunction`，并删除了原始类型前缀。

src/resolvers/Query.js

src/resolvers/interfaces.js

*本文没有直接讨论联合类型，但是它们非常类似于接口的* `*__resolveType*` *方法。*

*代码片段是为 node 8 及以上编写的。*

*如果您使用的是* ***Apollo 客户端*** *，请注意，内联片段中的接口和联合在开箱即用时无法正确解析。您需要基于 api 模式设置一个定制的片段匹配器。这在* *的* [*中有详细的解释。*](https://www.apollographql.com/docs/react/advanced/fragments.html#fragment-matcher)