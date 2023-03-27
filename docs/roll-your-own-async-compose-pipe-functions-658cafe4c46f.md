# 推出您自己的异步合成和管道功能

> 原文：<https://itnext.io/roll-your-own-async-compose-pipe-functions-658cafe4c46f?source=collection_archive---------0----------------------->

![](img/f9b0fa0a6ca228f32001ad707e6b1fdc.png)

受函数式编程启发的函数`compose`和`pipe`非常棒，但是它们只支持同步函数。我将向您展示如何编写您自己的函数，这些函数将与同步和异步函数结合使用，如果这是您喜欢的类型的话。

# 酷故事兄弟，给我看看代码吧！

好吧，给你。

异步合成功能

异步管道功能

## 好吧，现在怎么办？

如果您来这里只是为了获得一段代码，这没什么不好意思的，请随意将它添加到您的项目中，继续您的生活！否则，如果你想知道它是如何工作的，请继续阅读。

它可能会有点毛，但对我来说是光秃秃的。如果你熟悉某些部分，就直接跳过。

这些功能要求`Promises`在您的应用程序环境中可用，无论是浏览器还是带有节点的服务器端。如果你正在为现代网络(不包括 IE)开发，承诺已经在大多数浏览器中得到支持。回到现实世界，你可能会使用一个 [polyfill](https://github.com/stefanpenner/es6-promise) 或者其他一些第三方库，比如 [bluebird](https://www.npmjs.com/package/bluebird) ，给你`Promise`支持。如果你正在读这篇文章，你可能已经熟悉了承诺，如果不是，这里有一个快速入门。

# 承诺的简化入门

`Promise`是一个表示将在未来某个时间完成的动作的对象。这意味着，你为一个异步动作创建你的`Promise`，就像一个 AJAX 请求，当它完成时，类似于使用一个成功回调，你调用它的`resolve`函数；如果失败，就调用它的`reject`函数。

`Promise`对象公开了映射到两个“回调”函数的两个函数，即:当您`resolve`承诺时将调用的`.then()`和当您`reject`承诺时将调用的`.catch()`。

还有很多事情要做，但是如果你明白这些，你就会做得很好。

# compose & pipe 是怎么回事？

仅在 Medium 上就有许多关于`compose`和`pipe`的错综复杂的文章，它们在解释这一点上做得很好。如果你现在懒得用谷歌搜索，看到你已经在这里了，我会试着解释一下:作文就像成年人的乐高。这是一种通过使用更小更简单的单元来构建更复杂的东西的方法。在函数式编程中，`compose`是将较小的单元(我们的函数)组合成更复杂的东西(你猜对了，另一个函数)的机制。

因此，`compose`接受一个函数列表并返回另一个函数，您可以用您的数据调用它。一个例子就能说明这一点:

为了简单起见，我们创建了 3 个执行简单算法的基本函数:`double`、`square`和`plus3`。这里没什么特别的，`double`取一个值乘以 2；`square`取一个值，然后把它和自己相乘，最后，`plus3`就是这么做的——它取一个数，然后加 3。

在第 5 行，我们使用我们的`compose`方法来创建一个新的`composedFunction`，它是由我们的 3 个更简单的函数组成的。每次调用这个`composedFunction`时，它都会将这 3 个函数应用到我们传递给它的任何值上。在第 7 行，我们用值 2 调用我们的`composedFunction`。为了形象化这里发生的事情，想象一下:

```
double(square(plus3(2)))
```

这很难看，但它表达了这一点。

值 2 被传递给产生 5 的`plus3`函数。这个结果成为我们的`square`函数的输入，这个函数又产生 25。最后，我们用 25 调用`double`函数，然后产生 50。

这显然是一个简单得可笑的例子，但要点是把复杂的问题想象成一堆你可以一起解决的小问题的总和。

现在，假设您有一个来自 API 调用的响应来检索产品列表，您想要映射每个产品并只提取某些属性，比如`title`、`description`和`price`。然后你还想过滤掉所有比`$5`便宜的产品，最后按字母顺序对结果进行排序。

以前的你会写 3 个函数:`pluckProperties`、`getCheap`和`sortByTitle`，然后像这样做:

```
const products = // array of product objectsconst result = pluckProperties(sortByTitle(getCheap(products)))
```

***额外提示:*** *在对数据集做任何事情之前，总是先进行过滤，这样你就可以遍历最少的条目。*

新的你将使用`compose`来创建你的可重用`getProducts`函数:

```
const getProducts = compose(pluckProperties, sortByTitle, getCheap);const products = // array of product objects
const result = getProducts(products)
```

希望你现在已经看到使用`compose`的好处了。否则，所有的希望都没了，我也帮不了你。笑话。继续思考它，看看它会在哪些地方帮助你写得更干净更可重用(阅读:**更好！日常工作中的代码。**

我们也将快速触摸`pipe`。它与`compose`完全相同，但它从左到右而不是从右到左应用功能。

使用`pipe`，我们的示例将如下所示:

```
const getProducts = pipe(getCheap, sortByTitle, pluckProperties);const products = // array of product objects
const result = getProducts(products)
```

`pipe`对于编写一步一步的程序非常有用，比如:

```
pipe(logUserIn, displayNotification, redirectToHomepage)(user)
```

# 异步`compose`和`pipe`有什么酷的地方？

传统上，`compose`和`pipe`只对可以传递输入值的同步函数起作用，它对输入做一些事情，然后返回一个输出。查看上面的`pipe`示例，您会同意`logUserIn`函数很可能是一个异步函数，因为您需要与服务器/数据库进行一些通信。这样就不行了。洗澡时该哭了。

让我们通过使用我们自己改进的`pipe`函数来解决这个问题:

```
pipe(logUserIn, displayNotification, redirectToHomepage)(user)
  .then(() => {
    // The user is logged in
    // the login notification has been displayed
    // and s/he has been redirected to the homepage
  })
```

就这样，你完了！

# 我们来分解一下

作为参考，这里再次是`compose`片段。

首先，我们看到`compose`是一个返回函数的函数(示例使用标准 JavaScript 函数):

```
const compose = function(...functions) {
  return function(input) {
    // ...
  };
}
```

它使用 ES2015 [rest 参数](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/rest_parameters)将所有传入的函数组合成一个数组`functions`。

要理解内部函数的主体，熟悉 JavaScript 固有的`[reduce](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce?v=b)`和`[reduceRight](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/ReduceRight)`数组方法是很重要的。

`reduce`遍历给定数组中的每一项，并对其应用一个函数，每个函数的结果作为下一项函数的输入。每当你有一个东西的列表，你想把它“简化”成一个单一的值，使用`reduce`或`reduceRight`方法。这些方法有两个参数，第一个是需要为数组中的每一项运行的函数，第二个是可选的起始值。

最简单的是把它想成一个`SUM()`或`TOTAL()`方法，例如:

```
const numbers = [ 1, 5, 9 ];
const total = items.reduce(function(sum, number) {
  return sum + number;
}, 0);// total = 15 (1 + 5 + 9)
```

每个函数的输出都是下一个函数的输入。对于数组(1)中的第一个数字，`sum`将是`0`，因为这是起始值，`number`将是`1`。第二个数(5)将把`1`作为`sum`的值，`number`将是`5`，产生`6`。因此，对于第三个数字(9)，`sum`将是`6`并且`number`将是`9`，这产生了`15`的最终输出。

`reduce`和`reduceRight`的唯一区别是`reduceRight`从右到左(从后到前)而不是从左到右(从前到后)遍历数组中的元素。

好了，现在我们理解了`reduce`和`Promises`，让我们把它们放在一起理解我们`compose`函数的最后一部分。

```
functions.reduceRight(
  (chain, func) => chain.then(func),
  Promise.resolve(input)
)
```

与我们在示例中遍历一个数组`numbers`不同，这里我们遍历一个数组`functions`。这里我们不是从值`0`开始，而是从一个`Promise`开始，它立即解析为我们调用组合函数时使用的值。

在我们的 reduce/accumulator 函数中，我们不是建立一个`sum`，而是把将按顺序解决的承诺链接在一起。为了使用我们的用户登录示例直观显示这一点，将会产生以下内容:

```
logUserIn(user)
  .then(displayNotification)
  .then(redirectToHomepage)
  .then(result => `Do whatever we want with the ${result}`)
```

# 包扎

希望你觉得这很有用，如果你想在你的项目中包含这些功能，要么复制粘贴，要么从 NPM 获取:

*   [https://www.npmjs.com/package/compose-then](https://www.npmjs.com/package/compose-then)
*   [https://www.npmjs.com/package/fp-pipe-then](https://www.npmjs.com/package/fp-pipe-then)

*如果你有更好的想法或者很酷的东西可以分享，请告诉我；很乐意向你学习。*