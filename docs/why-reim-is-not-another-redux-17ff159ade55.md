# 为什么雷伊姆不是另一个替身

> 原文：<https://itnext.io/why-reim-is-not-another-redux-17ff159ade55?source=collection_archive---------1----------------------->

一个(不)可变的状态管理库，简直太棒了

![](img/91f0546f6752477567fcd96f07ab828c.png)

图片由 [Artem Sapegin](https://unsplash.com/@sapegin) 在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 拍摄

如果你在过去几周一直关注 Twitter，你可能已经注意到 Redux 和 Mobx 之间的竞争对手。

事实上，由于 Redux 臭名昭著的样板结构，他们中的很多人都推荐从 Redux 迁移到 Mobx。

但是说真的，Mobx 是唯一的选择吗？如果我们使用 Mobx 的反应，为什么不直接使用 Vue？

## [**雷伊姆**](https://reimjs.gitbook.io)

> Reim.js 是一个(im)可变状态管理库。受 Redux 等优秀库的启发，但没有样板文件。

那是什么意思？

好吧，如果你以前用过 React / Redux / Unstated，雷伊姆会觉得一切都很熟悉。

# 概观

雷伊姆基于 3 个*部件*:

## **1。商店**

> Store 用于创建一个状态，该状态公开了用于更新状态的方法`setState`

## 2 + 3.订阅/连接+设置状态

# 副作用

雷伊姆提供了 [reim-task](http://npm.im/reim-task) ，而不是将副作用绑定到单个存储，这使得一个异步函数持有一个存储，以主动维护其状态

# 与其他框架一起使用

尽管雷伊姆最初是为 React 构建的，但它也可以很容易地与其他库一起使用。

## **Vue**

## Rxjs /角度

# React-specific:同步 react-路由器-v4

即使对于成熟的框架 Redux 和 Mobx，也很难将路由器状态与存储同步。

对于**雷伊姆**，你只需要:

希望我已经说服了你们中的一些人去深入了解雷伊姆。请随意查看[文档](https://reimjs.gitbook.io)，并提出[问题](https://github.com/IniZio/reim)以获得更多改进:)