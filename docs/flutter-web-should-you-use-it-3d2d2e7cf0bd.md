# Flutter Web:应该使用它吗？

> 原文：<https://itnext.io/flutter-web-should-you-use-it-3d2d2e7cf0bd?source=collection_archive---------8----------------------->

所以，我是温哥华舞团的组织者。你可以查看我用 Flutter web 创建的网站，作为其维护的一部分。

![](img/6088eaeebbd3b8c9637771520bca9453.png)

关于代码，最初的想法是公开代码，但由于我想完成的几个小的重构问题，我推迟了它。但是最近我们的空闲时间达到了 100 多人，我对此感到非常兴奋，并决定在 BSD-2 许可下公开[的代码。你可以查看我在 github 上分享的](https://fluttervancouver.ca)[代码](https://github.com/MarsGoatz/birdforce)。此外，如果您有任何反馈，请随时联系我们。

虽然我在我们的网站上使用过 Flutter Web，但我并不推荐在这种情况下使用它。我宁愿使用 Angular、Vue 或 React，因为它们有更好的体验、速度和性能。在目前的状态下，我会用 Flutter Web 来将你的移动应用扩展到 Web。

例如，使用 Angular，Vue，React 或纯 HTML/CSS/JS 和 Bootstrap 等用于登录页面，然后您可以将您的登录按钮链接到您的 Flutter 应用程序。

请注意，这并不像你想象的那么简单。将你的移动应用扩展到网络需要努力，不是手工，而是设计。为什么？因为有两个主要原因，第一，网络上有更多的空间，第二，你用于移动应用的所有依赖可能在网络上不可用。

房地产问题是所有 web 框架的通病。解决房地产问题的方法当然是利用反应能力。我不会在这里深入探讨，但是你可以看看我写的这篇深入的响应教程。你可以下载代码并在任何尺寸的屏幕上运行，你可以检查它的响应。另外，[https://flutter Vancouver . ca](https://fluttervancouver.ca)的代码也是完全响应的，但是只使用了 [MediaQuery](https://api.flutter.dev/flutter/widgets/MediaQuery-class.html) 进行响应。

解决第二个问题，即依赖性的方法是通过抽象。举个例子，假设你想在你的应用中使用谷歌地图，你将不得不对[移动应用](https://pub.dev/packages/google_maps_flutter)和[网络](https://pub.dev/packages/google_maps_flutter_web)使用不同的依赖关系。这里有一个非常好的[堆栈溢出答案](https://stackoverflow.com/questions/61108935/handling-mobile-only-plugins-on-flutter-web-platform-gracefully)，指导你解决这个问题。请去投票支持这个答案。

在文章结束之前，关于导航需要注意一点。如果你使用 Nav 1.0，那么你还需要分别为网络和手机开发解决方案，但如果你使用 Nav 2.0，它有所有必要的工具来确保你的 Nav 解决方案既适用于手机也适用于网络。与 nav 1.0 相比，Nav 2.0 的学习曲线有点陡峭。

如果您对 Flutter web 有更多问题，请随时联系我们。干杯！

请在推特上关注我们:[https://twitter.com/fluttervancity](https://twitter.com/fluttervancity)

加入我们的行列:

[](https://join.slack.com/t/fluttervancouver/shared_invite/zt-dtlz4grr-mqfJO5cuH5Xq5PjHaqa4fQ) [## 加入 Flutter 温哥华的 Slack

### Slack 是一种与团队沟通的新方式。它比电子邮件更快、更有条理、更安全。

join.slack.com](https://join.slack.com/t/fluttervancouver/shared_invite/zt-dtlz4grr-mqfJO5cuH5Xq5PjHaqa4fQ)