# 记录反应方式

> 原文：<https://itnext.io/logging-the-reactive-way-f8d888c997df?source=collection_archive---------5----------------------->

![](img/e69cfd39cab14ec5297fa8dfa97ddfa1.png)

在 Flutter 应用程序构建的开发者方面，人们可以通过问一组最简单的问题来确定要实现和构建什么。在飞镖和颤振中，无功的哪一部分没有完全实现；你需要建立什么来处理这个事实。

在 UI 视觉方面，它是这个问题的间接答案。也就是说，使用我们在飞镖和扑动中对 reactive 的改进，我们可以构建哪些更强大的 UI 视觉效果和对象。

在本文中，我将向您展示如何实现反应式影响日志记录的 Dart 和 Flutter，并为我们提供线索，说明我们需要为 Flutter 应用程序实现什么样的日志记录基础架构。

# **无功背景**

**无功最初出自微软的一个名叫 LINQ 的研究项目。当然，Q 部分被脸书重新发现为 GraphQL。这就是我们以前在以前的语言中遇到的反应。**

在基于堆栈的 OOP 中，我们有可怕的死亡之钻，我们试图通过使用依赖注入解决方案来避免它。在 Reactive 中，我们遇到了同样的问题，通过堆栈传递更新更改来评估程序。

我对 Flutter 内部的研究表明，UI 依赖，即依赖图，仍然没有与 Dart 和 Flutter 反应式实现中的数据流状态传播完全集成。

这不仅要求有一个日志记录基础设施，还要求我们实现的日志记录基础设施对其数据流使用反应式编程，以便我们除了调试日志记录之外，还可以进行应用程序日志记录。

因此，我将向您展示一种实现日志基础设施的方法，该基础设施使用底层的反应式编程来传输日志事件数据流。

# **我们的依赖关系**

**这些是我们将用来实现反应式日志记录基础设施的依赖项:**

