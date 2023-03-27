# 介绍 Saturn-GQL:开发 GraphQL API 的自以为是的方法

> 原文：<https://itnext.io/introducing-saturn-gql-an-opinionated-way-to-develop-graphql-apis-d99bf4d0790e?source=collection_archive---------3----------------------->

![](img/3df9b4410841e1aa35b2cc2b4ee2f7b1.png)

[阿波罗 17 号](https://en.wikipedia.org/wiki/Apollo_17)土星五号火箭由不知名的 NASA 雇员拍摄，由基普·蒂格扫描—[http://www.hq.nasa.gov/alsj/a17/images17.html](http://www.hq.nasa.gov/alsj/a17/images17.html)(图片编号 KSC-72PC-589)直接链接:[http://www.hq.nasa.gov/alsj/a17/ap17-KSC-72PC-589.jpg](http://www.hq.nasa.gov/alsj/a17/ap17-KSC-72PC-589.jpg)，公共领域，[https://commons.wikimedia.org/w/index.php?curid=452085](https://commons.wikimedia.org/w/index.php?curid=452085)

> TL；DR: Saturn-GQL 提供了一个接口来帮助你组织你的 GraphQL 端点。它是一个自以为是的库，在你的 GraphQL API 中实施模块化。它利用了 GraphQL 模式语言，这使得它非常适合利用 Apollo graphql-tools 库的项目。

> [点击这里在 LinkedIn 上分享这篇文章](https://www.linkedin.com/cws/share?url=https%3A%2F%2Fitnext.io%2Fintroducing-saturn-gql-an-opinionated-way-to-develop-graphql-apis-d99bf4d0790e%3Futm_source%3Dmedium_sharelink%26utm_medium%3Dsocial%26utm_campaign%3Dbuffer)

GraphQL 已经迅速成为创建 API 的头号工具。与旧的方式相反，客户机能够指定它想要接收什么数据，这有助于开创软件开发的新时代。有了理解客户机请求的能力，服务器现在可以限制它查询的数据，从而提高性能。但是这篇文章不是关于 GraphQL 的。它是关于让 GraphQL 成为一个充满活力的生态系统的工具。

正如开源领域最好的想法一样，围绕 GraphQL 及其参考实现已经创建了一个充满活力的工具生态系统。Apollo(由 Meteor Development Group 创建)是一家将自己定位为 GraphQL 生态系统中开源工具和付费产品的首选目的地的公司。

`graphql-tools`就是这样一个开源库。它来自 Apollo，是一种构建 GraphQL 模式的自以为是的方法。它允许将业务逻辑(如何查询数据存储和返回数据)从 GraphQL 模式定义中分离出来(例如，如何进行变异、查询、自定义标量等..是在 GraphQL 中定义的)。

在我工作的开源团队中，我们开发并开源了 [Saturn-GQL](https://github.com/electric-it/saturn-gql) ，它建立在阿波罗`graphql-tools`提出的想法之上。 **Saturn-GQL 提供了一个自以为是但非常简单的接口来生成 GraphQL 模式**。只需将包含 GraphQL 端点的顶级目录传递给它(后面的例子)，并在另一端获得一个用 GraphQL 模式语言编写的完全充实的模式。例如，这可以传递给`graphql-tools`中的`makeExecutableSchema`工具，通过您的 HTTP 服务器提供服务。

正如我提到的，Saturn-GQL 是创建 GraphQL API 的一种自以为是的方式。如果您在一个大型团队中工作，或者有跨项目团队，那么利用像 Saturn-GQL 这样有主见的库可以减少总的开发时间，并且可以为项目的新(内部)队友提供一个更平滑、更有效的入门过程。让我们看一个简单的例子。

首先，用`npm install saturn-gql`安装土星-GQL

下面展示了一个示例目录结构，它将作为 GraphQL API 服务器的基础。

```
myapp
|___ src/
  |___ app.js
  |___ api/
    |___ users/
      |___ index.js
      |___ type.js
      |___ queries.js
      |___ mutations.js
      |___ resolvers.js
    |___ teams/
      |___ index.js
      |___ type.js
      |___ queries.js
      |___ mutations.js
      |___ resolvers.js 
```

这里是固执己见的部分。为了使用 Saturn-GQL，上面目录结构中列出的文件必须具有类似于下面代码示例 [](#c8d2) 的内部结构。我们将创建上面的`users`集合的示例实现。

```
################
# type.js
################export const type = `
  type User {
    id: Int!
    firstName: String
    lastName: String
  }
`;

export const typeQuery = `
  users: [User]
  getUser(id: Int!): User
`;

export const typeMutation = `
  addUser(newUser: User!): User
`;
```

现在已经定义了您的类型，我们可以创建包含应用程序业务逻辑的文件。

```
################
# queries.js
################export const queries = {
  users(_) { return database.getUsers(); },
  getUser(_, { id }) { return database.getUserById(id); },
};
```

下面是一个示例`mutations.js`文件:

```
################
# mutations.js
################export const mutations = {
  addUser(_, { user }) => {
    if (!user) {
      throw new Error('Can\'t create user with empty user object');
    }
    const newUser = database.createUser(user);
    return newUser;
  },
};
```

在每个集合中有一个创建自定义解析器的选项，但是为了简洁，我将只留下一个到官方 Saturn-GQL 文档的链接。最后只需将其全部导出到 index.js 文件中。**index . js 文件导出的项目的命名很重要**。如果它们的名字和下面的不一样，土星将不会像预期的那样工作。就像我说的，[它非常固执己见](https://github.com/electric-it/saturn-gql/blob/master/src/index.js#L26)。

```
################
# index.js
################import { type, typeMutation, typeQuery } from './type';
import { queries } from './queries';
import { mutations } from './mutations';export {
  type,
  typeMutation,
  typeQuery,
  queries,
  mutations,
};
```

所以现在你要做的就是创建一个新的`Saturn`对象并运行`makeSchema()`。然后你可以把它从`graphql-tools`传递给`makeExecutableSchema()`函数…

```
import Saturn from 'saturn';
const saturn = new Saturn(`${__dirname}/api`);// Graphql Schema
const schema = makeExecutableSchema(saturn.makeSchema());export default schema;
```

…您就可以愉快地构建 GraphQL API 了！然后，您可以将它导入到 Apollo express-graphql 服务器，或者您计划为 graphql 端点提供服务的服务器。

希望你喜欢我们的新图书馆！就像我上面提到的，这是一种非常固执的实现 GraphQL 的方式，当然也不是唯一的方式。Saturn-GQL 只是为 GraphQL 生态系统增加价值的众多包中的一个。它的开发是为了帮助我们促进不同开发团队之间的合作和理解，同时快速加入新的团队成员。我们希望它也能为你的团队做同样的事情！

谢谢你坚持到现在。我们对扩大和扩展功能持开放态度，所以请随意[提出拉请求](https://github.com/electric-it/saturn-gql)！

`mutations.js`和`resolvers.js`文件是可选的，因为你并不总是需要一个目录中的变异或自定义解析器。