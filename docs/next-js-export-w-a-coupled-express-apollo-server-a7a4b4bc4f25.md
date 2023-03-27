# Next.js 导出带有耦合的 express/apollo 服务器。

> 原文：<https://itnext.io/next-js-export-w-a-coupled-express-apollo-server-a7a4b4bc4f25?source=collection_archive---------2----------------------->

我最近喜欢使用的典型堆栈是 Next.js，带有一个定制的 express 服务器，该服务器中有一个 Apollo GraphQL 层，这样我就可以向同一个服务器发出请求。

这意味着该应用高度互联，微服务的概念已经过时。不管怎么说，总算摆脱了——它们增加了成本。

基线服务器是这样的

阿波罗服务器看起来像这样:

其中类型、上下文等在其他地方定义，就像普通的 GraphQL 设置一样。不需要在这里介绍。因此，我们的 Next.js 应用程序从这个服务器上消费数据，然后 Next 通过 Apollo 客户端进行各种魔法使其同构。

现在，为了在导出过程中实际使用这些数据，只需要一些非常基本的脚本:

并且在 package.json 中，您会想要使用`ts-node export.ts`。它在做什么应该很明显:

*   启动一个准系统 express 服务器。
*   连接 Apollo 服务器链接
*   开始`next export`构建过程
*   完成后，它会清理并关闭服务器

当服务器启动(在端口 3k 上)时，我们的客户端链接将使用 next 用来构建导出的数据。

作为参考，[这里是我通常为 Next.js](https://gist.github.com/OutThisLife/587f6fc89f72fc3345b1d44880c8ae70) 设置的 apollo 客户端。