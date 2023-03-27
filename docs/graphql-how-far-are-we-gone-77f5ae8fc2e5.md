# GraphQL:我们走了多远…

> 原文：<https://itnext.io/graphql-how-far-are-we-gone-77f5ae8fc2e5?source=collection_archive---------2----------------------->

… ApolloGraphQL、Graphcool、urql、石墨烯…

![](img/5a08f6b57950258ce5829c3cec2a763a.png)

[*点击这里在 LinkedIn* 上分享这篇文章](https://www.linkedin.com/cws/share?url=https%3A%2F%2Fitnext.io%2Fgraphql-how-far-are-we-gone-77f5ae8fc2e5)

G raphQL 正变得迅速流行，工具每天都在开发。在本文中，我将试着从“*我自己的观点*来分析，并不要求绝对/完全了解所有可用的工具。一些最前沿的常用工具。最值得注意的是由 [*ApolloGraphQL*](https://www.apollographql.com/) 、 [*Graphcool*](https://www.graph.cool/) 和[*FormidableLabs*](https://github.com/FormidableLabs)*(urql)提供的工具。*

这些工具变得势不可挡，如果不把可能做同样事情的大量包堆在一起，要决定合适的工具可能有点困难。

我可能有点偏见，因为我使用 GraphQL 的经验是在 Javascript 环境中(Node JS 和 React)。

尽管使用了脸书的 React，我还没有尝试过使用 [*Relay*](https://github.com/graphql/graphql-relay-js) ，从我看过的几篇评论来看，与其他工具相比，比如 Apollo 的客户端工具，它似乎有点复杂。

我也没有直接使用过[*GraphQL JS*](https://github.com/graphql/graphql-js)*，所以我要重点介绍的工具是那些利用了这些最初开发的技术并使在客户端和服务器端开发 graph QL 应用变得更加容易的工具。*

*非常感谢参与创建这些工具的人们。*

***脸书***

*目前正在设计的大多数工具都严重依赖于 facebook 开发的库。*

*[*graphql*](https://www.npmjs.com/package/graphql)*(graph qljs)——*graph QL 的 JavaScript 参考实现，graph QL 是脸书创建的 API 的查询语言。*

*以编程方式创建架构*

```
***import** { graphql, GraphQLSchema, GraphQLObjectType, GraphQLString } **from** 'graphql';var schema **=** **new** GraphQLSchema({
  query**:** **new** GraphQLObjectType({
    name**:** 'RootQueryType',
    fields**:** {
      hello**:** {
        type**:** GraphQLString,
        resolve() {
          **return** 'world';
        }
      }
    }
  })
});*
```

*使用模式语言创建模式*

```
*var { graphql, buildSchema } = require('graphql');

// Construct a schema, using GraphQL schema language
var schema = buildSchema(`
  type Query {
    hello: String
  }
`);*
```

*Lee Byron 试图解释为什么以及何时以编程方式创建模式或使用模式语言。看看这个 [***github 讨论***](https://github.com/graphql/graphql-js/issues/223#issuecomment-248647850) 。*

*我们还可以使用这个包来运行对模式的查询。*

```
*var query **=** '{ hello }';graphql(schema, query).then(result => {
  console.log(result);  // Result: *data: { hello: "world" }*
}*
```

*[*express-graph QL*](https://www.npmjs.com/package/express-graphql)—graph QL HTTP 服务器中间件。*

```
*const express **=** require('express');
const graphqlHTTP **=** require('express-graphql');const app **=** express();app.use('/graphql', graphqlHTTP({
  schema**:** /* The defined schema */
}));app.listen(4000);*
```

*结合上面的两个库，我们可以创建一个模式并将其与服务器连接(connect，express，Restify)。*

*[*graphiql*](https://www.npmjs.com/package/graphiql)*-*如果您想在不创建前端客户端应用程序的情况下测试您的服务器，那么这是一个完美的包。可以在 express-graphql 中将 graphql 设置为 true 来启用它。*

```
*...
app.use('/graphql', graphqlHTTP({
  schema**:** /* The defined schema */,
  graphiql: true
}));
...*
```

*[**Apollographql**](https://www.apollographql.com)*

*Apollo 社区已经成为 GraphQL 工具的顶级创造者之一。以下是一些工具及其使用案例。*

****服务器****

*[*【graph QL-tools*](https://github.com/apollographql/graphql-tools)—graph QL 的基础是一个使用 [*GraphQL SDL(模式定义语言)*](https://blog.graph.cool/graphql-sdl-schema-definition-language-6755bcb9ce51) *编写的模式。*模式定义需要与解析器连接，解析器提供方法 *getBooks* 和 *createBook(…)* 的实际实现，以创建可执行的模式。*

**graphql-tools* 就是能让我们做到这一点的工具。这个工具还有一些其他的功能，但是到目前为止这个功能还是很突出的。*

*GraphQL 模式*

*[*apollo-server*](https://github.com/apollographql/apollo-server/)—当 graphql-tools 创建模式时，Apollo-server 需要将其连接到服务器(Express、connect、哈比神、Koa …)。这是一个非常大的软件包，我建议你不要安装它。最好的方法是使用它的 [*变体*](https://github.com/apollographql/apollo-server/tree/master/packages) 取决于你的服务器，它采用*阿波罗-服务器- <变体>例如阿波罗-服务器-快递，阿波罗-服务器-koa，…**

*使用 **executableSchema，**我们现在可以如下连接到 express 服务器。*

*有了 apollographql 的上述工具，我们就可以使用 graphql 创建一个功能齐全的服务器端应用程序。*

****客户端****

*Apollo 社区还提供了一些工具，可以在客户端与不同的框架、库或普通 javascript 一起使用。最值得注意的工具是 [*阿波罗-客户端*](https://github.com/apollographql/apollo-client) 和 [*反应-阿波罗*](https://github.com/apollographql/react-apollo) 。*

*那么这两个工具有什么区别。来自[*learnapollo*](https://www.learnapollo.com/tutorial-react/react-01)*:**

*   *核心包，公开了提供核心功能的普通 JS Apollo 客户端。*
*   *`react-apollo`——React 集成公开了可用于包装其他 React 组件的`ApolloProvider`，允许它们发送查询和变化*

*从 [*阿波罗-客户端回购*](https://github.com/apollographql/apollo-client) :*

*来自 Apollo Client 是一个全功能的缓存 GraphQL 客户端，集成了 React、Angular 等等。*

*到目前为止，react 的集成是以 react-apollo 的形式提供的。其他一些框架集成仍在开发中。*

*对于任何 JS，尤其是 React dev，下面的用例应该非常简单。*

*不幸的是，对于 Redux 的粉丝来说，阿波罗 1.x 有一个 Redux 的实现，但在阿波罗 2.0 中它被放弃了，取而代之的是 [*阿波罗缓存内存*](https://www.npmjs.com/package/apollo-cache-inmemory) 。( [*见此处为什么*](https://github.com/apollographql/apollo-client/issues/2509#issuecomment-343963114) )。 *createReactInterface* 也由[*Apollo-link-http*](https://www.npmjs.com/package/apollo-link-http)中的 *HttpLink* 取代。*

*内存缓存可以按如下方式使用:*

```
*...
import { InMemoryCache } from 'apollo-cache-inmemory';
import { ApolloClient } from 'apollo-client';// Create client that connects to the server
const client = new ApolloClient({
  link: HttpLink({ uri: 'https://api.graph.cool/simple/v1/__PROJECT_ID__'}),
  cache: new InMemoryCache()
})*
```

*[](https://www.npmjs.com/package/graphql-tag)**——*这个和 react-apollo 一起使用，在客户端解析 graphql 查询。下面的例子，一个来自 [apollographql](https://github.com/apollographql/react-apollo) 的片段展示了如何使用来自 *react-apollo* 和 [*graphql-tag*](https://www.npmjs.com/package/graphql-tag) 的函数将一个功能组件连接到 graphql。非常类似于 Redux 中的 connect。**

```
**import { graphql } from 'react-apollo';
import gql from 'graphql-tag';

function TodoApp({ data: { todos, refetch } }) {
  return (
    <div>
      <button onClick={() => refetch()}>Refresh</button>
      <ul>{todos && todos.map(todo => <li key={todo.id}>{todo.text}</li>)}</ul>
    </div>
  );
}

export default graphql(gql`
  query TodoAppQuery {
    todos {
      id
      text
    }
  }
`)(TodoApp);**
```

**还有 [*apollo-ios*](https://github.com/apollographql/apollo-ios) 和[*Apollo-Android*](https://github.com/apollographql/apollo-ios)*面向原生手机 app 开发者的客户端工具。***

***到目前为止，我将谈到阿波罗号上的那些工具。关于阿波罗工具的完整教程可以在[*learnapollo*](https://www.learnapollo.com/)*(注:依赖阿波罗 1.x)* 找到。***

**[**Graphcool**](https://www.graph.cool/)**

**Graphcool 是另一个推动 GraphQL 福音的酷社区。他们开发了一些令人敬畏的工具，从 *graphql-framework* 、[、 *graphql-playground* 、](https://github.com/graphcool/graphql-playground)[、 *graphql-yoga* 、](https://github.com/graphcool/graphql-yoga)到超级令人敬畏的 *Prisma* 。开发很快，在某个时候，社区对 graphcool-framework 和 Prisma 感到困惑。[下面就由](https://www.graph.cool/forum/t/graphcool-framework-and-prisma/2237?u=nilan)好好解释一下 [*尼兰*](https://github.com/marktani) 它们到底是什么以及它们有什么区别。**

**根据我的观察，Graphcool 的一些工具利用了 Apollo 的工具，我认为这在开源社区合作时是非常酷的。Graphcool 的另一个好处是，在开发期间，它提供了一个免费的层服务来托管 graphcool 和 Prisma 应用程序。(救了我们这些付不起 AWS 的穷人)。那么，让我们来看看“*浅深度*”中的一些工具。**

*****GraphLQL 游乐场*****

***大多数人把它们称为 GraphQL IDEs，我更喜欢 GraphQL 查询编辑器；这里的查询是指查询、变异或订阅。如果你玩过 GraphQL，那么你可能已经和 graph QL 交互过了。GraphQL Playground 非常相似，但有一些额外的功能。***

******Graphcool 框架******

**Graphcool 框架[是一种开源技术](https://github.com/graphcool/graphcool-framework/)，它提供了基于自以为是的堆栈的 GraphQL 后端解决方案。它使用 graphcool CLI 工具来搭建、管理和部署应用程序。使用 Graphcool，可以在我们的数据库上直接生成 API，我们可以直接从客户端应用程序访问它。目前，它只支持 MySQL，但其他数据库的集成正在进行中。从 [***npm 包***](https://www.npmjs.com/package/graphcool) ***，*** 我们可以轻松设置一个简单的应用程序，并在不到 5 分钟的时间内完成部署。**

**[***棱镜***](https://www.prismagraphql.com/docs)**

**就像 Graphcool Framework 一样，Prisma 也公开了整个数据库及其方法，但增加了一个应用层，我们可以在其中指定如何使用数据库方法。**

*****棱镜装订*****

**[Prisma Binding](https://github.com/graphcool/prisma-binding) 是一个非常强大的工具。当使用 Prisma CLI 创建应用程序时，它会公开一个数据库 API，但除此之外，它还会使用 [graphql-yoga](https://github.com/graphcool/graphql-yoga) 创建一个 GraphQL 服务器，该服务器会自行公开另一个 API。Prisma Binding 所做的是使我们能够将这两个 API 结合起来，这样我们就可以用我们的服务器 API 创建我们自己的方法并调用数据库 API。**

**问题是为什么？当我们在正常情况下(REST)创建数据库时，我们不会将所有关于记录的字段/数据暴露给客户端，我们会过滤掉一些可能敏感的数据，如密码。通过 Prisma 绑定，我们可以定义自己的类型，从 CRUD API 中过滤掉我们不想要的类型。现在，客户不能直接访问我们的 CRUD API，我们可以定义自己的认证系统，并包含所有我们喜欢的复杂功能。**

**[***GraphQL 瑜伽***](https://github.com/graphcool/graphql-yoga)**

**这个工具是建立在`[apollo-server](https://github.com/apollographql/apollo-server)`、`[graphql-subscriptions](https://github.com/apollographql/graphql-subscriptions)`和其他一些工具之上的。使用 yoga，我们可以很容易地设置一个已经设置了订阅的 GraphQL 服务器。**

**[**可怕的实验室**](https://github.com/FormidableLabs)**

**在做一些研究的时候，我偶然发现了 Apollo 的一个工具: [apollo-boost](https://www.npmjs.com/package/apollo-boost) 。我还没有机会使用它，但是这里有一篇关于这个工具的很好的文章。那么，为什么阿波罗助推火箭如此强大呢？如果你读过那篇文章，你可能会知道它为什么被开发出来。**

**FormidableLabs 开发了[***urql***](https://github.com/FormidableLabs/urql)***，*** GraphQL 客户端，公开为一组 ReactJS 组件。仅仅浏览一下 urql repo 的自述文件就可以看出一些非常有希望的东西；如果你管理了 react-apollo，那么这应该会容易一点。**

**还有很多像 Graphene 这样的工具为 Python 和 JS 提供了实现。请随意探索和使用它们。**

**现在，如果您开始使用 GraphQL 应用程序，我建议使用以下格式。**

**计算机网络服务器**

*   **使用 apollo-server 的一个变种设置一个应用程序，并使用 PubSub 自行配置订阅。**
*   **使用 graphql-yoga 轻松设置，并开始使用已经设置的订阅。**
*   **graph QL(graph qljs)+express-graph QL**

**客户**

*   **对于 react 应用程序，apollo-client + react-apollo 运行良好。**
*   **React + urql。**

**至于 Graphcool Framework 和 Prisma，我会建议密切关注它们的更新，直到一切平衡。它们是正在快速发展变化的工具。由于这些变化，他们文档中的一些链接仍然是断开的，但这不应该阻止任何人使用它们。**

**我会感谢你的掌声、评论、纠正、建议和积极的批评。**