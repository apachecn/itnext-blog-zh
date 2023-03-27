# 构建您的第一个 GraphQL 服务器

> 原文：<https://itnext.io/building-your-first-graphql-server-d5c4f88f5e82?source=collection_archive---------4----------------------->

在本文中，您将学习像`schema`和`resolver`这样的概念，最后您将学习如何创建一个 GraphQL 服务器并查询它。这涵盖了必要的基础知识。如果你已经知道了基础知识，看看[用 nodejs + express 开发一个 GraphQL 服务器](https://medium.com/@noringc/building-a-graphql-server-with-node-js-and-express-f8ea78e831f9)

![](img/2558d91fd0e9d11ebe723812547a3aaf.png)

定义模式有不同的方式。使用适合你的方法

*   **Raw 方法**，这使用了由`graphql`库公开的原语
*   **GraphQL 查询语言定义**，这里我们将使用方法`buildSchema()`，这是更容易使用的方法

原始方法旨在向您展示引擎盖下发生了什么。如果您只想知道如何用查询语言定义构建您的模式，那么请跳到本文的`buildSchema`部分

# 模式内容

该模式可以说由两部分组成:

*   **模式定义**，通过定义一个模式，我们定义了存在的实体以及可能进行的不同查询和变化
*   **解析器**，解析器是用户进行查询或变异时调用的函数。解析器与第三方 API 或数据库对话以执行操作，并最终向调用用户返回一个答案

## 模式定义

模式几乎包含以下概念:

*   **查询**，这是可查询的数据，可以是无参数的，也可以是带参数的
*   **自定义类型**，如果您对提供的基本类型不满意，您可以构建自己的类型
*   **突变**，这些是应该改变数据的“方法”，比如创建、更新或删除
*   **解析器**，这些是将数据返回给用户的函数
*   片段，这是一个可重用的模式，你可以在你的定义中的很多地方使用它
*   **别名**，这是一个重命名机制，允许我们重命名一个列
*   指令，可以把它想象成`if/else`，它允许你决定在你的查询中包含/排除哪些列

## 下决心者

解析程序是架构实例的一部分，但不是架构定义的一部分。解析器只是在您尝试调用查询或变异时做出响应的函数。解析函数要么直接用数据响应，要么用承诺响应。

# 原始的方法

在我们下面将要描述的*原始方法*中，模式定义和解析器是结合在一起的。我们将在一个定义中定义一个可以调用的资源及其对应的解析器。

要创建一个模式，您需要创建一个类型为`GraphQLSchema`的实例。创建这个实例有不同的方法，但是让我们先看看最原始的方法。在这种方法中，我们有以下类型`GraphQLSchema`:

```
new GraphQLSchema(options)
```

`options`属于`GraphQLSchemaConfig`类型，如下图所示:

```
export interface GraphQLSchemaConfig extends GraphQLSchemaValidationOptions { query: Maybe<GraphQLObjectType>; mutation?: Maybe<GraphQLObjectType>; subscription?: Maybe<GraphQLObjectType>; types?: Maybe<GraphQLNamedType[]>; directives?: Maybe<GraphQLDirective[]>; astNode?: Maybe<SchemaDefinitionNode>; extensionASTNodes?: Maybe<ReadonlyArray<SchemaExtensionNode>>;}
```

正如我们在上面看到的，我们现在唯一需要给它的强制字段是`query`。这是简单的询问/问题，我们可以看到它们属于`GraphQLObjectType`类型。所以让我们开始构建这样一个实例:

```
import { graphql, GraphQLSchema, GraphQLObjectType, GraphQLString,} from “graphql”;const schema = new GraphQLObjectType({ name: “RootQueryType”, fields: { hello: { type: GraphQLString, resolve() { return “world”; } } }});
```

上面我们可以看到我们已经创建了查询`hello`。`hello`被分配了一个对象，所以我们来分解这个对象:

*   **类型**，这是数据类型，要么是原始类型，要么是自定义类型
*   这是一个解析函数，简单地说，如果有人调用我，我应该用什么来回答

我们还可以像`description`一样声明更多的字段，但这两个是最重要的。

## 查询我们的模式

下一步是告诉 graphql 这个模式，以便我们可以使用它:

```
let query = `{ hello }`;graphql(schema, query).then(result => { res.json(result);});
```

上面你可以看到我们用两个不同的参数调用了`graphql`:

*   **模式**，我们刚刚定义的模式
*   **查询**，这是一个我们刚刚初始化的变量，它包含了 GraphQL 查询语言

让我们放大`query`看看它做了什么:

```
{
  hello
}
```

上面我们简单地查询了一个已知的查询`hello`，它会用`world`响应，因为查询它会调用`resolve()`方法。很好，我们刚刚在 GraphQL 中创建了一个`hello world`。暂停以获得效果:)

