# 使用 React/Redux 的验收测试驱动开发—第 1 部分

> 原文：<https://itnext.io/acceptance-test-driven-test-with-react-redux-part-1-7ae8cb4fab00?source=collection_archive---------2----------------------->

![](img/b279120f59d4fbe8d092b9bf73368491.png)

"六扇颜色各异的门叠放在一起。"由[莫仁许](https://unsplash.com/@moren?utm_source=medium&utm_medium=referral)上 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)

***更新 1*** : *本文是一个系列的一部分，查看完整系列:* [第一部分](https://medium.com/@juntao.qiu/acceptance-test-driven-test-with-react-redux-part-1-7ae8cb4fab00)、[第二部分](https://medium.com/@juntao.qiu/acceptance-test-driven-test-with-react-redux-part-2-127949a6e47e)、[第三部分](https://medium.com/@juntao.qiu/acceptance-test-driven-test-with-react-redux-part-3-903e1e58e706)、[第四部分](https://medium.com/@juntao.qiu/acceptance-test-driven-development-with-react-redux-part-4-5db545953ed3)和[第五部分](https://medium.com/@juntao.qiu/acceptance-test-driven-development-with-react-redux-part-5-995577d28eff)。

***更新 2*** :我出版了一本名为 [*用验收测试驱动开发构建 React 应用*](https://leanpub.com/build-react-app-with-atdd) 的书，涵盖了更多关于 ATDD 和 React 的话题和实践，[请查看](https://leanpub.com/build-react-app-with-atdd)！

# 验收测试驱动开发简介

在这个系列中，我们将通过一个真实的例子来学习如何使用`Acceptance test driven development`来开发一个 web 应用程序。如你所知，设计一个好的例子最困难的是你必须同时平衡简单性和复杂性。

如果你的例子太简单，读者在尝试应用书中学到的方法时会遇到很多意想不到的问题；另一方面，如果例子太复杂，学习曲线将会非常陡峭，他们可能不得不学习许多无用的东西，只是为了跟随例子。

为了解决这个问题，我将在本系列中向您展示如何使用主流技术开发一些非常经典的场景，包括:

*   前端项目脚手架
*   异步请求处理
*   组件化
*   客户端路由
*   状态管理
*   自动化测试
*   模拟后端服务

# 技术堆栈

具体来说，我们将使用以下库

# JavaSCript 库

*   反应
*   反应路由器
*   Redux
*   Axios

我们将使用`React`作为客户端渲染库，`React-router`用于客户端路由，`Redux`用于状态管理，`axios`用于网络请求。对于自动化测试，我们引入了以下库:

# 测试

*   操纵木偶的人
*   酶
*   玩笑
*   json-server

来自 Google 的`Puppeteer`作为端到端测试(验收测试)，`Jest`作为测试运行者，`Airbnb`的`Enzyme`用于测试`React`组件，最后`json-server`用于模拟后端 API。

由于 ATDD 只是另一种类型的 TDD，它遵循红绿重构循环，可以这样描述:

1.  编写一个失败的验收测试
2.  用最简单的方法让它过去
3.  基于代码味道(如硬代码、幻数等)进行重构
4.  基于新的需求添加新的测试(如果我们需要新的验收测试，回到 1，否则过程就像传统的 TDD 一样)

对于我们将在本系列中开发的应用程序，它有 3 个特性，即:

# 功能 1—图书列表

在现实世界中，一个特性的粒度要大得多，通常一个特性可能包含许多用户故事，比如图书列表、分页、列表中一本书的样式等等。让我们假设一个故事只有一个故事:

我们可以用这种形式描述用户故事:

`As a user, I want to see a list of books, so that I can learn something`

验收标准是:

*   假设系统中有`10`本书，用户应该在页面上看到`10`项
*   在每个项目中，应该显示以下信息:书名，作者，价格，评级。

# 功能 2 —图书详情

`As a user, I want to check the detail of a book, so that I can quickly get the brief without reading through it`

验收标准是:

*   用户单击图书列表中的项目，将被重定向到详细信息页面
*   详细页面上有:书名、作者、价格、描述、评论

# 功能 3 —搜索

`As a user, I want to search book by its name, so that I can quickly find what I'm interested in`

验收标准是:

*   用户键入`Refactoring`作为关键字
*   只有名称中包含`Refactoring`的书籍才会显示在书籍列表中

在本系列的后续文章中，我们将逐步添加功能，并尝试严格应用 ATDD。