# firestore Flutter 后端。浏览器内代码实验室。第一部分。

> 原文：<https://itnext.io/firestore-backend-for-flutter-in-browser-code-lab-part-1-1b94d7839b94?source=collection_archive---------0----------------------->

# 动机

在**谷歌云平台**和 **Firebase** 动物园上，Firestore 是一个相对较新的野兽。这是一个*云数据库(更准确地说是*面向文档的数据库*)，针对*与它的数据一起工作。简而言之，如果有人在 Firestore 中更改了一个值，所有连接的设备和 Web 应用都会立即收到关于这一更改的通知。**

**对于经验丰富的开发人员来说，这意味着他们不需要编写传统的连接到数据库的 REST 端点和一些**消息系统**如 RabbitMQ 或 Kafka 来通知客户端数据是否已经更改。**

**Firestore 为各种 IT 创业公司提供了一个非常有利可图的开端。所以你，作为一个创业者，可以从零成本开始，测试用户的牵引力。有了用户牵引，你可以带来投资者的钱，并随着你的创业成长而付费。**

**Flutter 对 Firestore 的支持是一流的。**

**通常，这样的文章应该在这一点上做一些艰难的选择，在中级/高级颤振开发人员和“新手”之间进行选择，考虑在本地机器上预先安装颤振开发 IDE 或工具链的重要性。**

**希望我们生活在第三个千年，能够使用强大的在线编码工具。**