## 引入自定义类型

现在我们可以在 GraphQL 中使用很多原语，我们刚刚看到了一个`String`类型。有时候，这真的很有限，我们想要构造自己的类型。接下来让我们开始吧:

```
let humanType = new GraphQLObjectType({ name: "Human", fields: () => ({ id: { type: GraphQLString }, description: { type: GraphQLString }, name: { type: GraphQLString } })});
```

上面我们使用`GraphQLObjectType`来创建我们的自定义类型。属性`fields`允许我们列出我们的类型应该具有的所有不同属性，如`id`、`description`和`name`。

好了，我们有了一个自定义类型，让我们通过在我们的模式中使用它来使用它，如下所示:

```
let schema = new GraphQLSchema({ query: new GraphQLObjectType({ name: “RootQueryType”, fields: { hello: { type: GraphQLString, resolve() { return “world”; } }, person: { type: humanType, resolve() { return people[0]; } } } })});
```

上面我们添加了类型为`humanType`的`person`，因此使用了我们的自定义类型。因为我们添加了`person`作为可查询项，所以我们现在可以用这种类型扩展我们的查询，就像这样:

```
let query = `{ hello, person { name, description } }`;graphql(schema, query).then(result => { res.json(result);});
```

正如您在上面看到的，我们现在添加了以下内容:

```
{ person { name, description }}
```

我们正在查询`person`，我们知道 person 的类型是`humanType`，而`humanType`有字段`id`、`name`和`description`，我们选择选择其中的一个子集，即`name`和`description`。

到目前为止，完整的代码如下所示:

```
const people = [ { id: 1, name: “chris”, description: “viking” }, { id: 2, name: “maxim”, description: “viking” }, { id: 3, name: “sherry”, description: “viking” }, { id: 4, name: “ana”, description: “viking” }];let humanType = new GraphQLObjectType({ name: "Human", fields: () => ({ id: { type: GraphQLString }, description: { type: GraphQLString }, name: { type: GraphQLString } })});let schema = new GraphQLSchema({ query: new GraphQLObjectType({ name: "RootQueryType", fields: { hello: { type: GraphQLString, resolve() { return "world"; } }, person: { type: humanType, resolve() { return people[0]; } } }})});let query = `{ hello, person { name, description } }`;graphql(schema, query).then(result => { res.json(result);});
```

## 引入列表类型

到目前为止，我们已经描述了如何使用像`String`这样的原语，以及如何创建自定义类型`humanKind`。让我们看看如何使用列表类型。要使用列表类型，我们需要创建类型为`GraphQLList`的东西。让我们将它添加到现有的模式中，就像这样:

```
let schema = new GraphQLSchema({ query: new GraphQLObjectType({ name: "RootQueryType", fields: { hello: { type: GraphQLString, resolve() { return "world"; } }, person: { type: humanType, resolve() { return people[0]; } }, people: { type: new GraphQLList(humanType), resolve() { return people; } } }})});
```

上面我们添加了使用`GraphQLList`的`people`，你可以看到我们像调用方法一样调用它，并向它传递我们的自定义类型`humanType`来说明这是什么类型的列表。这就是事情的全部。

# buildSchema —使用查询语言定义

到目前为止，我们一直在以最痛苦的方式定义一个模式，使用像`GraphQLObjectType`、`GraphQLList`等类型，低级类型迫使我们写很多来产生我们的模式。

使用这种方法，我们将把模式定义从解析器中分离出来，所以我们需要单独创建它们

当然，这有些可读性，但有一个更好的方法。更好的方法是使用一个名为`buildSchema()`的方法，它将产生一个`GraphQLSchema`的实例，并使我们能够以一种可读性更好的方式定义模式。让我展示给你看:

```
import { buildSchema } from ‘graphql’;var schema = buildSchema(` type Query { hello: String, }`);var root = { hello: () => { return 'world' }}let query = `{ hello, person { name }, people { name, description } }`;graphql({ schema, source: query, rootValue: root}).then(result => { console.log(result);});
```

