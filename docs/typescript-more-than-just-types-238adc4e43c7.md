# TypeScript:不仅仅是类型

> 原文：<https://itnext.io/typescript-more-than-just-types-238adc4e43c7?source=collection_archive---------4----------------------->

## ES6 增加了 5 个出色的功能

![](img/538be3483f2fe8d05487d294ac476441.png)

JavaScript 占据了我的心，并且可能永远如此。但是，这并不是说我不知道它的缺点和改进的机会。像大多数语言一样，有一些好的部分，也有一些怪异的部分。虽然我喜欢这种松散类型语言的自由，但我也承认它需要你放弃一些控制权。为了夺回部分控制权，林挺变得相当受欢迎。但是它仍然不能像强类型语言那样给你控制权。幸运的是，我们有打字稿。

如果您曾经使用过面向对象的语言，TypeScript 会感觉非常熟悉——这不是巧合。微软使用 C#等语言中的许多相同模式和功能创建了 TypeScript(他们也保留了这些模式和功能)。然而，还是有一些不同之处。我喜欢 TypeScript 的一点是，在所有重要的方面，它仍然*像 JavaScript 一样。*

顾名思义，TypeScript 对 JS 的主要补充是** drumroll *……*types！其中包括:`number`、`string`、`object`，甚至还有泛型。然而，我喜欢 TypeScript 的原因是它对语言的其他补充。增加了一些内容，使 JavaScript 更具可读性，并鼓励代码的行为更可预测。

在我的团队中，我们已经开始将我们的三个回购转换为 TypeScript，在每次转换过程中，我们都很容易发现一些错误，否则很难找到。自从使用了 TypeScript，我们的速度更快了，我们的 Pull 请求变得更加不言自明。

但是炒作够了！如果你想看我列出的 TypeScript 提供的 5 大附加功能，请继续阅读。

# 1.访问修饰符

如果你熟悉其他面向对象的语言，比如 C#，你肯定会遇到像`public`、`private`和`protected`这样的属性访问关键字。令(许多人)沮丧的是？)开发人员来说，在 ES6 中将类引入 Javascript 时，没有访问修饰符。这是 Javascript 避开了看起来面向对象，但又忠于其原型之间的界限的又一个例子。

幸运的是，TypeScript 引入了所有这三个关键字来提供对类属性访问的更多控制——让您能够决定如何访问类属性。这对项目的未来维护者来说也是一个很好的信号，即某些属性永远不会在创建它们的上下文之外使用。这种微妙的交流本身就为长寿项目创造了奇迹。开发商不再需要搜索所有的物业用途来确定其预期用途。

截至发布时，普通 JavaScript 中的所有类属性都是隐式的`public`。您可以使用一些模块魔法来创建`private`变量的*功能*，但是它从来没有像 TypeScript 的访问修饰符那样优雅地完成工作。有一个向类添加私有属性的提议，但是值得注意的是，在这个阶段，这个提议与 TypeScript 的实现有很大的不同。

![](img/8be2b060983fbeb35db9feeffe92a655.png)

```
⛔️️ *no output*(Line 17 will result in a TypeScript error because we set the variables **name** and **sizeInMeters** to *private*. In that case, even child classes cannot access the variable. This can be fixed by making the variables *protected*.)
```

# 2.只读属性

如果一个私有变量有 getter 但没有 setter，它就隐式地变成只读的。然而，TypeScript 太棒了，它添加了一个`readonly`关键字来显式地创建一个只读的类属性。

虽然这个关键字实际上只是语法上的糖，但它非常可爱。它减少了冻结变量值所需编写的代码量。这有助于使您的代码可读性更好，并再次向未来的开发人员清楚地传达您的意图。

![](img/8be2b060983fbeb35db9feeffe92a655.png)

```
⛔️️ *no output*(This snippet will not compile because we've explicitly told the class that **age** is a *readonly* variable.)
```

# 3.抽象类和接口

抽象类是 JS 生态系统的另一个强大补充。如果你不熟悉，你可以把它们看作是部分定义的类。但是，真正的力量在于它们的可扩展性。

