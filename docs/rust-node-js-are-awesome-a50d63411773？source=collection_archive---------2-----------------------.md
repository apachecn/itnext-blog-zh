# Rust + Node.js 太牛逼了！

> 原文：<https://itnext.io/rust-node-js-are-awesome-a50d63411773?source=collection_archive---------2----------------------->

## 使用 Rust 对 Node.js 进行超快速、低要求、计算密集型操作

![](img/a0f323ee1f6a461c35cd4e8cb16275dc.png)

照片由[马修·施瓦茨](https://unsplash.com/@cadop?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 拍摄

> [*点击这里在 LinkedIn*](https://www.linkedin.com/cws/share?url=https%3A%2F%2Fitnext.io%2Frust-node-js-are-awesome-a50d63411773%3Futm_source%3Dmedium_sharelink%26utm_medium%3Dsocial%26utm_campaign%3Dbuffer) 上分享这篇文章

## 为什么不只是 Node.js

我只想说，我爱 Js！的怪癖(`1 == "1" // true`)，每周的新框架，`npm install meaning-of-life`，以及 ***X*** 如何比 React.js 快了 ***Y*** 个数量级😆！不管怎样，当我在`v0.9`时代听说 Node.js 的时候，拥有 js 全栈似乎是一个梦想！我和 Express.js 的恋情就是从那里开始的。我试过 RoR 和拉韦尔，但是简单地从客户端复制/粘贴业务逻辑到服务器就足够了，反之亦然。

但是说实话，Node.js 并不是最快的，糟糕的内存管理也是一个问题。我在一个项目中遇到过这种情况，在我廉价的 EC2 实例上，我的节点进程耗尽了内存。有了一些 C 和嵌入式开发的背景知识，*明显的*问题是 Node.js 的垃圾收集器，而*合乎逻辑的*解决方案是使用一种我可以手动分配内存的语言(只是…不是 C！).

就在那时，我听说了街区里新来的小子，拉斯特！我不会在这里歌颂 Rust，因为那将是一个完全不同的帖子，但可以说它是 Stack Overflow 上最受欢迎的语言。它对我来说也不是完全陌生的(我看着你*走)*或者是一个坚实但过于复杂的解决方案(… *C++* )。

长话短说，使用 Rust，花费了大约 40s 和大约 400MB 内存的节点进程现在需要大约 3s 和大约 16MB。😎

`this`，尽管这很好，但有点复杂，因为前面提到的过程的任务是从一些 CSV 生成 JSONs，然后将它们发送到不同的 Express.js API，为客户端提供服务。

当我在这个 [*周在 Rust*](https://this-week-in-rust.org/) 中看到[这篇文章](https://keminglabs.com/blog/building-a-fast-electron-app-with-rust/)时，我产生了这种想法，作者在这里介绍了 [Neon](https://github.com/neon-bindings/neon) ，一个库(？)从 Node.js 调用 Rust 函数，在他的例子中是从一个电子应用程序调用，但这让我想到我也可以从一个 Express.js 应用程序调用 Rust 函数。

花了一点时间来了解 Neon 是如何工作的，因为没有太多如何使用 Neon 的例子(我知道…我很快就会做出贡献！).我发现 Neon 在了解了它的一般概念之后，变得非常有逻辑性和功能性，并且它提供了非常好的从 Js 来回传输数据的方法。在一些情况下，我发现使用一个简单的`.split(",").collect();`来传递普通字符串并把它们转换成 Rust 上的向量更容易。

一个例子是需要从 Js 向 Rust 传递一个多维数组。`JsArray` API 可能有点难以理解，所以我发现将数据编码为 DSV(分隔符分隔的值)更容易，因此像这样的数组`[1, 2, [3, 4]]`被编码为`1|2|3,4`，并且使用 Rust 上的`.split()`方法将它恢复为向量也很节省。

我将与你分享的最后一件事是，我很高兴地发现 Rust 语法与 Js 有多接近。以至于没费多大力气就把我的 Js 算法移植到 Rust 了。

一些 JS 对一些 Rust 代码

## 结论

在这一切之后，我最终得到了一个 Express.js API，它使用原始 CSV 文件的启动时间为 JSONs 的生成变得没有必要),响应时间为 8ms！*(不包括网络延迟)。*这与以前的版本相比，以前的版本采用了一个单独的过程来从 CSV 生成 JSONs，然后以大得多的延迟为它们提供服务。性能提升并不是唯一的优势，流程整合简化了部署策略、故障排除和维护。锈也是一种快乐的工作！看一看玩得开心！🤓