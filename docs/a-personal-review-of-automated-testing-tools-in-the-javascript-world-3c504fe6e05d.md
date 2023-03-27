# JavaScript 世界中自动化测试工具的个人评论

> 原文：<https://itnext.io/a-personal-review-of-automated-testing-tools-in-the-javascript-world-3c504fe6e05d?source=collection_archive---------0----------------------->

## 我的一些开源工具的“2 美分”,我已经使用或已经使用，以确保软件开发项目的质量

![](img/9e83504cf15c7ca2171847861355e6f7.png)

JavaScript 代码

在这篇文章中，我将谈谈我使用支持 **JavaScript** 的工具进行**测试自动化**的经历。在这篇文章的第一部分，我将讨论图形用户界面或 **GUI** 的**自动化测试工具。在文章的第二部分，我将谈论用于**集成测试自动化**或 **API 测试**的工具。在文章的第三部分，我将讨论用于**单元测试**的工具。在最后一部分，我将讨论用于静态代码分析的工具。**

## 第 1 部分—图形用户界面(GUI)的自动化测试

## [Selenium Webdriver](https://github.com/SeleniumHQ/selenium/wiki/WebDriverJs)

我对 **Selenium Webdriver** 的 **JavaScript** 版本的体验在使用该框架时可能完成的事情方面相当满意，它是 **GUI 测试**的一个强大选项，满足了对多种浏览器的支持需求，如 **Chrome** 、 **Firefox** 、 **Safari、**和 **Internet Explorer** 。

