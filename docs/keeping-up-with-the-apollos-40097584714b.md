# GraphQL:跟上阿波罗的步伐

> 原文：<https://itnext.io/keeping-up-with-the-apollos-40097584714b?source=collection_archive---------1----------------------->

![](img/5dcd6585e97ceffd1b6a1a2099df2b52.png)

在[上一篇文章](/graphql-how-far-are-we-gone-77f5ae8fc2e5)中，我谈到了围绕 GraphQL 萌芽的不同技术。今天我要特别关注一下[阿波罗 GraphQL](https://www.apollographql.com/) 的作品。这是一个开发人员社区，它似乎站在客户端实现的前沿，并为他们的技术提供了机会。这是一件好事，因为 Apollo GraphQL 的实现非常简单，易于上手。但是新技术带来了新的挑战。

## 问题

[React Apollo 2.1](https://dev-blog.apollodata.com/introducing-react-apollo-2-1-c837cc23d926) 已经发布，有一些有趣的事情需要了解。以前的版本采用了更类似 Redux 的方法将组件连接到 Apollo 客户机。好处是它是 100%向后兼容的。下面是我使用 [react-apollo 2.0.4](https://github.com/kimobrian/GraphQL-Course/blob/lesson3F/packages/client/package.json#L26) 的一个项目的摘录。这段代码从 GraphQL 服务器获取图书列表。请注意我们是如何将组件连接到客户机的。

```
...import gql from "graphql-tag";
import { graphql } from "react-apollo";...export const getBooksQuery = gql`  
  query getAllBooks {    
     books: fetchAllBooks {      
        id      
        title      
        description    
      }  
   }`;const BooksComponent = (props) => {  
  let { error, loading, books } = props.data;  
  if (error) return <div> An Error Occurred</div>;  
  else if (loading)    
       return ( <div style={styles.loaderSection}> 
                   <CircularProgress />                        
                </div>    
              );  
   else return ( /* Render the list of books here */ );
}; const BooksComponentWithData = graphql(getBooksQuery)(BooksComponent);export default BooksComponentWithData;
```

通过使用`graphql`的上述连接，我们可以访问组件内部的三个重要道具，我们可以使用它们向用户显示适当的消息。

```
let { error, loading, books } = props.data;
```

我们也可以用`withApollo`代替`graphql`来连接组件。

我跳过了次要的细节和导入，这是 Github 上的完整组件。

[](https://github.com/kimobrian/GraphQL-Course/blob/lesson3F/packages/client/app/components/BooksComponent.jsx) [## kimobrian/graph QL-课程

### 一个完整的 GraphQL 课程 monorepo

github.com](https://github.com/kimobrian/GraphQL-Course/blob/lesson3F/packages/client/app/components/BooksComponent.jsx) 

现在这里有一个来自 Apollo GraphQL 迁移文档的例子，差别非常明显，而且更“有反应”。

```
import { Query } from "react-apollo";
import gql from "graphql-tag";

const ExchangeRates = () => (
  <Query
    query={gql`
      {
        rates(currency: "USD") {
          currency
          rate
        }
      }
    `}
  >
    {({ loading, error, data }) => {
      if (loading) return <p>Loading...</p>;
      if (error) return <p>Error :(</p>;

      return data.rates.map(({ currency, rate }) => (
        <div key={currency}>
          <p>{`${currency}: ${rate}`}</p>
        </div>
      ));
    }}
  </Query>
);
```

我们仍然可以像以前一样访问加载、数据和错误。注意来自`react-apollo`的`Query`组件。这里的另一个优点是 Apollo Client 自动缓存数据，因此如果您运行查询两次，就不会产生加载指示符。

上述实现将在组件加载时自动触发查询。如果您希望查询由用户手动触发，该怎么办？这就是我们得到[](https://www.apollographql.com/docs/react/essentials/queries.html#manual-query)****的地方。**这是我从阿波罗文档中借用的另一个片段。在这种情况下，我们可以在 render prop 函数中直接访问客户端。**

```
import React, { Component } from 'react';
import { ApolloConsumer } from 'react-apollo';

class DelayedQuery extends Component {
  state = { dog: null };

  onDogFetched = dog => this.setState(() => ({ dog }));

  render() {
    return (
      <ApolloConsumer>
        {client => (
          <div>
            {this.state.dog && <img src={this.state.dog.displayImage} />}
            <button
              onClick={async () => {
                const { data } = await client.query({
                  query: GET_DOG_PHOTO,
                  variables: { breed: "bulldog" }
                });
                this.onDogFetched(data.dog);
              }}
            >
              Click me!
            </button>
          </div>
        )}
      </ApolloConsumer>
    );
  }
}
```

## **突变**

**下面是一个示例组件，它通过变异创建一个新的图书记录。**

```
import gql from "graphql-tag";
import { graphql } from "react-apollo";
import { getBooksQuery } from "./BooksComponent"; /* Re-use the
          query from the Query section *//* Create the mutation */export let createBookMutation = gql`  
  mutation createBook($title: String!, $description: String!) {
    createBook(title: $title, description: $description) {
       id      
       title      
       description          
  }  
}`;class NewBook extends Component {
  ...
  createBook () {
    ... 
    /* We can access the mutation method through the props*/
    this.props.***createNewBook***(this.state.title,    this.state.description).then(data=> {/* Logic for result */});
    ...
  } render() {/* Render Logic(Form to create book) */}
}const NewBookComponentWithData = graphql(createBookMutation, {          props: ({ mutate }) => ({    
    ***createNewBook***: (title, description) => mutate({ 
      variables: { title, description },          
      update: (store, { data: { createBook } }) => { 
         /*Read the data from our cache for this query.  */        
         try {            
            const data = store.readQuery({ query: getBooksQuery }); 
            /* Add our book from the mutation to the end */  
            data.books.push(createBook);
            /* Write our data back to the cache. */
            store.writeQuery({ query: getBooksQuery, data });
         } catch(err){   
            console.log("Cache Error:", err);
         }
      }, 
    })
   })
  })(NewBook); export default NewBookComponentWithData;
```

**这是 Github 上的完整组件。**

**[](https://github.com/kimobrian/GraphQL-Course/blob/lesson3F/packages/client/app/components/NewBook.jsx) [## kimobrian/graph QL-课程

### 一个完整的 GraphQL 课程 monorepo

github.com](https://github.com/kimobrian/GraphQL-Course/blob/lesson3F/packages/client/app/components/NewBook.jsx) 

即使删除了乐观 UI 部分，连接看起来仍然太多。这相当于`mapDispatchToProps`，与其对应的查询相比，看起来并不简单。

下面是 react-apollo 2.1 中类似于实现的 [Apollo GraphQL 文档](https://www.apollographql.com/docs/react/essentials/mutations.html)的摘录。

```
import gql from "graphql-tag";
import { Mutation } from "react-apollo";

const ADD_TODO = gql`
  mutation ***addTodo***($type: String!) {
    addTodo(type: $type) {
      id
      type
    }
  }
`;

const AddTodo = () => {
  let input;

  return (
    <Mutation mutation={ADD_TODO}>
      {(***addTodo***, { data }) => (
        <div>
          <form
            onSubmit={e => {
              e.preventDefault();
              ***addTodo***({ variables: { type: input.value } });
              input.value = "";
            }}
          >
            <input
              ref={node => {
                input = node;
              }}
            />
            <button type="submit">Add Todo</button>
          </form>
        </div>
      )}
    </Mutation>
  );
};
```

上面的实现看起来更加清晰和直接。

> 在上面的代码片段中，我们将一个用`gql`函数包装的 GraphQL 变异字符串传递给`this.props.mutation`，并向`this.props.children`提供一个告诉 React 要呈现什么的函数。`Mutation`组件是使用[渲染属性](https://reactjs.org/docs/render-props.html)模式的 React 组件的一个例子。React 将调用您提供的 render prop 函数和一个包含加载、错误、被调用和数据属性的对象。
> 
> 首先，创建您的 GraphQL 变体，将其包装在`gql`中，并将其传递给`Mutation`组件上的`mutation` prop。`Mutation`组件也需要一个函数作为子组件(也称为渲染属性函数)。render prop 函数的第一个参数是 mutate 函数，这是告诉 Apollo 客户端您想要触发一个突变的方式。变异函数可选取`variables`、`optimisticResponse`、`refetchQueries`、`update`；然而，您也可以将这些值作为道具传递给`Mutation`组件。在示例中，注意使用了 mutate 函数(名为`addTodo`)来提交包含变量的表单。
> 
> render prop 函数的第二个参数是一个对象，它包含您对`data`属性的变异结果，以及用于`loading`的布尔值，如果变异函数是`called`，除了`error`。如果您想忽略变异的结果，将`ignoreResults`作为一个道具传递给变异组件。

更新缓存的示例代码可能如下所示:

```
...
<Mutation
      mutation={ADD_TODO}
      update={(cache, { data: { addTodo } }) => {
        const { todos } = cache.readQuery({ query: GET_TODOS });
        cache.writeQuery({
          query: GET_TODOS,
          data: { todos: todos.concat([addTodo]) }
        });
      }}
    >
...
```

实现非常类似于以前的版本，只是更干净。

## 阿波罗-助推

以前，我们需要安装几个(很多)包来设置 Apollo 客户端。Apollo boost 带来了所有这些包，并且只允许安装一个包。

Apollo Boost 包含了一些我们认为对于使用 Apollo 客户端开发来说必不可少的包。这是盒子里的东西:

*   所有神奇的事情都发生在这里
*   `apollo-cache-inmemory`:我们推荐的缓存
*   `apollo-link-http`:用于远程数据获取的阿波罗链接
*   `apollo-link-error`:用于错误处理的阿波罗链接
*   `apollo-link-state`:本地状态管理的 Apollo 链接
*   `graphql-tag`:为您的查询&突变导出`gql`函数

比较以下不带和带 apollo-boost 的 npm 命令。

```
# Without apollo-boost
npm install apollo-client apollo-cache-inmemory apollo-link-http apollo-link-error apollo-link --save# With apollo-boost
npm i apollo-boost graphql react-apollo@beta -S
```

这使它变得容易，因为我们管理更少的包。

我希望这能让您了解 GraphQL 的客户端实现。

我将欣赏掌声，积极的反馈和批评。我目前也在关注 Graphcool 社区，并将发布任何重大变化的更新。**