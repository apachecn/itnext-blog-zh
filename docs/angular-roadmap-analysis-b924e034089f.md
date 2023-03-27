# 角度路线图分析

> 原文：<https://itnext.io/angular-roadmap-analysis-b924e034089f?source=collection_archive---------3----------------------->

## 让我们一起来探索新公布的角度路线图！

![](img/32bf9d2136c54e79c903d91f9f837a6c.png)

图片由[马克·柯尼希](https://unsplash.com/@markkoenig)提供

在本文中，我将浏览一下[新公布的角度路线图](https://blog.angular.io/a-roadmap-for-angular-1b4fa996a771?source=collection_home---4------0-----------------------)。我将分享我对刚刚宣布的一切的看法和希望。

从现在开始，[官方角度路线图](https://angular.io/guide/roadmap)应该每季度更新一次，这将让我们更好地了解未来。

# 触发因素和动机

最近几个月/几周，Angular 公司发生了一些令人悲伤的消息。许多优秀的人离开了 Angular 团队，社区一直在想为什么。Lars Gyrup Brink Nielsen 分享了他对此的看法。那些离开的人并不代表 Angular 是出于天赋(远非如此)，但是离开的人肯定增加了很多价值，也会被社区怀念！

正因为如此，许多人开始想知道 Angular 的未来会怎样。

几年前我去 Angular Connect 的时候，空气中弥漫着一股关于 Angular 的正能量，我们可以感觉到谷歌是如何确保 Angular 将被尽可能多的人使用。人们非常关注社区。

现在，几年过去了，这个社区仍然庞大而充满活力，但是我个人感觉到谷歌的活力和参与度大大降低了。但我很可能错了:)

我们也对 Angular 最近的发展有点失望。我不想抨击，因为作为一个框架架构师/维护者(尽管相比之下是一个很小的框架)，我能够理解与每个人在表面上看到的相比，这一切是多么的困难。像 Angular CLI 这样的工具可能感觉很“基础”，但是需要太多的工程设计，一旦你意识到这一点，它几乎是令人恐惧的；-)

通过发布路线图，Angular 团队试图以更开放的方式与我们取得联系。当然，目标也是为了让我们放心，Angular 仍然活蹦乱跳，前途一片光明。我当然希望如此！；-)

Angular 拥有坚如磐石的基础，在前端“战争”中仍然有牌可打。它有自己的特点、优点和缺点，对企业来说有很多明显的优势。如果谷歌继续投资，那么它将继续造福每个人。

无论如何，毫无疑问，尽管 Angular 是开源的，但它也是谷歌许多网络应用的引擎，因此该项目受到严格控制，不能以不利于谷歌利益的方式进行。

让我们深入了解路线图！

# 在发展中

## 拜拜行动积压

正如我在关于 Angular 10 的文章中所讨论的，Angular 团队已经为许多问题的解决付出了很多努力。有了路线图，我们现在对它代表谷歌的投资有了更清晰的认识。Angular 团队一半的工程能力现在专门用于问题/ PRs 的分类。

由于这项正在进行的工作，团队将对情况、重要问题和社区请求有更清晰的认识。我们必须等到下一个路线图更新，看看这项工作对框架的优先级和下一个里程碑有什么影响。

## 支持 TypeScript 4.0