其他相关点是社区支持；它是 GitHub 和**会议**上的 [**活动项目。2017 年我有机会参加了柏林的**](https://github.com/SeleniumHQ/selenium/graphs/code-frequency)**[**SeleniumConf**](https://www.seleniumconf.de)，除了更新自己，也学到了很多。**

关于文档，直到最近，它还不是最好的，但是**版本 4** 正在推出优秀的 [**文档**](https://seleniumhq.github.io/docs/) 并且坚持 W3C WebDriver 的**规范。对那些感兴趣的人来说，下面还有一个**相关内容**。**

**Selenium** 的另一个优势是支持在不同浏览器中执行相同的测试用例。非常适合测试聊天和视频会议应用。

## [量角器](http://www.protractortest.org/#/)

**量角器**是我比较有经验的 **GUI 自动化测试工具**。自 2014 年以来，我一直在使用这个框架，在 Medium 上用英文 [**写内容，在 WordPress**](https://medium.com/search?q=protractor%20walmyr) 上用葡萄牙文 [**写内容，在 YouTube**](https://talkingabouttesting.com/?s=protractor) [**上分享**](https://www.slideshare.net/walmyrlimasilvafilho/presentations) **[**视频，在 events**](https://www.youtube.com/user/wlsf82/videos) 上演讲[**，指导 QA 从业者专门研究这个主题，维护**](https://www.slideshare.net/walmyrlimasilvafilho/presentations) **[**量角器助手**](https://www.npmjs.com/package/protractor-helper) 库，创建这个库是为了帮助其他专业人士写作 写两本书(一本用 [**英文**](https://leanpub.com/end-to-end-testing-with-protractor) 另一本用 [**葡萄牙文**](https://www.casadocodigo.com.br/products/livro-protractor) ，两本书都是经验教训的集合)，把项目以 [**codelab**](https://github.com/wlsf82/protractor-and-webrtc) 和 [**架构建议**](https://github.com/wlsf82/protractor-components-and-page-objects) 的形式保存在 **GitHub** 上，并在 [**谈论测试学校**](http://talkingabouttesting.coursify.me/courses/arquitetura-de-testes-com-protractor) 授课，如果我还没有😀****

与 Selenium 相比，**量角器的一个显著优点是，**是一个更简单的语法[](https://www.protractortest.org/#/webdriver-vs-protractor)**，你可以用更少的代码做同样的事情，有助于可读性和可维护性。**

**对于习惯于对库进行**单元测试的开发人员来说，比如 **Jasmine** 或者 **Mocha** ，两者都受到**量角器**的支持，第一个是默认的。****

**关于文档，我最喜欢的部分是 [**API**](http://www.protractortest.org/#/api) ，它总是随着新版本一起更新。**

****量角器**也支持市面上使用最多的浏览器，还有 **Selenium** 。**

****TypeScript** 支持是它的另一个优势。**

**最后主要由 [**Angular**](https://angular.io) 团队维护，由 **Google** 支持。**

## **[柏树](https://www.cypress.io)**

****Cypress** 是我目前正在使用的**端到端测试**工具(在一些个人项目中)。到目前为止，引起我注意的是它出色的 [**文档**](https://docs.cypress.io/guides/overview/why-cypress.html#In-a-nutshell) 、易用性，它支持对 REST API 的 [**测试，开发工具如**与 Chrome 开发工具**、**](https://docs.cypress.io/api/commands/request.html#Syntax) **[**集成调试**](https://docs.cypress.io/guides/guides/debugging.html#Using) 、 [**交互模式**](https://docs.cypress.io/guides/core-concepts/test-runner.html#Overview) 、 [**存根**](https://docs.cypress.io/api/commands/stub.html#Differences) 支持、[](https://docs.cypress.io/api/commands/exec.html#Syntax)****

## ****[背刺](https://garris.github.io/BackstopJS/)****

******BackstopJS** 是我用过的最直白的 [**截图对比测试**](https://talkingabouttesting.com/?s=backstop) 工具，也叫**视觉回归测试**。****

****我已经使用 **BackstopJS** 大约一年了，我很高兴。它拥有优秀的 [**文档**](https://github.com/garris/BackstopJS) ，是 GitHub 上相对活跃的 [**项目。它**](https://github.com/garris/BackstopJS/graphs/code-frequency) **[**支持在 Docker 容器**](https://github.com/garris/BackstopJS#using-docker-for-testing-across-different-environments) 内运行测试，支持 [**木偶**](https://github.com/GoogleChrome/puppeteer) 引擎。******

**我对 **BackstopJS** 用户的两个“贡献”是[**back stop-config**](https://github.com/wlsf82/backstop-config)项目( [**包含在工具**](https://github.com/garris/BackstopJS#tutorials-extensions-and-more) 的官方文档中)和使用 BackstopJS 进行可视化回归测试的 [**课程(只有葡萄牙语版本)。**](http://talkingabouttesting.coursify.me/courses/testes-de-regressao-visual-com-backstopjs)**

## **第 2 部分——集成测试自动化(API 测试)**

## **[jsdom](https://github.com/jsdom/jsdom)**

****Jsdom** 是一个可以和这里将要讨论的其他工具结合使用的库，比如 **SuperTest** 和 **Chai** 。使用 **jsdom** ，可以模拟 web 浏览器的子集来测试应用服务器，而不需要真正的浏览器。**

**用 **jsdom** 库可以做的一个例子是**解析由 **REST API 调用**返回的 HTML** ，并执行类似于 **querySelector** 函数的东西来返回一个特定的 HTML 元素，然后断言该元素具有给定值。例如，可以使用 **Chai** 库来执行这种验证。**

## **[超级测试](https://www.npmjs.com/package/supertest)**

****SuperTest** 用于**测试 HTTP REST 调用**，允许你测试**获取**、**发布**、**放置**、**删除、**和**修补**的请求。**

**可以使用该库完成的一些可能的测试有:**

*   **验证 **HTTP 请求**的响应是否返回正确/预期的**状态代码**；**
*   **验证 **HTTP reques** t 的响应在其标头中返回了正确的位置值；**
*   **例如，结合像 **jsdom** 和 **chai** 这样的工具，它可以断言在 **POST** 请求后使用特定 CSS 选择器返回的 HTML 元素的数量。**

## **[柴](https://www.chaijs.com)**

****Chai** 是一个**断言库**，不仅适用于节点环境，也适用于 web 浏览器，因此这个工具使得执行类似于 **jsdom** 和 **SuperTest** 中提到的检查成为可能。**

**当用于**集成测试**时，可以用 **Chai** 库执行的断言的一些例子是:**

*   **验证用 HTTP 请求的返回填充的特定变量包含给定的字符串或等于给定的字符串；**
*   **验证用 HTTP 请求的返回填充的特定变量具有一定的长度；**
*   **验证用 HTTP 请求的返回填充的特定变量是否具有特定的属性。**

## **[柏树](https://www.cypress.io)**

**正如在关于使用 **Cypress** 的**自动化 GUI 测试**一节中简要讨论的，除了通过图形用户界面测试交互之外， [**还可以测试 REST API**](https://docs.cypress.io/api/commands/request.html#Syntax)。**

**可以用 **Cypress** 进行的 **API 测试**的一些例子有:**

*   **验证状态代码；**
*   **验证 URL 重定向；**
*   **HTTP 请求的响应体验证。**

## **第 3 部分——单元测试自动化**

## **[磁带](https://github.com/substack/tape)**

****Tape** 是我用过的最直接的**单元测试库**。这样的库已经附带了断言列表，这简化了测试过程，不需要导入特定的库来执行预期结果的断言。**

**[**这里有一个 GitHub**](https://github.com/wlsf82/age-discoverer) 上的示例项目，我在那里使用 **Tape** 库来练习一些技术，比如 **TDD** ，增加**代码覆盖率**，**重构**，**清理代码**，等等。**

## **[茉莉](https://jasmine.github.io)**

**Jasmine 框架使得**行为驱动开发** ( **BDD** )在单元级别上的实践成为可能。**

**该工具的一个显著优势是**简化的语法**，它允许测试**易于编写和读取**。**

## **[摩卡](https://mochajs.org)**

**我的印象是， **Mocha** 框架是 **JavaScript** 开发者在谈论**单元测试**时使用最多的。**

**这个工具以一种简单的方式在节点和浏览器环境中支持**异步测试**。**

**它的许多功能包括:**

*   **支持**异步测试**，包括**承诺**；**
*   **使用运行单个测试。仅()，或者使用跳过单个测试。skip()；**
*   **代码覆盖率报告。**

## **[Node.js 断言](https://nodejs.org/api/assert.html)**

****Node.js 断言**是 **Node.js** 模块，用于**单元测试**中的断言。**

**它最显著的优点是已经和 **Node.js** 一起安装，不需要额外安装。**

## **[柴](https://www.chaijs.com)**

**正如在关于**集成测试**的章节中已经讨论过的， **Chai** 是一个**断言库**，用于节点环境，也用于 web 浏览器。**

**除了已经提到的，使用 **Chai** 库，除了支持使用插件之外，还可以以 **BDD** 或 **TDD** 的风格进行断言，以扩展其功能。**

## **[酶](https://github.com/airbnb/enzyme)**

****Enzyme** 是 Airbnb 推出的**单元测试实用程序**，用于对 **React** 组件执行断言。**

**可以用这种工具进行测试的一个例子是表面渲染一个**反应**组件(浅层渲染)并用一个特定的 CSS 选择器检查返回的元素数量。**

**[**这是 GitHub**](https://github.com/wlsf82/hackernews) 上的一个示例项目，我使用**酶**对 React 组件进行简单的**测试。****

## **[笑话](https://jestjs.io)**

****Jest** 是脸书推出的**测试框架**，其中 **React** 社区主要用于**快照测试**，但也可以用于需要使用**测试替身**的测试，如**模拟**。**

**它的一些优点是:**

*   **因为它是一个完整的框架，所以不需要安装其他的库。使用 **Jest，**您编写测试和断言，执行它们，然后可视化测试执行报告；**
*   **它已经支持**代码覆盖率**报告，不需要为此安装其他特定的库；**
*   **开始使用它不需要配置。**

**在**酶**测试部分的 **GitHub** 上的同一个示例项目中，我也使用了 **Jest** 框架进行**快照测试**。**

## **[伊斯坦布尔](https://istanbul.js.org)**

****伊斯坦布尔** **不是** **单元测试** **工具**而是**从**单元测试**中提取代码覆盖度量**的工具。**

**它的主要目的是清楚地说明应用程序代码的哪些部分是通过测试执行的，或者不是通过测试执行的，这使得识别风险区域和潜在的改进区域成为可能。**

**在同一个示例项目的**磁带**部分中，我使用**伊斯坦布尔**来提取**代码覆盖** **度量**。**

## **第 4 部分—静态代码分析工具**

## **[标准 JS](https://standardjs.com)**

****StandardJS** 是我所知道的最简单的静态 **JavaScript** **代码分析**的选项，有助于**代码格式化**和**林挺**，使得在整个项目代码中遵循相同的**风格指南**成为可能，而不需要任何配置。**

## **[ESLint](https://eslint.org)**

**ESLint 是一个**代码林挺**库，它使开发团队能够定义他们的**风格指南**并确保遵循这样的指南。**

**在 **ESLint** 中我非常喜欢的一个特性是自动修复，它自动纠正一些简单的规则，比如制表符的空格数、单引号或双引号、语句结尾缺少分号等。**

**今天到此为止。**

**你呢，你在不同的应用级别使用哪些工具进行**测试自动化**和**静态代码分析**？留下评论。**

**你喜欢你读的东西吗？请在您的社交网络上分享这些内容来帮助我。我会很感激的！**

**下次再见，祝你考试顺利！✅ 👋**

**这篇博文最初是在 **WordPress** 上用葡萄牙语发表的，在这里 可以看到 [**。**](https://talkingabouttesting.com/2018/10/05/um-review-pessoal-de-ferramentas-para-testes-automatizados-no-mundo-javascript-parte-1/)**