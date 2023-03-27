# JavaScript 特征:修改全局原型的干净方法

> 原文：<https://itnext.io/straits-9ef2b9a563cd?source=collection_archive---------1----------------------->

![](img/287cb891aa338477650cab9d251719b2.png)

## 有了 ES6，终于可以以一种干净的方式向 Object.prototype、Array.prototype 和所有其他对象添加方法了

特征是大多数现代语言用来实现多态性而不依赖于继承的特性。

我们将介绍它们，展示它们可以用 JavaScript 解决哪些问题，并建议一种实现它们的方法。希望你觉得有趣！

# 关于 JavaScript 和现代语言的几句话

在过去的 5 年里，JavaScript 一直是我的首选语言，尽管它有着臭名昭著的怪癖。

它写起来很快，运行起来很高效，但最重要的是，它是*现代*。这种语言本身以及它的生态系统使用了很棒的特性和解决方案，是我们所知道的最好的。

JavaScript 支持面向对象的编程，并使用鸭类型来实现多态性。这种方法比继承更加强大和灵活，但是它仍然有一些重大缺陷。

实现 OOP 的一个更好、更现代的方法是使用 traits。通过 duck typing 和原型继承在 JavaScript 中实现体面的特征曾经存在问题，但是自从该语言的最新版本(ECMAScript 6)添加了新的原始数据类型(`symbol`)以来，情况有所改善。

但是很少有人使用这种令人兴奋的语言特性，这也是这篇文章的动机所在。

# 什么是特质？

