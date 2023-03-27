# 🔥2022 年 Angular 14 期待什么:微前端来了吗？🤫

> 原文：<https://itnext.io/what-to-expect-from-angular-14-in-2022-is-micro-frontend-coming-7932566f773?source=collection_archive---------0----------------------->

Angular 13 最近发布了，但变化更为进化，我们能期待更多来自 **Angular 14** 的**革命性**更新吗？

![](img/4d420e683278ba1bc40d009cc1833d4b.png)

*   [**简介**](#a789)
*   [**三件大事纷至沓来**](#f827)
*   [**下一个版本会支持微前端吗？**](#68db)
*   [**结论**](#ac4d)
*   [**了解更多**](#ac4d)
*   [**你真了不起🤩**](#57f7)

# 介绍

在 Angular 13 发布之后，围绕它的宣传还不够，尽管它确实值得更多的关注。像 *TypeScript 4.4* 和 *RxJs 7.4* 支持这样的更新听起来不错，但是蛋糕上的主要樱桃是**构建缓存**，它将构建时间提高了 68% 。但是什么时候才能期待一个真正的大事件呢？自从引入 Ivy 以来，这个更新将会再次震撼前端社区。

*一起来了解一下吧！*

# 3 件大事即将来临

Angular 团队定期分享其即将发布的路线图。他们从社区反馈中收集信息，并与内部功能路线图相结合。让我们一起来看看他们中最精彩的。

**更好打字@angular/forms。**这些变化将影响*反应式*类型模型，并将使类型检查比以前更具冲击力。该团队的目标是实现与其他 angular 版本兼容的类型系统，并且不会导致退化。这种特性将允许开发人员在开发期间发现问题，并显著改进类型检查。

**独立组件脱离模块。**让我们试着回答这个问题，*哪个是唯一一个组件不是“复用单元”的前端框架？*当然，你知道答案——只有 Angular 框架是围绕*模块*构建的。在 angular 中，模块实际上充当了"*复用单元*"的角色，比如 Angular 库正在发布 NgModules 或者 NgModule 是懒加载的主要单元。这种概念不允许使用指令、管道，更重要的是，不允许单独使用模块范围内的组件。

事实上，模块是 Angular 中的核心构建块，这为开发经验设置了一系列限制:

*   *组件*必须是*总是依赖于*模块并且是模块的一部分，不能是独立的；
*   *复杂的 API* 围绕*加载*和*渲染* *组件。*例如，使用`bootstrapModule()` vs `bootstrapComponent()`
*   在优化构建性能的过程中，角度工具紧密依赖于模块。

主要的工作是改变架构，将*组件*、*指令*和*管道*放在框架的中心。简单来说，你可以将 ***导入*** *组件、指令、管道* ***和*** ***直接使用*****。**

**最后一个——也是 Angular 社区创造了一个故事并期待已久的——微前端支持。**

# ****下一个版本会支持微前端吗？****

**关于 Micro Frontend 有很多讨论，要想知道为什么它很酷，为什么 Angular 是它的完美框架，先看看另一篇文章:**

**[](/how-micro-frontend-changes-the-future-of-angular-bb4deb2cfdad) [## 🔥微前端如何改变 Angular 的未来？

### 让我们看看为什么 Angular 最适合微前端

itnext.io](/how-micro-frontend-changes-the-future-of-angular-bb4deb2cfdad) 

在引入*模块联盟*之后，Webpack 已经接近了未来。它允许您在一个应用程序中拥有多个独立的构建。来自 NX 的团队也将其集成到他们的命令行中，这使得引导微前端应用程序变得更加容易。最后重要的是，所有这些工具都与有角度的建筑完美结合。如果您对如何自己构建它感到好奇，请查看这篇文章。

[](/building-angular-micro-frontend-with-ngrx-state-sharing-and-nx-cli-7e9af10ebd03) [## 🔥利用 NgRx 状态共享和 NX cli 构建角度微前端

### 如何在几乎不编码的情况下构建健壮的微前端架构；)

itnext.io](/building-angular-micro-frontend-with-ngrx-state-sharing-and-nx-cli-7e9af10ebd03) 

你可能会说，这一切都很酷，但是在下一个版本中，我们能从 Angular 团队那里期待什么呢？目前，他们正在研究最好的抽象，以便更好地支持微前端。这是可以理解的，因为这种支持将需要 angular 框架的核心进行巨大的遗留变化。这项调查包括密切关注独立部署，以及大规模微前端应用程序的开发，并找到将它们与 angular 架构集成的方法。我们只能猜测，他们与 Webpack 团队和 NX 团队合作，使集成过程更有成效。

# **结论**

关于 Angular 14 或任何其他即将发布的版本是否支持微前端的信息非常少。这绝对是 Angular 和整个前端社区的下一个大事件。也很明显，这么大的一步，不是短时间能做到的。另一方面，Angular 团队一直关注并回应社区的需求和反馈。而激起围绕它的讨论，会让 Angular 团队把它的优先级放得更高。 ***如果你想参与而又等不及微前端支持的话，就在 Angular 鼓掌吧👏文章、评论或转帖吸引更多关注本话题*** 。*敬请关注并关注我的* [*媒体*](https://medium.com/@easy-web/subscribe) *和* [*推特*](https://twitter.com/EasyWebOrg) *获取关于微前端和 web 技术的最新消息。*

# **了解更多**

[](/how-micro-frontend-changes-the-future-of-angular-bb4deb2cfdad) [## 🔥微前端如何改变 Angular 的未来？

### 让我们看看为什么 Angular 最适合微前端

itnext.io](/how-micro-frontend-changes-the-future-of-angular-bb4deb2cfdad) [](/building-angular-micro-frontend-with-ngrx-state-sharing-and-nx-cli-7e9af10ebd03) [## 🔥利用 NgRx 状态共享和 NX cli 构建角度微前端

### 如何在几乎不编码的情况下构建健壮的微前端架构；)

itnext.io](/building-angular-micro-frontend-with-ngrx-state-sharing-and-nx-cli-7e9af10ebd03) [](/how-to-scale-angular-without-limits-e6a6ca15111) [## 🔥如何无限制地缩放角度

### 极限总是在你的头脑中，清空你的头脑和秤

itnext.io](/how-to-scale-angular-without-limits-e6a6ca15111) 

# 你真了不起🤩

感谢本周加入我的这些了不起的人🤗，有你在我的网络中，我感到很受鼓舞，并继续为热情的 web 开发人员发展社区。**别忘了订阅，每周都会收到更多 web devs 的见解，稍后见**😉。

[](https://medium.com/@easy-web/subscribe) [## 每当维塔利·舍甫琴科发表文章时，就收到一封电子邮件。

### 每当维塔利·舍甫琴科发表文章时，就收到一封电子邮件。通过注册，您将创建一个中型帐户，如果您还没有…

medium.comv](https://medium.com/@easy-web/subscribe) [](https://medium.com/@easy-web/membership) [## 通过我的推荐链接加入 Medium 维塔利·舍甫琴科

### 作为一个媒体会员，你的会员费的一部分会给你阅读的作家，你可以完全接触到每一个故事…

medium.com](https://medium.com/@easy-web/membership) 

[sebastian higuita](https://medium.com/u/1bd0883739f5?source=post_page-----7932566f773--------------------------------), [Manoj Kumar](https://medium.com/u/1ec26ace3af5?source=post_page-----7932566f773--------------------------------), [Beck Chen](https://medium.com/u/21a22c8fe0db?source=post_page-----7932566f773--------------------------------), [Anatoliy Targhoniy](https://medium.com/u/2a21dd1968e3?source=post_page-----7932566f773--------------------------------), [Sequestro Shedir Pharma](https://sequestroshedirpharma.medium.com/), [Giorgi Nutsubidze](https://medium.com/u/44028eb64892?source=post_page-----7932566f773--------------------------------), [Vivek Gupta](https://medium.com/u/5d2985194f99?source=post_page-----7932566f773--------------------------------), [情封](https://medium.com/u/5dc6d2d4fd54?source=post_page-----7932566f773--------------------------------), [Jasonprogrammer](https://medium.com/u/6731c428f33c?source=post_page-----7932566f773--------------------------------), [Bazla Kausar](https://medium.com/u/848e8d226503?source=post_page-----7932566f773--------------------------------), [Evgenia Maksymiv](https://medium.com/u/90822231ba89?source=post_page-----7932566f773--------------------------------), [Mouhamedyaya](https://medium.com/u/946183af0794?source=post_page-----7932566f773--------------------------------), [mohamed enab](https://medium.com/u/99fc41323224?source=post_page-----7932566f773--------------------------------), [Andrew Trots](https://medium.com/u/dd257f371e28?source=post_page-----7932566f773--------------------------------), [dhrumil shah](https://medium.com/u/f02aa01bf533?source=post_page-----7932566f773--------------------------------), [Aravindan Mahadevan](https://medium.com/u/d448572bcc04?source=post_page-----7932566f773--------------------------------)**