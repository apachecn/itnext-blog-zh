# 类和继承:JavaScript ES6 特性系列(第 8 部分)

> 原文：<https://itnext.io/classes-and-inheritance-javascript-es6-feature-series-part-8-4a81fa3adf0f?source=collection_archive---------0----------------------->

# 原型还在…引擎盖下

![](img/38cc522e25fc8586140b88cc63b47849.png)

埃里克·麦克林在 [Unsplash](https://unsplash.com/s/photos/open-hood-of-car?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上的照片

# 介绍

这些作品背后的灵感很简单:仍然有很多开发人员对 JavaScript 感到有点困惑——或者至少在行为上有些古怪。

尽管如此，根据维基百科[的数据，JavaScript 为 1000 万个最受欢迎的网页中的 95%提供了动力。](https://en.wikipedia.org/wiki/JavaScript)

由于它的使用量和受欢迎程度不断增加，我想提供一些我经常使用的 ES6+特性的文章和例子，供其他开发人员参考。

我们的目标是让这些文章简短、深入地解释对该语言的各种改进，我希望它们能启发您使用 JS 编写一些真正酷的东西。谁知道呢？在这个过程中，你甚至可能会学到一些新东西。😄

> 在我的第八篇博文中，我将深入探讨 JavaScript 类和基于类的继承，这是处理 JavaScript 现有的基于原型的继承系统的更简洁、更直接的方法。

# 在有类之前，就有了原型

在谈到 JavaScript 类之前，我必须先谈谈[原型](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Objects/Object_prototypes)——JavaScript 对象相互继承特性的原始机制。

深入研究 prototype 对象和基于原型的继承如何工作已经超出了这篇博客的范围，但是我会给你一个简要的概述。

JavaScript 中的每个对象都有一个**原型对象**，它作为一个模板继承方法和属性。同一个原型对象也可能有一个它继承的原型对象，等等。这有时被称为**原型链**。

当在对象上调用属性或方法时，浏览器首先检查实际对象上是否有该方法，如果没有，则通过原型对象检查该方法是否可用。如果该方法没有在对象的个人构造器上定义(它的原型对象来自那里)，浏览器检查另一个级别，看看*构造器的*原型对象是否有可用的方法。浏览器将一直这样做，直到到达以`null`为原型的顶部对象。

嗯…？

看看这个例子，这应该有助于说明我所说的。

*传统的基于原型的继承的例子*

```
function Superhero(superName, realName, powers){
  this.superName = superName,
  this.realName = realName,
  this.powers = powers
}const wonderWoman = new Superhero('Wonder Woman', 'Diana Prince', 'Strength and flight');console.log(wonderWoman); /* Superhero {
  superName: 'Wonder Woman',
  realName: 'Diana Prince',
  powers: 'Strength and flight' } */Superhero.prototype.equipment = 'Lasso of truth';console.log(wonderWoman.realName); // Diana Princeconsole.log(wonderWoman.equipment); // Lasso of truthconsole.log(wonderWoman.catchPhrase); // undefinedconsole.log(wonderWoman.hasOwnProperty('equipment')); // falseconsole.log(Superhero.hasOwnProperty('equipment')); // falseconsole.log(Superhero.prototype.hasOwnProperty('equipment')); // true
```

上面的例子是一个名为`Superhero`的构造函数，它定义了一个新的超级犯罪斗士。

我创造了`wonderWoman`，给了她`superName`、`realName`、`powers`的属性。然后，我给`Superhero`的原型对象添加了一个名为`equipment`的属性，值为`'Lasso of truth'`。

当我调用对象的`realName`属性时，在对象本身上发现了`"Diana Prince”`。当我调用`equipment`时，`"Lasso of truth”`是在对象构造器的原型上找到的。当我试图调用属性`catchPhrase`时，它返回为`undefined`，因为在原型链中不存在该属性。

我检查`'equipment'`的`wonderWoman.hasOwnProperty()`和`Superhero.hasOwnProperty()`是否都返回`false`，但是`Superhero.*prototype*.hasOwnProperty('equipment')`返回 true，这一行证明了这一点。

简而言之，这就是 JavaScript 基于原型的继承。请记住，当我们使用基于类的语法时，这是在后台发生的事情。

现在让我们开始新的 ES6 课程。

# 然后上课了

与其他面向对象编程(OOP)语言(如 Java)不同，Java 一直是基于类的，JavaScript 选择原型作为其处理继承的方式，但随着 ECMAScript 2015 的发布，类被引入到语言的语法中。

不过让我绝对明确地说:**类只是 JavaScript 现有的基于原型的继承**的语法糖衣。它们不是新的面向对象继承模型。

[类](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes)可能被定义为“特殊函数”，就像我在这里写的关于[的所有函数一样，它们可以用两种方式定义:类表达式和类声明。](/arrow-functions-javascript-es6-feature-series-pt-2-e8c31c823392)

## 类声明

定义类的第一种方法是使用**类声明**。要声明一个类，可以在类名中使用`class`关键字。

*类声明的例子*

```
class Book {
  constructor(title, author) {
    this.title = title;
    this.author = author;
  }
}const novel = new Book('Moby Dick', 'Herman Melville');console.log(novel); // Book { title: 'Moby Dick', author: 'Herman Melville' }
```

需要注意的一点是，与函数声明不同的是，**类声明没有被悬挂**。你必须首先声明你的类，然后访问它，否则你会得到一个`ReferenceError`抛出。

## 类别表达式

另一种定义类的方法是使用**类表达式**。类表达式可以是命名的，也可以是未命名的。如果一个类表达式是命名的，那么命名的类表达式的名字是局部的。

*未命名和已命名的类表达式示例*

```
// unnamed class expression
let Drama = class {
  constructor(title, author){
    this.title = title;
    this.author = author;
  }
};console.log(Drama.name); // Drama// named class expression
let Comedy = class Book2 {
  constructor(title, author){
    this.title = title;
    this.author = author;
  }
};console.log(Comedy.name); // Book2
```

好了，创建类的基础已经介绍完毕，是时候深入到本质细节了。

# 类体和方法定义

再次类似于函数，类的主体是包含在`{}`中的部分。这里是定义类成员如构造函数和方法的地方。

## 严格模式

你需要知道的第一件事是，类的主体总是在`strict mode`中执行。

这意味着，为了提高性能，这里编写的代码需要遵循更严格的语法，一些原本不引人注意的错误将被抛出，某些关键字被保留给 ECMAScript 的未来版本。

如果您一直在使用任何较新的 React、Angular 或 Vue 框架，那么您可能已经在默认的严格模式下进行开发了，所以这不会对您的开发方式造成太大的改变。

## 构造函数

在类的剖析中，下一步可能是构造函数。`constructor`方法是创建和初始化用`class`创建的对象的特殊方法。每个类中只能有一个名为“constructor”的特殊方法。如果一个类包含不止一次出现的`constructor`方法，将抛出一个`SyntaxError`。

构造函数也可以使用`super`关键字来调用超类的构造函数，但是我将在这篇文章的后面更详细地讨论这一点。

## 原型方法

原型方法也被称为[方法定义](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Method_definitions)，它们是分配给方法名称的函数的简写。这些往往构成了班级的主体。

*一个类的原型方法的例子*

```
class Book {
  constructor(title, author){
    this.title = title;
    this.author = author;
  } publicizeBook() {
    return `This book ${this.title} is written by renowned author ${this.author}.`;
  }
}const novel = new Book('Harry Potter', 'J.K. Rowling');console.log(novel.publicizeBook()); // This book Harry Potter is written by renowned author J.K. Rowling.console.log(Book.prototype.hasOwnProperty('publicizeBook')); // true
```

对于这个类`Book`，方法`publicizeBook()`被定义在它上面，所以任何用类`Book`创建的对象，也将自动拥有对它们可用的原型方法`publicizeBook()`。

并检查示例中的最后一行。我通过测试`Book`的原型对象`.hasOwnProperty('publicizeBook')`来检查`Book`类是否拥有`publicizeBook()`方法，它返回`true`。这表明它实际上是基于原型的继承发生在底层。

开始有意义了吗？

## 静态方法

当您使用`static`关键字时，它既可以应用于一个方法，也可以应用于一个类。静态方法在没有实例化它们的类的情况下被调用，并且**不能通过类实例被调用。这些类型的方法通常用于为应用程序创建实用函数。**

*一个类的静态方法的例子*

```
class Book {
  constructor(title, author){
    this.title = title;
    this.author = author;
  } static youMightLike(title, similarTitle) {
    return `If you like ${title}, you might also like ${similarTitle}.`
  }
}const novel = new Book('Moby Dick', 'Herman Melville');console.log(novel.youMightLike); // undefinedconsole.log(Book.youMightLike(novel.title, "A Midsummer Night's Dream")); // If you like Moby Dick, you might also like A Midsummer Night's Dream.
```

在这个例子中，如果您试图在实际的基于类的对象`novel`上调用静态方法`youMightLike()`，您只能得到一个`undefined`值。然而，如果你在类`Book`上用两个参数调用它，你将得到一个基于另一本书推荐一本书的响应。

## 当心用原型和静态方法装箱

当一个方法，不管是静态的还是非静态的，在没有分配给它的`this`值的情况下被调用时，在该方法中的`this`值将是`undefined`。这是因为类的主体总是在`strict mode`中运行，不管它是否被显式设置。

*类内未定义“this”的示例*

```
class Dog {
  eat() {
    return this;
  }

  static speak() {
    return this;
  }
}let shibaInu = new Dog();
console.log(shibaInu.eat()); // Dog {}let chowDown = shibaInu.eat;
console.log(chowDown); // undefinedconsole.log(Dog.speak.toString()); // Class Dog
let greet = Dog.speak;
console.log(greet); // undefined
```

如果用传统的基于函数的语法编写上面的类，那么方法调用中的自动装箱将基于初始的`this`值以非严格模式发生。如果初始值是`undefined`，`this`将被设置为全局对象。

*示例* `*this*` *如果没有分配函数*则取全局范围

```
function Dog() { };Dog.prototype.eat = function() {
  return this;
}Dog.speak = function() {
  return this;
}let husky = new Dog();
console.log(husky.eat()); // Dog {}let munch = husky.eat;
console.log(munch()); // global object (long list of available options)let bark = Dog.speak;
console.log(bark()); // global object again (long list of available options)
```

只是要意识到类和函数之间的这一点点差别，这样它就不会在几个小时内不知不觉地绊倒你。

## 实例属性

实例属性必须在类方法内部定义。

*类*中定义的实例属性示例

```
class Cat {
  constructor(eats, sleeps){
    this.eats = eats;
    this.sleeps = sleeps;
  }

  knocksThingsOver(obj) {
    return `Woops, the cat just knocked ${obj} over...`
  }
}
```

虽然静态数据属性和原型数据属性必须在类体声明之外定义:

*定义静态数据的示例&原型数据属性*

```
Cat.plays = true;Cat.prototype.eatsCatnip = 'Goes nuts for it';
```

## 字段声明(仍处于试验阶段)

这些下一个特性:公共和私有字段声明仍处于试验阶段(第 3 阶段)，由 JavaScript 标准委员会在 TC39 上提出。但是它们仍然值得了解，因为它们可能很快就会成为标准语法。

到目前为止，浏览器中的原生支持是有限的，但是这个特性可以通过一个构建步骤用于像 Babel 这样的系统，这要归功于您的 Webpack 配置。

**公开声明**

使用 JavaScript 字段声明语法，前面的`Cat`示例可以写成这样。

*类中公共声明的例子*

```
class Cat {
  eats = 'mice';
  sleeps; constructor(eats, sleeps){
    this.eats = eats;
    this.sleeps = sleeps;
  } knocksThingsOver(obj) {
    return `Woops, the cat just knocked ${obj} over...`
  }
}
```

通过预先声明字段，类定义变得更加自文档化，并且字段总是存在的。同样酷的是，字段可以声明有或没有默认值，就像函数中的默认值一样，我在这里写了关于[的内容。](/default-function-parameter-values-javascript-es6-feature-series-pt-3-bd8392a88a12)

**私下声明**

私有声明与公共声明在语法上没有太大的不同。但它们在用法和执行上略有不同。

私有字段只能在字段声明中预先声明。它们不能像普通属性那样在以后通过赋值来创建。

从类外部引用私有字段也是错误的；它们只能在类体内读取或写入。通过定义在类外部不可见的东西，你可以确保你的类的用户不能依赖于内部，这可能会改变版本。

*类中私有声明的例子*

```
class Cat {
  #eats = 'mice';
  #sleeps;constructor(eats, sleeps){
    this.#eats = eats;
    this.#sleeps = sleeps;
  }knocksThingsOver(obj) {
    return `Woops, the cat just knocked ${obj} over...`
  }
}
```

就像我之前说的，还没有广泛使用，我可能不会把它投入生产，但在未来的 ES 版本中要注意这些。

# 带有“扩展”的子类

在*类声明*或*类表达式*中使用`extends`关键字来创建一个类作为另一个类的子类。

*将一个类扩展为子类的例子*

```
class Animal {
  constructor(name) {
    this.name = name;
  } speak() {
    console.log(`${this.name} makes a noise`);
  }
}class Horse extends Animal {
  constructor(name) {
    super(name);
  } speak() {
    console.log(`${this.name} whinnies.`)
  }
}let thoroughbred = new Horse('Seabiscuit');
console.log(thoroughbred.speak()); // Seabiscuit whinnies.
```

类`Horse`扩展了另一个类`Animal`，以接受它的方法，由于`Horse`也有自己的构造函数，它需要在引用`this`之前调用`super()`。

如果你曾经使用过 React 的基于类的组件，那么`extends`的语法非常类似于 React 的基于类的组件。

*React 基于类的组件语法*

```
import React, { Component } from 'react';class Nav extends Component {
  // ...do something
}
```

同样值得注意的是，当父类和子类的方法命名相似时，子类的方法在调用时优先于父类的方法。

# 使用“Super”的超级类调用

现在我们回到上面提到过几次的那个`super`关键词。`super`关键字用于调用超类的相应方法。与基于原型的继承相比，这是一个明显的优势，因为从技术上来说，父类和子类的两个方法都可以通过使用`super`而不是一个或另一个来调用。

*从子类访问父类方法的例子*

```
class Animal {
  constructor(name) {
    this.name = name;
  } speak() {
    console.log(`${this.name} makes a noise`);
  }
}class Hippo extends Animal {
  speak() {
    super.speak();
    console.log(`${this.name} roars.`);
  }
}let hippo = new Hippo('Bertha');
console.log(hippo.speak()); // Bertha makes a noise.
// Bertha roars.
```

在这个例子中，`Hippo`的`speak()`方法实际上调用了`Animal`的`speak()`方法以及它自己唯一的方法，只需在`Hippo`的`speak()`中使用`super.speak()`即可。

这就是`super`的全部内容，一旦展示了一些例子，事情就没那么复杂了。对吗？

这是用 JavaScript 开发时，你需要知道的关于类的最重要的部分。干得好，坚持到了这篇文章的结尾！😃

# 结论

类在其他编程语言中已经存在了几十年，JavaScript 最终(在某种程度上)随着 ES 2015 的发布而出现。尽管 JavaScript 中的类在语法上看起来和其他 OOP 语言一样，但它仍然是真正基于原型的继承。

对于我们所有熟悉传统原型对象的人来说，这种思考方法和属性的新方式可能需要一点时间来适应，但是类提供了一些非常棒的改进，使我们的开发生活变得更加容易，我希望您能亲自尝试一下。

我这个系列的目的是深入解释您每天使用的 ES6 语法，以便您可以使用它们来获得最大的影响并避免常见的陷阱。

过几周再来看看，我会写更多关于 JavaScript 和 ES6 或其他与 web 开发相关的东西，所以请关注我，这样你就不会错过了。

感谢您的阅读，我希望您能在 JavaScript 中给 ES6 class-syntax 一个机会，让原型继承更容易。

如果你喜欢读这篇文章，你可能也会喜欢我在这个系列中的其他一些博客:

*   [内置模块导入导出:JavaScript ES6 特性系列(第七部分)](https://medium.com/better-programming/built-in-module-imports-and-exports-javascript-es6-feature-series-part-7-5f0864049e1f)
*   [增强的对象文字值速记:JavaScript ES6 特性系列(Pt 6)](/enhanced-object-literal-value-shorthand-javascript-es6-feature-series-pt-6-e00dfdc24f64)
*   [字符串模板文字:JavaScript ES6 特性系列(Pt 5)](https://medium.com/better-programming/string-template-literals-javascript-es6-feature-series-pt-5-a40e55a5485b)
*   [Spread & Rest 参数:JavaScript ES6 特性系列(Pt 4)](/spread-rest-parameters-javascript-es6-feature-series-pt-4-c9e9f0c0228f)
*   [默认函数参数值:JavaScript ES6 特性系列(Pt 3)](/default-function-parameter-values-javascript-es6-feature-series-pt-3-bd8392a88a12)
*   [箭头功能:JavaScript ES6 特性系列(第二部分)](/arrow-functions-javascript-es6-feature-series-pt-2-e8c31c823392)
*   [Var，Let & Const: JavaScript ES6 特性系列(Pt 1)](/var-let-const-javascript-es6-feature-series-pt-1-fa603567809e)

# 参考资料和更多资源

*   对象原型，MDN 文档:[https://developer . Mozilla . org/en-US/Docs/Learn/JavaScript/Objects/Object _ prototypes](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Objects/Object_prototypes)
*   类，MDN 文档:[https://developer . Mozilla . org/en-US/docs/Web/JavaScript/Reference/Classes](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes)