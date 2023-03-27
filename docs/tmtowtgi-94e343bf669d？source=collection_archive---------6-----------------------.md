# TMTOWTGI

> 原文：<https://itnext.io/tmtowtgi-94e343bf669d?source=collection_archive---------6----------------------->

![](img/d297462bff405d04e890ec6dc37d8244.png)

由[克里斯蒂安·斯塔尔](https://unsplash.com/photos/8S96OpxSlvg?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)在 [Unsplash](https://unsplash.com/search/photos/different?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 拍摄的照片

Perl 普及了这个概念*有不止一种方法可以做到这一点*(这取决于你问谁，要么很棒，要么只是令人困惑)。JavaScript 同样擅长给初学程序员两种方法去做几乎任何事情。[在之前的一篇文章](http://deadlysyn.com/blog//development/2018/02/11/idea-to-app-part-3)中，我们简要评估了如何为查询一个 API 实现简单的 HTTP GETs 作为对所有 Perl 僧侣的敬意，让我们探索一下如何有不止一种方法来获得它(或者 post、PUT…无论什么)。

注意:我倾向于不在 JavaScript 中使用分号…这并不是想要冒犯或参与任何圣战。在你认为合适的地方随意想象分号，或者更好的是[观看这个既有趣又有启发性的独白](https://youtu.be/Qlr-FGbhKaI)。

'注意':请带着幽默感阅读这篇文章或我创作的任何东西。:-)

# 语境

作为一个方便的用例，我们将构建一个极简的比特币价格检查器，因为，好吧，每个人都在做比特币的事情，同行的压力是真实的。我们将使用 Coindesk API。

为了简洁起见，我不会在所有示例中重复这一点，但我们在 JavaScript 的顶部声明了一些重要的常数，它们指向我们将使用当前比特币价格更新的 API 端点和元素:

“const”确保全局值不会被覆盖。

对于下面的例子，也使用了普通的 HTML 样板文件…下面是它的样子，这样 JavaScript 更有意义:

这里没什么可看的…

是的，就是它…相当可怕，但用户界面不是重点，我们不想让它分心。事实上，这应该更简单——我开始陷入对 HTML 和 CSS 无休止调整的黑洞。:-)您需要注意的是，我们设置了一些独特的元素`id`以使我们的选择器查询更容易，并导入我们稍后将使用的 jQuery 和 Axios。

… [或者直接用出处，Luke](https://github.com/deadlysyn/bitcoinGetter) (或者 Lukette，或者随便你的名字，发色，原籍国，人称代词等等。恰好是——source 对我们所有人都同样好)。

# XHR

首先，让我们回到过去……AJAX(异步 JavaScript 和 XML)刚刚出现的时候。显然，它仍然是新的，因为今天它将被称为 AJAJ(现在什么不喷 JSON？).那时还没有*单页面应用*和其他现在普遍接受的热门应用，所以异步 HTTP 请求的能力显然让人们非常兴奋。

严格来说， *X* 不仅仅是*代表 XML，更确切地说是 [XMLHttpRequest](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest?redirectlocale=en-US&redirectslug=DOM%2FXMLHttpRequest) 。今天，这个内置对象仍被用于在 JavaScript 中发出 HTTP 请求。幸运的是，现代实现使用 JSON，因为它在 JavaScript 生态系统中更有意义。我相信你已经注意到， *XMLHttpRequest* 如果我们将其缩写为 *XHR* 会更好用，所以这是通常的做法(谁不喜欢缩写词)。*

你说这是很好的琐事素材，但是它是如何工作的呢？给你:

使用 XHR 发出 HTTP GET 请求。

这没什么好奇怪的，假设你至少明白一些事情:

*   在做任何有用的事情之前，我们需要[实例化](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Working_with_Objects)一个新的 XHR 对象(因此使用了`new`)。
*   `onreadystatechange` ( *love* 那个名字)是一个事件处理程序，每当我们的新 XHR 的`readyState`属性发生变化时就会调用它。
*   `readyState`有[五种可能的状态](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/readyState) (0-4)，其中`4`表示请求完成。

如果你发现`open`和`send`令人困惑，[文档帮助](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest?redirectlocale=en-US&redirectslug=DOM%2FXMLHttpRequest)…但是只要把`open`想象成*建立请求*，并且`send`，嗯，*发送请求*。一旦发送，状态变化将传播，假设它完成(我们得到一个`200 OK` ) `PRICE.textContent`被更新。

为什么要从 xhr 开始？它们是下面所有事物建立的基础元素！

# 取得

另一种本地方式(除了我们过去讨论过的`https.get`之外)是[获取](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)。这是这种语言的一个新的补充，从它对承诺的使用可以看出。呜啊啊承诺，你说！在我们了解一个不幸的秘密之前，让我们先看看它的运行情况(如果*秘密*是文档中描述的*行为的另一个名称):*

使用 Fetch 发出 HTTP GET 请求。

很整洁，是吧？这里要注意的主要事情是使用`.then`和`.catch`来处理承诺，以及内置的`.json()`方法，它取代了我们上面使用的`JSON.parse()`。不错，干净，现代…但是还没有很好的支持。[获取 API 文档](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)显示 IE 在这一点上完全被冷落，这对你来说可能是一个阻碍。

# jQuery

优秀的老 jQuery。如果说我们当中还有比这更同时受到尊敬和批评的人，那我还没见过……至少从上次关于 vim vs emacs 的对话之后没有。虽然，从技术上来说，我想这是一个关于两个同样受辱的工作狂的对话(取决于你和谁说话)。我跑题了。

虽然没有以前那么流行，但是 jQuery 仍然是全功能的，在某些情况下非常有用，考虑到过去的流行程度，您可能需要支持它，而不管个人意见如何。郑重声明，我并不反对——只是要确保在引入一个大的依赖项之前，您需要做的不仅仅是发出一个 HTTP 请求！

这是:

好的 ol' jQuery 使 HTTP 请求变得容易。

`querySelector`不见了，或者更准确地说……它被伪装成了`$`。不同的语法，但是我们仍然将一个[匿名函数](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions)绑定到一个点击监听器。在内部，我们使用 [jQuery 的](https://api.jquery.com/jQuery.getJSON) `[getJSON](https://api.jquery.com/jQuery.getJSON)` [方法](https://api.jquery.com/jQuery.getJSON)将 HTTP 响应转换成有用的对象。这是 jQuery 的两大优势——不缺少文档或实用方法！

# Axios

最后，但同样重要的是， [Axios](https://github.com/axios/axios) 是一个非常受欢迎的 HTTP 客户端库(也就是说，在撰写本文时，GitHub 上有近 4 万个 stars)。它是基于承诺的，轻量级的，并且得到了很好的支持。

Axios 非常受欢迎是有原因的！

承诺处理看起来与前面的例子相似，但是没有任何对 JSON 相关方法的引用……因为库自动完成了转换。相当甜蜜！Axios 也是高度可配置的，并且支持实际上很容易理解的并发性:

Axios 让并发变得简单。

# 摘要

所以，现在你有了…比任何人用 JavaScript 发出 HTTP 请求都多的方法。不用说，上面的例子都是客户端的，但是也可以在 [Node.js](https://nodejs.org) 中轻松完成。对于需要导入的，只需将`<script></script>`标签替换为`npm install axios`或类似标签即可。

最后但同样重要的是，我想对 Udemy 的高级 Web 开发人员训练营大声疾呼。这里的用例与例子是从那里获得的东西的融合。如果你正在寻找一个关于现代 web 开发的好的概述，这是一个很棒的课程——我与 Udemy 或课程作者没有任何关系……所以希望这能有所帮助。:-)

感谢阅读！

PS:我的例子使用 USD 只是因为它是我的本地货币，并保持代码有点干净…在撰写本文时， [Coindesk API](https://www.coindesk.com/api) 可以返回 EUR、GBP 和 USD。作为一个有趣的练习，[克隆回购](https://github.com/deadlysyn/bitcoinGetter)并重构以支持您喜欢的，或者让用户选择所需的货币！

[最初发布于此处](http://deadlysyn.com/blog//development/2018/02/21/TMTOWTGI) …