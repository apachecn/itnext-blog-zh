# Gatsby.js 在我们的 React SSR 上每月节省 400 美元

> 原文：<https://itnext.io/gatsby-saved-400-month-react-server-side-rendering-5456f036ab6a?source=collection_archive---------0----------------------->

![](img/6197b2b90b5032919e99f882b2ce31c3.png)

每一个关心 SEO 的 React.js 网站所有者都应该有一个服务器端渲染设置，这需要大量的资源，即使有一些缓存层，有时它也会阻塞整个基础设施，特别是如果它是像我们([hexometer.com](https://hexometer.com))这样的网站，在一天的特定时间内流量会动态增加。

SSR 的主要问题是它是一个阻塞操作，当它执行时，整个 Node.js 进程不会获得任何新的请求，这就是为什么我们必须有许多 Node.js SSR 进程来处理传入的请求。CDN 缓存当然有助于减少 SSR 过载，但有时当有新的更新或我们必须为认证用户提供内容时，SSR 会破坏我们的基础设施，因为它还需要来自我们 API 的一些数据。

基本上每次当用户导航到我们的网站时，它发送 1 个 SSR 请求，然后 SSR 发送 1 个 API 请求，然后在浏览器中，我们的 react 应用程序发送第二个 API 请求来比较缓存的数据。很明显，React SSR 只是为了让 SEO 页面可用而浪费大量资源:/

因此，我们开始研究其他解决方案，这将只需要静态内容，没有 SSR 可用。你可以猜到我们找到了 Gatsby.js！

# 为什么 Gatsby.js 与众不同

Gatsby.js 背后的主要思想是构建静态编译的 React + GraphQL 应用程序，您可以编译一次并通过 CDN 缓存提供它，而无需任何 SSR 后端。我们已经有了 GraphQL 后端，所以这对我们的用例来说是完美的。

不能把 Gatsby.js 插到现有的 React + Apollo 应用中。它需要自己的设置，并且因为它正在编译为静态呈现的 HTML + JS，所以您必须定制 Gatsby GraphQL 端点，以便在 Gatsby.js 构建过程中交付您的 API 数据。

```
// gatsby-node.js
const query = `
  query {
    .... your custom query here from your API endpoint
  }
`;

exports.sourceNodes = ({ actions, createNodeId, createContentDigest }, configOptions) => {
  return fetch(getQueryURL(query))
    .then(response => response.json())
    .then(res => {
      createNode({
        ... node data from response JSON
      })
    });
}
```

**注意** Gatsby.js 有两个阶段

1.  构建静态内容，包括外部数据，下载并呈现为 html 内容
2.  作为一个常规的 React 应用程序在浏览器中运行，您可以使用 React Apollo 发出一个动态 GraphQL 请求，包括身份验证！

# 我们如何将 UI 应用程序迁移到 Gatsby

老实说，这不是一个快速的迁移过程，它需要大量的代码迁移和需要考虑的事情，特别是我们学到了很多关于如何设计 React 组件以要求静态构建的知识。

**首先，我们制作了一个基本的 Gatsby.js 应用程序，其中包含了我们应用程序迁移的所有构建模块，然后我们开始复制/粘贴我们的 React 组件，包括我们应用程序所需的特定 NPM 包。在这一阶段没有比赛可担心，感谢上帝我们有一个非常原子的组件基础，我们真的有一个巨大的优势！**

****其次，**我们不得不移动一些静态查询，这是我们 React 应用程序渲染的一部分，具体来说，我们有一些动态内容，如[顶级技术堆栈应用程序列表](https://hexometer.com/most-popular-tech-stacks)，这些内容必须可供搜索引擎机器人使用。因此，我们只是添加了静态查询，并对页面查询做了一点调整，以满足 Gatsby.js 页面静态查询的概念。**

**应用程序的其余部分是相同的，我们有 React Apollo，它仍然是 React Apollo，用于网站分析请求或用户验证等动态内容，令人惊讶的是，我们甚至不需要在那里做任何更改。**

****最大的问题**是动态导航，因为我们有基于 API 内容和结构的页面导航，而且因为 Gatsby.js 本身是静态导航的，所以很难理解你实际上必须在 other 中创建一个基于 API 内容的页面才能使用那个 URL。**

```
const pageResult = await fetchQuery(TechStackCategoryQuery(category));
await createPage({
    path: `most-popular-tech-stacks/${cleanURLSpace(category)}`,
    component: path.resolve(`./src/components/tech-stack/stackByCategory.tsx`),
    context: {
      ...pageResult.data.TechStack,
      name: category,
      isDetails: false,
    }
});
```

**除此之外，应用内迁移没有任何问题。如果你想到，如果你只是让你的组件原子化，你实际上可以节省多少匹配时间，那就太棒了！**

# **实际节省**

**目前我们没有服务器端渲染，所以我们实际上没有任何服务器或服务来处理初始 UI 内容请求，它完全通过 Cloudflare CDN 提供服务！它只是一个静态文件，包含预先呈现的完整 HTML 内容。**

**仅迁移到 Gatsby.js 并从我们的基础架构中移除服务器端渲染流程这一项，我们平均每月就节省了 400 美元。你可以想象，我们的网站开始从许多不同的地区更快地加载比赛。**

**如果你喜欢这个故事，请分享你的掌声和推文链接！👏👏👏👏**