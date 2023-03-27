# 为什么我们公司用 TypeScript+Prisma+tRPC 代替 Golang+GraphQL

> 原文：<https://itnext.io/why-our-company-replaced-golang-graphql-with-typescript-prisma-trpc-ef56aaaa1c8c?source=collection_archive---------0----------------------->

## 我们对获得的好处非常满意

![](img/49a2619dfd24456fd861ea4c1a192631.png)

照片由[西格蒙德](https://unsplash.com/@sigmund?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄

# 介绍

我想和你们分享我们公司走向 tRPC 和 GraphQL 的历程。

我在一家意大利咨询公司工作，主要为客户制作 ERP ( *企业资源规划*)软件。

这些类型的软件通常用于 50%到 70%的 CRUD 操作。

因此，这些年来，我们一直试图将所有我们能做的事情自动化，因为它们很无聊而且重复，并且利用了在业务逻辑上节省的**时间。**

# 我们的堆栈

我们过去 3 年的堆栈是这样的(我已经简化了一点)

**后端(在 GCP 上大多是无服务器的，带有 CloudRun/CloudFunctions )**

*   [GraphQL](https://graphql.org/)([gqlgen](https://github.com/99designs/gqlgen)go 库，用于使用模式优先方法构建 graph QL 服务器)
*   [gRPC](https://grpc.io/) (用于微服务之间的内部通信)
*   发布/订阅(用于微服务之间的内部异步通信)

**前端**

*   React ( [NextJS](https://nextjs.org/) )
*   [素材 UI](https://mui.com/material-ui/getting-started/overview/) (或其他 UI 工具包)
*   GraphQL(带[编码](https://www.the-guild.dev/graphql/codegen)

注意:我已经写了一篇文章，展示了[如何一步一步地用 Golang 建立 gRPC 服务器](/build-grpc-server-with-golang-go-step-by-step-b3f5abcf9e0e)。看看这个。

# 冗长:我们在公司面临的问题(也发生在个人开发或小公司)

在后端，我们使用了 GoLang 的 ORM(称为 [Bun](https://bun.uptrace.dev/) )和 GqlGen 库来生成 schema first GraphQL 服务器。

我们的流程是这样的(简化):

1.  用 ORM 定义结构和查询(这里有很多重复的代码)
2.  GraphQL 模式的定义。
3.  GraphQL 解析器的实现。
4.  GraphQL 查询的代码生成和 React 前端的变异(带 typescript)
5.  在前端使用生成的查询/变异

在这个过程中，第一步和第三步对于 CRUD 操作来说极其冗长和乏味。

# 什么是 tRPC

tRPC 是一个轻量级的库，让你无需模式或代码生成工具就能构建完全类型安全的 APIs)。

它允许客户端和服务器之间的类型共享。

目前，GraphQL 是在 TypeScript 中实现类型安全 API 的主要方式(这太神奇了！).由于 GraphQL 是作为实现 API 的语言无关规范而设计的，所以它没有充分利用像 TypeScript 这样的语言的强大功能。

如果你的**项目是用全栈类型脚本**构建的，你可以在你的客户机和服务器之间直接共享类型**而不依赖于代码生成。**

# 什么是 Prisma

Prisma 是一种帮助应用程序开发人员更快构建和更少出错的 ORM。

它由以下部分组成:

*   **Prisma Client**:node . js&TypeScript 的自动生成和类型安全查询生成器
*   **Prisma Migrate** :迁移系统
*   **Prisma Studio** :查看和编辑数据库数据的 GUI

# 我们的选择:放弃 Golang 和 GraphQL，使用 Typescript ( node)和 tRPC + Prisma

通过 tRPC 使用 Prisma 和 typescript，我们可以重新定义我们的流程:

1.  创建 Prisma 模式:**我们定义我们的应用程序模型**

*Prisma 将用*[*Prisma-tRPC-generator*](https://github.com/omar-dulaimi/prisma-trpc-generator)为我们创建一切:查询、结构和 tRPC APIs

2.在前端使用 tRPC APIs(因为有 typescript，所以完全是类型安全的)

就是这样。😃

# 我们从这个选择中获得了什么

*   提高代码质量
*   交货速度
*   我们开发者的幸福
*   减少重复和枯燥的编码

# 结论

我们要离开 Golang 了(除了一些微服务，因为我们还爱 Golang)。

我们将离开 GraphQL(注意，Prisma 也可以用所有的 crud 操作生成一个 GraphQL 服务器)。

GraphQL 仍然是一个很好的工具，但有时有些矫枉过正，在小规模项目中没有必要使用它。

我们对自己做出的选择非常满意。

# 这里有一些我写的其他文章，我想你会觉得有用

[](https://www.klitonbare.com/blog/animate-svg-with-framer) [## 用 React 和帧运动制作 SVG 动画

### 你好👋在这篇文章中，我将向你展示我是如何为我的博客标识创建一个绘制效果的。在这里你将得到什么(你可以…

www.klitonbare.com](https://www.klitonbare.com/blog/animate-svg-with-framer) [](https://betterprogramming.pub/how-i-improved-my-react-code-readability-and-maintainability-with-conditional-rendering-94b32448bc70) [## 不使用条件呈现操作符编写更好的 React 代码

### 避免使用使代码不可读的三元运算

better 编程. pub](https://betterprogramming.pub/how-i-improved-my-react-code-readability-and-maintainability-with-conditional-rendering-94b32448bc70) [](https://javascript.plainenglish.io/tips-for-writing-better-react-code-ceb49e929001) [## 编写更好的 React 代码的技巧

### 你知道那些窍门吗？

javascript.plainenglish.io](https://javascript.plainenglish.io/tips-for-writing-better-react-code-ceb49e929001) [](/best-books-that-every-software-developer-must-know-8b96faff180d) [## 每个软件开发人员都必须知道的最佳书籍！

### 你的书架上有这些书吗？

itnext.io](/best-books-that-every-software-developer-must-know-8b96faff180d) [](/best-vscode-extensions-by-a-full-stack-developer-in-2022-f730037b6e0b) [## 2022 年全栈开发者评选的最佳 VsCode 扩展

### 作为一名完整的堆栈开发人员，以下是我挑选的最好的 VS 代码扩展！

itnext.io](/best-vscode-extensions-by-a-full-stack-developer-in-2022-f730037b6e0b)