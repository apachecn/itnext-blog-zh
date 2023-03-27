# full page . js–Dart 入门

> 原文：<https://itnext.io/fullpage-js-get-started-in-dart-ac383e7bf458?source=collection_archive---------3----------------------->

## 了解如何在 Dart 中使用这个 JS 表示库

![](img/22f12677ec28eee1bb876739a7f5c9eb.png)

在今天的教程中，我们将学习如何在 Dart web 项目中使用 fullPage.js 演示库。我们将使用 Dart 团队的 **js** 包来实现这一点。

# 设置

安装 [**stagehand**](https://pub.dartlang.org/packages/stagehand) 并生成 web 项目:

```
**$** pub global activate stagehand
**$** mkdir my_project && cd my_project
**$** stagehand web-simple
```

修改 **pubspec.yaml** 文件的`dependencies`部分:

```
**dependencies**:
  **js**: ^0.6.0
```

并更新您的依赖关系:

```
**$** pub get
```

以下是完整的视频教程，请按照设置步骤操作:

→ [**在 YouTube 上观看**](https://youtu.be/qwpAEcSl43o)
→ [**获取源代码**](https://github.com/graphicbeacon/js-dart-interop-samples/tree/master/fullpage)

# 结论

我希望这是有见地的，你今天学到了一些新东西。

**订阅** [**我的 YouTube 频道**](https://www.youtube.com/channel/UCHSRZk4k6e-hqIXBBM4b2iA?view_as=subscriber) **用 Dart 学习全栈 web 开发。**

**喜欢，分享一下** [**跟我来**](https://twitter.com/creativ_bracket) 😍有关 Dart 的更多内容。

# 进一步阅读

1.  [js 包](https://pub.dartlang.org/packages/js)
2.  [如何在 Dart 应用中使用 JavaScript 库](https://dev.to/graphicbeacon/how-to-use-javascript-libraries-in-your-dart-applications--4mc6)
3.  [**fullPage.js 库**](https://alvarotrigo.com/fullPage/)