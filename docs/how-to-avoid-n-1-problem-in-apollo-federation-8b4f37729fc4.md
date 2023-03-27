# 如何避免阿波罗联邦中的 N+1 问题

> 原文：<https://itnext.io/how-to-avoid-n-1-problem-in-apollo-federation-8b4f37729fc4?source=collection_archive---------3----------------------->

如果你使用了 [GraphQL](https://graphql.org) ，你很可能遇到或听说过 N+1 问题。

![](img/694efd231fb029c9e4b722f1644f1034.png)

# 这是什么？

当我们执行下面的查询来获取前 100 篇评论和相应的作者姓名时，我们首先进行一次调用来从数据库中检索 100 条评论记录，然后对于每篇评论，我们对数据库进行另一次调用来获取给定作者 ID 的用户详细信息。

**查询**

```
{
  top100Reviews {
    body
    author {
      name
    }
  }
}
```

**模式**

```
const typeDefs = gql`
  type User {
    id: ID!
    name: String
  } type Review {
    id: ID!
    body: String
    author: User
    product: Product
  } type Query {
    top100Reviews: [Review]
  }
`;
```

**解析器**

```
const resolver = {
  Query: {
    top100Reviews: () => get100Reviews(),
  },
  Review: {
    author: (review) => getUser(review.authorId),
  },
};
```

要打很多电话。也是很大的网络开销。

[DataLoader](https://github.com/graphql/dataloader) 可以通过**将用户详细信息的每个单独加载合并**到单个调用中来帮助您减少开销。我们需要做的只是提供一个批处理函数的实现。

```
**const DataLoader = require('dataloader');
const userLoader = new DataLoader(ids => batchGetUsers(ids));**const resolvers = {
  Query: {
    top100Reviews: () => get100Reviews(),
  },
  Review: {
    author: (review) => **userLoader.load**(review.authorId),
  },
};
```

当每个`userLoader.load(review.authorId)`被调用时，它被分批并最终调用`batchGetUsers()`。因此，不需要 100 次单独调用数据库来获取用户详细信息，同样的事情只需一次调用就可以完成。

# N+1 和阿波罗联邦有什么关系？

Apollo 中查询规划器部分地但不是全部地为您处理 N+1。以[联邦演示](https://github.com/apollographql/federation-demo)为例，我们有一个**用户服务**，它只处理用户。

```
// User serviceconst typeDefs = gql`
  type User @key(fields: "id") {
    id: ID!
    name: String
    username: String
  }
`;const resolvers = {
  User: {
    __resolveReference(object) {
      return getUser(object.id);
    }
  }
};
```

另一个**评论服务**只处理评论，尽管在`Review`中我们引用了`User`类型，我们知道它将由用户服务处理。

```
// Review serviceconst typeDefs = gql`
  type Review @key(fields: "id") {
    id: ID!
    body: String
    author: User
    product: Product
  }

  extend type User @key(fields: "id") {
    id: ID! @external
  } extend type Product @key(fields: "upc") {
    upc: String! @external
  } extend type Query {
    top100Reviews: [Review]
  }
`;const resolvers = {
  Review: {
    author(review) {
      return { __typename: "User", id: review.authorId };
    }
  },
  // ...
};
```

在网关处，这些独立的模式被联合成一个单一的模式，如果我们像以前一样执行相同的查询，`top100Reviews`查询被委托给**审查服务**并由其解析，而解析`author`被委托给**用户服务**。

```
{
  top100Reviews {
    body
    author {
      name
    }
  }
}
```

起初，我们可能会怀疑解析`author`会多次调用**用户服务**，但事实并非如此。我们可以通过在**上下文**选项中记录传入的请求体来验证声明。

```
// User serviceconst server = new ApolloServer({
  schema: buildFederatedSchema([{ typeDefs, resolvers}]),
  **context: ({ req }) => {
    console.log(JSON.stringify(req.body, null, 2));
  },**
});
```

我们看到查询规划器将所有的用户 id 批处理到一个数组中，并将一个由**表示的数组**发送给负责解析引用的**用户服务**。

```
{
  "query": "query ($representations: [_Any!]!) {\n  _entities(representations: $representations) {\n    ... on User {\n      name\n    }\n  }\n}",
  "variables": {
    **"representations": [
      {
        "__typename": "User",
        "id": "1"
      },
      {
        "__typename": "User",
        "id": "2"
      },
      {
        "__typename": "User",
        "id": "1"
      },
      {
        "__typename": "User",
        "id": "2"
      },
**      // ... too long to list them all
    ]
  }
}
```

然后，**用户服务**将映射每个单独的 ID，并调用`__resolveReference()`来解析每个用户 ID。尽管网关不会多次调用**用户服务**，但是由于`getUser()`，解析引用仍然需要多次调用数据库。

```
// User serviceconst resolvers = {
  User: {
    __resolveReference(object) {
      return **getUser(object.id)**;  // **Bad**
    }
  }
};
```

# 解决方案

我们可以通过使用相同的`userLoader.load()`而不是`getUser()`来解决这个问题。数据加载器批量处理所有用户 id，并高效地对数据库进行一次调用。

```
// User serviceconst resolvers = {
  User: {
    __resolveReference(object) {
      return **userLoader.load(object.id);**  // **Good**
    }
  }
};
```

# 摘要

我们解释了 N+1 查询的缺陷以及如何使用数据加载器解决这个问题。我们还发现，Apollo Gateway 通过批处理引用有效地处理了对服务的委托，但是我们的解析器仍然需要显式地使用数据加载器来避免对数据库的多次调用。