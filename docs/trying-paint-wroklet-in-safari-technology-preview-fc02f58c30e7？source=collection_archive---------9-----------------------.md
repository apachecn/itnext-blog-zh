# 尝试在 Safari Technology Preview 中绘制 Worklet

> 原文：<https://itnext.io/trying-paint-wroklet-in-safari-technology-preview-fc02f58c30e7?source=collection_archive---------9----------------------->

![](img/1efb3c9665547281e9cb0b3b56cd4ad6.png)

撒哈拉沙漠探险

作为 CSS Houdini 的忠实粉丝，我很高兴 Safari 团队决定在开发中采用 Paint API。第一个实现是 Safari Technology Preview (TP) 69 附带的。但是在 72 版本的 changelog 中，我看到了令人兴奋的消息——可以将`<image>`作为输入属性传递给 Paint Worklet🤩。我想在此时此地玩它。不幸的是，这并不容易。

# 初始设置

第一个挑战是进行初始配置🤓。App Store 里找不到 Safari TP，需要从 developer.apple.com 的[手动下载。好的一面是它应该可以从 App Store 获得更新。然后你需要找到如何启用实验 API。作为一个 Chrome 用户，我已经开始寻找一些标志设置标签，但没有运气。要在 Safari TP 中启用画图 API，你需要进入*“开发”*“➡”*“实验功能”*菜单并切换*“CSS 画图 API”*选项。在我开始我的实验之前，我决定尝试我之前已经](https://developer.apple.com/safari/download/)[创建过的演示。他们在 Chrome 中工作，但是他们中的一些人需要 Chrome Canary，因为那里使用了一些其他的胡迪尼部件。所以我已经导航到基本的](https://vitaliy-bobrov.github.io/blog/exploring-the-css-paint-api/)[画图小工具演示](https://vitaliy-bobrov.github.io/css-paint-demos/hello-world/) …它死了😭。

# 死亡演示调查

屏幕上不是四个圆圈，而是白色背景。这意味着回退值没有应用，因为我已经使用了黑色。那很有趣，🧐.当我打开检查器时，我看到带有`paint`功能的背景被应用到元素上。然后我打开控制台，出现了一个错误——`ReferenceError: Can't find variable: paint`。这令人困惑，我没有在 Worklet 类定义中使用任何名为“paint”的变量。所以我尝试在控制台中手动加载它:

```
CSS.paintWorklet.addModule('paint.js');
```

又出现了同样的错误。Hm，`addModule`方法应该接受带有自定义 painter 实现的文件路径。有一个与安全相关的限制，类似于 Web 和服务工作者，所有的工作小程序代码都应该写在一个单独的文件中。他们有完全不同的 JavaScript 上下文，不应该访问任何全局数据。所以我尝试了不同的路径变化:

```
CSS.paintWorklet.addModule('/paint.js');
```

我又出错了，但是是一个不同的错误。似乎这个函数试图将 path 解释为一个代码。在[发行说明](https://webkit.org/blog/8547/release-notes-for-safari-technology-preview-72/)中有修改过的 Safari 源代码的链接。所以我决定在那里找到一个解释。和 Chrome 一样，Safari 也是用 C++写的。我不是这种语言的大专家，但至少能明白是怎么回事。

# 源代码 Safari

我已经开始探索开放与 Paint API 相关的变化的链接。我检查的第一件事是测试，因为它们是当前实现的最佳来源。它们只是 HTML 文件，看起来就像这样(为了显示要点，我将示例做得更短):

工作流代码存储在带有自定义 mime 类型“text/worklet”的脚本标签中，然后文本内容被传递给`importWorklet`助手。这证实了我的猜测，我继续查找源代码。过了一段时间，我在 [Worklet.cpp](https://trac.webkit.org/browser/webkit/trunk/Source/WebCore/worklets/Worklet.cpp?rev=239067) 文件中发现了一条很棒的评论:

```
// FIXME: We should download the source from the URL 
// [https://bugs.webkit.org/show_bug.cgi?id=191136](https://bugs.webkit.org/show_bug.cgi?id=191136)
```

所以我跟踪了 WebKit Bugzilla 中 bug 的[链接。评论解释了一切:](https://bugs.webkit.org/show_bug.cgi?id=191136)

> *目前，对 Worklet::addModule(字符串 url)的调用使用 url 作为代码。它应该按照规范异步获取脚本。—贾斯汀·米肖，2018–10–31*

`addModule`方法不是获取文件，而是从字符串中解析代码🤦‍♂️.

# 工作区

我的发现意味着我需要用 painters 将我的 JS 文件转换成一个字符串，但仅限于 Safari。直到他们根据[规范](https://www.w3.org/TR/css-paint-api-1/)实现该方法。有几种可能的解决方案:

1.  以字符串形式编写代码😅
2.  使用自定义 mime 类型在脚本标记中编写代码🤡
3.  手动获取 JS 文件并转换为字符串🤠

我决定采用第三种选择。我想检测 Safari 浏览器，请求一个脚本，并将其转换为字符串。

# 检测 Safari

在当前版本的演示中，我在注册 Paint Worklet 之前使用了功能检测:

之后，我需要检测 Safari 浏览器。经过短暂的调查，我使用了下一个用户代理检查片段:

这个有点混乱，但是 Safari 的用户代理应该包含“Safari”，而不是“Chrome”。因为 Chrome 两者都有🤣。

# 请求文件

这是最简单的任务，因为 Safari 支持大多数现代的 JavaScrip 特性。我使用 Fetch API 来请求文件。然后我将响应解析为`Blob`:

我们不允许在异步函数之外使用`await`,这就是为什么我将调用包装在 async IIFE 中的原因。

# 将文件转换为字符串

我发现，将`Blob`转换成字符串的最简单方法是使用文件阅读器 API。这个 API 的语义看起来与现代的不同，因为它非常古老。首先，我们需要创建`FileReader`实例。然后监听“load”事件。只有在这之后，我们才能开始阅读过程:

这是最后一个阶段，几乎所有的演示都可以在 Safari 中运行🎉。一些问题仍然存在，我会在最近的时间内修复它们。这是一个有趣的旅程，但正如我在源代码中发现的那样，Safari 团队实现了许多 Chrome Canary 中没有完成的功能。但这是一个完全不同的故事，我一定会在不久的将来分享我的实验。下面是生成的加载代码:

# 结论

制作一些未记录的和实验性的东西总是探索 Web 平台 API 的好方法。最重要的是，它教会我们保持评论和测试与时俱进，因为它们是真理的主要来源。玩得开心👻！

*原载于*[*vitaliy-bobrov . github . io*](https://vitaliy-bobrov.github.io/blog/trying-paint-wroklet-in-safari-tp/)*。*