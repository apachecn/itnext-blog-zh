# React 应用中针对较小软件包的 10 个提示和技巧

> 原文：<https://itnext.io/tips-tricks-for-smaller-bundles-in-react-apps-58d1b20c9c0?source=collection_archive---------1----------------------->

![](img/ae64b378e9328187d5159a83df34892b.png)

在这篇文章中，我将尝试分享一些技巧和优化技巧，当你的目标是最小化你的 React 应用程序的内存占用时，你应该考虑这些技巧和技巧。我相信在阅读完这篇文章之后，您将能够将您的包大小减少至少 5–10 %,因为我将从传统的技巧开始，然后转到边缘情况的微优化。我还想指出，我将经历的大部分事情并不是 React 特定的，而是适用于所有使用 Webpack 构建的 JS 应用程序。

# 我为什么要在乎？

如果你想知道，那么你的 JS 越早到达你的用户的浏览器，你的 React 应用就能越早启动，用户就能越快地与你的应用完全交互。尤其是在 B2C 应用程序中，从 [TTFB](https://en.wikipedia.org/wiki/Time_to_first_byte) 或 [FCP](https://gtmetrix.com/blog/first-contentful-paint-explained/) 节省的几百毫秒，有时可以产生数百万的公司收入。这一点非常重要，以至于公司付给性能工程师很多钱来获得如何使他们的网站更快的建议。此外，你也要尊重你的用户的数据计划，确保他们在访问你的网站时不会消耗太多的 MB，同时也要确保他们有时很差的网络不会妨碍他们访问它。一般来说，用户越早看到屏幕上的东西越好。如果您正在进行 SSR，那么这应该不会对您造成太大的影响，但是它仍然会给您的用户带来问题，因为您从服务器上下载的 HTML 是不可交互的，直到 JS bundle 对它进行了合并。

我可以一直说下去，但废话少说。我答应给你提示，所以让我们直接进入:

## 1.代码精简

这是显而易见的，但我不得不包括它。如果你正在使用[创建-反应-应用](https://github.com/facebook/create-react-app/)，那么你做的一切都是正确的。如果你不是，那么确保你缩小你的 JS，JSON & HTML 和散列/缩短你的 CSS 类名。

## 2.比起类，更喜欢带钩子的函数

类往往有许多额外的样板文件，而钩子就在那里，这样你可以用更少的代码实现同样的事情。更少的代码=更小的包。如果你是大型应用程序的一部分，请记住这一点。

## 3.更喜欢预先行动而不是反应

在极少数情况下，你的应用没有使用任何花哨的 React API，你可以使用 [Preact](https://preactjs.com/) 来代替 React。这是 React 的轻量级版本，体积缩小了近 90%，但仍然可以做 React 本身可以做的大多数事情。点击[此处](https://preactjs.com/guide/v8/differences-to-react/)查看其支持的完整功能列表。

## 4.精确代码翻译

如果你使用 Babel 来传输你的代码，确保你只针对你的用户使用的浏览器。许多项目已经配置了 babel 来生成与旧浏览器兼容的代码，这导致了代码冗长。确保根据您的需求指定 [browserlist](https://github.com/browserslist/browserslist) 或者让 babel 为最新一代的浏览器显式生成代码(参考您的用户分析),这肯定会减少您的 JS 占用空间。

## 5.避免在不需要时导入整个库

如果你使用的是不支持树抖动的大型库，确保你只导入**你真正需要的**。例如，如果你使用`lodash`只是为了`get`函数，那么你应该写`import get from 'lodash/get'`而不是写`import { get } from 'lodash'`。前者会导入库的所有模块，而后者只会导入`get`，其他什么都不会。具体到`lodash`，有一个 [webpack 插件](https://github.com/lodash/lodash-webpack-plugin)会把前一种类型的导入转换成后一种，这样你在开发的时候就不用考虑那样的事情了。一般来说，为了适应它，我会推荐手动操作，因为你最终将不得不为没有相应插件的其他库这样做。

## 6.选择满足您需求的最小库

如果您在一个项目中与许多其他开发人员一起工作，他们中的一些人可能会包括他们自己熟悉的第三方库。虽然这些库确实解决了这个问题，但是它们中的许多有时可能是多余的。以 [moment](https://momentjs.com/) 为例，这是一个日期时间操作库，它的大小高达 66KB(！)当 gzipped。大多数时候，人们使用 moment 只是为了以一种漂亮的格式显示一个日期。另一种选择可能是 [dayjs](https://github.com/iamkun/dayjs) ，它在 gzip 压缩时小于 2KB，并且可能同样解决手头的问题。

要调查一个现有的项目是否容易出现上述情况，您可以使用 Webpack 的[包分析器插件](https://www.npmjs.com/package/webpack-bundle-analyzer)，它将显示您所有包的树形图。在树状图上看到太大的东西？检查代码，看看它是如何被使用的。如果您仅将它用于琐碎的任务，那么:

1.  检查你是否能写普通的 JS 来解决这个问题。
2.  如果没有，请检查是否可以找到满足您需求的替代库。要检查替代品是否是一个好的候选者，在 [Bundlephobia](https://bundlephobia.com/) 中输入它的名字，看看是否有(足迹)收益从转换到它。
3.  如果没有任何替代方案(或者替代方案的规模同样庞大)，问问你自己使用这个包的特性给项目增加了多少价值。在某些情况下，您可以更改需求，以便完全省略该包。公平地说，只有当一个包的内存非常大，而你只利用了它的一部分模块时，才应该考虑这个问题。

## 7.将代码分割成多个包(也称为代码分割)

让我们把这件事解决掉，好吗？如果你不做，那就去做。如果你已经在做了，问问你自己你是否做得最好。老实说，进行代码拆分没有“正确的方法”，但是**只要不加载超过呈现页面所需的内容**、**、**就可以了。尽管适合每种业务的方法各不相同，但两(2)种核心方法是:

*   基于路由的代码分割
*   基于特征/组件的代码拆分

第一种方法为每条路线(应用程序中的每个不同页面)创建一个包，当 React 模块在页面之间的可重用性不高时，这是您的最佳方法。页面越“不同”越好。例如，如果您有一个产品页面和一个产品详细信息页面，它们不共享通用的 UI 元素，那么基于路由的代码分割可能是您的一个好选择。你最终会得到两(2)个包——每页一个——生活会很美好。当您引入重用产品页面使用的相同 UI 元素的用户页面和用户详细信息页面时，事情就变得糟糕了。在这种情况下，执行基于路由的代码分割会给你留下四(4)个彼此相关的包，并且获取它们的网络请求可能会超过它们被分割的好处。为了实现基于路径的代码分割，您可以使用`React.lazy`导入应用程序的每个页面，然后使用`<React.Suspense />`包装:

出于某种原因，Medium 没有显示完整的代码。[查看全部要点](https://gist.github.com/3nvi/8a9cd756cb05e7c11d9a9f3925fba695)

每个动态`import`语句将产生不同的包。通过向 bundle 添加一个昵称，并在想要分组的内容上使用相同的昵称，可以确保两(2)个或更多动态导入包含在同一个 bundle 中。例如，在上面的代码中，我们可以添加:

```
const ProductListPage = React.lazy(
  () => import(/* webpackChunkName: "pages" */ './ProductListPage')
); const ProductDetailsPage = React.lazy(
  () => import(/* webpackChunkName: "pages" */ './ProductDetailsPage')
);
```

这将指示 webpack 从这两个页面中创建一个块(bundle)。

基于特性的代码分割适用于在页面中重用大量代码库的应用程序，因此上述基于路径的方法不适用。这种方法关注于组件/元素/特性，这些组件/元素/特性通常只在某些情况下才需要，除非要向用户展示，否则不应该加载。例如，以谷歌的天气小工具为例:

![](img/49e3b770552a7097e337617819a523c1.png)

大多数时候，用户只是搜索一些东西是不会看到这个的，所以谷歌为什么要把它加载到它的 JS 包中呢？为什么不在用户将要看到的时候才加载呢？这就是基于特性的代码分割。要实现这一点，如果特性是 React 组件，您可以使用`React.lazy`，如果特性基于非 React 包，您可以使用动态 webpack 导入(`() => import('...')`)。

当然，不是所有特性都应该是代码分离的。一些功能非常小，以至于额外的网络请求(DNS 解析、SSL 握手、下载时间等)的成本。)超过了代码拆分的好处。为了知道某件事是否应该进行代码拆分，访问 [Bundlephobia](https://bundlephobia.com) 看看是否值得。根据经验，任何低于 1KB 的文件在 gzipped 时都是不值得的。

## 8.优化压缩

你在压缩你的内容吗？如果你不确定，那么你很可能不确定，除非你的 CDN 自动为你确定。最近，像 [Cloudfront](https://aws.amazon.com/cloudfront/) 这样的 cdn 自动对你的所有内容进行 gzip 压缩，并提供给所有使用 gzip 作为内容编码请求头的浏览器。问题是，gzip 不是目前最好的压缩算法，因为 [brotli](https://github.com/google/brotli) 比 gzip 占用的内存少 15-20 %,几乎有 92%的浏览器支持 gzip。如果你不关心旧的浏览器，今天就把你的压缩算法换成 brotli，因为所有主要的 cdn 都是最新的，可以适当地为它服务。如果你真的关心旧的浏览器，保留你的 gzip，但是确保你有一个 brotli 压缩的包可以让浏览器解析它。

如果 brotli 压缩不是一个选项(因为业务或基础设施的限制)，那么您可以对您的 gzipped CSS 包进行漂亮的优化。诀窍是确保 CSS 规则总是按字母顺序排序。Gzip 喜欢重复，它可以通过适当索引的可重用关键字来优化压缩。让你的规则像这样排序将会导致 1%到 3%的 CSS 包变小。如果你觉得很懒，你甚至可以使用一个 webpack 插件来自动排序。

## 9.包括每个库的单个版本/实例

假设您在项目中使用了最新版本的`apollo-client`。因为您为一家希望快速推出产品的初创公司工作，所以您决定使用 [Amplify](https://aws-amplify.github.io/) 来处理存储、认证等。Amplify 内部也使用`apollo-client`，但是因为 AWS SDKs 不经常更新它们的依赖项，所以它们使用库的特定**版本****，这不是最新的。您最终得到的是您的包中两个不同版本的`apollo-client`。**

**解决这个问题的一个方法是确保降级，以便使用与 Amplify 相同的版本(如果这是一个选项的话)。另一种方法是派生 [Amplify 项目](https://github.com/aws-amplify/amplify-js)并手动更新所需的依赖项。最后，有一种非常讨厌和棘手的方法来解决这个问题，这种方法只有在软件包的 API 在不同版本之间没有变化的情况下才有效(大多数时候这意味着主版本是相同的)。你可以做的是使用一个 [webpack 别名](https://webpack.js.org/configuration/resolve/#resolvealias)来将`apollo-client`的导入映射到你已经安装的`apollo-client`的版本(最新版本)。这意味着每次 Amplify 试图导入库的本地副本时，它的导入将被重新路由并映射到另一个副本。在代码中，应该是这样的:**

```
... // other webpack configuration,
**resolve**: {
*/*
make sure that all the packages that attempt to resolve the   following packages utilise the same version, so we don't end up bundling multiple versions of it. Unfortunately, both `aws-appsync` and `aws-amplify` install explicit versions of the their dependencies whose minor version doesn't match. This enforces all if the packages to use the same version 
*/
  alias: {* **'apollo-client'**: path.*resolve*(**__dirname**, **'node_modules/apollo- client'**),
  },
},
... // other webpack configuration
```

**这段代码说的是“嘿，每当有人做`import ... from 'apollo-client'`时，确保你总是从`./node_modules/apollo-client`目录解析这个导入”。我想强调的是，这个解决方案可能会引入一些问题，因为其他一些模块可能与最新版本的`apollo-client`不兼容，但是如果它们是兼容的，那么您可以从您的包中节省一些宝贵的 KB。**

**附:上面的例子不是随机的😜**

## **10.仅在需要时使用 React**

**这是一个奇怪的问题，但是本质上，如果你不需要 React，就不要马上把它放到客户端。例如，您可能有一个应用程序，它有一个大多数新用户最初会访问的登录页面。尽管您的登录页面可能是用 React 构建的，但用户可能不需要运行时，因为您的 SSR 返回的静态 HTML 已经足够了。我所说的本质上是一个没有客户端 React 依赖的服务器端渲染的登陆页面。然后，只有当用户切换到您的应用程序时，您才能加载 React 运行时。这是[网飞尝试过的](https://medium.com/dev-channel/a-netflix-web-performance-case-study-c0bcde26a9d9)方法，它导致了 JS 包大小的显著减少。**

# **结束语**

**本文的目的是提供一些减少应用程序内存占用的方法，从而缩短初始加载时间。虽然其中一些建议不容易实现，但我个人认为其他一些确实是微不足道的，并能带来立竿见影的效果。如果你觉得我错过了一些有趣的东西，请随意评论，我会把它添加到列表中。**

**感谢坚持到最后:)**

*****附言*** 👋 ***嗨，我是***[***Aggelos***](https://aggelosarvanitakis.me/)***！如果你喜欢这个，考虑一下*** [***在 twitter 上关注我***](https://twitter.com/AggArvanitakis) ***并与你的开发者朋友分享这个故事*😀****