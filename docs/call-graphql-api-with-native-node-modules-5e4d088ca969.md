# 使用 Node 调用 GraphQL API

> 原文：<https://itnext.io/call-graphql-api-with-native-node-modules-5e4d088ca969?source=collection_archive---------5----------------------->

## 使用本机模块

[点击这里在 LinkedIn 上分享这篇文章](https://www.linkedin.com/cws/share?url=https%3A%2F%2Fitnext.io%2Fcall-graphql-api-with-native-node-modules-5e4d088ca969)

乍一看，调用 GraphQL API 服务器似乎很复杂。有几个不错的选择，比如[阿波罗取](https://github.com/apollographql/apollo-fetch)。但是，如果您正在构建一个节点应用程序，并且想要一个使用本机节点模块的简单解决方案，这里是您的解决方案。在 Credit Karma，我们希望使用 GitHub GraphQL API 来搜索我们存储库的某些方面，所以我开发了一个简单的客户端，并想分享我的发现。

因此，在着手构建应用程序之前，我确立了以下目标:

*   它应该没有任何依赖性
*   应该能够加载以 GraphQL 模式语言编写的查询
*   查询应该能够使用从命令行传递的变量
*   支持在多个查询中使用的片段
*   通过多次调用同一查询来支持分页
*   用纯函数构建
*   (音乐)可设定的
*   可流式传输

在这个例子中，我们将调用 GitHub GraphQL API 来获取一个存储库的观星者列表。我们将发送的主要查询:

```
query StarGazers($owner: String!, $name: String!, $cursor: String) {
  repository(owner: $owner, name: $name) {
    stargazers(first: 100, after: $cursor) {
      ...starGazerFields
    }
  }
}
```

我不打算讨论 GraphQL 语言的细节或如何构建查询，但是如果您是 GraphQL 新手，下面的文章可以提供一些见解:

*   [graph QL 简介](http://graphql.org/learn/)
*   [查询和突变](http://graphql.org/learn/queries/)
*   [传递参数](http://graphql.org/graphql-js/passing-arguments/)
*   [GraphQL 模式语言备忘单](https://wehavefaces.net/graphql-shorthand-notation-cheatsheet-17cd715861b6)

让我们从分解我在库中创建的不同函数开始。请记住，目标之一是提供一个纯函数库，将这些函数组合在一起，以便轻松地发出请求。

# loadQuery

我们的目标之一是使用 GraphQL 模式语言文件来编写查询。上面的查询示例中展示了其中一个文件的示例。我们需要从构建一个从磁盘加载 GraphQL 文件的函数开始

```
const { readFile } = require('fs')
const { resolve } = require('path')

const loadQuery = (fileName) => new Promise((res, reject) => {
    readFile(resolve(fileName), 'utf-8', (err, data) => {
        (err) ? reject(err) : res(data)
    })
})
```

您会注意到，我将标准的文件系统 readFile API 包装在一个 promise 中。这样做的好处之一是使用。then()符号。稍后会详细介绍。

现在我们有了一个加载查询文件的工具，您应该检查查询中片段的使用:

```
...starGazerFields
```

但是这个片段没有在查询文件中定义，因此如果我们将这个查询发送到服务器，它将返回一个错误。下面是这个片段的一个示例:

```
fragment starGazerFields on StargazerConnection {
    pageInfo {
        endCursor
    }
    nodes {
        company
        bio
        name
        location
    }
}
```

# appendQuery

为了使原始查询能够工作，我们需要在将查询发送到服务器之前将上面的片段附加到查询中。我为这个用例创建了以下函数:

```
const appendQuery = (fileName) => (query) =>
    loadQuery(fileName).then((results) => query + results)
```

这种语法可能看起来有点奇怪，但它是我试图实现的可组合性的一部分。appendQuery 函数接受一个文件名并返回另一个函数，该函数接受原始查询并将文件追加到该查询中。在本文的后面，我将提供一个例子来说明这些是如何组合在一起的。

# 建筑主体

现在我们已经加载了查询，我们需要构建将要发送到 GitHub 的 http POST 的主体。根据是否需要向查询传递变量，我为此创建了两个不同的函数:

```
const buildBody = (query) => Promise.resolve({
    body: JSON.stringify({query})
})

const buildBodyWithVariables = (variables) => (query) => Promise.resolve({
    body: JSON.stringify({query, variables})
})
```

# 构建标题

一旦我们有了一个字符串体，我们就可以构建请求所需的 header 对象。

```
const buildHeaders = ({user, token}) => ({body}) => Promise.resolve({
    body,
    headers: {
        'Content-Type': 'application/json',
        'Content-Length': body.length,
        'User-Agent': `${user}`,
        'Authorization': `token ${token}`
    }
})
```

**观察**:你可能已经注意到，在这个函数以及前两个函数中，我使用了 Promise.resolve 来立即返回承诺的结果。我继续使用承诺来保持可组合性与查询加载器一致。

# buildRequestOptions

既然我们已经构建了请求的有效负载，我们可以定义我们想要调用的端点的细节。

```
const buildRequestOptions = ({headers, body}) => Promise.resolve({
    body,
    options: {
        method: 'POST',
        host: 'api.github.com',
        path: '/graphql',
        port: 443,
        headers
    }
})
```

**观察**:我对函数参数使用了对象销毁。这是识别调用该函数所需的对象结构的一个很好的方法。

# 生成请求

让我们将请求发送到 GraphQL API 服务器。我将来自服务器的结果批量处理成一个 JSON 对象结果。

```
const { request } = require('https')

const makeRequest = ({options, body}) => new Promise((resolve, reject) => {
    request(options, (res) => {
        let result = ''
        res.on('data', (chunk) => result += chunk)
        res.on('end', () => resolve(JSON.parse(result)))
        res.on('error', (err) => reject(err))
    }).write(body)
})
```

**观察**:我再次将 Node 的标准请求方法的细节包装在一个承诺中，使其在用于流时更具可组合性。

# 尝试这一切

那么，我们如何使用所有这些函数来发出 GraphQL 请求呢？

```
loadQuery('./starGazerFragment.gql')
    .then(appendQuery('./searchStarGazers.gql'))
    .then(buildBodyWithVariables({owner: 'hapijs', name: 'hapi'}))
    .then(buildHeaders({user: '...', token: '...'}))
    .then(buildRequestOptions)
    .then(makeRequest)
    .then((results) => console.dir(results, {depth: null}))
```

这可能看起来相当复杂，有几个移动的部分，但是，它提供了过程中的任何步骤都很容易被替换的灵活性，使其高度可定制。

# 使其可流动

这可以使用流式节点本地库轻松完成。下面我把前面的例子变成了一个函数，我正在一个可读的流中使用它。

```
const { Readable } = require('stream')

const sendRequest = () => loadQuery('./starGazerFragment.gql')
    .then(appendQuery('./searchStarGazers.gql'))
    .then(buildBodyWithVariables({owner: 'hapijs', name: 'hapi'}))
    .then(buildHeaders({user: '...', token: '...'}))
    .then(buildRequestOptions)
    .then(makeRequest)

const createStream = () => new Readable({
    objectMode: true,
    read(opts) {
        if (!this.requestCompleted) {
            return sendRequest().then((data) => {
                this.requestCompleted = true
                this.push(data)
            })
        } else {
            this.push(null)
        }
    }
})
```

# 结束语

尽管有许多不同的库可用于发出 GraphQL 请求，但我发现构建一个简单的 60 行模块而不依赖外部是令人满意的。我已经使用这个小库成功地执行了几个不同的 GitHub GraphQL 查询。API 的可组合性导致了一些样板文件，但是 API 的灵活性似乎提供了一个很好的平衡。

请在下面的评论中告诉我你的想法，或者直接在 Twitter @nancenick 上与我联系。