# 面向 2020+的 JavaScript 新特性

> 原文：<https://itnext.io/javascript-new-features-for-the-next-2020-7be2200d4995?source=collection_archive---------1----------------------->

## 接下来几年你需要知道的几个特性

下一个十年就在眼前，我们必须不断更新我们的技能，所以我将与你分享我认为对于用 JavaScript 制作的新项目来说了解哪些特性是很重要的。在许多情况下，我会避免使用 codepen，因为许多功能都很新，但是我会一如既往地给你最好的官方文档。

![](img/4d5930827bcee17554770f39edbcae97.png)

# 可选链接

JavaScript 是一种非常异步和灵活的好语言，但是这些特性使得 JS 很容易崩溃。这个 [**可选链接**](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Optional_chaining) 是从一个对象调用属性的一个好选择，但是不需要所有东西都爆炸。例如，如果您这样做:

在使用之前，不需要使用 [*类型的*](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/typeof) 验证或[*hasownproperty()*](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/hasOwnProperty)来处理 *pt* (葡萄牙语)中的*字*的存在，只需调用 new 运算符就足以更好地处理*未定义的*或*空值*而不会死亡。

```
console.log(lang?.pt?.words) 
// undefined but everything else is ok
```

不会返回 ***类型错误:无法读取对象*的属性【pt】***错误，只能读取未定义或空值。即使这样也会很有用，在 *if* 语句中:*

```
if(lang?.en?.words){
    // Do something 
}else{
    // Do another thing
}
```

当我们在处理大型嵌套对象时，这个特性的真正威力就显现出来了。为了不被复杂性所淹没，我们可以这样做:

```
console.log(lang?.en?.words?.rudeWords?.['a bad word'])
```

在 79 Chrome 版本上，这个功能只在实验中可用，但是你可以在这里跟踪状态，看看什么时候在大多数浏览器中完全实现。

# 零融合算子

在我们需要改进数据验证的地方，JavaScript 为我们带来了 [**？？**](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Nullish_coalescing_operator) 操作员验证数据存在。它的工作方式类似于 OR 运算符，但不会因为错误值而失败，请看下面的示例:

正如我们所看到的，当 OR 运算符将零作为假时，即使零是一个有效的数字，我们的 nullish 朋友也不会忽略它。要返回 false 必须有一个 *null* 或*未定义的*数据值。我想到的第一个例子是这个操作符要解决的点赞、点击、浏览量的计数器……在计数中，零是很重要的。

目前[在](https://caniuse.com/#feat=mdn-javascript_operators_nullish_coalescing)还不可用，但在下一个版本中，将完全支持 Firefox v72 和 Chrome v80 的更新。

# [原生文件系统 API](https://wicg.github.io/native-file-system/)

这种 API 为开发人员提供了与用户本地设备中的文件进行交互的机会，为他们打开了一个新的可能性世界。创建 IDE，创建自己的 office 软件，如 Microsoft Office library，开发文件编辑器、文本查看器、图像编辑器，更集成的游戏引擎，甚至像 Medium 这样的应用程序可以将故事的草稿保存在本地计算机中……所有这些都直接在我们的浏览器中运行，这至少非常有趣。

原参考:[https://web.dev/native-file-system/](https://web.dev/native-file-system/)

这个 API 的一个大问题是它打开了安全漏洞，即使 Chrome 不允许用户选择 System32 这样的敏感路径，这也不意味着用户是完全安全的。这就是为什么这个可能需要一点时间来完全实现，现在只有 chrome 支持这个功能，但你必须启用*文件系统 api* 标志。

# 数字分隔符

你能快速读出这个数字，不是吗？，嗯但是这个另外一个 **1143484622** 号呢？。当然，我相信你可以，但是如果有很多其他类似的大数字呢？我们知道这些数字，但在脚本代码中很难快速阅读它们，有时甚至会令人困惑，正如[原始命题](https://github.com/tc39/proposal-numeric-separator)所说:

嗯，为了帮助我们更容易地阅读数字，已经引入了[数字分隔符](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Lexical_grammar#Numeric_separators):

忽略红色突出显示，是一个有效的 JS 语法

如果你曾经使用过像 Stripe 这样的 API，管理过你的应用程序中的金额，使用过数学语法…你会发现这些符号非常有用，你现在就可以使用它们，但请查看[兼容性图表](https://caniuse.com/#feat=mdn-javascript_grammar_numeric_separators)以获得更好的参考。

# 动态导入

几年前，在我们的脚本中调用外部 JS 文件是完全不可避免的，实际上是不可能的，因为所有的坏人都在那里，现在时代变了，坏人还在那里，但我们的浏览器有一个标准，并准备在 JavaScript 文件/标签内执行 [**动态导入**](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import#Dynamic_Imports) 。

简单到调用方法 Import，然后解决 promise，您就可以使用手中的模块了。首先，这可以大大提高网站的加载速度。

静态导入已经有一段时间了，但动态加载模块是相对较新的， [Chrome 从 v77 开始就支持这个特性](https://caniuse.com/#feat=es6-module-dynamic-import)，但直到今天才变得更加相关。

# 结论

您可以关注 ECMAScript 新特性的[](https://github.com/tc39/proposals)**TC39 提案，以便充分了解 JS 在不久的将来会走上什么样的道路。永远保持你的技能不断提高，并为下一年 JavaScript only 将不断增长做好准备。**

> ***参考文献:***
> 
> **web . dev→[https://web.dev/](https://web.dev/)**
> 
> **MDN 网络文档→[https://developer.mozilla.org/en-US/](https://developer.mozilla.org/en-US/)**
> 
> **我能用→[https://caniuse.com/](https://caniuse.com/)吗**
> 
> **发展到→[https://dev.to/](https://dev.to/)**
> 
> **TC39 提案→[https://github.com/tc39/proposals](https://github.com/tc39/proposals)**