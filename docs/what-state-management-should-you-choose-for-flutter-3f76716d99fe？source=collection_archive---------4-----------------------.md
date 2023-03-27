# 颤振应该选择什么状态管理？

> 原文：<https://itnext.io/what-state-management-should-you-choose-for-flutter-3f76716d99fe?source=collection_archive---------4----------------------->

![](img/f5b3d6451ad6cd3daaaa462e3344d911.png)

[斯科特·格雷厄姆](https://unsplash.com/@sctgrhm?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍照

随着 flutter 的发展，状态管理解决方案的数量也在快速增长。当我第一次开始探索 Flutter 时，只有几个选项，现在有 Redux、ScopedModel、Provider、BLoC、RxDart、States Rebuilder、Get，以及许多我无法立即列出的选项。

虽然我没有尝试上面列出的所有方法，但是我使用了 BLoC、Provider、States Rebuilder，并且至少研究了其他方法。如果你想要 TLDR，他们都很棒，你不会错的。所有这些都很有可能解决你想要解决的问题。然而，他们每个人都有自己的差异，使他们独一无二。这是我迄今为止进入状态管理的旅程，以及我如何缩小适合我的解决方案。

重要的事情先来。如果你正在从 angular 或 react 转移，并习惯于 Redux 或 RxJS，那么你可能应该坚持你所知道的(除非你好奇并渴望探索其他选项)。第二，如果你试图构建一个简单的没有很多接口的应用程序，就不需要任何疯狂的状态管理解决方案。SetState 就够了！即使是更大的项目，如果你勤勤恳恳，setState 也足够强大。

但我的国家管理之旅是从 BLoC 开始的。至少可以说，在了解什么是国家管理之前就加入集团是一个挑战。但是在 ResoCoder 的大量教程和惊人的 BLoC 文档之后，我能够把它们拼凑起来。然而，bloc 有两件事困扰着我。

1.  状态如何总是被流式传输。从技术角度来看，我不认为这有什么不对，但我不明白为什么这是必要的，尤其是对那些不常改变的州来说。
2.  为了让最简单的状态也能工作，需要添加多少样板文件。

由于这两个原因，下一个合乎逻辑的步骤是提供者。一开始用 Provider，就爱上了。你感觉对你的代码和正在发生的事情有了更多的控制。这看起来比 BLoC 更简单，而且你不需要不断地到处传输数据。对于提供商来说，唯一的大败笔是……文档远不如 BLoC 那样丰富和有条理，有时很难完全理解。很多学习必须来自谷歌搜索和观看其他人的视频。如果 Provider 有 BLoC 级文档，那将使它成为一个比现在更大的包。让我对 Provider 感到恼火的最后一个吹毛求疵的问题是，实际上不能有两个相同类型的 Provider 实例。有一个相对简单的解决方法，即创建多个保存该类型的类，但这不是最干净的解决方案。

从那以后，我开始听说一个叫做 states_rebuilder 的包。现在前两个是“Flutter Favorites”包(一些包获得了 Flutter 团队批准的徽章),所以我有点担心 states_rebuilder，因为它不是。我也知道 flutter 社区在 Twitter 上很强大，我关注过 BLoC 和 Provider 的作者，他们都很活跃。states_rebuilder 作者不太活跃。但后来，我开始真正进入包裹，我所有的怀疑都飞出了窗外。它做 provider 所做的一切，除了状态是纯 dart 类。不需要扩展 ChangeNotifier 和 notifyListeners()或任何东西。当您想要使用它时，您只需将它包装在一个 ReactiveModel 中，您就拥有了所需的一切。除此之外，它还命名了 injections:)。我对供应商吹毛求疵的小问题解决了。就其他方面而言，它与 Provider 非常相似。您注入依赖项并使用定制的小部件构建 UI。我还需要对 states_rebuilder 做更多的挖掘，但是到目前为止，它看起来是我最喜欢的选项之一。

我的下一步是“获取”包。这个引起我注意的原因是，除了作为一个状态管理解决方案，它还有一个导航解决方案。我一直认为在 flutter 中导航比实际需要的要困难一些。我将在接下来的几周内研究这个包，所以如果你想了解更多，请跟我来！

感谢您的阅读！

youtube:业余程序员

instagram: @amateurcoder

推特:@tadaspetra