# Javascript 中的一对概念可能会让您感到困惑——number . isNaN()和 isNaN()

> 原文：<https://itnext.io/pair-of-concepts-may-confuse-you-in-javascript-number-isnan-and-isnan-556d14448dbb?source=collection_archive---------3----------------------->

![](img/01495f216a51d4159398f9518d5d549b.png)

马库斯·斯皮斯克在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

JS 有很长的发展历史(和混乱),有一些混乱的遗产，不能被删除以保持一致性，只能通过新的特性/功能来改进。这导致了开发者的困惑。这个系列被写成[笔记，供我自己](https://techika.com/blog)和其他人理解这些概念并避免开发中的错误。

# **什么是** `NaN ?`

`NaN`是*的简写，而不是数字*，在 1985 年制定的【IEEE 浮点运算标准(IEEE 754–2008)】的[中有规定。在 Javascript 上下文中，它是全局对象的一个“*属性。换句话说，它是全局范围内的变量。*”。它具有以下特点:](https://en.wikipedia.org/wiki/IEEE_754-2008_revision)

*被认为是`Number`类型
*等于`Number.NaN`
* `NaN`是 JavaScript 中唯一不等于自身的值。是福尔西

看看一些例子，想想结果！？

```
*console*.log(*NaN* === *NaN*) // false*console*.log(*NaN* == *NaN*) // false*console*.log(*NaN* !== *NaN*) // true*console*.log(*NaN* != *NaN*) // true*console*.log(typeof(*NaN*)) // numbera = *NaN*;
```

**全球 isNaN()**

正如你所看到的`NaN`甚至不能和它本身相比较，那么我们如何检测一个变量是否是`NaN`，在 ES6 之前我们可以使用函数`isNaN()`来考虑下面的例子。

```
isNaN(*NaN*); // trueisNaN('NaN');   // trueisNaN(undefined); // trueisNaN({}); // trueisNaN('Techika.com'); // trueisNaN(''); // falseisNaN('12abcd') // true
```

为了理解这种行为，我们需要理解它是如何正常工作的。

根据[MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/isNaN):*当 isNaN 函数的参数不是 Number 类型时，值首先被* **强制为** `**Number**` *。然后测试结果值以确定其是否为* `*NaN*`

然后，许多人认为它对于非数字参数的行为令人困惑，并可能导致意想不到的结果。因此，ECMAScript 2015 (ES6)中引入了新的函数来解决这个问题。

**Number.isNaN()**

它是来自原始包装对象——Number 的静态函数。该函数最重要的特性是它**不会强制将参数转换成数字**。因为`NaN`是 JavaScript 中唯一不等于自身的值，所以`Number.isNaN()`被认为是必要的。

```
*Number*.isNaN(*NaN*);        // true*Number*.isNaN(*Number*.*NaN*); // true*Number*.isNaN(0 / 0);      // true// e.g. these would have been true with global isNaN()*Number*.isNaN('NaN');      // false*Number*.isNaN(undefined);  // false*Number*.isNaN({});         // false*Number*.isNaN('Techika.com');   // false*Number*.isNaN(''); // true*Number*.isNaN('12abcd') // false
```

**结论**

从我个人的角度来看，`isNaN()`可能不像很多人想的那样是一个 bug，但是当你想专注于价值的检测时，它是可以考虑的。问题是我们需要理解它的机制，它会尝试将参数转换成`Number`。为了可靠起见，当我们想确保它的参数为`Number`以进行比较时，我们应该实现`Number.isNaN()`。

*本帖原帖来自* [*我的博客。*](https://techika.com/2021/05/19/isnan-vs-number-isnan/)