使用抽象类，您可以定义子类将继承的公共功能。但是，您也可以创建契约，子类*必须满足这些契约才能被实例化。这一切都要追溯到面向对象的继承概念，以及在父类中对公共功能进行分组。*

最重要的是，这个特性有助于提高代码的可重用性(并且再次清楚地传达了开发人员的意图)。为了在普通 JavaScript 中实现相同的功能，您必须检查父构造函数中的每个方法，或者实现另一个聪明的解决方案——这两种方法都可能需要大量的注释来解释。

TypeScript 还增加了对接口的支持，您可能会比抽象类使用更多的接口。它们是您学习使用 TypeScript 的第一批东西之一，因为它们能够代替`type`定义。在比较抽象类和接口时，要记住的最重要的事情是:*接口只定义子类必须实现的方法和属性，而抽象类也可以包括具体的实现逻辑(读:包含代码的方法)。*如果这还不清楚，请查看下面的代码，查看两者的示例:

![](img/8be2b060983fbeb35db9feeffe92a655.png)

```
Hi! My name is Home and I'm going 1028.99 mph!
```

# 4.名称空间

我争论过是否在本文中包含名称空间，因为尽管它们很强大，但它们在 TypeScript 中的实现有点…笨拙。它们之所以强大，是因为它们可以将常见的逻辑归为一类。您可能会想“是的，但这就是 JS 模块的用途。”你说得对！然而，名称空间非常棒，因为它们可以跨越多个文件。

因此，当我说名称空间在 TypeScript 中很笨拙时，我指的是导入语法。您可能熟悉 CommonJS 和 AMD 模块导入语法，但是 TypeScript 不使用它们。相反，它推出了自己的语法，更像 HTML 脚本标签，而不是你在 Javascript 中见过的任何东西。我希望未来版本的 TypeScript 使用更容易接受的语法。但是现在，这就是我们所拥有的！

![](img/8be2b060983fbeb35db9feeffe92a655.png)

```
The galaxy is 13.772 billion years old, and contains 100 billion galaxies.
The answer is and always will be 42.
```

# 5.装修工

装饰者是我最喜欢的 TypeScript 新增功能之一。然而，我应该在使用它们之前提醒你，因为它仍然是一个实验性的特性。而且，我的意思是*非常具有实验性——事实上，你必须用`experimentalDecorators`标志明确地启用它。*

自 2017 年 4 月以来，Decorators 一直是 Javascript 的拟议补充，但他们目前正处于批准过程的第二阶段——这意味着尽管他们可能会成为官方规范，但最终的实现可能会发生变化。你可以在这里查看当前的提议状态: [tc39 / proposal-decorators](https://github.com/tc39/proposal-decorators) 。

装饰器只是一种用函数名注释代码的方式，以便扩展其功能或提供元数据。这允许你写清楚和简洁的抽象。

您可以在 TypeScript 中修饰以下内容:

*   类别定义
*   性能
*   方法
*   访问器(getter/setter)
*   因素

如果我描述每种类型的装饰器，这篇文章会变得很长，所以我将把重点放在类定义上。当您想从多个类中抽象出一些公共的构造函数级逻辑时，类定义装饰器非常有用。您最常看到的是在包中使用它们来提供一种简单的方法来扩展您自己的类。

下面的例子展示了一个名为`Frozen`的装饰函数，它冻结了类的构造函数及其原型。正如您将在下面看到的，所有的类定义装饰器都接受构造函数作为参数。

第一个文件是一个示例`tsconfig.json`，它通过将`experimentalDecorators`设置为 true 来启用装饰器。

![](img/8be2b060983fbeb35db9feeffe92a655.png)

```
⛔️️ *no output*(The class has been frozen and therefore cannot be modified)
```

就是这样！我希望这个快速进入 TypeScript 的旅程已经激发了您的兴趣。如果你想了解更多关于 TypeScript 的知识或者开始你自己的项目，我建议你去官方的 [TypeScript 网站](https://www.typescriptlang.org/)看看。

如果你喜欢这篇文章，请考虑留下一些掌声，这样我就知道这种类型的内容是有价值的！👏