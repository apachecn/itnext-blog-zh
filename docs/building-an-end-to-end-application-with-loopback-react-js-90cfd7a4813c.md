# 使用 LoopBack & React.js 构建端到端应用程序—第 3 部分:GitHub API 结果中的分页

> 原文：<https://itnext.io/building-an-end-to-end-application-with-loopback-react-js-90cfd7a4813c?source=collection_archive---------5----------------------->

![](img/c0d3f1eba9559532ba53227e8b143c0e.png)

照片由[昂素敏](https://unsplash.com/@kaunglay1?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)在 [Unsplash](https://unsplash.com/s/photos/pages?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄

[在之前的](https://mobilediana.medium.com/building-an-end-to-end-application-with-loopback-react-js-part-2-creating-service-proxy-7ffac2bd7980)中，我们在一个回送应用程序中创建了连接到 GitHub APIs 的服务代理。默认情况下，结果每页包含 30 个项目。我们可以增加这个数字，但最多只能增加到 100。在本文中，我将向您展示如何遍历页面以获取我们需要的所有项目。

如果你想看看完成后的应用程序是什么样子，你可以点击[这里](https://github.com/dhmlau/loopback4-example-github)。

# **关于 GitHub API 分页的基本知识**

在我们开始之前，让我们看看 GitHub API 中分页是如何工作的。

根据这个关于 GitHub API 分页的文档[https://docs . GitHub . com/en/rest/guides/traversing-with-pagination](https://docs.github.com/en/rest/guides/traversing-with-pagination)，分页信息在`Link`头中。例如

```
Link: <https://api.github.com/search/code?q=addClass+user%3Amozilla&page=2>; rel="next",
  <https://api.github.com/search/code?q=addClass+user%3Amozilla&page=34>; rel="last"
```

这意味着，如果一个`Link`头存在，我们可以使用`rel="next"`的 url 作为下一个 API 调用来获得下一组结果。为了获得所有结果，我们将继续检查响应上的`Link`头，直到不再有“next”链接。

# 回顾一下我们为数据源创建的内容

在回顾我们之前创建的[数据源](https://github.com/dhmlau/loopback4-example-github/blob/main/src/datasources/githubds.datasource.ts)时，有两点需要注意:

*   `fullResponse:true`:这是必需的，因为分页信息在响应头中
*   `getIssuesByURL`函数:由于下一组结果的 API 端点已经可用，如果我们有这个额外的函数，可能会更容易。

# 在 GHQueryController 中添加新函数

我们将在`GHQueryController`中做一些改变，这样我们就可以继续评估`Link`头以获得下一页结果。

我们将创建以下 3 个函数:

*   这实际上与分页无关，而是整理我们之前创建的代码，因为这会在多个地方被调用。
*   `getNextLink`:这个新函数检查`Link`头并找到与`rel="next"`相关的 url。当我们到达最后一页时，这个函数返回 null。
*   `getIssueByURL`:这个函数映射到我们之前创建的服务代理。在我们获得“下一个”链接后，这个函数将被调用以获得更多的结果。

# 把东西放在一起

现在我们已经准备好了一切，让我们对`getIssuesByLabel`函数进行更多的修改，以利用我们在上一节中创建的函数。

```
 [@get](http://twitter.com/get)('/issues/repo/{repo}/label/{label}')
  async getIssuesByLabel(
    [@param](http://twitter.com/param).path.string('repo') repo: string,
    [@param](http://twitter.com/param).path.string('label') label:string): Promise<QueryResult> {
    let result:QueryResponse = await this.queryService.getIssuesByLabel(repo, label);
    let queryResult = new QueryResult();
    queryResult.items = [];
    queryResult.total_count = result.body.total_count;
    result.body.items.forEach(issue => {
      this.addToResult(issue, queryResult);
    });// check if there is next page of the results
    const nextLink = this.getNextLink(result.headers.link);
    if (nextLink == null) return queryResult;
    await this.getIssueByURL(nextLink, this.queryService, queryResult);
    return queryResult;
  }
```

可以找到完整的控制器:[https://github . com/dhmlau/loopback 4-example-github/blob/main/src/controllers/GH-query . controller . ts](https://github.com/dhmlau/loopback4-example-github/blob/main/src/controllers/gh-query.controller.ts)。

# 让我们来测试一下！

现在，如果我们用相同的值调用端点(就像我下面的这个)，你会得到和以前相似的结果，只是如果`total_count`大于 30，你在结果中得到的`items`大小将和`total_count`相同。

![](img/c1776de942ad8f16271d3a9e4a87f5e1.png)

# 结论

到目前为止，我们已经创建了一个 REST API，它将 GitHub org+repo 名称和标签作为路径参数，并返回一个符合标准的问题(和拉请求)列表。在本文中，我们遍历结果页面来获取所有项目，而不是前 30 个项目。

这个 API 有点原始，但我认为它是构建一个与该 API 对话的前端应用程序的良好起点。我们总能不断改进应用程序。

# 参考

本博客系列之前的文章:

*   第 1 部分—为 GitHub API 创建数据源:[https://mobilediana . medium . com/building-an-end-to-end-application-with-loopback-react-js-7a 22d 726 c35d](https://mobilediana.medium.com/building-an-end-to-end-application-with-loopback-react-js-7a22d726c35d)
*   第二部分—创建服务代理:[https://mobilediana . medium . com/building-an-end-to-end-application-with-loopback-react-js-part-2-Creating-Service-Proxy-7 ffac 2 BD 7980](https://mobilediana.medium.com/building-an-end-to-end-application-with-loopback-react-js-part-2-creating-service-proxy-7ffac2bd7980)