[Traits](https://en.wikipedia.org/wiki/Trait_%28computer_programming%29) 是一种向现有类型添加语义的方式，不会有无意干扰现有代码的风险。

特征是继承的替代方法，作为实现多态性的方法，所有现代语言都有特征:想想 go、haskell 和 rust，仅举几个例子。

JavaScript 是一种[原型](https://en.wikipedia.org/wiki/Prototype-based_programming)、[鸭型](https://en.wikipedia.org/wiki/Duck_typing)语言，这使得程序员可以通过向现有原型添加新属性来做一些非常类似的事情。但是这种简单添加有很大的问题，应该避免。事实上[你永远不应该修改你不拥有的类型的原型](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Inheritance_and_the_prototype_chain#Bad_practice_Extension_of_native_prototypes)，这就是为什么大多数库在没有实际修改那些类型的情况下，尽力为现有类型提供新的功能。

JavaScript 的最新版本(即 ECMAScript 6)增加了一种新的原语数据类型`[symbol](https://developer.mozilla.org/en-US/docs/Glossary/symbol)`，可以用来有效地实现 traits。一个`symbol`基本上是一个惟一的标识符，可以作为一个属性使用，并且永远不会与其他任何东西冲突。在 ECMAScript 6 中，他们需要一种方法来扩展标准类型，而不破坏与现有代码的兼容性。这就是他们引入`symbol`的原因，他们确实用它来实现特征。然而，他们没有宣传这一新功能，也没有让 traits 成为一等公民，而是让`symbol`留在暗处。标准将这个特性称为*协议*，而不是*特征*，其中一个例子是[迭代协议](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols)。

除了缺乏指导方针和好的例子，另一个问题使得很难使用`symbol` s 作为特征:缺乏好的语法来这样做。

但是说够了！让我们看一个例子。

# 为什么我们需要特质？一个例子

假设您需要一个序列化程序来将数据转换成可以存储在某个地方的字符串。

`JSON.stringify()`不够好，因为它不支持“复杂”的对象(尝试字符串化一个圆形对象、`RegExp`、`Map`等，你会看到)。

那么，让我们写自己的序列化函数吧。我们希望支持[原始数据类型](https://developer.mozilla.org/en-US/docs/Glossary/Primitive) ( `boolean`、`number`、`string`等)、[内置类型](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects) ( `Array`、`Map`、`RegExp`等)，以及我们或其他人定义的类。有些东西可能无法序列化，比如`Function` s 或`Promise` s，这没关系:如果我们要序列化其中一个，我们会抛出一个错误。

我们试图做的是，给大多数类型添加一个新的序列化逻辑，不管它们是由我们定义的，由其他人定义的，还是内置在语言中的。这种逻辑需要为每种类型定制。

我们可以通过给每样东西添加一个新的`serialize`方法来实现这一点。这样`var.serialize()`将为任何变量`var`返回一个可序列化的表示。

让我们试一试…

它打印出`Person("peoro", 32){objects:[{"a": true,"re": /^...$/g}]}`，这正是我们想要的。

太神奇了！不是吗？

不完全是。

我们修改了我们没有所有权的现有类型。那叫[猴子打补丁](https://en.wikipedia.org/wiki/Monkey_patch)。它似乎可以工作，但是一旦有人试图在实际应用程序中使用我们的序列化程序，就会产生许多严重的问题。让我们来看几个:

*   尝试序列化对象`{serialize:true}`:您会得到一个错误，因为这样的对象的`serialize`属性覆盖了`Object.prototype.serialize`。
*   其他人可能会定义不同的`serialize`方法来序列化不同格式的对象。我们的序列化程序和他们的不兼容；如果两者都被加载到同一个项目中(即使是作为一个间接依赖)，事情会以意想不到的方式发生。
*   尝试在普通对象上使用`for...in`进行迭代:您也将迭代`Object.prototype`的`serialize`属性。这将打破现有的大部分代码。

monkey patching 的问题是我们正在修改全局数据:任何不是我们编写的函数都可能依赖于对我们可能已经破坏的现有对象的假设，

这就是为什么库(包括可能强加自己标准的大型库——比如 jQuery 或 lodash)不会修改内置类型的原因。他们宁愿公开自由函数(比如 lodash)，或者封装现有对象的包装器，并且只向它们的包装器添加新方法(比如 jQuery)。

值得注意的是，这些库选择的解决方案非常有限:很难对包装器和自由函数的行为进行专门化。当你写它们的时候，你可能会硬编码一个`if`的瀑布来支持一堆类型，但是以后它就不能被扩展了。您将无法使它们的函数与您的自定义类型一起工作。

作为一个练习，尝试定义一个`serialize`函数，它能够在不修改现有对象及其原型的情况下序列化几种类型。然后尝试让库的用户能够添加对他们自己类型的支持，或者添加到现有的第三方对象。你可能考虑的大多数解决方案(例如使用`Type → serializationFunction`地图)可能会导致进一步的意想不到的问题。

这就是我们需要帮助的地方。欢迎来到特质的世界。

# 如何在 JavaScript 中实现特征

一个`symbol`是 ES6 中引入的原始类型，可以用作对象的键，它保证永远不会与其他任何东西冲突:如果`sym`是一个`symbol`，访问`object[sym]`的唯一方法就是使用`sym`本身。另外，`for...in`循环不会迭代`symbol` s。

以这种方式工作并不是巧合:它们被添加到标准中的原因和我们需要它们的原因完全一样。ECMAScript 6 想要给现有类型添加新的功能，但是，正如我们所看到的，如果不冒破坏现有代码的风险，这是不可能的。

关于标准如何使用它们的例子，请看[可迭代协议](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols)。引入了一个新符号`Symbol.iterator`。实现它的类型可以使用`[for...of](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...of)` [语法](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...of)进行迭代。此类符号在`Array`、`TypedArray`、`String`、`Map`、`Set`上实现，您也可以在自己的类型上实现。

我们的序列化程序应该实例化一个`serialize`符号，使用它，并公开给每个人使用:

我们的用户可以在他们的类型上实现相同的`serialize`符号并使用它。其他现有的代码都不会受到影响。

然而，这并不是最终答案...它们非常强大，被标准用来实现特征，但是涉及它们的文档数量很少。几乎没有指南，很少有教程或文章解释它们是什么以及如何使用它们。结果是很少有模块在使用它们。

另一个问题与它们的语法有关:没有特殊的语法来使用它们，在某些情况下，这成为一种痛苦。

想象一个 [lodash-traits](https://www.npmjs.com/package/lodash-traits) 库，它为每个 lodash 函数提供并实现了一个`symbol`。

你不能只做:

你必须做…

这很快就会变得不舒服。

这就是为什么，除了特征，我们提出了一个新的语法，旨在帮助特征的发展。

# 特征的更好语法

我们提出了一个语言扩展，海峡语法，能够以如下方式编写前面的代码片段:

这是什么意思？

`use traits * from traitSet;`意味着我们将寻找物体内部的符号`traitSet`。我们称`traitSet`为*特征集*。`object.*key`意味着我们正在使用特征集中的符号`key`访问`object`。

实际上:

大致相当于:

我们将按照以下方式编写模块的序列化部分:

这种语法与 ECMAScript 6:

可以同时使用多个特征集中的特征，如果某个特征被复制或丢失，我们会收到一个错误。

该语法旨在…

*   将`symbol`变成 JavaScript 的一等公民。
*   使特征更容易声明和使用。
*   避免范围和特征变量之间的冲突和错误。

目前可以使用这种语法开发代码，并使用一个 babel 插件将其转换成标准的 JavaScript:[@ straits/babel](https://www.npmjs.com/package/@straits/babel)。

# 文章结论和海峡介绍

希望现在清楚了为什么需要特征，如何使用`symbol` s、原型遗传和鸭子分型来创建它们，以及如何舒适地使用它们。

straits 项目提供了许多公共函数来帮助声明和使用特征、特性和特性集。

如果您想尝试一下，只需在一个空目录中运行`npm init @straits`:它将建立一个准备使用新语法的项目。然后运行`npm install`，一切都准备好了:`npm start`将运行`src/index.js`，一个可以玩的 hello-world。

如果你喜欢特征，你应该在你的项目中使用[海峡语法](https://straits.github.io/syntax/)。这对你的用户来说是完全透明的，因为你将在 npm 上发布的代码是透明的:标准的、常规的 JavaScript。你的模块的用户可以自由选择是否也使用这个语法，或者手动使用`symbol` s，甚至通过自由函数。查看一下 [lodash-traits'](https://github.com/peoro/lodash-traits/blob/master/test/index.js) `[test/index.js](https://github.com/peoro/lodash-traits/blob/master/test/index.js)`，看看使用 traits 的模块如何在使用或不使用 straits 语法的情况下使用。

如果您想了解一些依赖于 traits 的项目，请查看:

*   [lodash-traits](https://www.npmjs.com/package/lodash-traits) :包装 [lodash](https://lodash.com/) 功能的特性集。
*   [粉笔-特性](https://www.npmjs.com/package/chalk-traits):特性集包装[粉笔](https://www.npmjs.com/package/chalk)功能。
*   [容器](https://www.npmjs.com/package/scontainers):一个强大的、高性能的库(尽管仍在 alpha 中),用于处理数据集合。
*   [ESAST](https://www.npmjs.com/package/esast) :一个以舒适的方式操作 JavaScript AST 的库(即没有包装对象)。

*原载于*[*straits . github . io*](https://straits.github.io/)*。*