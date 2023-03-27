# 如何将 Node.js 与 TypeScript 一起使用

> 原文：<https://itnext.io/how-to-use-node-js-with-typescript-98e16b35518a?source=collection_archive---------4----------------------->

![](img/fed7a0463baf85b14103cd7dbacd1c9b.png)

## 用 TypeScript 编写更好的 Node.js 应用程序

[**TypeScript**](https://www.typescriptlang.org/) 继续改进，在 web 开发中得到更多的适应。如果你是 Angular 的开发者，那么你已经习惯了打字稿。Vue.js 和 React 也可以很好地配合 TypeScript 使用。

**打字稿的优势:**

*   静态类型有助于**在执行代码之前尽早**发现问题。
*   静态类型化**提高了代码的可读性**并且可以作为文档。
*   Java 或 C# 对**语法感觉很熟悉。**
*   TypeScript 允许你在你的代码中使用最新的 JavaScript 特性，这些代码甚至可以在旧的浏览器中运行。
*   许多库提供**内置类型**或[在明确类型化的库中有类型](http://definitelytyped.org/)可以很容易地使用。
*   像 IntelliJ 或 Visual Studio 代码这样的 ide 对 TypeScript 有很大的支持。

**打字稿的缺点:**

*   TypeScript 有一个(小)**学习曲线**。
*   不是每个图书馆(尤其是老图书馆)都有打字。不过，您可以自己创建类型。
*   对于小项目来说，打字稿可能太多了。

但是，TypeScript 也可以在后端使用。 [**Node.js**](https://nodejs.org/en/) 对于后端来说是个不错的选择，尤其是如果你懂 JavaScript 的话。Node 最受欢迎的选择是 [**Express**](https://expressjs.com/) ，一个简单易用的 web 框架。

## 如何在节点中使用 TypeScript(带 Express)

1.  [**设置一个快速项目**](https://expressjs.com/en/starter/installing.html) 如果你还没有。
2.  **安装类型脚本**作为开发依赖:`npm i typescript --save=dev`。
3.  **安装以下类型**以在您的 IDE 中获得最佳的类型脚本支持:`npm i @types/node @types/express --save-dev`。
4.  **重命名你的*。js* 文件(如路线)到*。ts*** 使用 TypeScript。
5.  可选:**安装**[ts-node-dev](https://www.npmjs.com/package/ts-node-dev):`npm i ts-node-dev --save-dev`。这通过在您更改代码时自动重启服务器来改进您的开发过程。这类似于使用普通 JavaScript 的 [*nodemon*](https://github.com/remy/nodemon) 。
6.  可选:为了检查我们的代码是否正确编译，我们应该**在 CI** 上运行构建。在 *package.json* 中运行一个生成 JavaScript 代码的 TypeScript 构建的简单脚本可能是这样的:`"build:ci": "**tsc**"`。
7.  可选:**使用**[**Jest**](https://jestjs.io/)**进行单元和 API 测试**。测试应该是软件开发项目的基本部分。Jest 是一个可行的解决方案，因为它提供了很好的测试体验，并且与 TypeScript 配合良好。[这里有一篇关于如何用 TypeScript 和 Node.js 使用 Jest 的文章](/test-driven-develop-your-api-with-jest-supertest-in-node-js-7e1c6489b0a6)。

## 结论

感谢您阅读这篇关于如何在 TypeScript 中使用 Node.js 的文章。如您所见，用 TypeScript 设置 Node.js 项目很容易。就我个人而言，我已经成为 TypeScript 的忠实粉丝，因为它可以在任何 web 项目中使用，并且极大地改善了我的开发体验。如果你有任何问题，请在评论中告诉我。