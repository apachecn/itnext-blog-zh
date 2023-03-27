# 禁用 WordPress 网站上的类别、标签和作者页面

> 原文：<https://itnext.io/disable-categories-tags-and-author-pages-on-your-wordpress-website-7d2166f9b613?source=collection_archive---------2----------------------->

![](img/5615cc3106d4eccb6d61089c66f72036.png)

Arnel Hasanovic 在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

我在网上找不到任何简单的超快解决方案，所以这是我的。

它将带你:

*   1 行实际代码
*   1 分钟

它假设:

*   你的 WordPress 网站托管在一个 Apache 服务器上
*   您的 Apache 服务器支持`mod_rewrite`
*   您可以完全访问`.htaccess`

如果用户试图访问一个类别/标签/作者页面，你可以将他们重定向到主页。

# 逐步地

1.  通过 FTP 连接到您的 web 服务器。
2.  在 WordPress 安装的根目录下下载`.htaccess`文件(通常是`/www`
3.  打开文件进行编辑。
4.  将此块复制粘贴到文件的开头:

第 3 行是唯一需要记住的东西

5.保存、关闭并上传文件到您的网络服务器。

# 它是如何工作的

*如果您有 1 分钟的时间来深入分析一个简短的解释，这是正确的地方。*

*   `Line 1 and 4`:告诉 Apache 只有当`mod_rewrite`模块存在时才执行这个块。
*   `Line 2`:告诉 Apache 激活重写引擎。这个引擎可以随时重写 URL。
*   `Line 3`:我们按照这个通用语法定义一个规则:

`RewriteRule patternToMatchAgainst whereToRedirect [Options]`

*   `^(author|tag|category)\/.+$`:这是要匹配的模式。

> 如果你不熟悉正则表达式，看看这篇文章

**这种模式意味着:**如果请求看起来像`/author/anything`或`/tag/anything`或`/category/anything`，那么，重定向到某个地方。

*   `/`:这是我们在模式匹配的情况下重定向用户的地方。它是你的 WordPress 网站的根。这意味着如果有人试图导航到一个类别/标签/作者页面，他将被重定向到主页。

# 感谢阅读

希望这篇小教程对你有所帮助，解决了这个共同的需求。

运行我自己的 WordPress 网站，我更喜欢几乎不安装插件，这就是为什么配置 Apache 似乎是可行的方法。

查看我的最新教程:

[](https://dmware.fr/the-easiest-tutorial-on-react-router-v4-youll-ever-find-build-a-single-page-application-in-seconds/) [## React Router v4 上最简单的教程——用…构建一个单页应用程序

### 我找了将近一整天才清楚 React 路由器将如何帮助我建立我的…

dmware.fr](https://dmware.fr/the-easiest-tutorial-on-react-router-v4-youll-ever-find-build-a-single-page-application-in-seconds/) 

请随时联系 **david.mellul@outlook.fr**

![](img/12af5639b8feac4b4a1ea6ccecdf5336.png)

[吴怡](https://unsplash.com/@takeshi2?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上的《白色马克杯里的卡布奇诺，白色泡沫艺术在木桌上》