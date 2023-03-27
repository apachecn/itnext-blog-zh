# 2023 年的 Swift 语言项目

> 原文：<https://itnext.io/swift-language-project-in-2023-b964515f4764?source=collection_archive---------0----------------------->

Swift 集团在 2023 年的工作重点是什么，我们可以期待新版本的哪些语言功能。

![](img/f0a805f2e038f0b9c798271857fd7ff0.png)

【https://verticalcoding.com】从现在开始，文章也可以在我的个人网站上找到，以后会有越来越多策划的内容:[](https://verticalcoding.com/swift-project-in-2023/)**强烈推荐你关注我的网站:)**

# *新团体正在出现*

*核心团队(Swift 语言开发人员的主要团队)开始进一步[重组，成立了几个工作组:](https://www.swift.org/blog/language-workgroup/)*

*   *[语言工作组](https://www.swift.org/language-workgroup/)*
*   *[网站工作组](https://www.swift.org/website/)*
*   *[文档工作组](https://www.swift.org/documentation-workgroup/)*
*   *[C++互操作工作组](https://www.swift.org/cxx-interop-workgroup/)*

*因此，所有对将 Swift 发展成一种语言感兴趣的人都可以加入七个工作组中的一个，这七个工作组由上一段中指定的工作组和两个最老的工作组组成:*

*   *[服务器上的 Swift](https://www.swift.org/server/)*
*   *[多样性](https://www.swift.org/diversity/)*

# *新的核心团队领导*

*新的核心团队领导宣布了。*

# *2023 年焦点快速演进*

# *并发*

*Swift 团队将致力于以`Sendable`协议、`actor`、`async/await`的形式完成 Swift 5.5 对并发性的语言支持。他们将关注:*

*   *固定螺纹安全孔*
*   *修复跨角色调用错误*
*   *允许`non-Sendable`值在隔离层之间移动的可能性*

*[链接到 Swift 论坛的主题](https://forums.swift.org/t/swift-concurrency-roadmap/41611)*

# *无商标消费品*

*下一个主要焦点将是**仿制药**的发展，这个过程将会持续几年。它将包括为编译器和运行时添加基本实现，以支持新的泛型特性。例如，允许元组有条件地符合协议。*

*[链接到 Swift 论坛的主题](https://forums.swift.org/t/manifesto-completing-generics/1656)*

# *所有权——对那些乡巴佬来说*

*对于那些不熟悉 Rust 或所有权话题的人来说，这里有一个在 [Rust](https://www.youtube.com/watch?v=VFIOSWy93H0) 中关于它的视频。但是为什么要在 Swift Evolution 中谈论 Rust 编程语言呢？虽然我真的相信 Rust 的流行也通过 Swift 语言的想法创造了新的思维模式。因此，今年 Swift 团队将专注于:*

*   *添加通过`borrow`和`take`转移所有权的选项*
*   *添加对不可复制类型的基本支持。这将使您能够限制关键值的生命周期。看起来像是之前提到的 Rust 中的一个同名特性。*

*[关于主题](https://forums.swift.org/t/se-0377-borrow-and-take-parameter-ownership-modifiers/61020)的 Swift Evolution 链接*

*[链接到 Swift 论坛的主题](https://forums.swift.org/t/se-0377-borrow-and-take-parameter-ownership-modifiers/61020)*

# *宏指令*

*对于那些不熟悉其他编程语言的宏的人。宏在很高的层次上，在编译时获取程序的部分源代码，并将其翻译成其他源代码，然后编译到程序中。如果你想了解更多关于宏及其在 Swift 中的用法，我强烈推荐这个 Github [gist](https://gist.github.com/DougGregor/4f3ba5f4eadac474ae62eae836328b71) 。2023 年，Swift 语言团队将关注这一主题，我非常期待看到他们将会创造出什么。*

*[链接到 Swift 论坛的主题](https://forums.swift.org/t/a-possible-vision-for-macros-in-swift/60900)*

# *C++*

*C++仍然是最常用的系统语言之一，因此难怪 Swift 团队希望将 c++集成到您的 Swift 代码中变得更加简单。今年 Swift 团队和 C++语言的主要关注点是:创建更多关于 C++ API 集成到 Swift 的文档，反之亦然，并稳定当前在 Swift 中使用 C++和在 c++中使用 Swift 的原型互操作性。*

# *包注册表*

*Swift 团队将专注于创建将 SPM 从源代码控制生态系统过渡到基于注册的生态系统(如 Cargo、NPM 或 Maven)所需的技术组件。*

# *工作组工作*

*每个工作组都有自己明年的目标，但我不会写下来，因为这些目标非常广泛，如果你对它们感兴趣，请访问源文章:[https://www.swift.org/blog/focus-areas-2023/](https://www.swift.org/blog/focus-areas-2023/)或[[# New groups happing]]中提到的工作组链接之一。*

*来源:*

*   *https://www.swift.org/blog/focus-areas-2023/*
*   *【https://apple.github.io/swift-evolution/ *
*   *[https://forums.swift.org/](https://forums.swift.org/)*