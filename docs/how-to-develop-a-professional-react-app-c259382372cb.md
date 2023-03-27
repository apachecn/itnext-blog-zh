# 如何开发专业的 React 应用

> 原文：<https://itnext.io/how-to-develop-a-professional-react-app-c259382372cb?source=collection_archive---------0----------------------->

你是否希望获得成为 6 位数前端工程师的技能，并掌握非常受欢迎的 React.js 生态系统？

*如果是的话，那就在 ProfessionalReactApp.com*[](https://professionalreactapp.com/special-offer)**报名参加一个令人兴奋的新培训项目吧！**

**避免错误——比如大量的技术债务——从专业人士那里获得指导，学习如何以正确的方式设计你的应用。最快、最简单、最省事的学习方法是让专家指导您掌握应用程序。**

*![](img/dc8857852b0383898ac25b862f12d3d5.png)*

*ProfessionalReactApp.com*

# *介绍*

*React 及其相关 JavaScript 库的生态系统拥有 ***巨大的市场份额*** *，*并且它正被用于驱动许多成功的 SaaS(软件即服务)产品的前端。*

*但是如果你想得到一份非常受欢迎的开发人员的工作，你需要知道的不仅仅是反应。*

*如果你想在现实世界中加入一个高技能工程师团队，或者如果你想成为一名为客户工作的独立顾问，下面这篇文章是你应该知道的指南。*

# *反应元件的思考与发展*

*![](img/d71fc7424ef6f2136a77cbb0ab3a1b7e.png)*

*所有用红色标出的都是 React UI 组件。许多组件都是高度可重用的。*

*今天的现代界面是根据单个 UI 组件构建的。这些组件通常(但不总是)被设计成高度可重用的。因为我们开发界面时考虑到了可重用性，所以对复杂前端的可管理性的改进是**巨大的**(“我一直喜欢这个词……‘巨大’……所以很少有机会在一个句子中使用它”！).*

*当你使用像 React 这样非常流行的库时，你制作那些可重用组件的方式是标准的和可预测的。可重用性和可预测性允许产品和团队扩展。使用 React 这样的库，可以更容易地在整个应用程序或一套 N 个应用程序中实现一致的体验。*

*React 在不断发展，不断改进其语法，提高性能，现在它完全免费使用，没有任何附加条件(MIT 许可证)，它有一个庞大的第三方组件生态系统，您可以利用它，许多团队都在使用它。因此，这是你的武器库中拥有的一项有利可图的技能。*

*新开发人员也更容易学习。唯一的问题是弄清楚所有这些不同的组件是如何编排和组合在一起的，以便产生一个功能齐全的应用程序。换句话说，所有这些独立的 React UI 组件是如何接收数据并相互通信的？*

*这就是 Redux 的用武之地。*

# *管理应用程序状态和数据流*

*![](img/0fbf00bacbfcee3fdd0d26baa53b3f1e.png)*

*流经专业 React 应用程序的所有数据都包含在内存中一个名为“Redux Store”的对象中。*

*一旦你开始使用高度可重用的 React 组件构建你的应用，下一步就是弄清楚如何 1) *管理流经你的应用的数据*和 2) *管理* *应用的状态*。*

*输入 **Redux** 。Redux 是一种高级的*设计模式*，它附带了一组 JavaScript 库，您可以使用这些库来实现它。还有 MobX 和 Flux，但 Redux 是目前最受欢迎的。*

*你看，一旦你有了一个由许多小组件组成的界面，它们都需要以某种方式相互通信。他们需要将面向用户的数据输入其中。如果用户在一个组件中做了一些改变底层数据的事情，这种改变需要反映在组成整个应用程序的其他组件中。*

