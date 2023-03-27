# JavaScript 基础:理解 ES6

> 原文：<https://itnext.io/javascript-fundamentals-understanding-es6-583efbe43c4f?source=collection_archive---------4----------------------->

![](img/bf126436d3b0c7e57cdb092014dc0a74.png)

ES6(或 ECMAScript 2015)是 ECMAScript 编程语言的第 6 个版本。ECMAScript 是 2015 年发布的 JavaScript 的标准化版本，因此得名:ECMAScript 2015！

ES6 的一个很大的优点是，它允许我们以一种更现代和可读的方式编写代码。它还引入了许多新特性，比如:let/const、arrow 函数、新的“this”作用域、类、getter/setter、模块、模板文字、默认参数、扩展运算符、析构、for..循环和更多！

🤓*想与 web dev 保持同步吗？*
🚀*想把最新消息直接发到你的收件箱里吗？
🎉加入一个不断壮大的设计师&开发者社区！*

**在这里订阅我的简讯→**[**https://ease out . EO . page**](https://easeout.eo.page/)

现在让我们来看看这些新功能..

# `let`和`const`

ES6 引入了两个新的定义变量的关键字:`let`和`const`。虽然`var`仍然可以被使用，但是随着现代编码标准的推进，它已经不被鼓励了。让我们比较一下每种情况的一些用例..

`var`是**函数作用域**。

`let`是**块作用域**。

`let`变量的工作方式与`var`相似，但有一个关键区别。还要注意，任何用`let`声明的变量都将是**可变的**(它们的值可以改变)。

例如，如果我们用`let`在 for 循环中声明变量，在 if 或普通块中，它不能“转义”块，而`var`被提升到函数定义。

`const`就像`let`，但是**是不可变的**(算是吧！).

它主要用于在变量中存储一个不会被改变的值。`let`和`const`的主要区别在于，一旦你用`const`给一个变量赋值，你就不能再给它赋值了。

然而，仅仅因为用`const`声明了一个变量并不意味着它是不可变的，它只意味着值不能被重新赋值。

从技术上来说，引用本身是**不可变的**，但是变量持有的值不会变成不可变的。

在今天的 JavaScript 中，你很可能会看到很少甚至没有`var`声明，现在的惯例是只使用`let`和`const`。

# 箭头功能

箭头函数改变了大多数 JavaScript 代码的外观(和工作方式！).它们是编写函数的更简洁的语法，来自:

```
// Old Syntaxconst something = function something() {
  //...
}
```

可读性更强的:

```
// New Syntaxconst something = () => {
  //...
}
```

语法有两个新部分:

*   `const something = ()`
*   `=> {}`

第一部分是我们的变量声明和函数赋值，即`()`。它只是将变量设置为一个函数。

第二部分，即`=>,`是声明函数的主体。箭头后面是包含正文的花括号。

如果函数体是一行代码，箭头函数会更简单:

```
const something = () => doSomething()
```

此外，如果只有一个参数，可以这样写:

```
const something = param => doSomething(param)
```

具有旧语法的函数将像以前一样继续工作。然而，当我们向更现代和简洁的语法前进时，ES6 箭头函数是更好的选择！

## 关于范围的注释

对于箭头函数，`this`范围是从上下文中继承**的。**

记住对于常规函数，`this`总是指最近的函数。

我们遇到的问题现在已经解决了，你不会再发现自己在写`var that = this`了！

# 班级

类是面向对象编程(OOP)的核心。在 ES6 中，JavaScript 中引入了类。本质上，它们是内部工作的语法糖，但它们改变了我们构建许多程序的方式。在构造函数的帮助下，类可以用来创建新的对象(注意:每个类只能有一个构造函数)。

让我们看一个例子:

```
class Person {
  constructor(name) {
    this.name = name
  } hello() {
    return 'Hello, I'm ' + this.name + '.'
  }
}class Musician extends Person {
  hello() {
    return super.hello() + ' I am a musician.'
  }
}let prince = new Musician('Prince')
Prince.hello()// output: "Hello, I'm Prince. I am a musician."
```

类没有显式的类变量声明，但是您必须在其构造函数中初始化任何变量。

# Getters 和 setters

在一个类中，我们可以选择使用“getters”和“setters”。它们是 ES6 中引入的有用特性，在使用类时特别方便:

参见下面的例子，**没有**getter 和 setters:

```
class People {constructor(name) {
      this.name = name;
    }
    getName() {
      return this.name;
    }
    setName(name) {
      this.name = name;
    }
}let person = new People("Taylor Phinney");
console.log(person.getName());
person.setName("Fred");
console.log(person.getName());// Output:
Taylor Phinney
Fred
```

从上面我们可以看到，在我们的`People`类中有两个函数，用于获取和设置人名。

现在让我们看看如何使用 ES6 的 getters 和 setters 来实现这一点:

```
class People {constructor(name) {
      this.name = name;
    }
    get Name() {
      return this.name;
    }
    set Name(name) {
      this.name = name;
    }
}let person = new People("Taylor Phinney");
console.log(person.Name);
person.Name = "Fred";
console.log(person.Name);
```

现在你会看到在类`People`中有两个函数具有“get”和“set”属性。“get”属性用于获取变量的值,“set”属性用于设置变量的值。

并且调用`getName`函数不带括号。另外`setName`调用时不带括号。就像给变量赋值一样！

# 模块

模块对于在我们的项目中将代码组织成逻辑文件非常有用。在 ES6 之前，有许多我们在整个社区使用的模块解决方案，例如 RequireJS、CommonJS 和 AMD。

在 ES6 中，我们已经将这些标准化为统一的格式。

在 ES6 中，每个模块都在自己的文件中定义。一个模块中的函数和变量在另一个模块中根本不可见，除非它们已经被导出。

**导入**是通过`import ... from ...`构造完成的:

```
import * from 'mymodule'
import React from 'react'
import { React, Component } from 'react'
import React as MyLibrary from 'react'
```

当**导出**时，您可以使用`export`关键字编写模块并将任何内容导出到其他模块:

```
export var number = 10
export function bar() { /* ... */ }
```

要在不同的模块中使用导出的变量，可以使用`import`。

# 模板文字

模板文字允许我们以比过去更好的方式处理字符串。请参见下面创建字符串的示例:

```
const myString = `My string`
```

*注意:*我们使用反勾````而不是单引号或双引号。

模板文字提供了一种将表达式嵌入字符串的方法，通过使用`${variable}`语法有效地插入值:

```
const name = 'John'
const string = `Text ${name}` //Text John
```

在 ES6 之前，我们会使用`+`操作符来连接字符串，或者在字符串中使用变量。模板文字可读性更好！

当然，您也可以执行更复杂的表达式:

```
const string = `text ${1 + 2 + 3}`
const string2 = `text ${doSomething() ? 'x' : 'y' }`
```

我们也可以将字符串跨越多行，就像这样:

```
const string3 = `This
stringis neat!`
```

这比旧的多行语法更容易编写:

```
var str = 'One\n' +
'Two\n' +
'Three'
```

通过使用模板文字，我们可以极大地提高代码的可读性。

# 默认参数

对于 ES6，函数现在支持默认参数:

```
const myFunction = function(index = 0, testing = true) { /* ... */ }
myFunction()
```

默认参数允许您预先定义一个参数，这在许多情况下很有用。在 JavaScript 中，我们的函数参数默认为 undefined。我们现在能够设置不同的默认值。

在 ES6 之前，我们通过测试默认函数体中的参数值来定义默认参数，如果未定义，则分配一个值**。**

# 扩展运算符

Spread 操作符(`...`)是 ES6 提供的一个操作符，它允许我们很容易地从一个数组中获得一个参数列表。

让我们从一个数组示例开始:

```
const a = [1, 2, 3]
```

您可以创建一个新数组，如下所示:

```
const b = [...a, 4, 5, 6]
```

您也可以创建数组的副本:

```
const c = [...a]
```

这也适用于克隆对象:

```
const newObj = { ...oldObj }
```

使用字符串时，spread 运算符可以用字符串中的每个字符创建一个数组:

```
const hello = 'hello'
const imArrayed = [...hello] // ['h', 'e', 'l', 'l', 'o']
```

我们现在还可以以一种非常简单的方式使用数组作为函数参数:

```
const f = (arg1, arg2) => {}
const a = [1, 2]
f(...a)
```

# 析构赋值

JavaScript 中的析构本质上是将复杂的结构(比如对象或数组)分解成更简单的部分。通过析构赋值，我们将数组对象“解包”到许多变量中。

我们没有为对象的每个属性分别声明变量，而是将值放在花括号中来访问对象的任何属性。

例如，让我们从一个对象中提取一些值，并将它们放入命名变量中:

```
const person = {
  firstName: 'Bill',
  lastName: 'Billson',
  famous: false,
  age: 45,
}const {firstName: name, age} = person
```

`name`和`age`包含我们想要的值。

语法适用于数组:

```
const a = [1,2,3,4,5]
const [first, second] = a
```

该语句通过从数组`a`中获取索引为 **0，1，4** 的项目来创建三个新变量:

```
const [first, second, , , fifth] = a
```

# For-of 循环

在 ES6 之前，通常使用`forEach()`回路。他们很好地服务于目的，但没有提供打破的方法，就像`for`循环总是有。

ES6 给了我们`**for-of**` 循环，它结合了`forEach`的简洁和突破的能力:

```
// Iterate over a valuefor (const x of ['a', 'b', 'c']) {
  console.log(x);
}// Get the index, using `entries()`
for (const [i, x] of ['a', 'b', 'c'].entries()) {
  console.log(i, x);
}
```

***你准备好让你的 JavaScript 技能更上一层楼了吗？*** *今天就开始用我的新电子书吧！无论你是想学习你的第一行代码，还是想扩展你的知识面并真正学习基础知识..*[*《JavaScript 掌握完全指南》*](https://gum.co/mastering-javascript) *带你从零到英雄！*

![](img/dde515044536421c6c999650977f80c4.png)

*现已上市！👉*[https://gum.co/mastering-javascript](https://gum.co/mastering-javascript)

# 结论

我们走吧！我们已经看了很多 ES6 的新特性，比如:let/const、arrow 函数&它的‘this’作用域、类、getter/setter、模块、模板文字、默认参数、spread 运算符、析构和 for..循环的。

所以我们现在可以实现这种现代语法，并将我们的 JavaScript 带到下一个层次！

我希望这篇文章对你有用！你可以在媒体上跟我学。我也在[推特](https://twitter.com/easeoutco)上。欢迎在下面的评论中留下任何问题。我很乐意帮忙！

# 关于我的一点点..

嘿，我是提姆！👋我是一名开发人员、技术作家和作家。如果你想看我所有的教程，可以在我的个人博客上找到。

我目前正在构建我的[自由职业者完整指南](http://www.easeout.co/freelance)。坏消息是它还不可用！但是如果你对它感兴趣，你可以[注册，当它可用时会通知你](https://easeout.eo.page/news)👍

感谢阅读🎉