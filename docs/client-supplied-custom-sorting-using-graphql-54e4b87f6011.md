# 使用 GraphQL 的客户端提供的自定义排序

> 原文：<https://itnext.io/client-supplied-custom-sorting-using-graphql-54e4b87f6011?source=collection_archive---------5----------------------->

![](img/7ca7da9cd3f554ba1c504fdea69f1b46.png)

> 本故事被移至:[https://jonathancardoso . com/en/blog/client-supplied-custom-sorting-using-graph QL/](https://jonathancardoso.com/en/blog/client-supplied-custom-sorting-using-graphql/)

根据客户端发送的参数，对从 API 返回的数据进行排序的需求并不少见。在 REST 中，我们可以只使用查询字符串，但是在 GraphQL 中如何做呢？

大多数数据库允许同时对多个字段进行排序，并且排序顺序可以各不相同，一种将此转换为 GraphQL 的方法，也是我一直在使用的方法，是使用枚举和对象输入参数。

## 例子

假设我们有一个解析器`getUsers: [User!]!`，它从 MongoDB 返回用户，我们按照`name`和`createdAt`对结果进行排序:

```
const users = await ctx.db.users.find().sort({
  name: 1, // ascending
  createdAt: -1 // descending
});
```

但是我们希望允许客户端指定将要排序的字段，以及它们各自的顺序。

首先，我们可以有一个`Direction`枚举，它将有值`ASC`和`DESC`:

```
enum DirectionEnum {
  ASC
  DESC
}
```

使用`graphql-js`,这可以定义如下:

```
import { GraphQLEnumType } from 'graphql';export const DirectionEnumType = new GraphQLEnumType({
  name: 'DirectionEnum',
  values: {
    ASC: {
      value: 1,
    },
    DESC: {
      value: -1,
    },
  },
});
```

现在让我们创建一个 enum 来表示允许在我们的`User`上排序的字段:

```
enum UserSortFieldEnum {
  NAME
  CREATED_AT
}
```

在`graphql-js`中:

```
import { GraphQLEnumType } from 'graphql';export const UserSortFieldEnumType = new GraphQLEnumType({
  name: 'UserSortFieldEnum',
  values: {
    NAME: {
      value: 'name',
    },
    CREATED_AT: {
      value: 'createdAt',
    },
  },
});
```

现在，为了将这两者连接在一起，我们将制作一个名为`UserSort`的[输入类型](https://graphql.org/graphql-js/mutations-and-input-types/):

```
input UserSort {
  field: UserSortFieldEnum!
  direction: DirectionEnum!
}
```

`graphql-js`:

```
import { GraphQLInputObjectType, GraphQLNonNull } from 'graphql';

import { DirectionEnumType } from './DirectionEnum';
import { UserSortFieldEnumType } from './UserSortFieldEnum';const UserSortInputType = new GraphQLInputObjectType({
  name: 'UserSort',
  fields: () => ({
    field: {
      type: GraphQLNonNull(UserSortFieldEnumType),
    },
    direction: {
      type: GraphQLNonNull(DirectionEnumType),
    },
  }),
});
```

那么我们的`getUsers`解析器将变成:

```
getUsers(sort: [UserSort!]): [User!]!
```

现在，客户端可以查询它并指定他们需要的任何排序顺序:

```
query GetUsers {
  getUsers(sort: [
    {
      field: NAME,
      direction: DESC,
    },
    {
      field: CREATED_AT,
      direction: ASC,
    }
  ]) {
    # ...
  }
}
```

如果您正在使用 MongoDB，并且打算开始使用它，`@entria/graphql-mongo-helpers`有一个非常简单的助手可以将客户端提供的`sort`参数转换成可以在 MongoDB `sort()`方法上使用的参数。

[](https://github.com/entria/graphql-mongo-helpers) [## entria/graphql-mongo-helpers

### graphql-mongo-helpers。在 GitHub 上创建一个帐户，为 entria/graphql-mongo-helpers 的开发做出贡献。

github.com](https://github.com/entria/graphql-mongo-helpers) 

它被称为`buildSortArg`，在我们的`getUser`解析器上使用它会是这样的:

```
// ...
import { buildSortFromArg } from '@entria/graphql-mongo-helpers'; // ... inside getUser resolver:
const users = await ctx.db.users.find().sort(buildSortFromArg(args.sort));
```

该库还分别为方向枚举、`DirectionEnum`和`DirectionEnumType`导出了 SDL 定义和 GraphQLObject。

让我知道你对这个想法的想法！我的推特: [@_jonathancardos](https://twitter.com/_jonathancardos)