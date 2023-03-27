# JavaScript 基础:语法和结构

> 原文：<https://itnext.io/javascript-fundamentals-syntax-structure-5e9badd0cc4f?source=collection_archive---------2----------------------->

![](img/2909592fabd31ddc7d28bcb701ba5b1b.png)

和任何语言一样，编程语言是由一系列规则定义的。规则(或语法)是我们编写代码时遵循的，它构成了程序的逻辑结构。

> 这篇文章标志着我将撰写的关于学习 JavaScript 基础知识的系列文章的开始。如果你想继续追踪，请务必跟我来！

让我们直接进入 JavaScript 的构建模块。我们将会看到值(文字&变量)，骆驼大小写，unicode，分号，缩进，空白，注释，区分大小写，关键字，操作符和表达式！😅

通过花时间学习基础知识，我们将朝着构建更具功能性和可读性的代码前进！

🤓*想了解最新的 web 开发吗？*🚀*想要将最新消息直接发送到您的收件箱吗？
🎉加入一个不断壮大的设计师&开发者社区！*

**在这里订阅我的简讯→**[**https://ease out . EO . page**](https://easeout.eo.page/)

# JavaScript 值

在 JavaScript 中有两种类型的值:固定值(或**文字值**)和变量值(**变量**)。

## 文字

文字被定义为在我们的代码中编写的值，比如数字、字符串、布尔值(真或假)，以及对象和数组文字(现在还不要太担心这些)。一些例子包括:

```
10          // A number (can be a decimal eg. 10.5)
'Boat'      // A string (can be in double "" or single '' quotes)
true        // A boolean (true or false)
['a', 'b']                           // An array
{color: 'blue', shape: 'Circle'}     // An object
```

注意:我将在下一篇文章中详细讨论数据类型，敬请期待！

## 变量

变量是存储数据的命名值。在 JavaScript 中，我们用关键字`var`、`let`或`const`声明变量，并用等号`=`赋值。

例如，`key`被定义为一个变量。其被赋予值`abc123`:

```
let key;key = abc123;
```

什么时候用`var`？不要。它只应该在处理遗留代码时使用。这是 ES6 之前的旧语法。

什么时候用`let`？如果您的变量需要在程序中更新，请使用它(它可以被重新分配)。

什么时候用`const`？如果您的变量包含一个常量值，请使用它。它必须在申报时分配，并且不能重新分配。

# 骆驼箱

如果我们的变量名由一个以上的单词组成呢？例如，我们如何声明一个希望命名为“名字”的变量？

我们可以使用连字符吗？例如`first-name` 不，`-`是为 JavaScript 中的减法保留的。

**下划线呢？我们可以，但是这有可能会让我们的代码看起来混乱不堪。**

解决办法？**骆驼案**！如`firstName`。第一个单词是小写的，任何后续单词的第一个字母都是大写的。*这是社区内的惯例。*

注意:然而，用下划线大写命名你的`const`变量也是可以接受的，比如`const DEFAULT_PLAYBACK_SPEED = 1;`这样会让其他人清楚这个值是固定的。否则就坚持骆驼案！

# 统一码

JavaScript 使用 unicode 字符集。Unicode 涵盖了几乎所有的字符、标点符号和符号！检查[完整参考](https://unicode-table.com)。这很棒，因为我们可以用任何语言写我们的名字，我们甚至可以使用表情符号作为变量名(为什么不呢？🤷🏻‍♂️).

# 分号

JavaScript 程序由大量被称为语句的指令组成。比如:

```
// These are all examples of JavaScript statements:let a = 1000;a = b + c;const time = Date.now();
```

JavaScript 语句通常以分号`;`结尾。

然而，**分号并不总是强制性的！如果你不使用 JavaScript，它不会有任何问题。**

```
// Still perfectly valid!let a = 1000a = b + cconst time = Date.now()
```

但是，在某些情况下，它们是强制性的。例如当我们使用一个`for`循环时，就像这样:

```
for (i = 0**;** i < .length**;** i++) { 
  // code to execute
}
```

然而，当使用 block 语句时，分号是*而不是*包含在花括号之后，例如:

```
if (name == "Samantha") {
  // code
}                           // <- no ';'//or,function people(name) {
  // code
}                           // <- no ';'
```

但是，如果我们使用一个对象，例如:

```
const person = {
  firstName: "Samantha",
  lastName: "Doe",
  age: 30,
  eyeColor: "blue"
}**;** // the ';' is mandatory
```

那么就需要我们的`;`！

随着时间的推移，你会开始记住分号在哪里可以用，在哪里不可以用。在此之前，明智的做法是在所有语句的末尾使用它们(block 语句除外！)

此外，无论如何都要使用它们，这确实是一个常见的惯例，它被认为是一个很好的实践，因为它减少了出错的可能性。

*注意:一旦你真正开始使用 JavaScript，就开始使用像*[*ESLint*](https://eslint.org/)*这样的 linter。它会自动发现你的代码中的语法错误，使生活变得更加容易！*

# 刻痕

理论上，我们可以在一行上编写一个完整的 JavaScript 程序。然而，这几乎是不可能读取和维护的。这就是我们使用线条和缩进的原因。让我们以条件语句为例:

```
if (loginSuccessful === 1) {
  // code to run if true
} else {
  // code to run if false
}
```

在这里我们可以看到，块中的任何代码都是缩进的。在这种情况下，它是我们的注释代码`// code to run if true`，然后`// code to run if false.`你可以选择用几个空格(通常是 2 或 4 个)或者一个制表符来缩进你的行。完全是你的选择，主要是要一致！

如果我们正在嵌套我们的代码，我们应该像这样进一步缩进:

```
if (loginAttempts < 5){
  if (loginAttempts < 3){
    alert("< 3");
  } else {
    alert("between 3 and 5");
  }
} else {
  if (loginAttempts > 10){
    alert("> 10");
  } else {
    alert("between 5 and 10");
  }
}
```

通过应用缩进，你将拥有更干净、更易维护、更易读的代码！

# 空格

JavaScript 只要求关键字、名称和标识符之间有一个空格，否则任何空格都会被完全忽略。这意味着就语言而言，以下语句之间没有区别:

```
const visitedCities="Melbourne, "+"Montreal, "+"Marrakech";
const visitedCities = "Melbourne, " + "Montreal, " + "Marrakech";
```

我相信你会发现第二行可读性更强。另一个例子是:

```
let x=1*y;       
let x = 1 * y; 
```

同样，第二行更容易阅读和调试！因此，请随意以一种有意义的方式来分隔您的代码！因此，这也是空白的一种可接受的用法:

```
const cityName         = "Melbourne";
const cityPopulation   = 5000001;
const cityAirport      = "MEL";
```

# 评论

注释是不可执行的代码。它们对于解释程序中的一些代码很有用。还可以“注释掉”一段代码以防止执行——通常在测试一段替代代码时使用。

JavaScript 中有两种类型的注释:

```
// Comment goes here/* Comment goes here */
```

第一种语法是单行注释。评论放在`//`的右边

第二个是多行注释。注释放在星号之间`/* here */`

更长的多行注释:

```
/* This is 
a comment spanning
multiple lines */
```

# 标识符

在 JavaScript 中，变量、函数或属性的名称被称为标识符。标识符可以由字母、数字、`$`和`_`组成。不允许使用其他符号，并且它们不能以数字开头。

```
// Valid :)Name
name
NAME
_name
Name1
$name// Invalid :(1name
n@me
name!
```

# 区分大小写

JavaScript 是区分大小写的！名为`test`的标识符不同于`Test`。

下面将抛出一个错误:

```
function **test**() {
  alert("This is a test!");
}function showAlert() {
  **Test**();                     // error! test(); is correct
}
```

为了确保我们的代码是可读的，最好尝试改变我们的名字，这样就不会发现标识符看起来太相似。

# 保留字

JavaScript 中有许多单词不能用作标识符。这些词是语言保留的，因为它们有内置的功能。比如:

```
break, do, instanceof, typeof, case, else, new, var, catch, finally, return, void, continue, for, switch, while, debugger, function, this, with, default, if, throw, delete, in, try, class, enum, extends, super, const, export, import.
```

参见保留关键字的完整[列表。](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Lexical_grammar#Reserved_keywords_as_of_ECMAScript_2015)

你当然不需要记住这些话！如果您在指向某个变量时遇到任何奇怪的语法错误，您可以检查列表并更改名称。

# JavaScript 运算符

算术运算符`+ -` `*`和`/`主要用于在 JavaScript 中执行计算，例如:

```
(2 + 2) * 100
```

赋值运算符`=`用于给变量赋值:

```
let a, b, c;
a = 1;
b = 2;
c = 3;
```

# JavaScript 表达式

表达式是指我们将值、变量和运算符组合起来计算一个新值(称为评估)。比如:

```
10 * 10    // Evaluates to 100let x = 5
x * 10     // Evaluates to 50const firstName = "Samantha";
const lastName = "Doe";
firstName + " " + lastName;    // Evaluates to: Samantha Doe
```

***你准备好让你的 JavaScript 技能更上一层楼了吗？今天就开始使用我的新电子书吧！无论你是想学习你的第一行代码，还是想扩展你的知识面并真正学习基础知识..*[*《JavaScript 掌握完全指南》*](https://gum.co/mastering-javascript) *带你从零到英雄！***

![](img/dde515044536421c6c999650977f80c4.png)

*现已上市！👉*[https://gum.co/mastering-javascript](https://gum.co/mastering-javascript)

# 结论

我们走吧！本文旨在提供 JavaScript 基本语法和结构的概述。我们已经研究了许多常见的约定，但是，请记住，您可以有一定的灵活性——尤其是在具有自己特定标准的协作环境中工作时。

语法和结构对于我们程序的功能以及代码的可读性和可维护性都有重要的作用。

我希望这篇文章对你有用！你可以在媒体上跟随我。我也在[推特](https://twitter.com/easeoutco)上。欢迎在下面的评论中留下任何问题。我很乐意帮忙！

这个系列的下一个.. [JavaScript 基础知识:数据类型](/javascript-fundamentals-data-types-29ba4a49470d)

# 关于我的一点点..

嘿，我是提姆！👋我是一名开发人员、技术作家和作家。如果你想看我所有的教程，可以在我的个人博客上找到。

我目前正在构建我的[自由职业者完整指南](http://www.easeout.co/freelance)。坏消息是它还不可用！但是如果你对它感兴趣，你可以[注册，当它可用时会通知你](https://easeout.eo.page/news)👍

感谢阅读🎉