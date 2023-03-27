# 使用 GraphQL 和 Apollo 实现自动打字

> 原文：<https://itnext.io/automated-typescript-typing-with-graphql-apollo-4ee2a92c2fda?source=collection_archive---------2----------------------->

![](img/389326c4cbf1d0d1625863b2c8af69bd.png)

照片由[大卫·托里斯](https://unsplash.com/@djjabbua?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)在 [Unsplash](https://unsplash.com/s/photos/apollo?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 拍摄

将 TypeScript 与 GraphQL 结合使用的一种方法是基于 GraphQL 模式生成一个 TypeScript 类型库，为我们可能在代码中使用的所有对象创建接口，这确实很好。

然而，有一种更安全的方法可以做到这一点，那就是在每种情况下都有唯一的类型，而不是到处都可以使用的共享类型。在本文中，我将解释这种方法的错误以及如何改进它的机制。

> 所有的代码都可以在这里找到:[https://github.com/thomasparsons/graph-apollo-typing](https://github.com/thomasparsons/graph-apollo-typing)

## 生成和使用类型的第一种方法

生成类型的一种机制是使用与上述项目中的*本机*文件夹中的`yarn generate-types`调用类似的方法:

`cd native && yarn generate-types`

这将从图中生成的模式文件转换为匹配的类型，因此:

```
type DogInfo {
  name: String!
  age: String
}
```

被转换为:

```
export type DogInfo = {
  __typename?: 'DogInfo',
  name: Scalars['String'],
  age?: Maybe<Scalars['String']>,
};
```

那么在代码中，可能会有这样的内容:

```
import {DogInfo} from "../generated/types"interface Props {
  dogInfo: DogInfo
}export default function DogInfo({dogInfo}: Props) {
  const {age, name} = dogInforender (
    <Text>{name}: {age}</Text> 
  )
}
```

## 这和阿波罗有什么关系

如果上面是 redux，那就 *ok* ，可能不完美，但是还可以……但是，在使用 GraphQL 时，由于响应是针对请求的数据的，所以需要更精确的输入，所以我们来看一个查询；在这个例子中，我们期待一组狗，包括它们的年龄和名字。

```
query getDogs {
  getDogs {
    dogInfo {
      age
      name
    }
  }
}
```

在这种情况下，最初的输入就可以了，但是，如果从查询中删除了属性:`age`，因为可能只需要名称，那么可能会出现一些问题。由于类型化在接口上仍然有`age`值，一个工程师在以后修改代码时，可能会尝试并呈现那个 *age* 值，但现在还不清楚它是否会有问题。

```
const {age, name} = dogInforender (
  <Text>{name}: {age}</Text> // this won't work as expected!
)
```

## 如果类型是从查询中生成的会怎么样？

除了使用上述模式之外，还有一种替代方法……

```
yarn generate-query-typesor:apollo codegen:generate generated --localSchemaFile=./generated/schema.json --target=typescript
```

本质上，这将为每个查询生成一组唯一的类型(突变等)。)以便可以导入生成的类型，有点像这样:

```
import {getDogs_getDogs_dogInfo} from "../Queries/generated/getDogsinterface Props {
  dogInfo: getDogs_getDogs_dogInfo
}export default function DogInfo({dogInfo}: Props) {
  const {age, name} = dogInforender (
    <Text>{name}: {age}</Text> 
  )
}
```

*注意:实际的组件逻辑不一定会改变*

在 IDE 或 CI 中，很快就会发现，如果查询中没有请求`age`值，就会出现 lint 错误，因为该值不在类型中。如果随后添加了`age`值，并且更新了类型，linter 就会通过。

同样，如果一个新值被添加到图中还不可用的查询中，那么 GraphQL lint 将失败，Apollo 类型生成器也将失败。

## 不利之处

有几个负面因素…

*   太丑了。毫无疑问，`import {getDogs_getDogs_dogInfo} from "../Queries/generated"`不是最吸引人的代码编写方式…
*   它需要运行大量的脚本来保持一切都是最新的，特别是`apollo codegen`命令来保持查询的输出类型是最新的。(你可以使用`--watch`命令来省去一些麻烦)
*   我不得不在`eslint`中关闭`camelcase`规则，因为目前没有办法在不使用不同的包(例如[https://graphql-code-generator.com/](https://graphql-code-generator.com/))的情况下移除下划线

## 普通类型还有位置吗？

现在最有可能的是需要一个已经就位的系统，但是，不是在从图中获取数据的时候，图中的数据可能占组件数据的 80%,另外 20%是状态管理，等等。为此，当然有必要进行类型化，尽管有人可能会认为形式、状态和突变应该使用或扩展自动突变类型。

![](img/f76f690bb0e4b45fc796036850c76b92.png)