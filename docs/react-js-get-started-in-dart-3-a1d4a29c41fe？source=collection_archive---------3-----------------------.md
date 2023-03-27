# react . js–快速入门

> 原文：<https://itnext.io/react-js-get-started-in-dart-3-a1d4a29c41fe?source=collection_archive---------3----------------------->

## 了解如何以“ *Darty* 风格使用视图库

![](img/68800be81ad817c0b65926668c1a0cb1.png)

在 [**第 2 部分**](/react-js-get-started-in-dart-2-74f956316b3c) 中，我们继续查看 React 文档示例，同时在 Dart 中实现它们。我们从重构`createReactClass`函数开始，使用**命名参数**，试图简化组件的编写:

```
// web/components/ticker.dart
...
...
var **Ticker** = createReactClass(
  **getInitialState**: () => makeJsObject({
    "**seconds**": 0,
  }),
  **componentDidMount**: (ReactClassInterface self) {
    **self.interval** = Timer.periodic(Duration(seconds: 1), (_) => **self.tick()**);
  },
  **componentWillUnmount**: (ReactClassInterface self) {
    **self.interval.cancel()**;
  },
  **render**: (ReactClassInterface self) => React.createElement(
    'div',
    null,
    ['Seconds ${**getProperty**(**self.state**, "**seconds**")}'],
  ),
  **methodMap**: {
    "tick": (ReactClassInterface self) {
      **self.setState**((dynamic state) {
        var seconds = getProperty(state, "seconds") as int;
        return makeJsObject({
          "**seconds**": **seconds + 1**,
        });
    });
  }
});
```

[**查看来源**](https://github.com/graphicbeacon/reactjs-get-started-in-dart/blob/part-2/web/components/ticker.dart#L6-L30)

它的用法是:

```
// web/main.dart**ReactDOM.render**(
  React.createElement(
    **Ticker**,
    null,
    null,
  ),
  **querySelector('#output2')**);
```

→ [**在 YouTube 上看 Part 3**](https://youtu.be/kmCJ-qkBjrc)
→[**获取源代码**](https://github.com/graphicbeacon/reactjs-get-started-in-dart)

在这最后一部分，我们将使用 [**react**](https://pub.dartlang.org/packages/react) 包来构建其他示例。react 包为构建定制组件提供了更加友好的 API:

```
import 'dart:async';

import 'package:react/react.dart';

class **TickerComponent** extends **Component** {
  Timer interval;

  tick() { ... }

  @override
  Map **getInitialState**() => {'seconds': 0};

  @override
  **componentDidMount**() { ... }

  @override
  **componentWillUnmount**() { ... }

  @override
  **render**() => div({}, 'Seconds ${state["seconds"]}');
}

var **Ticker** = **registerComponent**(() => **TickerComponent**());
```

点击完整视频了解更多信息:

→ [**在 YouTube 上观看**](https://youtu.be/kmCJ-qkBjrc)
→ [**获取源代码**](https://github.com/graphicbeacon/reactjs-get-started-in-dart)

# 结论

我希望这是有见地的，你今天学到了一些新东西。

**订阅** [**我的 YouTube 频道**](https://www.youtube.com/channel/UCHSRZk4k6e-hqIXBBM4b2iA?view_as=subscriber) 未来 React 视频通知。谢谢！

**喜欢，分享** [**关注我**](https://twitter.com/creativ_bracket) 😍有关 Dart 的更多内容。

# 进一步阅读

1.  [反应**包**包](https://pub.dartlang.org/packages/react)
2.  [如何在 Dart 应用中使用 JavaScript 库](https://dev.to/graphicbeacon/how-to-use-javascript-libraries-in-your-dart-applications--4mc6)
3.  [**采用 Dart 的全栈 web 开发**](https://bit.ly/fullstackdart)