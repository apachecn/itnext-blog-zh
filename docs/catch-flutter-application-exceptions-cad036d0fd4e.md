# 捕捉颤振应用异常

> 原文：<https://itnext.io/catch-flutter-application-exceptions-cad036d0fd4e?source=collection_archive---------4----------------------->

一些捕捉应用程序异常的秘密技巧！

![](img/dd73c2470a4023a19781cbd83e1e69b0.png)

有两种情况你需要得到反馈来帮助改进应用程序代码。第一个用的显然是日志。第二个阶段是发生罕见应用程序异常的时候。

我将向您展示一些秘密的开发人员权力移动，不仅收集应用程序异常，还向您介绍异步代码行为的配对等待。和往常一样，flutter 项目的设置是基于我的 flutter_setup 项目 git repo 的，在 CodeX(【https://medium.com/codex/flutter-perfect-setup-c5462b412f78】)上发表的我的文章 Flutter Perfect SetUp 中有详细介绍，这两个链接都在本文末尾的参考资料部分。

# **背景**

让我介绍一些 API 设计问题。此时，我们正处于设计应用程序 API 的非 UI 部分的开始。我们想要一些纯粹的函数或者稍微脏一点的函数，来帮助我们在创建 UI 代码之前在应用程序代码中设置所有的样板文件。

在应用程序异常处理中，我们需要一个 API 来获取特定的颤振构建模式。颤振建立模式有:
1。调试模式
2。轮廓模式
3。发布模式

flutter SDK 框架在基础 API 包中设置布尔值，表示这些 flutter 构建模式:

[https://API . flutter . dev/flutter/foundation/foundation-library . html](https://api.flutter.dev/flutter/foundation/foundation-library.html)

API 设计的一部分是我们对碰巧使用 API 的人进行编程。在这种情况下，我们希望用最少的认知负荷来理解，因为当我们下一次看这个 API 的代码时，我们不会完全记住它。由此，我可以用两个东西:
1。布尔值
2。变量的可读性。

让我通过一些 flutter_app_exceptions 代码，build_modes dart 包来说明这一点:

请注意，我将我的布尔值分为两种语言形式，一种是 inDebugMode 类型，另一种是 isInDebugMode 类型。此外，请注意，我将内部布尔隐藏为私有布尔，因为我们不想在应用程序代码中错误地访问它们，因为这样我们就不会有一个指示 flutter 构建模式的布尔集，就像我们希望这个特定的包 API 那样。

现在我们有了这个 build_modes 包 API，我们可以使用它来帮助建立一个适当的方法来实现应用程序异常处理，包括向应用程序用户显示一个错误消息。首先，我们需要了解飞镖摆动区。

# **省道区域**

在大多数 Flutter SDK 示例中，您看不到实现区域的全部原因是，所有这些示例都是基本的应用程序，在大多数情况下只能同步运行。当您知道应用程序的某个部分将异步执行代码时，您可以使用区域。如果你的应用程序使用异步代码，提示线索是有任何期货调用或使用。

基本上，它在一个区域中运行应用程序，这样当未捕获的异常发生时，应用程序仍将运行。好吧，但是我们为什么需要这个？Dart 在 Dart 1.9 中引入了异步支持，下面是我们过去必须手动写出每个可能的异常的方法:

[https://dart.dev/guides/libraries/futures-error-handling](https://dart.dev/guides/libraries/futures-error-handling)

在那种情况下，我们在将来分离同步和异步代码时总是有很多问题，并且求助于大量的包装代码和额外的异常编写。
新的异步支持很好，因为我们要写的异常代码少了很多。

要实现一个区域，我们创建:

```
runZonedGuarded<Future<void>>(() async {
});
```

现在，让我们走进 Dart 中的基本异步编程。

# **飞镖异步编程**

Future 是类，future 是实例，表示异步操作。await 和 async 关键字允许我们声明异步函数。或者，换句话说，我们没有一个仅使用 Future 的异步完整函数，因为我们必须至少将它与 async 配对，并且还经常等待。

通过使用 wait，我们强制函数等待完成，然后将未完成的 future 实例转换为最初返回的具有结果或错误的已完成 future。在我们的主代码块中，由于我们不返回任何东西，即 Future <void>，我们只需使用 await 关键字来获取异常和堆栈跟踪输出。</void>

现在，我可以解释用于报告应用程序异常的两个小部件。

#向应用用户报告应用异常

当遇到应用程序异常时，你的痛苦的应用程序用户应该知道你足够关心及时修复应用程序异常。您可以通过实现一个向用户显示异常消息的错误(异常)小部件来给人这种印象:

在主街区，我们有:

```
ErrorWidget.builder = (FlutterErrorDetails details) {
      if (isInDebugMode){
          return ErrorWidget(details.exception);
      }

      return Container(
      alignment: Alignment.center,
      child: const Text(
        'Error!',
        style: TextStyle(color: Colors.yellow),
        textDirection: TextDirection.ltr,
      ),
    );};
```

我们要覆盖的 ErrorWidget 类是这样的:

[https://API . flutter . dev/flutter/widgets/error widget-class . html](https://api.flutter.dev/flutter/widgets/ErrorWidget-class.html)

细节。异常消息内容来自此 FlutterError 类:

[https://API . flutter . dev/flutter/foundation/flutter error-class . html](https://api.flutter.dev/flutter/foundation/FlutterError-class.html)

基本上，FlutterError 值就是抛出的 Flutter 异常。因此，显示给用户的消息将是以下格式

```
Flutter Exception   Error!
```

是啊，你抓到我了！我很懒，没有在这里使用 dart 三元运算符。因此，这可能是您的家庭作业，将块转换为而不是 if-conditionals，而是使用 dart 三元运算符。

由于我们只向控制台报告，因此我们不会覆盖 ErrorWidget，而是在调试模式下重复使用底层 FlutterError 函数来输出到控制台，并将详细信息返回到区域，以便通过应用异常跟踪提供程序进行应用异常处理。

和 our _reportError 函数将异常的详细信息传递给 describe 旁边的一个 app 异常跟踪服务。

# **_reportError 函数**

private _reportError 函数是这样实现的:

```
Future<void> _reportError(dynamic error, dynamic stackTrace) async {
  log('Caught error: $error');
  // Errors thrown in development mode are unlikely to be interesting. You
  // check if you are running in dev mode using an assertion and omit send
  // the report.
  if (isInDebugMode) {
    log('$stackTrace');
    log('In dev mode. Not sending report to an app exceptions provider.');return;
  } else {
    // reporting error and stacktrace to app exceptions provider code goes here
    if (isInReleaseMode) {
        // code goes here
    }
  }
}
```

然后，为了使其功能完整，您将实现一个函数来调用您正在使用的应用程序异常跟踪器提供程序的客户端 API，然后在 isinreasemode if-conditional 块中调用该函数。

# **结论**

通过展示应用程序样板代码中捕捉应用程序异常的基本部分，您可以了解 API 设计和异步编程的基本知识。当然，对于作业，你有把我的多个 if-conditionals 改成 dart 三元运算符的任务。但是，当你开始掌握 dart 时，这是一个很好的小任务。

接下来是混合一些高级日志。

# **资源**

文章的具体资源:

颤振 _ 设置 git[repo@gitlab.com](mailto:repo@gitlab.com)https://gitlab.com/fred.grott/flutter_setup

颤振完美设置文章@CodeX 媒体出版[https://medium.com/codex/flutter-perfect-setup-c5462b412f78](https://medium.com/codex/flutter-perfect-setup-c5462b412f78)

flutter _ app _ exceptions git reop @ git lab[https://gitlab.com/fred.grott/flutter_app_exceptions](https://gitlab.com/fred.grott/flutter_app_exceptions)

flutter error class @ flutter . dev[https://API . flutter . dev/flutter/foundation/flutter error-class . html](https://api.flutter.dev/flutter/foundation/FlutterError-class.html)

error widget class @ flutter . dev[https://API . flutter . dev/flutter/widgets/error widget-class . html](https://api.flutter.dev/flutter/widgets/ErrorWidget-class.html)

异步编程@ codelabs . dart . dev[https://dart.dev/codelabs/async-await](https://dart.dev/codelabs/async-await)

https://dart.dev/articles/archive/zones

一般颤振和飞镖资源；

扑社区资源[https://flutter.dev/community](https://flutter.dev/community)

https://flutter.dev/docs/get-started/install[SDK](https://flutter.dev/docs/get-started/install)

Android Studio IDE【https://developer.android.com/studio 

MS 的 Visual Studio 代码[https://code.visualstudio.com/](https://code.visualstudio.com/)

扑文件[https://flutter.dev/docs](https://flutter.dev/docs)

dart Docs[https://dart.dev/guides](https://dart.dev/guides)

谷歌 Firebase 移动设备测试实验室[https://firebase.google.com/docs/test-lab](https://firebase.google.com/docs/test-lab)

**商标公告**

Google LLC 拥有以下商标:飞镖，颤振，机器人，机器人，诺托。苹果公司拥有 iOS、MacOSX、Swift 和 Objective-C 的商标。苹果公司拥有 SF Pro、Sf Compact、SF mono 和 New York 字体的商标。JetBeans Inc .拥有 JetBeans、IntelliJ 和 Kotlin 的商标。甲骨文公司拥有 Java 商标。微软公司拥有微软视窗操作系统和 Powershell 的商标。Gradle 是 Gradle Inc .的商标。Git 项目拥有 Git 的商标。Linux 基金会拥有 Linux 的商标。智能手机 OEM 自有商标到其手机产品名称。尽我所能，我遵守上述商标的品牌和使用指南。

# **关于弗雷德·格罗特**

我是一个疯狂的 SOB，作为一名前 android 移动开发者，我开始写关于 flutter 移动应用程序开发、设计和生活的文章(见 Eff COVID 和 GOP[https://Fred grott . medium . com/Eff-COVID-and-the-GOP-e 912 db 0548 b 8](https://fredgrott.medium.com/eff-covid-and-the-gop-e912db0548b8))。我会达到关键的每月 100 万观众大关吗？坐下来看着它发生。在这些社交平台上找到我:

[](https://www.xing.com/profile/Fred_Grott/cv) [## 弗雷德·格罗特-安卓软件工程师-弗雷德·格罗特

### 信息、投资组合和价值:或者弗雷德·格罗特先生。

www.xing.com](https://www.xing.com/profile/Fred_Grott/cv)  [## 弗雷德·格罗特-首席执行官-首席采购官-设计师-格罗特| LinkedIn

### 查看弗雷德·格罗特在全球最大的职业社区 LinkedIn 上的个人资料。弗雷德有 3 份工作列在他们的…

www.linkedin.com](https://www.linkedin.com/in/fredgrottstartupfluttermobileappdesigner/) 

[https://keybase.io/fredgrott](https://keybase.io/fredgrott)

[https://twitter.com/fredgrott](https://twitter.com/fredgrott)