[](https://pub.dev/packages/logging) [## 日志| Dart 包

### 默认情况下，日志包不会对日志消息做任何有用的事情。您必须配置日志记录级别…

公共开发](https://pub.dev/packages/logging) [](https://pub.dev/packages/logging_appenders) [## 日志记录 _appenders | Dart 包

### 用于日志记录附加器的本机 dart 包，可用于日志记录包。它目前包括用于以下目的的附加器:本地…

公共开发](https://pub.dev/packages/logging_appenders) 

并在您的 pubspec 中使用它们:

```
 logging: ^1.0.1logging_appenders: ^1.0.0
```

**由于我们需要进行调试和应用程序日志记录，我们需要某种方法来检测应用程序创建时的构建模式。**

# **构建模式检测**

**基础 Dart 包有构建模式；但是我更喜欢一组更具语义的变量，所以我将构建模式变量细化为:**

这展示了如何在 Dart 中使用一些奇特的操作符，所以让我来解释一下。

让我们来看看 isInDebugMode 函数:

我将 _inDebugMode 布尔值设置为 false。所以我在 return 语句中抓取 KDebugMode 的时候，用设置为 true 来评估 KDebugMode，重新设置为 false 来评估。这不是你陈述的唯一方式，因为这也是一样的:

```
**return kDebugMode ?(_inDebugMode!) : (_inDebugMode)**
```

**我更喜欢另一个，因为它更容易让初学飞镖和颤振的开发者理解。那么为什么不用 if 语句来代替呢？您应该使用它，因为在 Dart 中它比使用 if 语句执行得更快。**

现在是伐木的时候了。

# **测井代码**

**现在回到 Dart 和 Flutter 的反应实现上来。Dart 反应式训练轮是一些原始流和汇实现的形式。这意味着，如果我们找到一个使用流的日志插件，那么我们会得到一个即时的反应数据流连接。这甚至意味着我们可以将这些流路由到控制台和其他日志记录 appender 输出。**

Dart 日志记录包确实使用了开发人员包中的 Dart 日志函数，但同样重要的是，它使用流来构建其日志事件数据流。和日志记录附加器都有构造日志记录输出的方法。

第一部分是使用应用程序标题来设置一个命名的记录器

**这将我们需要的应用标题设置为我们的日志名称:**

**现在我有两个任务要完成，日志包的设置和日志附加器包的设置。因为 appLogger 是我通过日志包访问日志方法时唯一需要的对象引用，所以我需要一个 void 标记的函数，而不是使用类构造函数:**

**我正在关注的设置文档有:**

[](https://pub.dev/packages/logging) [## 日志| Dart 包

### 默认情况下，日志包不会对日志消息做任何有用的事情。您必须配置日志记录级别…

公共开发](https://pub.dev/packages/logging) [](https://pub.dev/packages/logging_appenders) [## 日志记录 _appenders | Dart 包

### 用于日志记录附加器的本机 dart 包，可用于日志记录包。它目前包括用于以下目的的附加器:本地…

公共开发](https://pub.dev/packages/logging_appenders) 

完整的日志源代码是:

现在，发布设置是没有发出日志。要将它设置为在发布模式下发出日志，您需要:

```
**if(isInReleaseMode){****recordStackTraceAtLevel = Level.ALL;****Logger.root.onRecord.listen((record) {****if (record.error != null && record.stackTrace != null) {****log('${record.level.name}: ${record.loggerName}: ${record.time}: ${record.message}: ${record.error}: ${record.stackTrace}');**log(**// ignore: prefer-trailing-comma****'level: ${record.level.name} loggerName: ${record.loggerName} time: ${record.time} message: ${record.message} error: ${record.error} exception: ${record.stackTrace}');****} else if (record.error != null) {****log('level: ${record.level.name} loggerName: ${record.loggerName} time: ${record.time} message: ${record.message} error: ${record.error}');****} else {****log('level: ${record.level.name} loggerName: ${record.loggerName} time: ${record.time} message: ${record.message}');****}****});****MyReleaseLogAppender(formatter: const MyDevLogRecordFormatter())****.attachToLogger(Logger.root);****}**
```

**是啊，我要换个自己的源，:)！**

在 main 中调用它:

# **结论**

**因此，通过在 Dart 和 Flutter 中使用底层流反应式基础设施，我们可以使用一个日志记录基础设施来进行调试和应用程序日志记录。**

# 关于我，弗雷德·格罗特

我是一个改过自新的 Android、Java、Kotlin 和前端开发人员；一个改过自新的多动症患者，设计师和制造者。

前端应用程序开发和设计的有趣之处在于，我们是在给人类编程，而不是给计算机编程。它本身不是编码或设计；但是，编码和设计试图通过其人工制品创造一个故事来抓住人类，并说这样做！

我贡献了几个社区 Flutter 插件，包括:

颤振平台部件

[https://pub.dev/packages/flutter_platform_widgets](https://pub.dev/packages/flutter_platform_widgets)

捕手

【https://pub.dev/packages/catcher 号

我的一些有用的文章和指南:

反作用颤振训练轮

[https://medium . com/p/training-wheels-for-reactive-flutter-d1a e 35 c 47787](https://medium.com/p/training-wheels-for-reactive-flutter-d1ae35c47787)

我的专家项目设置，分层洋葱架构

[https://medium . com/geek culture/my-expert-project-setup-layered-onion-architecture-5dd 06 e 29 ee 9 f](https://medium.com/geekculture/my-expert-project-setup-layered-onion-architecture-5dd06e29ee9f)

深入状态

[https://medium . com/geek culture/deep-dive-into-state-34b 443 da 3573](https://medium.com/geekculture/deep-dive-into-state-34b443da3573)

选择颤振状态管理解决方案[https://medium . com/p/choosing-A-Flutter-State-Management-Solution-cccf 1 B2 AC F10](https://medium.com/p/choosing-a-flutter-state-management-solution-cccf1b2acf10)

我如何学会信任我的多动症

[https://medium . com/p/how-I-learn-to-trust-my-ADHD-DBF 4f 80518 cc](https://medium.com/p/how-i-learned-to-trust-my-adhd-dbf4f80518cc)

颤振完美设置

[https://medium.com/codex/flutter-perfect-setup-c5462b412f78](https://medium.com/codex/flutter-perfect-setup-c5462b412f78)

颤振专家 IDE 设置

[https://medium . com/geek culture/flutter-expert-ide-set-up-25791 ce 690 c](https://medium.com/geekculture/flutter-expert-ide-set-up-25791ce690c)

全局 Dart 文件是反模式

[https://medium . com/p/globals-dart-file-is-an-anti pattern-92975320 e30c](https://medium.com/p/globals-dart-file-is-an-antipattern-92975320e30c)

在 Flutter 中的函数式编程，用沙箱保护你的函数

[https://medium . com/p/functional-programming-in-flutter-sandbox-your-functions-ee 679d 3 db 7d 5](https://medium.com/p/functional-programming-in-flutter-sandbox-your-functions-ee679d3db7d5)

颤振应用的专家捕捉器设置

[https://medium . com/p/expert-catcher-setup-for-flutter-apps-a9ee 3a 6a 9 e 08](https://medium.com/p/expert-catcher-setup-for-flutter-apps-a9ee3a6a9e08)

我正在为图书开发和设计系列构建 Flutter 应用程序代码，网址为:

[https://github.com/fredgrott/ddi_flutter](https://github.com/fredgrott/ddi_flutter)

关注我的一些地方:

【https://keybase.io/fredgrott 号

【https://twitter.com/fredgrott 

[https://github.com/fredgrott](https://github.com/fredgrott)

[https://www.xing.com/profile/Fred_Grott/cv](https://www.xing.com/profile/Fred_Grott/cv)

[https://www . LinkedIn . com/in/fredgrottstartupfluttermobileappdesigner/](https://www.linkedin.com/in/fredgrottstartupfluttermobileappdesigner/)

为生活制造和创造…