*Redux 已经被证明是一种管理应用程序状态的*高效且可伸缩的*设计模式，它大量借鉴了函数式编程哲学。然而，对于新开发人员来说，它确实有一个学习曲线。但是一旦 [*你掌握了这个关键的设计模式*](http://professionalreactapp.com/) ，你就能很好地编写专业的 React 应用程序。*

*如果您不知道“应用程序状态”是什么意思，请不要担心。你可以在这里了解更多。只需知道，在从事专业的 React 项目时，您必须了解如何管理您的应用程序的状态。现在就投入时间学习 Redux 吧！*

# *Webpack 支持您的整个应用程序*

*![](img/c8fd98bd27c342565c645da237ff48eb.png)*

*主要 React 应用程序的 Webpack 配置通常分为两种不同的环境，开发和生产。*

*因此，你有一个用 ES6(JavaScript 语言的最新版本)编写的独立 React 组件组成的应用程序，它需要被编译成所有浏览器都能够解释的 JavaScript 版本。你还有 CSS、图片、字体、SVG 等资产。所有这些都需要处理和编译成最终输出，最终发送到用户的浏览器。*

*这就是 Webpack 的用武之地。Webpack 是一个资产捆绑器，由*驱动整个 React 应用*。简而言之，它采用所有的 JavaScript、CSS 和任何其他前端资产，并生成您在浏览器中看到的输出。它可以做一些高级的事情，比如[代码分割](https://webpack.js.org/guides/code-splitting/)来提高你的应用程序的性能，[延迟加载](https://webpack.js.org/guides/lazy-loading/)，[缓存](https://webpack.js.org/guides/caching/)，[树抖动](https://webpack.js.org/guides/tree-shaking/)，并且它可以帮助你[创建整个库](https://webpack.js.org/guides/author-libraries/)来跨团队分发。*

*因为 Webpack 一开始看起来很吓人，所以很多开发人员不知道如何使用它。他们中的许多人不知道如何为大型 React 应用程序构建 Webpack 配置。如果您正在为客户端构建 React 应用程序，您必须了解如何使用 Webpack 以任何所需的方式配置应用程序的功能。*

*了解 Webpack 是一项强大的技能，能让你从其他开发人员中脱颖而出。*

*如果 Webpack 对您来说是一个谜，您需要了解行业专业人员如何使用它来驱动真实世界的 React 应用程序，那么现在就[学习如何使用它](http://professionalreactapp.com/)并成为 Webpack 专家。*

# *大型 React 应用的 CSS 架构*

*![](img/a814d90d052de44dea153f9fcc1a02b0.png)*

*SASS ( [语法上令人敬畏的样式表](https://sass-lang.com/))配置，用于专业 React 应用程序的集中主题化。*

*我为许多初创公司工作过，我无法告诉你我遇到过多少次笨拙和难以管理的 CSS 代码。伙计们，外面有很多糟糕的代码。*

*初创公司几乎总是匆忙地尽快推出产品，以便在用户身上进行测试。在匆忙交付的过程中，他们中的一些人搞砸了他们的 CSS，直到它变成一个臃肿的大烂摊子，背负着沉重的技术债务。我曾经参与过一些项目，它们的 CSS 实现非常糟糕，以至于产品的性能受到了很大的影响。在一些非常糟糕的情况下，我几乎想离开。如果你是这样一个项目的开发者，你将不得不清理这些。*

*今天，专业的 React 应用程序使用复杂的工具链来编写 CSS 代码。他们用 SASS ( [语法上很棒的样式表](https://sass-lang.com/))来编程 CSS，用 CSS 模块来组织(我喜欢！)、实现响应视窗的媒体查询等等。*

*你能给你的团队带来的最大的好处之一就是知道如何正确的皮肤你的 React 应用。你的项目需要有一个专业的外观和主题，因为这是每个 UX 体验非常重要的一部分。*

*此外，你的设计者想要尽可能多的控制来改变整个应用程序的外观和感觉。因此，你应该学习如何为你的应用程序创建一个[可配置的主题，并以一种适合开发者和设计者的方式实现你的 CSS。](http://professionalreactapp.com/)*

# *自动化林挺*

*![](img/9ba4c89fed2cac14fcd8572ec2bf34d1.png)*

**皮棉，婴儿皮棉*。您的团队需要一个自动化的开发人员工作流来检查代码的健康和质量。一个大规模的 React 应用程序，由一组开发人员操作，必须遵循一个风格指南，并且必须不断地进行测试和质量检查。*

*为了保持代码健康和干净，每个 JavaScript 应用程序都应该被自动测试。在我的专业项目中，我将我队友的 linters 配置为在开发人员能够在他们的本地机器上提交代码之前自动执行*。这可以防止开发人员在开发周期的早期签入坏代码。**

*不仅如此，你可以配置你的 linter 来自动遵循指定的风格指南。很多团队使用 AirBnb 的 [JavaScript 风格指南](https://github.com/airbnb/javascript)。在我看来，AirBnb 风格指南非常严格，但如果它们惹恼了你，你可以放松一些规则。因此，使用 AirBnb 风格指南开始您的项目，并根据您团队的喜好进行定制。*

*您的专业 React 应用程序必须自动 lint 代码，并遵循标准的风格指南。事实上，每一个用 JavaScript 写的专业项目都必须 linted。这种语言有太多的陷阱，会诱使开发人员因为察觉到的语言缺陷而引入意想不到的错误。*

# *设计更高级别的组件*

*![](img/80096404489161045bf4781d21536fc2.png)*

*大型 React 应用程序包含更高级别的可重用组件，这些组件代表了常用的 UX 元素。比如情态动词。这些高级组件对于应用程序的整体体验非常重要，因此必须小心设计。*

*在我最后一次签约时，我为一家非常成功的 SaaS 产品公司工作，这家公司赚了很多钱。该公司有一个庞大的工程师团队，需要快速迭代功能。事实上，人们被解雇是因为管理层认为他们“不够快”！*

*这个特定的 SaaS 产品的一个主要技术问题是，没有人知道如何显示一个基本的模态。整个应用程序中有多个模态组件，所有这些组件都增加了整个 JavaScript 负载的大小。花了几个小时才弄明白如何做一些基本的事情，比如向用户显示一个模态(T2)。换句话说，产品的技术债务拖慢了开发者的速度，导致人们被*解雇*(不是开玩笑)。*

*我最终不得不为整个团队解决这个技术债务，并最终学到了一些宝贵的经验。更高级别的组件必须经过深思熟虑和精心设计。如果你需要学习如何为大规模的 React 应用程序构建一个高度可定制和可配置的模态组件，那么在这里查看我的教程。*

# *利用第三方组件库*

*![](img/72c46f1a70722e2da94d98a1c207091c.png)*

*新版本的 Material-UI 出来了！*

*在 React components 中构建您的界面最酷的事情之一是，有一个免费和开源组件的宝库可供您选择。*

*你甚至可以使用第三方用户界面库来应用你的整个应用程序的整体外观。一些团队利用 [Material-UI](https://material-ui.com/) ，一些团队利用 [React-Boostrap](https://react-bootstrap.github.io/components/alerts/) 。我都用过，都很棒。*

*如果你正在创造一个 SaaS 产品，而你手头拮据，也许你不能雇佣一个设计师。或者，您可能是一名主要关注工程的开发人员，对获得设计技能不感兴趣。如果是你，你所要做的就是将一个预先制作的库导入到你的应用程序中，并学习如何使用它，以便获得专业的外观和感觉。*

*根据我的经验，我发现即使是拥有专业设计人员的团队仍然会使用第三方库。只要可以进行风格上的调整，设计师很乐意利用第三方设计师的作品。毕竟对他们来说是少干活！*

*但你需要知道如何正确地将第三方库和设计系统整合到你的应用中。您希望能够重用所有这些预制的逻辑，但是您也需要能够使其适应产品的独特需求。*

# *测试对于开发团队来说是强制性的*

*![](img/430c328827fa8f587af6ecc711356740.png)*

*你必须为你的应用程序编写测试。我没开玩笑。随着您团队的应用程序规模的增长，没有测试的工作变得很可怕。*

*为您的 React 应用程序实现自动化测试套件非常重要。最终，你将面临决定什么时候写测试，什么时候不写。事实上，自动化测试编写中最大的挑战之一是确定何时编写有价值的测试。*

*管理层并不总是花时间来测试写作。不等等。他们根本没有为自动化测试写预算时间。他们可能从工程师那里听说这是一件很重要的事情，但是他们并不是每天都在编写和管理代码，所以他们不太关心。他们只是希望产品按预期工作，不希望有 bug，希望你快速实现新功能。*

*作为一名开发人员，您需要测试来确保事情按预期运行。随着时间的推移，许多其他开发人员会接触和编辑你的工作，你不希望他们破坏东西。*

# *用户界面组件库*

*![](img/0b4e6ddc256697d0ba5866dca72724a3.png)*

*可以用 [React 故事书](https://storybook.js.org/)创建一个团队的 UI 库；然而，我发现有些团队更喜欢其他选择。*

*现在来锦上添花。复杂的 SaaS 产品通常由多个应用程序组成。用户可能没有注意到，但有几个应用程序一起工作，以产生一个整体的体验。或者，一家公司可能有一套不同的软件应用程序，都需要遵循相同的 UX 主题和公司品牌。*

*这就是公司范围的 UI 库的用武之地。作为一名开发人员，如果你知道如何维护(和实现)一个 UI 库，一个开发团队可以利用它来构建他们的应用程序套件，你将拥有一笔巨大的财富。*

*为整个组织建立 UI 组件库是一项非常高级的技能。*

# *正确设置应用程序的基础*

*软件开发是一个漫长而充满挑战的旅程。除了你需要具备的所有硬技术技能，你还必须善于与人合作。没错，就算是软件工程也是以人为本的工作。*

*我不会骗你说所有的技术技能都很容易学会。很难。但是，如果你想获得丰厚的报酬，或者你想向世界释放你的创造力，那么用你需要的知识武装自己，做好这件事。*

*如果您是一名顾问或开发人员，并且您需要关于如何使用这个高级 React 样板文件的逐步指导，或者您想要了解所有移动部分(Webpack 配置、CSS 架构、目录结构等)。)然后报名参加我在 ProfessionalReactApp.com[的培训项目。当我们构建一个真正的应用程序时，我将指导你使用样板文件。](https://professionalreactapp.com/special-offer)*

*随着知识和技能而来的是 ***更大的自由*** 。*