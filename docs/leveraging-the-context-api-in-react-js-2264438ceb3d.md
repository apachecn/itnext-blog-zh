# 利用 React.js 中的上下文 API

> 原文：<https://itnext.io/leveraging-the-context-api-in-react-js-2264438ceb3d?source=collection_archive---------4----------------------->

在 [Twitter](https://twitter.com/chris_noring) 上关注我，很乐意接受您对主题或改进的建议/Chris

![](img/e01cd01e027ab52d9de569b9325de43a.png)

> React 上下文存在，因此您不必在每一级手工传递数据。上下文是关于向许多组件共享数据的

在本文中，我们将了解以下主题:

*   **上下文 API** ，它是什么，为什么使用它
*   **构建模块**，我们将通过上下文来了解不同的构建模块以及它们如何协同工作
*   **在动作演示**中，我们将看几个创建上下文的例子，我们将确保了解 Dynamix 上下文，以及更高阶的组件如何帮助清理我们的上下文代码

如果您曾经对本文中的代码示例感到困惑，请查看包含以下所有代码的演示:

 [## soft Chris/react-上下文-演示

### 这个项目是用 Create React App 引导的。在项目目录中，您可以运行:在…中运行应用程序

github.com](https://github.com/softchris/react-context-demo) 

这篇文章被移到了 https://softchris.github.io/pages/react-context.html