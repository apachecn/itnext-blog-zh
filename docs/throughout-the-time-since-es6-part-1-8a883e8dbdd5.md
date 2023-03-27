# 自 ES6 第 1 部分以来

> 原文：<https://itnext.io/throughout-the-time-since-es6-part-1-8a883e8dbdd5?source=collection_archive---------8----------------------->

嘿，JavaScript 伙伴们，ES6 发布已经快六年了。有一层 JS 开发者是伴随着 ES6 长大的，对之前的版本不太了解。但另一方面，我还是会遇到在日常工作中用 ES5 得心应手，不会过多使用 ES6 功能的人。这篇文章对于那些想回忆 ES 特性、学习新东西或者想看时间顺序的开发人员来说很有用。

几天前，我发明了一个虚拟时间机器，并邀请你和我一起回到过去，一年一年地去看看 ECMAScript 是如何随着时间的推移而变化的。

![](img/8f4ec1b8068c34febf790424a02af465.png)

3，2，1 波耶哈利

# 2015 年

现在是 2015 年，准确地说是 2015 年夏天。让我们四处看看。我们看到 es 世界的许多新变化。让我们仔细看看每一个。

## 变量声明的新关键字。

引入了两个新的关键字 ***const*** 和 ***let*** 。两者都是块范围的，没有提升。 ***const*** 声明不可变变量。

如果不想重新声明变量，最好使用 ***const*** 。

MDN: [设](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let)，[常数](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/const)。

## 字符串插值(模板文字)

有可能不使用字符串连接来编译动态[字符串](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals)。

[介绍*串音*介绍](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/raw)方法。

[MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals)

## 箭头功能。

引入了新的语法来编写函数。

但是使用箭头函数有一些限制:

*   没有自己的*这个，超*也不能作为方法使用
*   没有*参数*(应用参数的类似数组的对象)和 *new.target* (指示函数是否用 *new* 运算符调用)关键字。
*   不能用*绑定*，*调用*或*应用*的方法。
*   不能用作生成器[函数](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function*)或构造函数。

[MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions)

## 扩展参数处理

引入了默认参数、spread 和 rest 运算符。

MDN: [默认](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Default_parameters)，[休息](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/rest_parameters)，[价差](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax)。

## 数组和对象析构

引入了一种从数组或对象中提取数据的新方法。

在数组析构中，顺序很重要，它与数组中的值相同。在对象中，顺序无关紧要，但是如果对象中没有析构属性，它将返回 undefined。

[MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)

## JavaScript 类

在 JavaScript 语言中引入了类，但它只不过是基于原型的面向对象模式的语法糖。

[MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes)

## 新的数据结构:地图，集合，WeakMap，WeakSet

为常用算法引入了新的高效数据结构。

WeakMap 和 WeakSet 提供无泄漏的对象键边表。

MDN: [地图](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map)，[集合](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set)， [WeakMap](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WeakMap) ， [WeakSet](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WeakSet) 。

## 生成器和迭代器

介绍了生成器和迭代器

MDN: [生成器](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Generator)，[迭代器](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols)。

## 模块:导出和导入

介绍了从/向模块导出/导入值。

MDN: [出口](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/export)，[进口](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import)。

## 新的内置方法

引入了许多内置方法

MDN: [字符串](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)，[数字](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number)，[数组](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array)，[对象](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object)。

## 承诺

引入了一个新的异步编程库。

[MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)

## 代理人

引入了代理对象，允许为另一个对象创建代理，它可以截取和重新定义该对象的基本操作。

[MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy)

## 新数据类型符号

引入了新的数据类型*符号*。

[MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol)

## 结论

哇，很多新的惊人的选择。让我们稍作休息，跳到新的 2016 年。

如果您对本文或我的其他文章感兴趣，请随时关注我:

github:[https://github.com/NovoManu](https://github.com/NovoManu)

推特:[https://twitter.com/ManuSEngineer](https://twitter.com/ManuSEngineer)

这是所有的乡亲。
下一个 2016 年见。

![](img/103f09f833d8b5a0aba4cf9fab02d48f.png)