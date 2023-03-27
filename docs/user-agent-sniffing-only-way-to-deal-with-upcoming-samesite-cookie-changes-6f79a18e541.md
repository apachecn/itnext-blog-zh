# 用户代理嗅探处理即将到来的 SameSite Cookie 更改的唯一方法

> 原文：<https://itnext.io/user-agent-sniffing-only-way-to-deal-with-upcoming-samesite-cookie-changes-6f79a18e541?source=collection_archive---------3----------------------->

2020 年 2 月 4 日，Chrome 将对 SameSite cookies 的处理进行修改，要求任何人接受跨来源请求的 cookies。新 Chrome **要求的 cookie 格式不能被相当一部分老浏览器读取。**

![](img/e281882359b98addeee9972118edd0e5.png)

照片由[诺亚·布舍尔](https://unsplash.com/@noahbuscher?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄

解决这个问题的一个建议是为每个 cookie 设置两个版本，但是如下所述，这种方法对于许多应用程序来说是不可行的。这使得用户代理嗅探成为 web 开发中最容易出错的做法，也是唯一兼容跨来源 cookies 的方法。出于这个原因，我们将以一个全面的指南来结束这篇文章，该指南将使用户代理嗅探正确，以一种处理不兼容浏览器的完整列表的方式综合主要参与者的建议，并在今天使用的超过 40000 个独特的真实世界用户代理字符串的数据集上验证它。

十多年来，web 开发人员社区尽最大努力避免在 *JavaScript* 中进行用户代理嗅探，但是*服务器端*用户代理嗅探自 90 年代以来几乎没有进行过，即使在那时感觉也很糟糕。现在它又回来了，对于某些用例来说，它是强制性的。

首先让我们看看这是如何发生的。如果您只想了解如何可靠地解决这个问题，请向下滚动。

# 我们是怎么到这里的？

*   许多第三方登录的实现，如 OAuth，要求您接受从另一个站点发起的请求，并读取该请求中的 cookie。
*   [2019 same site 标准](https://tools.ietf.org/html/draft-west-cookie-incrementalism-00#section-3.1)强制要求不得在其他站点的帖子请求中发送 cookies，除非它们标有`SameSite=None; Secure`。如果没有设置这个属性，cookie 将默认为`SameSite=Lax;`，这意味着不会在跨源帖子上发送 cookie。`SameSite=None`必须与`Secure`属性一起设置，这意味着这些请求必须发生在 HTTPS。这将在 Chrome 80 中实现，Chrome 80 将于 2020 年 2 月 4 日为用户自动安装。Firefox 和 Edge 已经表示，他们也将采用这些变化，但尚未确定发布日期。
*   【2016 年 4 月 SameSite 标准的一个草案不知道`SameSite=None`，并规定当遇到未知值时(例如“None”)，cookie 应该被忽略。这是在 [Chrome 51-66](https://www.chromium.org/updates/same-site/incompatible-clients) 中实现的，以及从它派生的任何东西，比如 Android WebView 或 Chrome。Android 上的 UC 浏览器< 12.13.2 部分版本也有这种行为。
*   【2016 年 1 月的 SameSite 标准草案规定未知的 SameSite 值(例如“无”)应被视为`SameSite=Strict`。这个行为在 iOS 12 上的[任何浏览器和 MacOS 10.14 (Mojave)上的 Safari 上实现。Chrome 团队坚持认为这种行为是一个 bug，但它实际上符合这个特定版本的规范。](https://github.com/aspnet/AspNetCore/issues/14996)
*   对于 iOS 12 上的任何浏览器和 MacOS 10.14 上的 Safari，这种行为都与操作系统相关联——用户必须更新操作系统才能获得新的行为。苹果[不会将修复](https://bugs.webkit.org/show_bug.cgi?id=198181#c19)回移植到 iOS 12。在 iOS 12 上的 iPhone 5S、iPhone 6/6 Plus 和第 6 代 iPod Touch 的用户在任何情况下都无法升级到 iOS 13，因此他们必须购买新的设备才能获得新的 cookie 行为。

# 这对跨产地的饼干意味着什么

*   如果需要在 Chrome 80 中从跨站点请求中读取 cookies，需要用`SameSite=None; Secure`显式设置。就其本身而言，这是 cookie 行为的一个突破性变化。
*   如果你需要在 iOS 12、MacOS 10.14 上的 Safari、Chrome 51–66 或没有 same site 属性的 UC 浏览器< 12.13.2 for Android, you need to set them *上读取跨站点请求的 cookies。*
*   这两种行为是不相容的。因为一长串浏览器将`SameSite=None`视为`SameSite=Strict`或可以忽略的东西，所以没有办法设置一个跨站点请求发送的 cookie，既适用于这些浏览器，也适用于 Chrome 80。您必须在设置它时进行用户代理嗅探，或者采用双 cookie 方法，如下所述。在一个浏览器中有效的东西，在其他浏览器中就会失效。
*   如果您通过 OpenIdConnect、SAML 2.0、WsFederation 或任何其他方式使用第三方身份验证，并且需要读取在不同站点发起的请求中发布到您站点的 cookie，您会受到影响。

# 为什么会这样？

Chrome 引入的变化的重点是在默认情况下获得更强的跨站点请求伪造保护。这些变化本身就是突破性的——不管旧浏览器在做什么，Chrome 推动的变化意味着`SameSite=None; Secure`将不得不设置为有意的跨站点 cookie 传递，以处理 POST 请求。这些变化极大地降低了意外引入 CSRF 漏洞的可能性，因此它们显然有一定的价值。如果在 Netscape 1.0 中引入了这一点，提倡这一点是显而易见的。当这些突破性的改变在 cookies 引入四分之一世纪后被引入时，很明显会有很多东西因此而被打破。

更大的问题是，原来旧版本的规范与新版本的不兼容。一堆较旧的 Chrome 版本，以及 iOS 12 上所有遵循 2016 年标准的浏览器版本`SameSite=None`要么被忽略，要么被视为`SameSite=Strict`。2019 年规范的作者也是 2016 年规范的作者，但不清楚这些规范的互斥性是否被理解。

# 浏览器使用份额

根据 w3counter 的数据，4.75%的网页流量来自一台 iOS 12 设备。Caniuse.com 提供了每个 Chrome 版本的使用份额细分，并表示 Chrome 51-66 占总网络流量的 0.75%。根据来源不同，UC 浏览器被列为拥有 0.3-2.9%的全球市场份额，尽管这在不同国家有很大差异。例如，据报道它在印度占有 22%的份额。对于 UC 浏览器，源代码不会根据版本号进行区分。总之，使用 2019 年 12 月的统计数据，大约有 **6%的网络流量**将无法处理`SameSite=None; Secure`。即将到来的 2 月 4 日，自动更新的 Chrome 安装，约占网络流量的 55%，如果没有设置 T4，将拒绝跨来源传递 cookies。

# 谁需要关心这个？

首先，想想你是否需要从第三方的帖子中读取 cookies。如果没有，可以将 cookies 设置为`SameSite=Lax`(或者不设置，默认为 SameSite=Lax 行为)。如果您需要防止跨站点 GET 请求，请使用`SameSite=Strict`。

如果你真的需要来自其他网站的 cookies，你需要采取特殊的步骤。下面将解释两种处理方法。

# 有警告的解决方案:双饼干

解决这个问题的一种方法是读写两个不同的 cookie，一个有`SameSite=None; Secure`，一个没有，并在处理 cookie 时检查两者。您的 HTTP 应该是这样的。

```
HTTP/1.1 200 OK
Date: Fri, 17 Jan 2020 10:10:01 GMT
Content-Type: text/html; charset=utf-8
Set-Cookie: myCookie1=value; SameSite=None; Secure
Set-Cookie: myCookie2=value; Secure
```

这个*可能*对你来说是一个可行的解决方案，但是有几个原因说明这可能不是一个好主意，你应该三思而行。

1.  你要瞄准的浏览器之一，iOS 12 上的 Safari，对每个域的 cookie 大小有严格的限制。Safari 允许每个域总共有大约 4KB。如果您设置了两个 cookies，就将可用空间减半，很可能您会超过这个限制。这完全取决于你在饼干里储存了什么。
2.  改变饼干的制作方式*设定*有点复杂，但是很可能会集中到一个地方。然而，更改 cookie 的位置是*读取*可能会分散到代码库的更大部分。您还需要更改删除 cookie 的任何位置，并且在每种情况下，您都需要确保处理其中一个 cookie 存在、另一个 cookie 存在，或者两者都存在，或者都不存在的情况。总之，这有很大的复杂性，这种复杂性很可能与身份验证有关。那不是你想到处闯荡和冒险的地方。

虽然双 cookie 方法是谷歌推荐的，但是[ASP.NET 团队得出了另外一个结论](https://devblogs.microsoft.com/aspnet/upcoming-samesite-cookie-changes-in-asp-net-and-asp-net-core/)——他们发现采用用户代理嗅探方法将是更安全的方法。用户代理嗅探很难做对。很难涵盖所有相关的浏览器，而且很难做到在某种程度上你可以合理地确定它不会在未来的用户代理中被破坏。但是，当有人做对了，这就变成了一个任何人都可以使用的容易复制的解决方案。

# 另一个解决方案:用户代理嗅探

解析用户代理字符串很难，因为它实际上不遵循任何特定的格式，还因为每个浏览器都谎称自己是谁，并声明自己是许多其他浏览器。浏览器撒谎的原因是为了避开在解析用户代理字符串时做错事的糟糕代码。因此，因为用户代理字符串解析本来就很难，所以现在整个世界都在合谋让它变得更加困难。对于如何解析它们，真的没有规则，你必须依靠统计证据来看你是否得到了正确的结果。您需要对照已知的用户代理变体检查您的代码，并且足够具体地检查版本号，以便您可以确信它不会在未来的版本中崩溃。

我们通常会不惜一切代价避免用户代理嗅探，这是有充分理由的。然而，这些相同地点的问题是极端的，它可能是导致最少破坏的解决方案。

作为一个 JavaScript 错误记录服务， [CatchJS](https://catchjs.com/) 拥有大量关于用户代理的数据，以及一些解析这些数据的专业知识。以下是基于此的一些建议。首先，我们将概述如何可靠地检测 iOS 12、MacOS 10.14 上的 Safari 和 Chrome 版本 51-66 上的任何浏览器。然后，我们将对此进行扩展，以包括已知的不兼容浏览器的完整列表。

*   您可以通过在用户代理中查找子字符串“iPhone OS 12_”来可靠地检测运行 iOS 12 服务器端的 iPods 和 iPhone。
*   您可以通过查找子字符串“iPad；来可靠地检测运行 iOS 12 服务器端的 iPad。CPU OS 12_”。
*   上述两项检查也将可靠地检测 iOS 12 上的 Chrome。它通常还包括 iOS 12 上的网页浏览量，至少包括来自脸书、Instagram、Snapchat 和 Shopify 的应用。
*   MacOS 10.14 上的 Safari 可以通过查找*所有*的子字符串“OS X 10_14_”、“版本/”和“Safari”来可靠检测。
*   通过检查子字符串“Chrome/5”和“Chrome/6 ”,可以避免在 Chrome 50-69 上出现`SameSite=None`。我们只关心 Chrome 51-66，但是包括更多的版本是好的，因为这些在任何情况下都不会与`SameSite=None`有任何关系。还会包括 Chrome > = 500。假设目前的发布时间表是每八周发布一个主要版本，那么在这成为问题之前，你还有大约 64 年的时间。

上面的代码可以检测几乎所有的`SameSite=None`不兼容的浏览器，如果你想要一个覆盖 99 的低复杂度的解决方案，这是很好的。正在使用的浏览器的 X%。如果你想覆盖罕见的浏览器的长尾，下面的段落显示了如何。

上面忽略的一个浏览器组是在 Android 上安装的较旧的 UC 浏览器。在大多数市场，这可能没问题。检测它需要更昂贵的解析来提取版本号。浏览器通常会自动更新，并且在大多数市场的使用份额相对较低，因此对于许多网站来说，来自未更新的 UC 浏览器的访问份额非常小。然而，在一些亚洲国家，UC 浏览器是巨大的，你可能无法忽略旧的安装。在这种情况下，您可以使用下面的代码来可靠地检测不应该获得`SameSite=None`cookie 的安装。

在包括 UC 浏览器的检查之后，我们仍然忽略了 Chromium 的旧安装，以及 MacOS 10.14 上的嵌入式 web 视图。这些是条纹，但是可以被探测到。检测 Chromium 的方法与我们检测 Chrome 的方法类似，可以通过检查用户代理字符串是否包含“OS X 10_14_”并以“(KHTML，like Gecko)”结尾来检测 Mac 嵌入式浏览器。最后，给出了一个有点长，但完整的检查如下。

我们可以与其他来源给出的用户代理嗅探建议进行比较。

ASP.NET 博客[提供了 Azure Active Directory 团队使用的样本代码](https://devblogs.microsoft.com/aspnet/upcoming-samesite-cookie-changes-in-asp-net-and-asp-net-core/)来做这件事。它与我们上面的轻量级版本非常一致，因为它使用子串检查，并且只涵盖了旧的 Chrome、iOS 12 和 MacOS 10.14 的情况。

Chrome 团队提供了一个用户代理嗅探的[伪代码实现，它检测已知不兼容浏览器的完整列表。它依赖于运行多个正则表达式来提取和读取版本号。一个快速的非正式基准测试显示它大约慢了 2.5 倍。这可能根本不重要，取决于你的负载量。](https://www.chromium.org/updates/same-site/incompatible-clients)

我们在 40000 个不同的真实世界用户代理的数据集上比较了谷歌版本和我们版本的结果。它们的区别只是在边缘的边缘:例如，谷歌的 regex 要求在 Chrome/[versionNumber]之后有一个空格，因此不会识别(几乎没有使用过的)Chromium 衍生版本 Vivo 浏览器 6.0，该版本将 Chrome/[versionNumber]放在用户代理字符串的末尾。我们的版本需要在“iPhone OS 12_”中添加下划线，以便将来针对 iOS 120 进行验证，但因此错过了 iOS 12 上谷歌地图应用程序的嵌入式 web 视图，该视图报告自己为“iPhone OS 12.n”(带点)。这两者的使用份额基本为零。在支持未来的 iOS 120 与支持 iOS 12 上的谷歌地图 web 视图和完整版本号解析之间做出权衡，是受虐读者的一项练习。对大多数人来说，这两种解决方案都足够好了。

*最初发表于*[*【https://catchjs.com】*](https://catchjs.com/Blog/SameSiteCookies)*。*