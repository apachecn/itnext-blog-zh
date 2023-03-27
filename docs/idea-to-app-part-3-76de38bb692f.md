# 应用第 3 部分的想法

> 原文：<https://itnext.io/idea-to-app-part-3-76de38bb692f?source=collection_archive---------2----------------------->

![](img/d4193627b2c0c71f4431249f759011a7.png)

照片由[克莱门特·H](https://unsplash.com/photos/95YRwf6CNw8?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)在 [Unsplash](https://unsplash.com/search/photos/javascript?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄

# 介绍

在本系列的前两部分中，我们启动了我们的想法，并为响应式 webapp 准备好了基本的 UI。老实说，由于在新的工作中分心，完成这个部分花费的时间比我希望的要长……:-)我觉得我想分享的一些观点已经被遗忘了，所以这将是一个比我希望的更快的旅程。

[*点击这里在 LinkedIn 上分享这篇文章*](https://www.linkedin.com/cws/share?url=https%3A%2F%2Fitnext.io%2Fidea-to-app-part-3–76de38bb692f)

以防您忘记，以下是我们对 App 系列的*想法的概述:*

*   [构思](https://medium.com/@deadlysyn/idea-to-app-part-1-42ca01aba91d)
*   [UI、UX 和客户端关注点](https://medium.com/@deadlysyn/idea-to-app-part-2-ad040109ba97)
*   后端和 API(你在这里)

我原本也计划了一个*部署*部分…但是我决定这将是这个系列的最后一部分。也许我会在以后的系列文章中谈到典型的 web 和移动应用程序的部署方面。ChowChow 是一个部署在 Heroku 上的简单应用程序，所以没有太多的内容可以分享，除非是在[他们的优秀文档](https://devcenter.heroku.com/categories/reference)！

为了弥补这一点，基于我在工作中分配时间的方式(SRE 很有趣，经常让人难以割舍！)，实际编码，学习新东西和博客…一个更有趣的话题可能是围绕我们完成的应用和重构(添加更多的错误处理，更好地利用新的 ES 功能，如承诺，基于 [Airbnb 的风格指南](http://airbnb.io/javascript)的清理，等等。).

**记住** [**克隆资源库**](https://github.com/deadlysyn/chowchow) **跟着一起走……**

# 利用环境

在设计最现代的应用程序时，我们需要注意的第一件事就是管理敏感数据。通常的做法是从环境中读取这些值，然后可以通过凭证管理实用程序注入这些值，通过构建工具配置等进行设置。

利用环境不仅限于您不希望检查到版本控制中的敏感信息…它还可以被传递到控制行为(例如 dev vs prod)或读取动态信息，如监听地址。对于 [Node.js](https://nodejs.org) ，我们使用`process.env`来实现。

ChowChow 非常简单，我们没有太多的担心，但我们确实需要从环境中读取 IP 地址和端口(所以我的笔记本电脑和我的主机提供商都可以正常工作)。我们还有一个由[快速会话](https://www.npmjs.com/package/express-session)使用的秘密密钥(提供轻量级会话管理)。

从 Node.js 中的环境读取值

这个特殊的地址`0.0.0.0`只是监听所有可用的接口(可能是我家里笔记本电脑的回环或以太网地址，或者是像 Heroku 这样的平台上的容器的虚拟网卡)。我可以将它设置为`127.0.0.1`，让它在家里也能轻松工作，但在发布应用程序时通常会中断，所以`0.0.0.0`更容易管理。如果您担心绑定到笔记本电脑上所有可用的接口(希望您运行了防火墙)，您可以使用`127.0.0.1`，然后在启动时设置`IP`环境变量。`PORT`非常相似，所以我不会详述，只需注意如何传入一个`PORT`环境变量或者让它默认为`3000`。您可以通过 shell 将这些设置为`export NAME = value`或者像 Heroku 的[配置变量](https://devcenter.heroku.com/articles/config-vars)这样的机制。

最后但同样重要的是，`secret`将默认为`some random string`只是为了使开发更容易，但对于生产，我们将在我们的环境、容器、构建工具或托管提供者中设置`SECRET`环境变量…这样真正的秘密就不会被签入 GitHub。很简单，对吧？

# 开发与生产

我真的很喜欢学习 [Express](https://expressjs.com) ，但是如果你要发布一个真正的应用程序，你首先需要改变的事情之一(这不是秘密，所有的文档都明确说明了这一点！)是[快速会话的](https://www.npmjs.com/package/express-session)T0。这是原型开发的一个很好的起点，但是会在生产中泄漏内存(据我所知，它根本不关心到期)。

[后备存储有很多选择](https://www.npmjs.com/package/express-session#compatible-session-stores)(例如，您可能希望会话存储在 MongoDB 或 SQL 数据库中)，但是坚持我们简单的主题， [memorystore](https://www.npmjs.com/package/memorystore) 的工作方式与默认方式非常相似，没有内存泄漏。完美！让我们为 ChowChow 配置它:

配置备用会话存储

`secret`是我们从上面的环境中读取的值。根据[存储库文件](https://www.npmjs.com/package/memorystore)和[快速会议文件](https://www.npmjs.com/package/express-session)调整其他文件。

# 中间件

正如本系列前面提到的，我们为这个实验选择的技术栈是 Node.js 和 Express……在这个生态系统中，一个共同的主题是通过将执行繁重任务的功能分解到[中间件](https://expressjs.com/en/guide/writing-middleware.html)中来保持负责路由的代码整洁。

您可以将中间件功能链接在一起以获得灵活性，通过会话或返回值传递结果。让我们在我们的应用程序中看到一个简单的例子……首先，在`app.js`，负责显示用户附近随机选择的餐馆的`/random`路线看起来相当干净:

将繁重的逻辑转移到中间件可以保持路线的整洁

我们可以让我们的`app.post`路径包含`m.logRequest`和`m.parseRequest`中的所有逻辑，但是这将使代码更难阅读，并导致大量的重复(不是很[干](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself)！)像`logRequest`这样的东西，我们所有的航线目前都共享。

让我们深入了解一下`parseRequest` …

# API 争论

我们的大部分功能来自于 [Yelp 融合 API](https://www.yelp.com/developers/documentation) 。名字起得不好的`parseRequest`(老实说，出于代码*可能会*弄清楚的原因，当时感觉这是个好名字；如果没有，请在我们的重构列表中添加一项！)是不是*中间件*负责按摩我们 app 的输入(从[本系列前一部分](http://deadlysyn.com/blog/2018/01/20/idea-to-app-part-2)中讨论的`geolocation`获得的经纬度之类的东西)并获得 API 结果。

这仍然比我想要的要长，即使去掉了几行 helper 函数，但是这里有一个`parseRequest`的例子:

想象一下，如果这是你的路线代码？！？

这建立了 Yelp 的 API 搜索用户附近的食物所需的查询字符串…值得注意的是，通过使用[模板文字](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals)，使得`q`赋值*稍微*更易于管理。

一旦我们有了合适的查询，我们就用回调函数调用`searchYelp`(这可能需要一段时间)来检索我们的结果……如果发生了意想不到的事情，我们使用事实上的 [connect-flash](https://www.npmjs.com/package/connect-flash) 通过写入会话向用户显示消息(如果您克隆了 repo，您可以在`error` div 的`home.ejs`中看到一个如何显示消息的示例)。

这都是非常标准的(唯一的*技巧*是使用[滤镜](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/filter))，所以让我们通过看一下`searchYelp`来总结一下:

“发出 HTTP 请求的方法不止一种…”

首先，我们构建一个`options`对象来控制`https.get`的行为。大多数都是简单的`const`文件，但是`APIKEY`是从环境中读取的(毕竟，这是我们不想签入 git 的敏感数据！)经由熟悉的`process.env.API_KEY`。

最有趣的(也许？)这其中有一部分是使用了`https.get`。对于这个请求，我们可以使用任何数量的选项……我的第一直觉是使用熟悉的东西(在课堂上学到的)；[请求模块](https://www.npmjs.com/package/request)。除了最熟悉它之外，它还类似于 Python 的请求库。我们的应用程序的一个真正的缺点(试图优化简单性)是依赖的绝对数量。

其他一些选项有 [Axios](https://www.npmjs.com/package/axios) 、[将承诺包装在请求周围](https://www.npmjs.com/package/request-promise-native)或[fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch)…我选择`https.get`是因为它是标准库的一部分，我发现它将响应视为流的方式很有趣！你会用什么？

由于`https.get`响应是一个流，我们将`body`声明为一个块级变量，并向其追加数据，直到我们接收到`end`事件。然后我们用准备好的响应数据回电。

注意:我们甚至没有触及用 Javascript/Node 发出 HTTP 请求的许多方式……[看看这个](https://www.twilio.com/blog/2017/08/http-requests-in-node-js.html)。

# 摘要

诚然，这是一次闪电之旅，但现在我们已经正式窥视了幕后，看到了负责与 Yelp 对话和检索数据的中间件，这使我们的应用程序变得有用(或至少比无用略多)…我们还看到了如何快速解决默认`MemoryStore`的缺点，并利用 Node 的`process.env`轻松控制应用程序行为和保护敏感信息。

正如我前面所说，这可能是本系列的最后一篇正式文章……Heroku 简单地将部署做得如此简单，以至于没有什么值得写的，他们的文档已经比我希望改进的更好了！也就是说，我正在试验一个新的开发环境，包括 [VSCode](https://code.visualstudio.com) 、 [ESLint](https://www.npmjs.com/package/eslint) 和 [Airbnb 的风格指南](http://airbnb.io/javascript)……所以可能会重新讨论这个项目，以涵盖重构。

让这款应用成为真正的应用的最后一步将是更接近于原生的或渐进式的网络应用……这对我来说需要一点时间，因为我还在学习 [React-Native](https://facebook.github.io/react-native) (这不是唯一的路要走，只是我目前正在学习的一条路)。如果运气好的话，我会想出一个更有趣的前提来一起探索，一旦我走得更远并准备好合并其他技术。:-)

感谢阅读！

PS:如果你喜欢，一定要[看完整个系列](https://medium.com/@deadlysyn/idea-to-app-part-1-42ca01aba91d)！:-)

PPS:最初发布的[在这里](http://deadlysyn.com/blog)。