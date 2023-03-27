# 在 Apollo 客户端延迟加载 Apollo 链接

> 原文：<https://itnext.io/lazy-load-apollo-link-in-apollo-client-f548e0254a16?source=collection_archive---------2----------------------->

## 10 行代码有所帮助

![](img/0f75955ca7b15fea6adbb55456a92089.png)

# 介绍

这是一篇关于我的小图书馆的短文。

[Apollo 客户端](https://github.com/apollographql/apollo-client)是 GraphQL 的一个库。 [Apollo Link](https://github.com/apollographql/apollo-link) 是扩展 Apollo 客户端的接口。

通常，您会像这样初始化 apollo 客户机。

```
import { ApolloClient } from 'apollo-client';
import { InMemoryCache } from 'apollo-cache-inmemory';
import { HttpLink } from 'apollo-link-http';const cache = new InMemoryCache();
const link = new HttpLink({ uri });const client = new ApolloClient({
  cache: cache,
  link: link,
});
```

我想在另一个文件中定义链接，并延迟加载它，因为它不是一个 HttpLink，而是一个复杂的大链接。

# 如何使用

为此，我们使用[动态导入](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import)。

假设我们有一个导出阿波罗链接的`link.js`文件。如果能动态导入就好了。

```
import { lazy } from 'apollo-link-lazy';const link = lazy(() => import('./link'));
```

`import()`返回一个承诺，但是没有`await`。这怎么可能？

# 如何实施

有趣的是，阿波罗链接本质上是异步的。然而，它不是基于承诺的。它有可观察的界面。

所以，你所需要的就是将承诺转化为可观察到的东西。

这是代码。

```
import { ApolloLink, fromPromise, toPromise, Observable } from 'apollo-link';export const lazy = (factory) => new ApolloLink(
  (operation, forward) => fromPromise(
    factory().then((resolved) => {
      const link = resolved instanceof ApolloLink ? resolved : resolved.default;
      return toPromise(link.request(operation, forward) || Observable.of());
    }),
  ),
);
```

幸运的是，阿波罗客户端导出了`fromPromise`和`toPromise`实用函数。因此，它可以很容易地实现。

这里的一个小技巧是同时支持 ApolloLink 承诺和默认导出。

# 演示

我开发这个代码作为一个库。

[](https://github.com/dai-shi/apollo-link-lazy) [## 戴-史/阿波罗-林克-懒惰

### 这是一个很小的库，用于延迟加载阿波罗链接。它对代码分割很有用。npm…

github.com](https://github.com/dai-shi/apollo-link-lazy) 

可以安装使用。它支持 TypeScript。

这里还有一个 codesandbox 中的演示。

# 结束语

由于我的动机是代码分割，支持像 [React.lazy](https://reactjs.org/docs/code-splitting.html#reactlazy) 这样的默认导出实际上已经足够了。因为它也支持直接承诺，所以我们可以将它用于任何异步初始化，如下所示。

```
import { lazy } from 'apollo-link-lazy';const link = lazy(async () => {
  // await ...
  return new ApolloLink(...);
});
```

我希望这可以帮助其他尝试延迟加载 apollo 链接的开发者。

*原载于 2020 年 1 月 10 日 https://blog.axlight.com*[](https://blog.axlight.com/posts/lazy-load-apollo-link-in-apollo-client/)**。**