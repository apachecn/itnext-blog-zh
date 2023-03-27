# JavaScript 基础:数据类型

> 原文：<https://itnext.io/javascript-fundamentals-data-types-29ba4a49470d?source=collection_archive---------9----------------------->

![](img/21fccb935b9bde9d8af1adb6cddbf193.png)

让我们来看看 JavaScript 中的数据类型基础！首先，我们将了解每一种不同的数据类型，然后再看我们可以用来转换数据的方法(例如，我们如何将值转换成数字、字符串和布尔值)。

到本文结束时，您将对使用许多主要数据类型有更多的了解。这是一项需要掌握的基本技能。

🤓*想了解最新的 web 开发吗？*
🚀*想要将最新消息直接发送到您的收件箱吗？
🎉加入一个不断壮大的设计师&开发者社区！*

**在这里订阅我的简讯→**[**https://ease out . EO . page**](https://easeout.eo.page/)

# 什么是数据类型？

数据类型是特定数据类型的分类。我们有数字、布尔值(真或假)、字符串(用引号`‘’`或`“”`括起来的字符序列)和更复杂的数据类型，称为数组和对象(我们稍后将讨论这些)。

```
let a = 1;           // a is a number
let a = "Avocado";   // a is a string
let a = true;        // a is a Boolean
let a;               // a is undefined
```

例如，`a`，已经使用`let`关键字定义为变量。我们可以为它指定任何数据类型，甚至可以通过将其留空来初始化它。

**为什么有关系？**

当在变量中存储数据时，我们知道它的类型是很重要的，因为这决定了我们可以用它做什么！例如，你可以添加数字`1 + 1 = 2`，这很好。然而，如果您试图将数据类型为 string `“1” + “1” = 11`的数字相加，您的结果将是 1 *和* 1，而不是您预期的总和等于 2。现在让我们详细看看每种类型。

## 数字

JavaScript 中的数字可以用不带小数的*或*来写，比如:

```
let a = 1;
let b = 1.1;
```

它们可以用`e`指数来缩写，例如:

```
let million = 1000000;// or..let million = 1**e**6;
```

6 是 0 的数量，在这种情况下等于一百万。

在处理数字时，我们经常会遇到几个特殊值:`Infinity`、`-Infinity`和`NaN.`

如果您试图将一个数除以 0，例如:

```
3/0    // Infinity
```

结果将是无穷大。因为 JavaScript 计算的结果超出了其最大可能数量`9007199254740992`。相反的情况会产生:

```
-3/0   // -Infinity
```

`NaN`的值代表“不是一个数字”，意味着该值不被认为是一个数字。这将在非法表达式中生成，例如:

```
let a = 100 / "Greg";     // a will be NaN
```

当然，你不能用一个字符串来除一个数！

然而，在某些情况下，JavaScript 足够智能来转换您的数据类型，例如:

```
let a = 100 / "2"         // a will be 50
```

在这种情况下，JavaScript 将使用*类型强制*将您的“2”字符串视为一个数字。

## 用线串

如前所述，字符串是存在于单引号或双引号中的字符序列:

```
let foodChoice = 'Crunchy cheddar jalapeno cheetos';
let foodChoice = "Crunchy cheddar jalapeno cheetos";
```

字符串也不限于字母，数字和符号也是可以接受的。引号定义了我们的字符串数据类型。

至于使用单引号还是双引号，这实际上取决于个人偏好，一致性是代码中最重要的！

```
let eater = “Bruce”;
let foodChoice = "Crunchy cheddar jalapeno cheetos";alert(eater + " loves " + foodChoice + "!")// Bruce loves Crunchy cheddar jalapeno cheetos!
```

## 布尔运算

我们使用关键字`true`和`false`将变量设置为布尔数据类型。

```
let a = true;
let b = false;
```

在执行数学运算、确定表达式是真还是假时，布尔值尤其有用，例如:

```
10 > 5     // true, 10 is greater than 5
5 > 10     // false, 5 is not greater than 10
5 < 10     // true, 5 is less than 10
5 = 5      // true, 5 equals 5
```

如果我们将表达式赋给一个变量，比如:

```
let a = 10 > 5;   // true
```

当然，我们的变量`a`的值为 true。

当我们需要基于对真或假的评估来执行操作时，在程序中使用布尔。例如，收到的登录凭证评估为真吗？授予访问✔️.的权限还是它们是假的？拒绝访问❌.

## 数组

数组是一种稍微复杂一点的数据类型，但是它们确实很容易掌握！数组是一种用单个变量保存多个值的方式。例如:

```
let colors = ["red", "green", "blue", "yellow"]
```

如上例所示，使用方括号`[]`定义数组。我们给变量`colors`分配了一个数组，里面包含了我们的红色、绿色、蓝色和黄色的**元素**。

对我们的`colors`变量的调用将输出我们的整个`[“red”, “green”, “blue”, “yellow”]`数组。

数组的真正强大之处在于它们的内容可以被迭代，我们可以在数组变量中调用一个单独的项。为此，我们使用方括号内的索引号:

```
colors[0]     // red
colors[1]     // green
colors[2]     // blue
colors[3]     // yellow
colors[4]     // undefined
```

*注意:*我们的第一个数组元素的索引位置总是为 0。所以记得从 0 开始计数，而不是从 1 开始！

数组具有很大的灵活性，它们可以添加、删除和更改元素。现在让我们来看看最终的数据类型..

## 目标

对象数据类型通常用于保存大量相关数据。对象数据值存储在键/值对中，这些键/值对构成了存储和访问数据的逻辑方式，例如使用花括号`{}`:

```
let user = {firstName:"Jane", lastName:"Doe", age:34, location:"Vancouver"};
```

为了清楚起见，我们将它写成多行:

```
let user = {
    firstName: "Jane",
    lastName: "Doe",
    age: 34,
    location: "Vancouver"
};
```

我们上面的例子包含四个属性`firstName`、`lastName`、`age`和`location`。我们的属性可以是任何数据类型，可以使用 *objectName.property* 进行访问，如下所示:

```
user.firstName     // Jane
user.lastName      // Doe
user.age           // 34
user.location      // Vancouver
```

这就结束了我们对数据类型的第一次观察！现在让我们看看如何执行数据类型转换。

# 转换数据类型

正如我们现在看到的，数据类型是特定类型数据的分类。正是这种类型决定了可以对数据执行哪些操作。当我们编写程序时，我们经常需要转换我们的数据类型来执行任务。

JavaScript 足够智能，可以为我们转换一些值，这就是所谓的类型强制:

```
"15" - "5"    // 10
"15" / "3"    // 5
```

当我们的字符串对数字求值时，上面的内容将会起作用。然而，

```
"5" + "5"   // 55
```

如果我们使用`+`操作符，字符串将会连接起来！因此，如果我们试图在这种情况下依赖类型强制，我们会看到意想不到的结果。

因此，在编写自己的代码时，我们应该努力自己转换数据类型，从而减少潜在的错误。现在让我们来看看我们该如何去做！

## 将值转换为字符串

使用`String()`方法，我们可以显式地将值转换成字符串。

例如，让我们将*数字* 100 转换成*字符串文字*“100”。

```
String(100);     // "100"
```

让我们将一个*布尔值* false 转换成一个*字符串文字*“false”。

```
String(false);   // "false"
```

在变量中，我们可以将值转换成字符串，如下所示:

```
let name = 5000;       // 5000
name = **String(name);  ** // "5000"
```

我们可以使用`typeof`检查任何值的数据类型，例如:

```
**typeof** name;    // string
```

或者，我们可以更简洁地这样做:

```
let name = 5000;
name.toString();    // "5000"(5000).toString();  // also returns "5000"
```

不管我们选择哪种方法，通过使用`String()`或`n.toString()`，我们都可以显式地将数据转换成字符串值。

## 将值转换为数字

当需要将数值转换成数字时，我们以类似的方式使用`Number()`方法:

```
Number("2000");   // 2000
```

我们已经把字符串“2000”变成了数字 2000。

我们也可以转换一个布尔值:

```
Number(true);    // 1
Number(false);   // 0
```

但是需要注意的是，如果字符串中有任何字符或空格，我们就不能将数据类型转换成数字！如果尝试这样做，将返回一个`NaN`(不是一个数字)。

```
Number("one");      // NaN
Number("1,000");    // NaN
Number("1 1");      // NaN
Number("01-01-19"); // NaN
```

## 将值转换为布尔值

我们使用`Boolean()`将字符串和数字转换成布尔值。

如果**存在任何**值，它将被转换为`true`:

```
Boolean(100);     // true
Boolean("name");  // true
Boolean(" ");     // true, a space is considered a value
```

如果该值被认为是空的(`0`、an `empty string ("")`、`undefined`、`NaN`或`null`，它将返回 false:

```
Boolean(0);         // false
Boolean("");        // false, an empty string (with no space)
Boolean(undefined); // false
Boolean(NaN);       // false
Boolean(null);      // false
```

将数字和字符串转换成布尔值是开始将逻辑引入我们的编程的一种强有力的方法。例如，如果 Boolean 返回 false，我们可以检测输入表单上缺失的必填字段。

***你准备好让你的 JavaScript 技能更上一层楼了吗？今天就开始用我的新电子书吧！无论你是想学习你的第一行代码，还是想扩展你的知识面并真正学习基础知识..*[*《JavaScript 掌握完全指南》*](https://gum.co/mastering-javascript) *带你从零到英雄！***

![](img/dde515044536421c6c999650977f80c4.png)

*现已上市！👉*[https://gum.co/mastering-javascript](https://gum.co/mastering-javascript)

# 结论

理解和处理数据类型是使用 JavaScript 时需要掌握的一项基本技能。在本文中，我们研究了数据类型以及如何使用它们，既使用了类型强制，也隐式地执行了我们自己的类型转换。

我希望这篇文章对你有用！你可以[跟着我](https://medium.com/@timothyrobards)上媒。我也在[推特](https://twitter.com/easeoutco)上。欢迎在下面的评论中留下任何问题。我很乐意帮忙！

本系列的下一篇文章: [JavaScript 基础知识:使用字符串](/javascript-fundamentals-working-with-strings-3b14a9e08134)

# 关于我的一点点..

嘿，我是提姆！👋我是一名开发人员、技术作家和作家。如果你想看我所有的教程，可以在我的个人博客上找到。

我目前正在编写我的[自由职业者完整指南](http://www.easeout.co/freelance)。坏消息是它还不可用！但是如果这是你可能感兴趣的东西，你可以[注册，当它可用时会通知你](https://easeout.eo.page/news)👍

感谢阅读🎉