**我们将使用 [**FlutLab**](https://flutlab.io) 进行这个动手代码实验。你只需要一个浏览器和一些良好的互联网连接。**

# **我们的计划**

*   **我们将在您的浏览器中完成所有操作。**
*   **我们将创建一个 **Firebase** 项目(它是免费的)。**
*   **我们将配置简单的 **Firestore** DB(免费)。**
*   **我们将在 **FlutLab.io** 中玩经典的 Count-clicker Flutter 应用程序(也是完全免费的)。**
*   **我们将为您的 ***云*** Android 模拟器使用**cemete . io**。是的，我们将开始使用 Flutter/Android 应用程序。**

**对于给定的文章，我们的目标是稍微修改一下[流行的点击计数器颤动项目](https://dartpad.dev/b6409e10de32b280b8938aa75364fa7b)，并以*的方式将其连接到 ***Firestore 集合*** 。***

**Click-Counter 是一个简单的 Flutter 项目，带有一个有状态的小部件:**

**![](img/6fbb6d38f86ba7072e2f71b79ed44e91.png)**

**当您点击*加*按钮时，您将在*点击次数*字段中看到增加的值。我们的任务是将这个应用程序转换成一个“ ***全球点击器*** ”。以便全球所有人都可以共享这个点击计数器。这款应用的商业价值很低，但它有一个非常清晰的技术理念，与共享后端数据相关。**

**我们的验收标准将是以下工作测试。我们将在 Firebase 控制台中修改 Firestore 中的 clicks-number 的值，然后，我们将在云 Android 模拟器中的 Android 应用程序上看到这些变化。**

# **我们开始吧。创建 Firebase 项目。**

**去你的 [Firebase 控制台](https://console.firebase.google.com/)。如果这听起来不清楚，谷歌搜索“firebase 控制台”并点击这个结果:**

**![](img/3cd5cdfc2774b52a233163d387ab0882.png)**

**然后点击**p*lus*按钮创建一个新的 FireBase 项目:****

**![](img/60747f238ac45966cb3f7a87fa095b56.png)**

**然后经历 3 个初始化步骤，使用相当多的默认值。输入“Global Clicker”作为这个新项目的名称:**

**![](img/374f53cb934b32b240a50b2e1f36ff86.png)****![](img/7250c520a9a324bdabd51d223f192f45.png)****![](img/6d0cb0b5962d74d54e4fc3fe0dcb73f2.png)**

**最后，Firebase 将为您创建一个新项目:**

**![](img/485602c82161cb03c2b4d1de6f3e424f.png)**

# **将 Firebase 项目连接到您的 Android 应用程序。**

**在本文中，我们将尝试 Android 应用程序。那么我们就在 Firebase 端做相应的配置吧。点击“将 Firebase 添加到您的应用程序开始使用”下的 Android 图标:**

**![](img/1ac1faaba64c01eea17f06d0812e97ee.png)**

**下一步，将“com.corp.globalclicker”放入“Android package name”字段。此外，把“全球点击者”应用程序昵称的领域。点击“注册应用程序”按钮:**

**![](img/143316704f2ca0effbaed0c823597f1b.png)**

**您将被引导到下一步，在这里您必须下载名为***Google-services . JSON***的重要配置文件:**

**![](img/6fb1ce75e791f0ce024bdba31a17aea0.png)**

**下载这个文件。**

**目前，我们将切换到其他网页浏览器标签，并将创建我们的颤振项目。然后，我们将返回并完成这个 Firebase 配置。**

# **创建你的 Flutter 项目和工作环境。**

**传统上，Flutter 开发人员使用本地开发环境，例如，由 Intellij IDEA、Flutter SDK、Android SDK 和 Emulator 组成。本文是一个在线代码实验室。除了浏览器，你什么都不会有。我们将创建由纯在线工具组成的“云开发环境”。其中之一， [FlutLab.io](https://flutlab.io) 将是我们的 Flutter 代码编辑器和 Android 应用构建器。**

**去 [https://flutlab.io](http://flutlab.io) 注册你的免费用户。您将看到一些默认 Hello-world 项目。**

**![](img/559ee581ebd954faa2bc304e1d4ded54.png)**

**我们需要从点击计数器项目开始。单击当前项目名称左侧的加号图标:**

**![](img/f6d4e908ad7037e9f26082a78c67ad80.png)**

**在下一个对话框中，选择“代码库:有状态点击计数器”:**

**![](img/9c8a99344b00f92947d8602b42393392.png)**

**将这个项目命名为“Global Clicker”，并将“com.corp.globalclicker”作为它的包:**

**![](img/954ce76b842e21e08d02869872b185cf.png)**

**因此，您将在在线代码编辑器中看到这个有状态小部件的经典示例。**

**![](img/fe63b3c8e192e733b5acdd76d196bf91.png)**

**现在，让我们配置适当的颤振建设目标。单击左侧菜单中的齿轮图标:**

**![](img/3dd62181f2fc60163be24e641c323d89.png)**

**在出现的窗口中，选择 android-x86 作为目标。这种构建将生成 APK 文件(Android 可执行文件)，与 Android 模拟器兼容。**

**![](img/f7b3bd8c676bd05eccc1f210fbd92d3f.png)**

**现在是我们第一次建造的时候了！单击顶部菜单中的构建图标:**

**![](img/1a5edc933aadb67b40398adeafe32cb8.png)**

**FlutLab 正在为我们施展构建安卓 APK 的魔法:**

**![](img/956cc2eb9d46d76345440681d290e065.png)**

**大约一分钟后，构建成功完成:**

**![](img/4da123f96eb424bc5fe9061ecb93ad78.png)**

**点击底部框架中的“获取我的 apk”链接，将 APK 文件下载到您的本地文件系统中。**

**现在我们需要检查“云 IDE”的最后一部分。我们需要一个 Android 模拟器来测试我们的应用程序。在下一个浏览器标签中打开[https://ceite . io](http://appetize.io)。对于在线 Android 和 IOS 模拟器和设备来说，这是一个非常方便的服务。模拟器可以免费使用:**

**![](img/a86944e5c8ccf8e06f981cab9c11b3b5.png)**

**点击主菜单中的“上传”:**

**![](img/844e99185ca24ab35a769e9f45cf4f4f.png)**

**点击“选择文件”按钮，选择最近下载的 APK。开胃菜. io 开始上传:**

**![](img/962847d01fc1395b22f0513be83e3861.png)**

**上传成功。**

**![](img/adb1940c321998518b6986480c26d687.png)**

**在页面的步骤 2 部分输入您的电子邮件，然后单击“生成”:**

**![](img/38545284344e534ad67095ecb82be008.png)**

**稍后，您将收到一封新的电子邮件，其中包含指向您在云模拟器上的应用的链接:**

**![](img/1c574fffe3efa4d0882d53e5fb98fa91.png)**

**点击此链接并点击“点击播放”:**

**![](img/bafe83775b5b8406f2447980c431232e.png)**

**奇迹发生了！你可以看到和玩你的应用程序！按几次“加号”按钮，检查我们的应用程序是否按预期运行:**

**![](img/df3eb4cd666dec532a2d8ef354885a29.png)**

**现在我们已经让我们的 ***云飘起 IDE*** 准备工作了！**

# **将 Android 连接到 Firebase！**

**让我们将我们的 Flutter 项目连接到 Firebase。**

**让我们回到完成 Firebase 设置的阶段:**

**![](img/6fb1ce75e791f0ce024bdba31a17aea0.png)**

**下载 ***google-services.json*****

**![](img/799ddd38fa94aca266ca98623253f35a.png)**

**它必须出现在应用程序的 build.gradle 文件附近:**

**![](img/25313d2722b4d73c374ebde38db69154.png)**

**转到 Firebase 控制台中名为“添加 Firebase SDK”的下一步:**

**![](img/310d9061ddd21e4ebe1efc79622f485a.png)**

**让我们遵循建议，将 Firebase 工件添加到我们的 Gradle 系统中。这个系统由两个同名文件“build.gradle”组成。首先，添加类路径行**

```
**classpath 'com.google.gms:google-services:4.3.3'**
```

**在 build.gradle 文件中，位于“android”级文件夹中:**

**![](img/b16cb692b5b7eeb177b782734956da8f.png)**

**接下来，添加插件行**

```
**apply plugin: 'com.google.gms.google-services'**
```

**到第二个 build.gradle，位于“Android”one 中的“app”文件夹级别:**

**![](img/281f2b9cf680d520c70cbec4360db0a7.png)**

**确保在最后一个版本的依赖部分有这两行。**

```
**implementation 'com.android.support:multidex:1.0.3'implementation 'com.google.firebase:firebase-analytics:17.2.2'**
```

**![](img/78ec6b7fa631154059007d2f63ae9354.png)**

**此外，在同一个 build.gradle 中找到“defaultConfig”部分，并添加以下行:**

```
**multiDexEnabled true**
```

**![](img/6988d895752b8db2952ae7df3cbf190a.png)**

**允许在 Android 清单中使用互联网:**

```
**<uses-permission android:name="android.permission.INTERNET" />**
```

**![](img/d672e367f3b63c4b012156a6022a615e.png)**

**让我们仔细检查一下，我们的 Android 应用程序是否可以在更改后构建和启动。让我们构建 APK 并上传到我们的云模拟器:**

**![](img/b3913a308dc8734a748491d31a96a96e.png)**

**到目前为止一切顺利。**

# **创建 Firestore 系列**

**是时候创建和填充我们的数据库了！**

**正如承诺的那样，我们将使用 Firestore 来实现这一目的。**

**回到你的 Firebase 控制台，点击左边菜单中的“数据库”。您将会看到这个漂亮的橙色“云 Firestore”横幅:**

**![](img/3e9d0d0bf2d74ce8bd98a2c6413318bb.png)**

**单击“创建数据库”按钮。在下一个屏幕上，选择“以文本模式启动”。这将允许我们暂时跳过安全设置。但是我们稍后会回到这一节。**

**![](img/503acf7ec76c02138c480dfe90922f2d.png)**

**现在，选择 Firestore 服务器的位置。我们可以在这里保留默认设置:**

**![](img/c171f9040b48eecae5e0fc0fe4ff3902.png)**

**现在，Firestore 正在施展魔法，为您的新数据库提供资源:**

**![](img/0407582e07d59119e732dab2f918dda9.png)**

**过一会儿，就好了！**

**现在让我们配置数据库。Firestore 由按集合分组的文档组成。所以首先我们需要创建一个新的集合。点击“开始收集”:**

**![](img/c2993a6de1b697b091417d8046191350.png)**

**将这一新系列命名为“点击”:**

**![](img/d09d70a22134a30cf0572c2d407e3ceb.png)**

**现在我们必须配置这个集合中的文档:**

**![](img/016d64f6aa5bdf98030e99b163602f81.png)**

**将“plusButton”作为新文档的 Id:**

**![](img/9c050dc2865daab7d691f857b4be8d90.png)**

**添加数值字段“counter ”,并将 0 作为其值:**

**![](img/a20f0601afa116e7f39ed2bcdb519b0f.png)**

**你做到了！现在，您有了一个新的集合，其中有一个新的文档:**

**![](img/89b8cf04b7129141e02bbd9dc96613bd.png)**

**您可以添加和修改收藏中的文档。尝试在我们的文档中输入 100 而不是 0:**

**![](img/26838f5208566b05da8ce7facd8cb321.png)**

**并点击“更新”:**

**![](img/14d800c5e01ad5dad68d7d0ac11a5831.png)**

# **将 Flutter 连接到 Firestore**

**现在是最有趣的部分。我们将把 Flutter 和我们的 Firebase 系列整合在一起！**

**将 Firebase 和 Firestore 依赖项添加到 **pubspec.yaml** :**

```
**firebase_core:cloud_firestore:**
```

**![](img/6f15eea07c797007d0035752b3688846.png)**

**用以下代码片段替换 ***lib/main.dart*** :**

**这段代码是对我们原来的 main.dart 的快速修改，有 4 个地方:**

1.  **为了使代码更短，删除了所有注释；**
2.  **添加的导入:**

```
**import 'package:cloud_firestore/cloud_firestore.dart';**
```

**3.创建了顶层变量 ***fsi*** ，是 Firestore API 委托。我们可以使用它执行各种操作。**

```
**var fsi = Firestore.instance;**
```

**4.***_ increment counter***方法，该方法在最初的示例中用于增加点击的本地计数器，现在它从 Firestore 中的集合“clicks”和文档“plusButton”中读取数据。为此我们使用 ***fsi*** 对象:**

**![](img/ac36b11f9b074876cdf65bb725f18a7e.png)**

**关于 Firestore API for Flutter 的更多细节，我们将在下一篇文章中发现。现在，我们的重点是将所有需要的部分整合在一起。**

**让我们建立并上传 APK 到我们的云模拟器。刚开始就显示 10 点。这是因为我们把 10 作为 ***_m*** 变量的初始值:**

```
**int _m = 10;**
```

**![](img/0596055d6a9bc95aac2d9523bafa8ed7.png)**

**点击“加号”按钮。现在我们可以看到我们的 Firestore 系列节省了 100 美元！**

**![](img/34874ff38b50a529854c06a9d8852f9f.png)**

**整合已经完成。**

# **保存我们的项目**

**这是一次令人兴奋的旅行，不是吗？但是我们还没有完成我们的颤振代码。让我们在即将到来的变更之前存储我们的项目。**

**单击 FlutLab 左侧菜单中的“分支”图标:**

**![](img/db5752c910b0418b277dd0288c03cfe4.png)**

**选择“制作快照”:**

**![](img/378173192663b0acfc3860bfee2d8fc7.png)**

**给这个检查点起一个好名字，这样我们就可以很容易地回到我们项目的最后一个工作阶段:**

**![](img/19b596514a2ee5e012d1278bc09fd428.png)**

# **下一步是什么**

**恭喜你！你已经完成了第一个在线代码实验室与神话般的颤振和 Firestore。**

**在下一篇文章中，我们将播放和研究更多关于 Flutter-Firestore API、异步代码和流的内容。敬请期待！**