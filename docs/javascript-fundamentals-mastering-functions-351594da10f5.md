# JavaScript 基础:掌握函数

> 原文：<https://itnext.io/javascript-fundamentals-mastering-functions-351594da10f5?source=collection_archive---------0----------------------->

![](img/7057a74ed64b3f931bfdc23c46b6c87b.png)

函数可以被认为是 JavaScript 程序的核心构件之一。函数只是一组旨在执行特定任务的语句，只要被调用(或“调用”)就会被执行。

在这篇文章中，我们将看看定义函数，函数表达式，调用函数，作用域，嵌套，闭包，arguments 对象，函数参数&箭头函数！

🤓*想了解最新的 web 开发吗？*🚀*想要将最新消息直接发送到您的收件箱吗？
🎉加入一个不断壮大的设计师&开发者社区！*

**在这里订阅我的简讯→**[**https://ease out . EO . page**](https://easeout.eo.page/)

# 定义函数

函数定义(也称为函数声明或函数语句)总是以关键字`function` 开始。接下来是我们函数的*名*，任何*参数*括在括号里，然后是我们的 JavaScript *语句*(括在`{ }`里)，这些语句在我们的函数被调用时执行。

让我们看一个例子:

```
function multiply(a, b) {
  return a * b;
}
```

我们的函数`multiply`有两个参数(`a`和`b`)。花括号中包含一条语句，该语句返回第一个参数`a`乘以第二个参数`b`的结果。

# 函数表达式

还有另一种定义函数的方法，称为函数表达式。这些类型的函数实际上可以是匿名的。 他们不需要给名字。

例如，我们之前的`multiply`函数，也可以这样定义:

```
let multiply = function(a, b) { return a * b; };
let x = multiply(2,2); // x returns 4
```

函数表达式的一个典型用例可能是将一个函数作为参数传递给另一个函数。

我们也可以根据条件定义一个函数。例如，以下函数仅在`num`等于 1 时定义`addItem`:

```
let addItem;
if (num === 1) {
  addItem = function(shoppingList) {
    shoppingList.item = 'Apples';
  }
}
```

# 调用函数

当我们定义一个函数时，我们所做的就是给它一个名字&指定函数执行时做什么。所以要真正运行这个函数，我们需要在程序中调用它！

要调用前面的函数`multiply`，我们可以如下调用它:

```
multiply(2,2);
```

这里我们调用我们的函数，该函数被设置为接收两个参数，参数分别为`2`和`2`。该函数执行并运行其语句，返回`4` (2 乘以 2)的结果。

函数在被调用时需要在作用域内，但是函数声明可以被提升(出现在代码中调用的下方)，就像这样:

```
console.log(multiply(2,2));
/* ... */
function multiply(a,b) { return a * b; }
```

函数的作用域是声明它的函数，或者是整个程序，如果它是在全局级别声明的话。

*注意:*这只适用于函数声明。不是函数表达式！

# 功能范围

当变量在函数内部定义时，不能在函数外部的任何地方访问它们。因为这超出了范围。然而，一个函数能够访问在它被定义的范围内定义的所有变量和函数。所以在全局范围内定义的函数可以访问全局范围内定义的所有变量。在另一个函数中定义的函数也可以访问在其父函数中定义的所有变量以及父函数可以访问的任何其他变量。

让我们看一个例子..

```
// These variables are defined globally (global scope)let a = 10,
    b = 3,
    name = 'Bruce';// This function is defined globally function multiply() {
  return a * b;
}multiply(); // returns 30// Working with a nested functionfunction multiplyAssign() {
  let a = 20,
      b = 6;

  function add() {
    return name + ‘ received’ + (a * b);
  }

  return add();
}multiplyAssign(); // returns "Bruce received 120"
```

# 嵌套函数和闭包

所以你可以在一个函数中嵌套一个函数！嵌套(内部)函数是其包含(外部)函数的私有函数。它还形成一个*闭合*。闭包是一个表达式(通常是一个函数)，它可以有自由变量以及绑定这些变量的环境(它“关闭”表达式)。

由于嵌套函数是一个闭包，它能够“继承”其包含函数的参数和变量。

如下所示:

```
function addSquares(a, b) {
  function square(x) {
    return x * x;
  }
  return square(a) + square(b);
}
a = addSquares(2, 3); // returns 13
b = addSquares(3, 4); // returns 25
c = addSquares(4, 5); // returns 41
```

因为内部函数形成了一个闭包，所以您可以调用外部函数并为外部 ***和*** 内部函数指定参数！

# 关闭

正如我们现在所知道的，我们可以嵌套我们的函数，JavaScript 将给予内部函数对外部函数中定义的所有变量和函数的完全访问权(以及它可以访问的所有变量和函数)。

但是，外部函数不能访问内部函数中定义的变量和函数！当内部函数对外部函数之外的任何作用域可用时，就会创建一个*闭包*。

让我们看一个例子:

```
// The outer function defines a "name" variablelet user = function(name) {   
  let getName = function() {
    return name;                 // The inner function can access the "name" variable of the  outer function
  }// Return the inner function, and expose it to outer scopesreturn getName;            
}
makeAdmin = user('Steve Stevesson');

makeAdmin(); // returns "Steve Stevesson"
```

# 参数对象

任何函数的参数都保存在一个类似于*数组的*对象中。在函数中，可以按如下方式处理传递给它的参数:

```
arguments[i]
```

这里`i`是参数的第一个数字，从零开始。因此，传递给函数的第一个参数是`arguments[0]`。参数的总数将是`arguments.length`。

使用`arguments`对象，您可以调用比正式声明接受的参数更多的函数。如果您事先不知道将有多少个参数传递给函数，这通常很有用。您可以使用`arguments.length`来确定实际传递给函数的参数数量，然后使用`arguments`对象来访问每个参数。

例如，考虑一个连接几个字符串的函数。该函数唯一的形式参数是一个字符串，它指定了分隔要连接的项的字符。该函数定义如下:

```
function myConcat(separator) {
   let result = ''; // initialize list
   let i;
   // loop through arguments
   for (i = 1; i < arguments.length; i++) {
      result += arguments[i] + separator;
   }
   return result;
}
```

您可以向该函数传递任意数量的参数，它会将每个参数连接成一个字符串“list”:

```
myConcat(', ', 'fred', 'wilma', 'barney', 'betty');
// returns "fred, wilma, barney, betty, "myConcat('; ', 'marge', 'homer', 'lisa', 'bart', 'maggie');
// returns "marge; homer; lisa; bart; maggie; "myConcat('. ', 'jerry', 'kramer', 'elaine', 'george', 'newman');
// returns "jerry. kramer. elaine. george. newman. "
```

*注意:*`arguments`变量是“类数组”，但不是数组。它有一个编号索引和一个`length`属性。但是，它不具备所有的数组操作方法。

# 功能参数

有两种功能参数:*默认*参数和*剩余*参数。现在让我们来看一看每一个..

## 默认参数

在 JavaScript 中，函数的参数将默认为`undefined`。但是，在某些情况下，设置不同的默认值可能会很有用。这就是默认参数有用的地方。

这实际上很容易实现:

```
function multiply(a, b = 1) {
  return a * b;
}multiply(10); // 10
```

你可以简单地将`1`作为`b`在函数头中的默认值。

## 休息参数

rest 参数语法允许我们将不定数量的参数表示为一个数组。

在这个例子中，我们使用 rest 参数从第二个到最后收集参数。然后我们把它们乘以第一个。这个例子使用了一个箭头函数，我们接下来会看到它！

```
function multiply(multiplier, ...theArgs) {
  return theArgs.map(x => multiplier * x);
}let arr = multiply(2, 1, 2, 3);
console.log(arr); // [2, 4, 6]
```

# 箭头功能

与函数表达式相比，箭头函数的语法要短得多。例如，下面是一个常规函数:

```
function funcName(params) {
  return params + 10;
}funcName(2); // 12
```

这里是同样的功能，表示为一个*箭头功能*:

```
let funcName = (params) => params + 10funcName(2); // 12
```

只用一行代码就实现了相同的功能！相当整洁！

如果我们没有参数，我们将箭头函数表示为:

```
() => { statements }
```

当我们只有一个参数时，左括号是可选的:

```
parameters => { statements }
```

最后，如果要返回一个表达式，可以去掉括号:

```
parameters => expression// is the same as:function (parameters){
  return expression;
}
```

*注意:*与常规函数不同，箭头函数不绑定`this`。相反，`this`必然会保留其原本上下文中的含义。在下一篇文章中，我们将进一步了解“this”关键字！

# 预定义功能

非常值得注意的是，JavaScript 有许多内置函数！而且它们可能会节省你很多时间，特别是对于一些普通的任务。参见:
[https://www . tutorialspoint . com/JavaScript/JavaScript _ builtin _ functions . htm](https://www.tutorialspoint.com/javascript/javascript_builtin_functions.htm)

***你准备好让你的 JavaScript 技能更上一层楼了吗？*** *今天就开始用我的新电子书吧！无论你是想学习你的第一行代码，还是想扩展你的知识面并真正学习基础知识..*[*《JavaScript 掌握完全指南》*](https://gum.co/mastering-javascript) *带你从零到英雄！*

![](img/dde515044536421c6c999650977f80c4.png)

*现已上市！👉*[https://gum.co/mastering-javascript](https://gum.co/mastering-javascript)

# 结论

今天就到这里吧！我们已经学习了定义函数、函数表达式、调用函数、作用域、嵌套、闭包、arguments 对象、函数参数和 arrow 函数！

我希望这篇文章对你有用！你可以在 Medium 上[关注我](https://medium.com/@timothyrobards)。我也在[推特](https://twitter.com/easeoutco)上。欢迎在下面的评论中留下任何问题。我很乐意帮忙！

# 关于我的一点点..

嘿，我是提姆！👋我是一名开发人员、技术作家和作家。如果你想看我所有的教程，可以在我的个人博客上找到。

我目前正致力于建立我的自由职业者完整指南。坏消息是它还不可用！但是如果是你感兴趣的东西，你可以[注册，当它可用的时候会通知你👍](https://easeout.eo.page/news)

感谢阅读🎉