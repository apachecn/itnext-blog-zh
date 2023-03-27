# 如何在 AngularDart web 应用程序中使用 Toastr.js 通知

> 原文：<https://itnext.io/how-to-use-toastr-js-notifications-in-an-angulardart-web-application-d2b9b138f9e3?source=collection_archive---------2----------------------->

## 了解如何在 Dart web 应用程序中使用 JavaScript 库

![](img/673d374e4f65ab36541e5da99b67e236.png)

在这个视频中，我们将看看 Toastr 通知 jQuery 插件，并演示如何在 Dart web 应用程序中使用这个 JavaScript 解决方案。作为过渡到 [AngularDart 视频系列](https://youtu.be/9X88k6kTuNw)的一部分，我们将使用 Stagehand 脚手架工具引导一个样本 AngularDart 应用程序，并从那里开始。

以下是我们将在本课中讲述的内容:

1.  使用 VS 代码的 **Dart** 扩展快速设置项目
2.  添加 [**js**](https://pub.dartlang.org/packages/js) 包作为依赖项
3.  了解 AngularDart 项目的结构
4.  导入 Toastr.js 并实现我们的互操作逻辑
5.  在 AngularDart 应用中集成我们的互操作解决方案

→ [**在 YouTube 上观看**](https://creativebracket.com/how-to-use-toastr-notifications-in-an-angulardart-web-application/?preview_id=680&preview_nonce=53c6609c63&preview=true&_thumbnail_id=690)

# 解决方案

下面是我们实现的互操作文件:

```
// lib/src/interop/toastr.dart
**@JS()**
**library toastr_interop;**

import 'package:js/js.dart';
import 'package:js/js_util.dart';

**@JS()**
external **ToastrInterface** get toastr;

class **ToastrInterface** {
  external ToastrNotificationFn get **info**;
  external ToastrNotificationFn get **success**;
  external ToastrNotificationFn get **error**;
  external ToastrNotificationFn get **warning**;
  external Function get **remove**;
  external Function get **clear**;
}

typedef **ToastrNotificationFn** = Function(String message,
    [String title, dynamic options]);

// Converts a Dart Map object to a native JavaScript object
Object **mapToJsObject**(Map<String, dynamic> **dartMap**) {
  var jsObject = newObject();

  **dartMap**.forEach((**name**, **value**) {
    **setProperty**(jsObject, **name**, **value**);
  });

  return **jsObject**;
}
```

→ [**获取源代码**](https://github.com/graphicbeacon/js-dart-interop-samples/tree/master/toastr)
→ [**在 YouTube 上观看**](https://youtu.be/xegpncRwqGw)

我希望这很有见地，你今天学到了一些新的有趣的东西。如果您有任何问题或一般反馈，请在下面的评论中告诉我。*谢谢！*

# 进一步阅读

1.  [AngularDart 教程:让我们构建一个书签管理器](https://youtu.be/9X88k6kTuNw)
2.  [如何在 Dart web 应用程序中使用 JavaScript 库](https://creativebracket.com/how-to-use-javascript-libraries-in-your-dart-applications/)
3.  [Toastr.js 通知库](https://codeseven.github.io/toastr/)

# 分享是关怀🤗

如果你喜欢读这篇文章，请通过各种社交渠道分享。此外，检查并 [**订阅我的 YouTube 频道**](https://youtube.com/c/CreativeBracket) (也点击铃铛图标)上的 Dart 视频。

[**订阅我的电子邮件简讯**](http://eepurl.com/gipQBX) 下载我的免费 35 页【Dart 入门电子书，并在新内容发布时得到通知。

**喜欢，分享和** [**关注我**](https://twitter.com/creativ_bracket) 😍有关 Dart 的更多内容。