Angular 团队已经在忙于添加对 [TypeScript 4.0](https://medium.com/javascript-in-plain-english/whats-new-with-typescript-4-0-beta-a2e674846ef3) 的支持。

这没什么好惊讶的，但是知道一旦它发布了，我们应该很快得到支持，这太棒了。让我们的主框架与 TypeScript 保持同步对于生产力来说是非常好的。

顺便说一下，一定要看看[我写的关于打字稿 4](https://medium.com/javascript-in-plain-english/whats-new-with-typescript-4-0-beta-a2e674846ef3) 的文章；我已经涵盖了你需要知道的一切。

## e2e 测试策略的更新

Angular 问世时，量角器是端到端测试事实上的选择。从那以后， [Cypress](https://www.cypress.io/) 获得了很大的人气，并且是目前最好的选择。此外，现在还有像 [Nrwl NX](https://nx.dev/angular) 这样的工具的强大支持，这使得在我们的项目中引入和使用它变得轻而易举。

在此期间，我没有注意到量角器这边的很多演变(但我没有紧跟，所以不要犹豫，纠正我。

好消息是 Angular 团队将评估量角器的状态，观察生态系统的其他部分，并探索新的可能性。希望这将在未来导致这一领域的改进。

## 棱角分明的图书馆和常春藤

Angular 团队正在投资常春藤图书馆发行计划的设计和开发。正如路线图中所述，这将包括库包格式的更新，以便它使用 Ivy 编译，并最终让他们放弃视图引擎库格式以及 [NGCC](https://angular.io/guide/glossary#ngcc) 。

我不知道你怎么想，但我“讨厌”NGCC。当 CI 管道运行时，感觉很笨拙，看起来很痛苦。希望在不久的将来，我们能够忘记这一点；-)

这里有一个关于这个主题的公开 RFC:【https://github.com/angular/angular/issues/38366?s=09

## 评估未来 RxJS 的变化

如你所知， [RxJS](https://rxjs-dev.firebaseapp.com/) 是 Angular 的核心，是我们所有人的巨大胜利。尽管如此，框架的不同部分将受益于 RxJS 的更好使用/支持(例如，组件模板)。这就是为什么我们会看到像 [NGRX 组件](https://ngrx.io/guide/component)这样的库出现。

希望随着时间的推移，Angular 在这方面会有所改善。对我们来说不幸的是，现在 hat [Ben Lesh](https://twitter.com/BenLesh) 已经离开 Angular team(咩)，可能有点难到达那里了。

Angular 团队正在研究即将推出的 [RxJS v7](https://medium.com/@dSebastien/rxjs-7-in-depth-32cc7bf3e5c) 。在我之前的文章中已经提到过，我认为 Angular 不应该有太大的变化，但是团队也在考虑更多。

无论如何，我们至少会在 RxJS 7 发布时或发布后不久获得对它的支持，这本身就很酷。

## Angular 语言服务切换到 Ivy

目前，正如路线图中所述，语言服务仍然使用视图引擎编译器和类型检查；即使是常春藤的申请。

正如你所猜测的，这与理想相差甚远。通过将 Angular 语言服务切换到 Ivy 模板解析器，我们应该可以进一步改进模板的类型检查。而且，如你所知(嘿)，我喜欢我们得到更好的类型检查！

顺便说一下，如果你还没有启用[严格的模板类型检查](https://medium.com/javascript-in-plain-english/angular-template-type-checking-e2c99c50f999)，那么我真的建议你现在就启用。别读了，去做吧！我掩护你；-)

## 扩展组件管理最佳实践

随着 Angular 10 的[发布，Angular CDK 推出了](https://medium.com/javascript-in-plain-english/angular-10-in-depth-a48a3a7dd1a7)[组件测试工具 API](https://material.angular.io/cdk/test-harnesses) 。

新的 API 旨在帮助我们更容易地编写和维护组件测试。当然，现在还为时过早，所以模式和最佳实践必须被识别和巩固。

Angular 团队拥有第一手经验，他们将组件测试工具用于 Angular 材料组件。基于此，他们将改进 API，并为我们所有人阐明最佳实践。

正如路线图中提到的，他们也在推动在谷歌内部更多地采用这种方法。

## 支持本机受信任类型

在路线图中，还宣布谷歌现在正忙于增加对新的[可信类型 API](https://web.dev/trusted-types/) 的支持，这是网络平台的一部分。这真是令人兴奋的消息！

这个新的 API 旨在帮助我们防止基于 DOM 的跨站点脚本(XSS)攻击。希望大家都知道，即使在 2020 年，这种类型的攻击在 OWASP 十大攻击中也名列前茅。

可信类型 API 是在 Chrome 83 中引入的，其他很多版本都不支持，但是有一个 polyfill。不过，随着时间的推移，采用率肯定会提高。因为这对每个人都是双赢的。

您可以在此了解更多信息:

*   [https://web.dev/trusted-types/](https://web.dev/trusted-types/)
*   [https://github.com/w3c/webappsec-trusted-types](https://github.com/w3c/webappsec-trusted-types)
*   [https://developer . Mozilla . org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/trusted-types](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/trusted-types)

## 将 MDC 网整合到角形材料中

很长一段时间以来，谷歌的材料设计团队一直忙于创建自己的纯 CSS 材料设计实现。

当然，这在 Angular Material 所做的(即，最好遵循规范)和 MDC Web 实现(实际上应该与规范相当，因为它是由材料设计团队自己开发的)之间产生了差异。

此外，复制和维护基于相同规格的重复样式显然是对谷歌资源的浪费，因此他们尝试将角度材料与默认实现相匹配是有意义的。

虽然，我想这需要 Angular Material 团队做大量的工作，我想知道这对现有项目的影响。对于那些根据需求调整了样式的人来说，可能会有相当多的问题需要解决(取决于他们是如何做的)。

尽管如此，这种变化是为了更好！

## 角度版本控制和分支

Angular 团队正忙于整合现有存储库之间的发布管理工具。这将允许他们重用基础架构、统一方法并简化流程。

当然，还应该提高 Angular 释放过程的可靠性。

# 将来的

因此，这个长长的列表实际上涵盖了 Angular 团队的当前和短期活动。但是我们还有更多要讨论的:未来。

## 刷新介绍性文档

介绍性文档将被重新定义，并应更好地强调 Angular 的好处，如何探索其功能并提供指导。

就目前情况来看，Angular 文档已经相当扎实了，所以我对即将到来的东西充满热情！

## 反应式表单的严格类型

这个让我觉得有点难过。当然，我们今天不可能全部得到；我们知道工程需要时间。此外，让这样一个 API 进化不是轻松的任务。尽管如此，我还是希望看到这个在列表中更靠前的位置；-)

Angular Reactive forms 缺乏强类型是 Angular 多年来最大的问题之一。

希望将来情况会有所改善。在此期间，我们可以暂时切换到诸如 [ngneat/reactive-forms](https://github.com/ngneat/reactive-forms) 之类的库，它们可以暂时将我们从弱类型地狱中解救出来。

## Webpack 5 支持

正如我在关于 [Angular 10](https://medium.com/javascript-in-plain-english/angular-10-in-depth-a48a3a7dd1a7) 的文章中所写的，Webpack 将留在 Angular 领域。这让我们所有人都感到安心，尤其是那些不得不定制他们的构建的人。

在接下来的几个月中，Angular 团队将迁移到 Webpack 5，这将提高构建速度并减小包的大小。

## 提交消息标准化

这是我打算将来写的一个主题。提交消息标准化对于清晰性来说是非常棒的，并且是发行说明和变更日志生成的推动者。

Angular 在这方面做得很好，但团队打算更进一步，改进他们的提交消息，以带来更多的一致性。

## 可选 zone.js

我在本文前面提到的 NGRX component 之类的库可以帮助我们走向无区域应用。那就是不需要 [Zone.js](https://github.com/angular/angular/tree/master/packages/zone.js/) 就能发挥作用的角度应用。

不深究细节，如果我们简化一下，Zone.js 就是帮助 Angular 检测变化的库。它通过对许多 API 进行猴子修补来做到这一点，以便 Angular 在事情发生时得到通知。

不幸的是，Zone.js 有一些缺点，包括它不支持 async/await 语法，并且对包的大小有影响。

将来，Angular 团队希望让 Zone.js 成为 Angular 应用程序的可选选项。

不错！

## 移除旧视图引擎

你可以想象，一旦 Ivy 最终成为应用程序和库唯一需要的引擎，Angular 将最终能够放弃传统的视图引擎。

同样，这将减轻代码库，并导致更小的包大小。

## 角度开发工具

Angular 与 React 或 Vue 不相上下的一个领域是开发工具。当然，我们有很棒的命令行界面，但是一旦我们的浏览器打开了，我们就没什么可看的了。

据我所知，[占卜](https://augury.rangle.io/)几乎是我们唯一的工具，它与 Angular 10 不兼容(至少我上次检查时不是这样)。此外，它不是由 Angular 团队维护的。

在未来，我们可能最终会拥有更好的工具。如果有像 [React 开发工具](https://chrome.google.com/webstore/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi?hl=en)和 [Vue.js 开发工具](https://chrome.google.com/webstore/detail/vuejs-devtools/nhdogjmejiglipccpnnnanhbledajbpd?hl=en)这样的酷工具，那就太棒了。

## 可选 NgModules

这又是一个重大的声明。NgModules 是在 Angular“2”发布后不久推出的，目的是帮助我们更好地构建我们的应用程序，并允许延迟加载场景、性能改进等。

当然，NgModules 代表了额外的复杂性，从而增加了精神开销。

在未来，该团队显然想让 NgModules 成为可选的。这种设计上的改变可以为不同的(更简单的)开发体验铺平道路，我很好奇这对现有的 API 和应用程序意味着什么。

## 人体工程学组件级代码拆分 API

Angular 团队计划帮助我们进一步改善初始加载时间。为了实现这一点，他们希望为我们提供在组件级应用更细粒度代码拆分的方法。

同样，我很好奇这在实践中会是什么样子。

## 迁移到 ESLint

我很高兴看到这最后一点出现在短期路线图中。如您所知，TSLint 已被弃用，并将在年底停止支持。

它将继续工作，但在那之后就不会进化了。前进的道路确实是 [ESLint](https://eslint.org/) 。

Angular 团队将为现有的 Angular 应用程序准备迁移策略，并在 CLI 中引入对 ESLint 的支持。

即使过渡不一定是 1:1 和完全自动化的，这种 JS/TS 棉绒的整合对我们所有人都有好处。

# 结论

在本文中，我浏览了新发布的角度路线图。

正如我们所看到的，现在有很多事情正在进行，在不久的将来有很多想法可以进一步改进 Angular。对于 Angular 社区来说，这是一个非常积极的信号。

从更个人的角度来说，我有点失望，没有提到 Angular 团队本身，它的组织，团队中的许多离职(和新人？？)，还有，很简单，人的方面。我想象商业就是商业(你好甲骨文)，但与我在 2017 年看到的相比，这是令人失望的。但也许只是我？；-)

今天到此为止！

PS:如果你想了解大量关于软件/Web 开发、TypeScript、Angular、React、Vue、Docker/Kubernetes 和其他很酷的主题的其他很酷的东西，那么不要犹豫[去拿一本我的书](https://www.amazon.com/Learn-TypeScript-Building-Applications-understanding-ebook/dp/B081FB89BL)并订阅[我的简讯](https://mailchi.mp/fb661753d54a/developassion-newsletter)！