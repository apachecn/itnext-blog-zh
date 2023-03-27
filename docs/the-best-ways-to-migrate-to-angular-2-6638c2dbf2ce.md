# 迁移到 Angular 2 的最佳方式

> 原文：<https://itnext.io/the-best-ways-to-migrate-to-angular-2-6638c2dbf2ce?source=collection_archive---------3----------------------->

![](img/b0e8b4b5eb2cdef989a34d7d4328afc9.png)

Angular 开发者和投资 Angular 的企业在过去的一年里一直密切关注 Angular 2 的最新发展。与此同时，Angular 2 的稳定版本在漫长的等待后发布了。一些企业和 Angular 开发者已经卷起袖子将他们的 Angular 1 项目迁移到 Angular 2。

将 Angular 1 项目迁移到 Angular 2 可能没有您想象的那么简单。团队意识到这一事实后，已经在迁移计划中采取措施来简化未来的过渡。我写这篇文章的目的是为您提供迁移准备方面的建议，并分享实现这一目标的可能方法。

## **我为什么要将现有项目升级到 Angular 2**

首先，重要的是要考虑为什么升级到 Angular 2 会是一个好主意。

开发人员知道，构建 Angular 2 的想法是在 Angular 社区提出需求时形成的。随着这些需求和要求的出现，变化和区分角度 2 和角度 1 的特征浮出了水面。

在开发者中，众所周知的事实是 Angular 2 是一个主要关注性能的更新。Angular 社区发现了大规模和复杂应用程序的性能问题，并在这个问题上严重困扰了他们的大脑。在 web 工作人员和服务器端渲染的帮助下，他们似乎解决了复杂应用程序中出现的大多数性能问题。

通过关注可伸缩性，现在通过使用诸如 [RxJS](http://reactivex.io/) 和 [Immutable.js](https://facebook.github.io/immutable-js/) 这样的推送模型，大规模处理数据变得更加可行。

将现有项目升级到 Angular 2 还支持多平台使用，这使得对 web、移动 web、本地移动和本地桌面平台使用相同的代码成为可能。这种对多平台的关注是许多开发人员在效率上的一大胜利。

## 【TypeScript 如何帮助迁移

Angular 2 文档提供 TypeScript 作为默认编译器。TypeScript 是 JavaScript 的类型化超集，现有的 JavaScript 项目可以简单地通过从。js 到. ts。

除了获得 ES6 和 ES7 特性的好处，TypeScript 还通过为 JavaScript 启用静态类型来提高项目的可维护性。因此，与 JavaScript 相比，它创建了一个没有错误的环境。它还使外部和内部类型定义的代码更具可读性。类型注释是在项目中使用 TypeScript 的另一个原因，因为 JavaScript 目前没有与类型注释相同的构造。

许多企业已经用 TypeScript 编写了他们的 Angular 1 代码，这使得他们的代码库更易于维护，更安全，同时也为迁移到 Angular 2 做好准备。将为 Angular 1 编写的 TypeScript 类升级到 Angular 2 并不费力，因为 TypeScript 是一种面向对象的编程语言，Angular 2 将它作为默认编译器。

## **逐步升级您的 Angular 1 代码库**

我们可以开始享受 Angular 2 所提供的一切，而无需完全升级我们的 Angular 1 代码库。Angular 2 带来了用于迁移 Angular 1 项目的内置工具。

有效升级的最佳方式之一是增量升级，在同一个应用程序中同时运行两个系统，然后将 Angular 1 组件一个接一个地移植到 Angular 2。这使得在不干扰不同业务的情况下升级甚至广泛而复杂的应用程序成为可能，因为这项工作可以在一个时间框架内合作完成。Angular 2 中的升级模块旨在使增量更新保持一致。

通过遵循 [Angular 1 风格指南](https://github.com/johnpapa/angular-styleguide/blob/master/a1/README.md#single-responsibility)，你可以让你的应用程序处于更干净、更易维护的状态。将你的应用程序转换成[基于组件的应用程序架构](https://docs.angularjs.org/guide/component)将理想地让你建立一个组件树，从而最小化双向数据绑定。因此，您的应用程序将更紧密地与 Angular 2 保持一致。[基于特征的文件夹结构](https://github.com/johnpapa/angular-styleguide/blob/master/a1/README.md#folders-by-feature-structure)和[模块化](https://github.com/johnpapa/angular-styleguide/blob/master/a1/README.md#modularity)是在高层次抽象上定义相似原则的关键规则。

下一步，您可以使用一个名为 *UpgradeAdapter* 的升级模块，这是一个用于引导和维护同时支持 Angular 2 和 Angular 1 代码的混合应用程序的服务。这将是逐步升级 Angular 1 代码库的关键工具。

请记住，Angular 团队在 Angular 1 的每个主要版本中都引入了很棒的工具和改进，让您可以用 framework 的最新架构思想简化您的应用程序。他们在 1.5 版本中引入了组件方法，此外，他们最近的 1.6 版本包括禁用自动绑定和 ngModelOptions，这是您可以为 Angular 1 应用程序提供的最佳性能增强。

## **开始新项目**

Angular 2 测试版已经发布了很长时间，距离最终版本发布还没有一段时间。如果你出于任何原因不想开始 Angular 2 项目，你可能想通过使用 Angular 1 上的最新功能来接近 Angular 2 的思维模式。

我强烈建议您从 Angular 2 开始您的新项目，并关注 Angular 的新版本，以跟上它们的最新功能。这将为您打开强大的性能增强和可维护性的大门。

## **结论**

如果你正在考虑升级，你可以做很多事情将 Angular 1 代码库迁移到 Angular 2。您应该开始逐步升级您现有的代码，或者立即开始在您的新项目中编写 Angular 2 代码。

然而，如果你的应用没有被足够的测试覆盖，你可能会遇到很多问题，所以要注意，如果你的组件还没有被覆盖，考虑先用测试覆盖它们。

Angular 社区对 Angular 2 的最终版本非常兴奋，对于即将到来的版本，他们仍然如此。Angular 在 2017 年和 2018 年的下一个版本上有着积极的计划。对于使用 Angular 1 的公司来说，现在是时候制定他们的迁移计划了，如果他们还没有这样做的话。