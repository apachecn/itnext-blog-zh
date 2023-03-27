# Angular life:带 Angular Universal 的服务器状态代码

> 原文：<https://itnext.io/angular-life-server-status-codes-with-angular-universal-8b2b8f0163ff?source=collection_archive---------0----------------------->

显然，*浏览器*和*服务器*没有那么相似😅。

![](img/8779dd63383ba4e7b04e5390a1fd3cdb.png)

# #问题

既然您已经成功地为您的项目设置了***Angular Universal***，为了利用那种*甜美*和*快速*的服务器端渲染，您可以开始了🐾起来，你对自己说🧠:

> 我受够了。我可以专注于客户，有趣的部分！

# #大约两周后

谷歌总部的人🏢给了你一个提示。如果你的服务器在试图访问一部不存在的*电影* / *汽车* / *书籍*时返回 **404** ，你的 **SEO** 会变得更好，因为谷歌会停止抓取那些*坏的网址👎*(存在于互联网某处)。

同样，现在你发布了你网站的 **v2** ，你想从`movies?category=drama`重定向到`movies/drama`，但是你也不想失去街头信誉💰这些网页已经与谷歌。同一个好朋友👆告诉你，如果你做一个 **301 永久重定向**，谷歌会变得更聪明，重新分配已经赢得的信用到新的页面。

> 听起来棒极了！⭐️

你对自己说。

> 但是等等，我如何在我的 **Angular** 代码中使用不同的**服务器**状态代码？我不想让我的服务器突然知道我网站上的所有逻辑。嗯…🤔

# #解决方案

嗯，没那么难。显然，你可以*注入*来自`express`的**响应**，并根据你的喜好使用它。

解决方案的第一部分:**状态响应服务**:

这是面包和黄油🍞我们设置的🧈。一些🔑这里的要点是:

*   该响应是@ **可选的**注入参数
*   我们可以在响应上使用`statusCode`、`statusMessage`、`finished`和`end()`
*   **响应**是一个`express-engine/token`。因此，请确保您使用的是正确的导入

现在，为了让这个工作，我们必须确保在我们的**应用模块**或**核心模块**或*中提供这个服务，不管你的提供策略是什么*。沿着这条线的东西可能是:

# #怎么用？

现在，你如何使用它？好吧，随你怎么想。它可以设置 **404s** ，它可以设置 **301s** ，它可以设置 **500s** ，它可以吠叫🐶…等等，它还不能这么做…还不行！

这是一个非常简单的例子，说明如何在 **404** 场景中使用该服务。这将使一切短路，并将向浏览器返回一个简单的 **404** 和消息`Movie does not exist...yet!`。

这不是最好的解决办法。理想地💡，您可能希望让**某个组件**也听到这个消息，可能会在发布 **404** 时显示一个漂亮的 **UI** ，但是您明白这一点。世界是你的！🐚

# #想要更多？

如果你想看到更多关于这个的高级用例，请告诉我，这样我就可以做一些漂亮的 UI 处理之类的后续工作。

此外，如果你有问题，你知道在哪里可以找到我…在互联网上！🤓

再见。👋

# #延伸阅读？

一些其他角通用提示🖥:

1.  [角度寿命:用户首选语言带角度通用](https://catalincodes.com/posts/user-preferred-language-with-angular)

一些笔记📓我拍的:

1.  [注释:NativeScript + Angular](/notes-nativescript-angular-5ae7dbe18672?source=friends_link&sk=30e20b23026b8cf15fbabaed268f4687)
2.  [备注:React + Redux](/notes-react-redux-e0c7a4d62e69)

另外，如果你对🧪测试驱动开发感兴趣:

1.  角度世界中的测试驱动开发:第 1 部分
2.  [角度世界中的测试驱动开发:第二部分](https://catalincodes.com/posts/test-driven-development-in-an-angular-world-part-2)
3.  [角度世界中的测试驱动开发:第 3 部分](https://catalincodes.com/posts/test-driven-development-in-an-angular-world-part-3)

[](https://twitter.com/c5n_c8u) [## JavaScript 不可用。

### 编辑描述

twitter.com](https://twitter.com/c5n_c8u) 

# #结束通话