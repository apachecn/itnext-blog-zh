# 使用 EcmaScript 6 (ES6)有条件地添加对象属性

> 原文：<https://itnext.io/adding-object-properties-conditionally-with-ecmascript-6-es6-7e7611da242e?source=collection_archive---------1----------------------->

*来源:*【https://twitter.com/bmeurer/status/1137025197557669888】

*这是谷歌的一名工程师最近发布的一条推文，他正在为谷歌 Chrome 网络浏览器和 Node.js 开发 V8 引擎。这让我想到了哪些用例对我非常有帮助，我应该在这里与全世界分享它们。*

## *向对象添加属性*

*在 JavaScript 中，向对象添加属性非常容易。您基本上可以像下面的代码那样完成它。*

*但是想象一下，如果有特定的条件，就应该加上`lastName`。*

*这看起来还不错，但是如果我们把条件组合起来，并把它们添加到多个 if 块中会怎么样呢？*

*总体来看，它看起来很难读懂，而且一个包含这段代码的函数会增长很多，即使它只是在做简单的事情。*

*EcmaScript 6 (ES6)是一个新标准，它支持 JavaScript 语言中的许多新特性。有了这些特性，我们可以编写下面的代码。*

*`person`的值的结果将与上面提到的脚本中的结果相同。但是这是如何工作的呢？*

## *`&&`号操作员*

*`&&`是 JavaScript 中的逻辑运算符，通过使用`AND`来组合布尔值。它可以与两个参数一起使用。让我们假设那些参数是`a`和`b`。当我们应用`&&`参数时，它看起来像`a && b`。这将产生一个布尔结果，可以是`true`或`false`。如果两个参数都是`true`，则为`true`。如果其中一个或两个都是`false`，结果也将是`false`。*

*另一个特点是，当不为`a`或`b`使用布尔值时，可能会发生没有布尔值被返回的情况。我们可以使用浏览器控制台简单地检查这一点。*

*我们可以扩展这个特性，并返回一个非常好的对象。*

*但是我们需要将这个对象合并到上面的`person`对象中🤯*

## *合并对象*

*使用`Object.assign`功能合并对象在 JavaScript 中早已成为可能。*

*这里发生的是最后一个参数`{ lastName: ""}`被合并到对象`basePerson`中。如果属性相同，则最后一个对象将胜出并覆盖前一个对象的属性。`Object.assign`也是可变函数。这意味着它会将所有更改应用于函数所有参数中的第一个参数。为了避免对`basePerson`对象的更改，我们传递一个空对象作为第一个参数，这样就创建了一个新对象，而不是一个被重用的旧对象。*

*EcmaScript 6 (ES6)提供了一种更好的方式来完成这些合并。*

*这将产生与之前相同的具有属性`firstName`和`lastName`的人物对象。同样，这里最后的属性覆盖了第一个属性，但是我们仍然有一个问题，那就是将在`&&`解释中返回的对象应用到对象中，因为当前版本假设`lastName`只是一个键，并且有一个静态值。*

*我们可以传递一个要被析构的对象到 person 对象中来传递一个对象，而不是键和值。*

*现在这可能足以重构并放入一个变量，但是我们有一个问题，因为`const result = condition && { lastName: "" };`可能导致`false`或`{ lastName: "" }`。如果我们将对象`...{ lastName: "" }`传递给析构函数，一切都很好，但是如果我们传递类似`...false`的东西，会发生什么呢？*

*![](img/d1046d189e303df536b91aad6b45c04c.png)*

*JavaScript 很聪明*

*实际上，这在一个对象中使用时不会产生任何结果。这也意味着我们可以在大的析构功能中使用它。*

*这非常好，但是仍然可以通过内嵌`result`来改进。*

*这是一个关于如何在 JavaScript 中有条件地扩展对象的超级巧妙的例子，它真实地展示了当使用现代语言时，我们可以使用更简洁的语法来表达自己。*

*当您创建相同类型的对象时，这些函数变得特别重要，有时有一些属性，有时没有。考虑单元或集成测试的工厂函数。*

*![](img/9b78101ec5f387020fa5fc8b3ef7bdf2.png)*

> *感谢你阅读这篇文章。你真棒🤘*
> 
> *也可以查看我的其他博客文章，比如 JavaScript 中的[函数参数](https://medium.com/@kevin_peters/function-parameters-in-javascript-clean-code-4caac109159b)，[学习如何用真实世界的例子重构 Vue.js 单个文件组件](https://medium.com/@kevin_peters/learn-how-to-refactor-vue-js-single-file-components-on-a-real-world-example-501b3952ae49)或者[如何调试 JavaScript 应用程序和测试](/how-to-debug-javascript-applications-and-tests-9718fe610f7d)。*
> 
> *如果你有任何反馈或者想给这篇文章添加一些东西，请在这里评论。您也可以在 [twitter](https://twitter.com/kevinpeters_) 上关注我，或者访问我的[个人网站](https://www.kevinpeters.net/)来了解我的博客文章和更多内容。*