# Angular life:Angular Universal 的用户首选语言

> 原文：<https://itnext.io/angular-life-user-preferred-language-with-angular-universal-6d35548e0d48?source=collection_archive---------3----------------------->

![](img/c1f964741dc1a591650261efb2d57d8f.png)

在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上由[波莱特·沃图尔](https://unsplash.com/@vautourp?utm_source=medium&utm_medium=referral)拍摄的照片

用户说不同的语言🏴 🏳️

# #问题

既然你是角度宇宙的专家🤓(假设你读了[这个](https://catalincodes.com/posts/server-status-codes-with-angular)，你想让事情变得更好。你想确保用户得到应用程序📱(或者网站，无论你怎么称呼你的大脑🧠孩子)，如果可能的话，用他们自己喜欢的语言。

> **比如**:你支持**英文🇺🇸** 和**法文🇫🇷** ，但是你默认为**英文**并允许用户根据需要更改。虽然这很有效，但是如果支持的话，弄清楚用户想要什么并从一开始就给他们就更好了。👍

# #免责声明

现在，你的**产品负责人**不知道这件事(让我们称他为**罗宾**👱例如‍♂️)。所以，你不想打扰罗宾👱昨天，‍♂️谈到这些细节，你甚至不能确定他是否会把这些放在🗄️需要建设的一堆东西之上。长话短说:你想建这个🏗，并且您希望快速构建它⏩在雷达下。

# #解决方案

该解决方案有 3 个部分:

—服务合同，它将决定语言

—在浏览器上计算语言的服务

—在服务器上计算语言的服务

# #第一部分

契约需要是一个**抽象类**，因为*角度依赖注入*不**与**接口一起工作。这很简单👇：

这是**抽象**类。它也有一个支持语言的列表，它需要实现 **getLanguage** 方法。

# #第二部分

**浏览器实现。**这里，我们需要问一下浏览器🌐，用户首选语言是什么，看看我们是否支持。如果没有，使用一些默认值🇺🇸.这看起来会像这样👇：

这个使用**窗口**的🪟对象来询问什么是浏览器🌐语言。我们只使用前两个字符。接下来，我们检查是否支持这种语言，如果不支持，我们返回默认语言🇺🇸.

小🗒️注意，默认语言是一个**输入**为此方法。这样，需要它的地方，就完全控制了🎛️.

# #第三部分

**服务器** 🖥 **实现。**这就是事情变得棘手的地方🤡，或者说我是这样认为的。因为在🖥服务器上，我们没有真正的浏览器🌐，我们无法从那里获得用户偏好的语言。但是我们能做什么呢？

经过一点(灵魂)的探索🔎我发现浏览器🌐向它发出的每个请求添加一个名为' ***accept-language'*** 的头。这个头的值是一个逗号分隔的列表，按首选顺序列出了所有用户首选的语言。**非常有用！有了这些知识📚，我们可以实现服务器🖥版本的语言服务。它看起来像这样👇：**

如你所见，这里我们注入了**请求**，得到***【accept-language】***头，分割它，得到第一个值 *badabing，badabum 和 Bob 的你叔叔(Bob 是谁？)*。服务器🖥上有用户首选语言。

# `#Part 4`

是的，有第四部分。**把一切都绑在一起。我们现在唯一需要做的就是确保在正确的平台上提供正确的✅服务。类似这样的东西👇：**

当然，这是 **AppModule** 和 **AppServerModule** 的精简版本，但是您已经明白了……(如果没有，请告诉我，我会尝试更详细地解释这是如何工作的👨‍🏫).

# **#小纸条**

1.  您当然可以让服务不需要默认语言🇺🇸，并使用 **SUPPORTED_LANGUAGES** 数组中的第一项。
2.  实现之间的所有共同点都应该存在于抽象语言服务中。那是个好地方！🏨
3.  **！重要！**当你需要语言服务的时候，不要注入 **WebLanguageService** ，或者 **ServerLanguageService** 。总是注入 **LanguageService** ，因为 Angular 会计算出使用哪个实现。类似这样的东西👇：

# #想要更多？

让我知道，如果这还不清楚，你需要更多的例子，更多的用例等。

也可以随意标记🚩我的方法有什么问题，这就是我们如何在这方面做得更好👨‍🏫。

要善待对方！🧡

那都是乡亲们！👋

# #延伸阅读？

一些其他角通用提示🖥:

1.  [Angular life:带 Angular Universal 的服务器状态代码](https://catalincodes.com/posts/server-status-codes-with-angular)

一些笔记📓我拍的:

1.  [备注:NativeScript + Angular](/notes-nativescript-angular-5ae7dbe18672?source=friends_link&sk=30e20b23026b8cf15fbabaed268f4687)
2.  [备注:React + Redux](/notes-react-redux-e0c7a4d62e69)

另外，如果你对🧪 **测试驱动开发**感兴趣:

1.  [测试驱动开发:第一部分](https://catalincodes.com/posts/test-driven-development-in-an-angular-world-part-1)
2.  角度世界中的测试驱动开发:第 2 部分
3.  [角度世界中的测试驱动开发:第 3 部分](https://catalincodes.com/posts/test-driven-development-in-an-angular-world-part-3)

[](https://twitter.com/c5n_c8u) [## JavaScript 不可用。

### 编辑描述

twitter.com](https://twitter.com/c5n_c8u) 

# #结束通话