## **自定义类型**

要创建自定义类型，我们只需使用`type`关键字，并为我们的类型选择一个名称，就像这样:

```
var schema = buildSchema(` type Person { id: Int, name: String }, type Query { hello: String, person: Person }`);
```

你可以在上面看到我们做了两件事:

*   **创建**我们的类型`Person`
*   **用`Person`型的`person`延长**型的`Query`

这意味着我们现在需要为`person`添加一个解析函数，这样如果有人试图查询它，他们就不会得到错误，所以接下来让我们这样做:

```
const people = [{ id: 1, name: ‘chris’},{ id: 2, name: ‘maxim’}]var root = { hello: () => { return ‘world’ }, person: () => people[0]}
```

上面你可以看到我们创建了`people`数组，并且我们还扩展了我们的`root`，我们的解析器具有属性`person`。这里的最终结果是，现在有人可以查询`person`，它会工作

## **列表类型**

让我们在模式定义中添加另一个列表类型:

```
var schema = buildSchema(` type Person { id: Int, name: String }, type Query { hello: String, person: Person, people: [Person] }`);
```

上面我们已经添加了可查询的`people`，我们可以看到它是列表类型的，因为它使用了方括号`[Person]`。此后，我们需要为`person`添加一个我们现在已经习惯的解析器函数:

```
var root = { hello: () => { return 'world' }, person: (id) => people.find(p => p.id === id), people: () => people}
```

## **参数化查询**

到目前为止，我们已经看到了没有参数的简单查询，但是很可能我们需要向查询发送一些参数来过滤我们的响应。通过首先正确定义我们的模式，我们很容易做到这一点。我们已经有了一个现有的`person`属性，所以让我们更新那个属性以获取一个参数，就像这样:

```
var schema = buildSchema(` type Person { id: Int, name: String }, type Query { hello: String, person( id: Int!): Person, people: [Person] }`);
```

`person`现在看起来如下:

```
person( id: Int!): Person
```

这看起来像是方法调用的签名。

## **突变**

突变是我们改变应用程序中数据的方式，因此我们可以对资源执行创建、更新或删除等操作。要定义一个变异，我们只需要创建保留类型`Mutation`，就像这样:

```
type Mutation { // mutation // another mutation}
```

一旦我们有了以上这些，我们就可以开始考虑我们想做什么了。考虑到上面关于人员列表的叙述，能够向该列表添加人员是有意义的，因此让我们创建这样一个变体，因此我们将模式更新为:

```
input PersonInput { name: String!},type Mutation { addPerson(person: PersonInput!): String}
```

上面的内容告诉我们，我们能够调用一个名为`addPerson()`的方法/变异，它将一个`person`作为输入参数，并以一个字符串结束响应。我们在这里添加了一个新的类型`input`的构造。当处理突变时，如果我们想发送比原语更复杂的东西(字符串、布尔等)，我们需要构造类型`input`的东西。是的，我们的`PersonInput`看起来有点简单，在这种情况下一个原语可能就足够了，但是我选择向您展示如何发送一个更复杂的输入，如果您愿意的话。

下一步是为此定义一个解析器，这样如果有人调用`addPerson()`，我们就知道该怎么做:

```
const people = [{ id: 1, name: 'chris'}]const addPerson = (person) => {const nextId = people.length === 0 ? 1 : people[people.length -1].id + 1people = […people, {…person, { id : nextId } }]}var root = { hello: () => { return 'world' }, person: (id) => people.find(p => p.id === id), people: () => people, addPerson: (args) => { const { person } = args; addPerson(person); return 'success' }}
```

加上这一点，我们已经创建了一个特定的突变`addPerson()`，我们已经定义了一个解析器。

现在我们知道了创建 GraphQL 服务器的基础。

# **总结**

我们已经知道有两种主要的方法来声明一个模式:

*   **生法**，多向式
*   **使用`buildSchema()`的 GraphQL 模式定义**方法

我们还介绍了如何通过调用`resolvers`来回答查询，这些函数需要从某个地方获取数据。

最后，我们学会了如何创造突变。

接下来，我们将介绍如何创建一个模式，编写一些解析器，并启动和运行一个 GraphQL 服务器

[GraphQL + Node.js Express](https://medium.com/@noringc/building-a-graphql-server-with-node-js-and-express-f8ea78e831f9)