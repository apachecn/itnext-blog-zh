# firestore Flutter 后端。浏览器内代码实验室。第二部分。Web 应用程序。

> 原文：<https://itnext.io/firestore-backend-for-flutter-in-browser-codelab-part-2-web-app-8f1ceb2c4e44?source=collection_archive---------5----------------------->

![](img/f32c130c811caaca5bb1a0efa46dfd72.png)

# 欢迎回来！

在之前的[部分](/firestore-backend-for-flutter-in-browser-code-lab-part-1-1b94d7839b94)中，我们看到了如何将 Flutter Android App 连接到 Firestore。

开发了一个非常简单的应用程序来证明连接，并向 Firebase 后端发出一些简单的请求。

在这一部分中，我们将考虑一个更高级的业务应用程序，以及对 Firestore 更现实的请求。

此外，我们将为另一个平台开发:Flutter **Web** App。

与上一部分相同的是我们选择的 IDE——[flutlab . io](https://flutlab.io)，免费订阅的在线 Flutter IDE。

我会试着把这篇文章作为一篇独立的文章来写。你不一定要先阅读第一部分。我将展示完整的场景，但是对 Firebase 配置做了一些简化。

# 业务逻辑

我们将开发一个 [Scrum Poker](https://en.wikipedia.org/wiki/Planning_poker) Web 应用程序。

Scrum Poker 是一个非常重要的敏捷实践，所有团队成员都参与到开发任务的评估中。任务以故事点的形式进行评估，以类似斐波那契的格式指出工作的相对努力:1，2，3，5，8，13 …

在评估会议期间，团队成员做出他们的评估。每个团队成员都不能有偏见，这一点很重要。因为如果我们得出非常不同的估计，那么很可能团队不理解这个任务或者它被描述得很糟糕。

这就是为什么有一个领导团队成员，他结束评估会议，所有团队成员的所有评估都变得可见。

在这里，我们将关注支持 Scrum Poker 的应用程序的最简单的实现。它应该呈现非常简单的用户界面，但是具有完全实现的业务逻辑。

# 开始吧！

这个 CodeLab 的唯一先决条件是在 [https://flutlab.io](https://flutlab.io.) 注册一个用户账号(免费)。

打开您的仪表板。在这里你可以看到你目前的颤振项目。现在我们需要添加一个新项目。这个项目不是从头开始，而是从 FlutLab 的 WidgetBay 上已经实现的开源 Scrum Poker 派生出来的。单击左侧面板上的蓝色大按钮即可进入该页面:

![](img/08b5d5830966cc2c14ddab30b58de5ba.png)

在 WidgetBay 中，找到名为 Scrum Poker 的项目并单击它:

![](img/19669832a4a9a72490d6f1ae3b40d0ad.png)

在这里你可以阅读一些关于项目的信息，预览它作为一个弹出的网络应用程序，或者，在我们的例子中， ***把它*** 放到你的项目空间中。单击另一个蓝色大按钮:

![](img/a65706dbe6eb53713aa49e5677f8f89f.png)

因此，该项目将出现在您的仪表板中。单击以在 FlutLab 的代码编辑器中打开它。

![](img/0a7516a8ead100233822fe6513c46fed.png)

这个项目只是部分功能，因为它与 Firestore 没有真正的联系。这是有意为之，以便其他程序员可以将这个应用程序添加到他们自己的后端。我们开始吧。

转到 Firebase 控制台，创建一个新项目。你可以把它命名为“我的 Scrum Poker”或者任何你想命名的名字:

![](img/8c69fd26d6c06a87e6b9076499ad721f.png)

执行剩余的步骤(在第 1 部分中有详细描述)并创建项目:

![](img/88481e0d140ef8b067fac2a13ba82b7d.png)

现在，我们要将 Web App 连接到 Firebase，请单击此处的相应按钮:

![](img/524dc2d0d1155e9d43c241795a554229.png)

提供应用程序的昵称:

![](img/4deb8b19c97b0e4b8fcd4bb3338d1855.png)

并接收将您的应用程序连接到 Firebase 的配置片段:

![](img/ade1be255def73f4b8ce3cdf38d3e89b.png)

现在我们要做一件简单的事情:从这个页面复制最后一个 *<脚本>…</脚本>* 片段，包含:

```
<script> // Your web app's Firebase configuration var firebaseConfig = {
        ...
    }; ... firebase.analytics();</script>
```

现在，回到你的 FlutLab 项目，导航到它的 **web** 文件夹，打开【index.html】的**。你会发现类似的 *<脚本>…</脚本>* 片段填充了虚拟数据:**

**![](img/0588a49342059df6f18b6adaf0e892d1.png)**

**用之前复制的真实配置数据替换它！**

**现在下一步。让我们配置我们的云 Firestore 数据库。只需点击左侧菜单中的**数据库**项目。然后点击**创建数据库**按钮:**

**![](img/2cc2d643bd4890ad5392bff1956634d0.png)**

**您现在可以推迟安全配置，所以在下一个屏幕上选择“**在测试模式下启动**”选项:**

**![](img/e1a6b2190db176ff6dc9c9d5214c00c4.png)**

**创建数据库后，您可以选择创建新集合:**

**![](img/15712e641cd76c9c423013544874898e.png)**

**让我们将这个集合命名为“votes ”,并创建包含以下字段的第一个文档，这是非常自描述的:**

**![](img/431203d5818daf4fa5194a4cc5152f8b.png)**

**恭喜您，您刚刚完成了应用后端的配置！**

**在 FlutLab 代码编辑器中返回到您的项目。为项目构建选择合适的平台。点击左侧菜单中的**设置**图标:**

**![](img/f48c4abf51f605472154cd163188c957.png)**

**再次检查构建器目标是否设置为**网**并且**预览**是否打开**:****

**![](img/52ceb09e505611b654fd837ae5f1bca4.png)**

**现在是时候**建**这个项目了！点击顶部菜单:**

**![](img/91b09162c56ec2ba85d930987b768a9a.png)**

**您将看到一分钟的构建动画:**

**![](img/d3927ef86089c76375f071e0c649716b.png)**

**最后，您的 web 应用程序将出现在弹出窗口中:**

**![](img/050f29812031e2b6c2aecdf9d9565560.png)**

**让我们试试这个应用程序是如何工作的！输入您的**姓名**并点击“**会议主持人**”复选框。然后点击**加入会话**按钮:**

**![](img/b3335d9da46c4eb6daca4148e6ec5cb3.png)**

**现在你可以投票了！但是等一下，让我们添加几个你的“团队成员”。**

**![](img/222b4c97885c4cb0176ecb3af173f6b6.png)**

**点击预览窗口左侧的**获取链接**图标:**

**![](img/5a9d458ba6f68d43e1d029e6d60c5ac7.png)**

**新标签将在您的浏览器中打开。以“john”的身份加入投票环节:**

**![](img/368cfbd735cae8bd10364b1cb9a7cecf.png)**

**你可以再次分享这个链接，并以“bob”的身份加入。现在让我们模拟投票环节。以 john 和 bob 的身份在他们的浏览器选项卡中投票:**

**![](img/ddcbf6976973723ebe00655fd9be0d4a.png)**

**如您所见，团队成员看不到他们的投票。直到会议主持人“亚历克斯”投票:**

**![](img/69807c33dba439e58bf8d2d006b6d8e8.png)**

**在此之后，您可以在每个浏览器选项卡中看到投票会话的结果:**

**![](img/a65f9e28c9916ca8b0ee68cb46cac281.png)**

**所以我们可以证明应用程序的工作能力！**

# **它是如何工作的**

**让我们看看引擎盖下面。**

**让我们从最开始的单个 ***main.dart*** 模块开始:**

**![](img/f9bc9b8cd17fb5e53093bd6fb7b29bf3.png)**

**在这个模块的顶部，我们可以看到包含了 **cloud_firestore.dart** 库。**

**同样，全局变量 **fsi** 被创建并指向 **Firestore.instance** 。这个变量将被用作执行对 Firestore 后端的各种请求的网关。**

**下面你可以找到非常简单的代码结构。在根级别，我们有 **MyApp** StatelessWidget，它在其 **build** 方法中呈现 **MaterialApp** 。反过来，它在 **home** 部分有**my home page**——我们的两页应用程序的第一页。**

**我们可以跳过下一行非常标准的代码，直到下一个片段:**

**![](img/22da11972a35b19dbc43e42b443a249c.png)**

**其中，在提供了团队成员**姓名**和可选复选框后，我们可以看到绑定到“**加入会话**”按钮的方法 **goAhead** ()。**

**这个方法很重要。让我们看看它的代码:**

**![](img/a768f80e9d231165def8a1baea637d57.png)**

**它填充要存储到 Firebase 中的 **userData** 对象。它从 textEditingController 和 _checkboxValue 获取其字段。“估计”字段被设置为等于-1 的“未投票”状态。**

**对 Firestore 的第一个请求假装已经在“投票”集合中创建了具有给定团队成员名称的对象:**

```
fsi.collection('votes').document(textEditingController.text).updateData(userData)
```

**有两种可能的情况:**

*   **这样的一个文档确实被找到了，修改后我们去 **navigate()** 方法调用。**
*   **找不到文档，出现异常。所以我们试图发出另一个请求，用给定的名称创建一个新的**文档:****

```
fsi.collection('votes').document(textEditingController.text).setData(userData)
```

**如果成功，我们转到 **navigate()** 方法调用。或者我们会显示 Firestore 连接失败的警告对话框。**

****navigate()** 方法导致切换到我们 app 的第二页:**

**![](img/7c90a42453bb68411c37ab51a6ac3294.png)**

**它还有一个基于 StatefulWidget 和 Scaffold 的非常标准的结构。**

**脚手架的主体是这样的:**

**![](img/c2b680abb6f61d644d6e1b5ceacf9ba7.png)**

**它是一个 **StreamBuilder** 小部件，连接到 Firestore 的“ **votes** ”集合！**

**它的主要工作是填充 **_items** 集合，并将其提供给下游的 ListView.builder()方法。**

**这是来自 Firestore 的“阅读”部分。为了理解“编写”部分的概念，当投票被存储时，请看下一个代码片段:**

**![](img/c16efaf3cceded04f977ef9d16a7fd2e.png)**

**这就是 Scrum Poker 应用程序的主要逻辑！**

**现在你应该明白所有的部分是如何连接在一起的了。**

**你可以随意将这个应用扩展到许多有趣的方向:**

*   **创建永久团队，将他们的投票历史作为单独的集合**
*   **用一些好的图形设计来扩展用户界面**
*   **你自己的想法**

# **请继续收听**

**让我们保持联系。关于 Flutter/Firebase 的一些非常有趣的部分将很快以在线代码实验室的形式发布！**