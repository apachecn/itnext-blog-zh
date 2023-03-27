# 处理 GraphQL 错误的权威指南

> 原文：<https://itnext.io/the-definitive-guide-to-handling-graphql-errors-e0c58b52b5e1?source=collection_archive---------0----------------------->

![](img/f662d0a16f925e1fbd26a4659d22d0a3.png)

明白了吗？因为它们是错误...你扔出去，然后接住它们…没关系

[*点击这里在 LinkedIn 上分享这篇文章*](https://www.linkedin.com/cws/share?url=https%3A%2F%2Fitnext.io%2Fthe-definitive-guide-to-handling-graphql-errors-e0c58b52b5e1)

最近，我搞砸了，导致一个客户在使用我们的应用程序时出现白屏。像大多数应用程序一样，我们有一个初始的 GraphQL 查询来获取大量信息，包括所有通知的列表。其中一个通知引用了数据库中不再存在的字段(哎呀！).结果呢？GraphQL 是冠军，它将`data`和`errors`都发送给了客户端。但是客户端完全忽略了`data`,因为它将响应作为错误处理。事后看来，那是相当愚蠢的。这就像一个学生得不到 100 分就不及格一样。这是不对的。

GraphQL 发送`data`和`errors`的能力令人惊叹。这就像和一个真正的人对话:“嘿，马特，这是你想要的结果。除了任务区，我什么都给你了；我去查了一下，你的数据库里没有。”有了这些能力，我们可以在客户端上做一些非常酷的事情！不幸的是，大多数客户端代码归结为:

```
if (result.errors) throw result.errors[0]
```

这并不完美，但是如果我们没有抛出错误，那么就不会调用`onError`处理程序，这就是我将服务器验证错误传播到 UI 的方式。因此，在编写一个完美的服务器和不接收服务器错误之间选择，我选择了前者——它工作了将近 2 年！…直到它没有。

# 识别错误类型

为了确保我修复了根本原因，我开始研究我们在应用程序中抛出的所有类型的错误，以及其他人处理 GraphQL 错误的所有方式。在查询 GraphQL 服务器时，客户端可能会遇到很多错误。无论是查询、变异还是订阅，它们都属于 6 种类型:

1.  服务器问题(5xx HTTP 代码，1xxx WebSocket 代码)
2.  客户问题，如速率受限、未授权等。(4xx HTTP 代码)
3.  `query`缺失/格式不正确
4.  `query`没有通过 GraphQL 内部验证(语法、模式逻辑等。)
5.  用户提供的`variables`或`context`不正确，`resolve` / `subscribe`函数故意抛出错误(例如，不允许查看请求的用户)
6.  在`resolve` / `subscribe`函数中出现未被捕获的开发人员错误(例如，编写的数据库查询很差)

那么，这些错误中哪些是关键到可以忽略所有数据的呢？数字 1-3，因为它们甚至在调用 GraphQL 之前就出现了。第四，它也调用 GraphQL，但是只接收到`errors`的响应。对于 5–6，GraphQL 用部分`data`和一个数组`errors`来响应。有些人会把类型 5 和类型 2 混为一谈，例如，查询“点”的用尽可能会构成 HTTP 429(请求太多)。但说到底，最简单的答案是最好的:**如果 GraphQL 给你一个带有** `**data**` **的结果，即使那个结果包含** `**errors**` **，也不算错误。**没有根据错误类型改变 HTTP 代码，没有读取`errors`来决定特定错误有多“严重”,也没有读取`data`来查看它是否可用。我不在乎结果是不是`{data: {foo: null}}`。数据就是数据；任何在 GraphQL 返回后实现的任意空逻辑都是:任意的。

按照这个逻辑，错误类型 1–4 将作为错误发送到客户端，因为没有`result.data`。但是第 5-6 类呢？

# 不要故意在 GraphQL 中抛出错误

截至 2018 年 3 月，无论是[阿波罗](https://github.com/apollographql/apollo-client/issues/3034) - [客户端](https://github.com/apollographql/apollo-client/issues/3000) ( [包括](https://github.com/apollographql/apollo-client/issues/2810) [订阅-传输-ws](https://github.com/apollographql/subscriptions-transport-ws/issues/305) )还是[中继](https://github.com/facebook/relay/issues/1913) [现代](https://github.com/facebook/relay/issues/1816#issuecomment-304492071)在处理错误方面都不完美。Relay 的突变 API 接近于它的`onCompleted(result, errors)`回调，但是对于查询和订阅来说这是非常缺少的。阿波罗凭借其`ErrorPolicy`更加灵活；但是这两种方法都没有提供最佳实践，所以我提出了自己的方法:**如果查看者看到了错误，将错误作为一个字段包含在响应有效负载中。**例如，如果有人使用过期的邀请令牌，而你想告诉他们令牌已过期，你的服务器不应该在解析过程中抛出错误。它应该返回包含`error`字段的正常有效载荷。它可以像字符串一样简单，也可以像您希望的那样复杂:

```
*return* {
  error: {
    id: '123',
    type: 'expiredToken',
    subType: 'expiredInvitationToken',
    message: 'The invitation has expired, please request a new one',
    title: 'Expired invitation',
    helpText: 'https://yoursite.co/expired-invitation-token',
    language: 'en-US'
  }
}
```

通过在模式中包含错误，生活变得容易多了:

*   所有错误在到达客户端之前都会被清理并准备好供查看器使用
*   您不需要抛出一个 stringified 对象并在客户端解析它
*   你不必用 22 种不同的语言发送同样的错误(你知道你是谁)
*   您可以将相同的错误作为面包屑发送到您的错误日志记录服务
*   最重要的是，*你的 GraphQL* `*errors*` *数组不会包含任何面向用户的错误*，这意味着你的 UI 不会忽略它们！

对于突变(和订阅)，这是一个简单的销售。如果你遵循我的订阅混合方法，那就更容易了，因为你的订阅重用了你的突变负载。但是查询呢？现在 GraphQL 最佳实践中存在一个二分法:变异和订阅返回一个充满类型的有效载荷，但是查询只返回一个类型。以我的错误为例，想象一个请求，其中`team`成功了，但是`notifications`失败了:

```
mainQuery {
  team {  #succeeds
    name
  }
  notifications { #fails
    text
  }
}
```

为了避免丢失部分数据，我们认为整个事情是成功的，但是这样做，我们丢失了错误！怎样才能两者兼得？由于上面列出的原因，我们不能回头抛出错误，但是将每个对象包装成一个有效负载会非常难看:

```
mainQuery {
  teamPayload {
    error {
      message
    }
    team {
      name
    }
  }
  notificationPayload {
    error {
      message
    }
    notifications {
      text
    }
  }
}
```

虽然这并不理想，但只有当用户界面需要知道错误时才适用。听起来熟悉吗？它的功能就像 React 中的[错误边界:](https://reactjs.org/blog/2017/07/26/error-handling-in-react-16.html)

> 错误边界的粒度由您决定。您可以包装顶级路由组件以向用户显示“出错”消息，就像服务器端框架通常处理崩溃一样。您还可以将单个小部件封装在一个错误边界中，以防止它们破坏应用程序的其余部分。

因此，如果返回一个`null`或空数组就足够了，那么继续前进；但是将事件发送到您的异常管理器进行跟踪。如果您注意到某个特定的查询片段经常失败，那么您可以使用一个有效负载来包装它，以创建一个伪错误边界。虽然艺术多于科学，但这意味着我对所有 GraphQL 操作一视同仁，我不会不必要地膨胀我的整个模式。

现在说到信任客户端，如果客户端不应该看到它，您的服务器就不应该发送它，这就把我们带到了最后一个处理程序。

# 如何(对客户)隐藏自己的缺点

还记得过去的美好时光吗？那时*所有*的错误都是无意的。如今，玩 catch 出错比实际玩 catch 更常见(看看你对 v17 疯狂承诺的反应)。

在将我们有意抛出的错误重构到我们的响应有效负载中的常规字段后，任何剩余的错误都必须是无意的(即开发人员错误)，这意味着我们应该掩盖我们的踪迹，并用类似于“服务器错误”的模糊内容替换`message`。在完美的世界中，这些将被捕获、净化，并作为响应中的错误属性返回，但是您永远不会捕获所有的错误(因此您可以停止在 try/catch 中包装每一个语句)。我们仍然把真正的错误发送给我们的日志服务，这样我们可以在任何人知道它坏了之前修复它，但是客户端永远不会看到它，因为错误可能包括敏感的东西，比如我们在生产中使用的实际数据库查询。除了模糊的`message`，保留错误的`path`也是值得的，因为这将帮助我们确定错误发生在哪里。同样，简单是最好的:**对于** `**errors**` **中的每一个错误，发送一个通用的消息和路径到客户端，并附上部分数据。**

一旦结果出现在客户机上，它将被作为一个成功的请求来处理。您甚至可以忽略这些错误，这样就很好了(如果这是一个查询，您可能必须这样做！).然而，如果你想利用它，你仍然可以在任何有`errors`数组的地方引用它。将所有这些放在一起，这是它在 Relay Modern 中的样子:

```
*// Called for error types 1-4 (5xx, 4xx, missing/invalid query)
const* onError = (err) => {
  *this*.setState({err})
}

*const* onCompleted = (result, errs) => {
  *// Called for error type 6 (eg unexpected missing DB field)
  const* err = errs.find(({path}) => path.includes('approve'));
  *if* (err) {
    onError(err.message);
  }

  *// called for error type 5 (eg expired auth token)
  const* {approve: {error: {message}}} = result;
  onError(message);
}

commitMutation(env, {mutation, onCompleted, onError})
```

请记住，这对于突变非常有效，但是查询和订阅会吞下错误，除非它们被抛出，这意味着如果您希望它出现在您的 UI 中，您最好将它放在您的模式中！

# 结论

TL；博士；医生

*   如果 GraphQL 给你`results.data`，那就不是错误，不要扔在客户端。
*   如果查看器应该看到错误，则将错误作为响应有效负载中的一个字段返回。如果是查询，就做一个响应负载。
*   用通用消息替换任何剩余的 GraphQL `errors`,但是不要把它扔到客户端，也不要期望 UI 总是能够处理它。

无论是查询、变异还是订阅，我们确定了请求在返回*graph QL 响应时可能遇到的 6 种不同类型的错误。我们提出了策略来保证部分数据永远不会被客户端忽略。最后，我们确保查看者总是看到我们希望他们看到的错误(仅此而已！).我们还设法避免了像`GraphQLConnectionError`这样抛出自定义错误的又深又黑的兔子洞，尽管它们有缺点，但看起来还是很受欢迎。你如何处理错误？这是不是已经是常识了，我只是派对迟到了？让我知道。*