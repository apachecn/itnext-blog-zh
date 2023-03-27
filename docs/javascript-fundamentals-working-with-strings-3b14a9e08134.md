# JavaScript 基础:使用字符串

> 原文：<https://itnext.io/javascript-fundamentals-working-with-strings-3b14a9e08134?source=collection_archive---------6----------------------->

![](img/6f64f40484bf5a4ac7264c963839ee75.png)

JavaScript *字符串*用于存储和操作文本——它们可以由一系列字母、数字和/或符号组成。在这篇文章中，我们将首先看看我们如何创建字符串，连接字符串，我们将看到一些适用于字符串的基本语法规则。

然后，我们将继续讨论字符串操作，使用许多常见的方法，例如:`charAt()`、`indexOf()`、`slice()`、`length`、`toUpperCase()`、`toLowerCase()`、`split()`、`trim()`、`replace()`、`parseInt()`、`parseFloat()`和`Number()`。

> 🤓*想跟上网络发展的步伐吗？*
> 🚀想要最新的新闻直接发送到你的收件箱吗？
> 🎉加入一个不断壮大的设计师&开发者社区！
> 
> **在这里订阅我的简讯→**[**https://ease out . EO . page**](https://easeout.eo.page/)

# 创建字符串

有三种方法来编写字符串——用单引号`‘ ’`、双引号`“ ”`或反引号`` ``。让我们用每种技术给一个`myString`变量分配一些字符串:

```
let myString = **'**I am a string!**'**;
let myString = **"**I am a string!**"**;
let myString = **`**I am a string!**`**;
```

单引号和双引号在语法上没有区别，您可以根据自己的喜好自由选择——但是在项目中保持一致是个好主意！然而，反勾号(被称为*模板文字*)有一些超能力，我们将在本文后面看到。

当然，我们将字符串赋给变量的原因是，它们在我们的程序中更容易操作！

# 串联

我们使用*字符串连接*将两个或更多的字符串组合在一起，形成一个新的字符串。我们使用`+`操作符组合我们的字符串。

```
const firstName = 'Barry';
const secondName = 'Dingus';const concat = firstName + secondName;console.log(concat); // output: BarryDingus
```

这里，我们将存储在不同变量中的两个字符串合并成一个新的`concat`变量。然而，你会注意到我们在新变量的名字之间没有空格，因为字符串是首尾相连的。要添加空格，请使用一对引号`‘ ’`，如下所示:

```
const concat = firstName + **' '** + secondName;console.log(concat);// output: Barry Dingus
```

现在我们有了我们想要的输出！

## 与模板文字连接

当我们使用模板文字`` ``时，我们不再需要使用`+`操作符来连接。我们使用`${}`语法在字符串中包含表达式和变量。举个例子，

```
const fullName = 'Barry Dingus';
const location = 'Mullet Creek';const concat2 = `Welcome ${fullName}! We think ${location} is alright!`;console.log(concat2);// output: Welcome Barry Dingus! We think Mullet Creek is alright!
```

模板文字更容易读写。

# 语法规则

## 使用引号和撇号

知道引号`“`和撇号`‘`被 JavaScript 解释来标记字符串从哪里开始&结束。我们如何将它们包含在字符串中呢？例如:

```
const greeting = 'G'day Barry!';console.log(greeting);// Uncaught Syntax Error: Unexpected identifier
```

`’G**’**day Barry!'`中多出来的撇号就是问题。JavaScript 认为`‘G’`是字符串，并试图将其余部分解析为代码，从而产生错误。

在字符串中使用引号会导致相同的错误:

```
const quote = 'Barry said 'I lost my keys'';console.log(quote);// Uncaught Syntax Error: Unexpected identifier
```

我们有很多方法可以避免这些错误…

**改变你的语法**

使用相反的引号样式将字符串中的任何引号或撇号括起来，例如:

```
'G'day Barry!'     // error
"G'day Barry!"     // correct'Barry said 'I lost my keys''   // error
'Barry said "I lost my keys"'   // correct
"Barry said 'I lost my keys'"   // correct
```

虽然这种技术工作得很好。我们的代码最好更加一致。

**使用转义符** `**\**`

通过使用反斜杠，我们可以防止 JavaScript 将引号解释为字符串的结尾。\应该在任何单引号或双引号之前:

```
'G\'day Barry!'   // correct'Barry said \'I lost my keys\''   // also correct!
```

**使用模板文字**

如果我们在模板文字````中包含我们的字符串，我们的字符串将是有效的，它看起来也会更整洁:

```
`Barry said "I lost my keys" and "I'm now concerned"!`; // correct
```

## 更长的字符串

我们使用换行符`\n`或回车符`\r`，将较长的字符串分成多行:

```
const multipleLines = "This string is long\nso lets display it\nacross multiple lines.";// output:"This string is long
so lets display it
across multiple lines."
```

这工作很好，但它相当混乱。我们可以通过使用一些连接来整理它:

```
const multipleLines = "This string is long\n" +
"so lets display it\n" +
"across multiple lines.";
```

或者选择带有模板文字的金星:

```
const multipleLines = `This string is long
so lets display it
across multiple lines.`;
```

Back-ticks 来救援了！不需要转义或连接，换行符被保留，看起来更整洁。

# 使用字符串

在下一节中，我们将看看使用字符串的一些方法。

字符串中的每个字符都有编号，从`0`开始。例如:

```
"This is my string!" 
```

第一个字符是索引位置`0`的`T`。

最后一个字符是索引位置`17`的`!`。

这些空间在`4`、`7`和`10`处也有索引位置。

我们可以像这样访问任何字符:

```
"This is my string![8]"     // m
```

## charAt()

我们可以类似地使用`charAt()`方法。它返回字符串中指定索引处的字符，例如:

```
"This is my string!.charAt(8)"     // m
```

## 索引 Of()

如果我们想返回一个索引号，我们可以使用`indexOf()`。

它返回子串在字符串中开始位置的索引，如果没有找到子串，则返回`-1`。它区分大小写。

```
const str = "This is my string!";str.indexOf('s');       // returns 3 (the first 's' instance)str.indexOf('my');      // returns 8str.indexOf('My');      // returns -1str.lastIndexOf('s');   // returns 11 (the last instance)
```

您可以使用条件来测试子字符串是否存在:

```
if (str.indexOf('my') > -1) {
	console.log('Success! It contains the string');
}
```

我们知道，当什么都没有找到时，`-1`会产生结果，所以从逻辑上讲，任何大于`-1`的结果都意味着子串存在。

## 切片()

要获得从某个特定字符开始的字符串部分(和可选的*结尾的部分),我们可以使用`slice()`。*

这个方法可以有两个参数。第一个是开始位置(`0`当然是第一个字符)，第二个*可选*参数是结束位置。如果任何一个参数是负数，它将从字符串的末尾开始并向后工作。

```
const str = "This is my string!";str.slice(5);         // "is my string!"
str.slice(5, 10);     // "is my"
str.slice(0, -8);     // "This is my"
```

## 求字符串的长度

要找到一个字符串的长度，我们可以使用`length`属性。它返回字符串中的字符数。

```
"This is my string!".length;     //18
```

`length`属性返回从`1`开始的实际字符数，在我们的例子中是`18`，而不是从`0`开始到`17`结束的最终索引号。

## 转换为大写或小写

两个方法`toUpperCase()`和`toLowerCase()`是格式化我们文本的有用方法。

`toUpperCase()` 将字符串中的所有文本转换为大写。

```
const str = "This is my string!";
const upper = str.toUpperCase();
console.log(upper);// "THIS IS MY STRING!"
```

`toLowerCase()` 将字符串中的所有文本转换成小写。

```
const str = "This is my string!";
const lower = str.toLowerCase();
console.log(lower);// "this is my string!"
```

## 拆分()

要将一个字符串分成几个部分并把内容插入到一个数组中，我们可以使用`split()`方法。

我们提供的参数是`delimiter`，我们用来分割字符串的字符。

在下面的例子中，让我们使用空白字符`“ “`作为分隔符:

```
const str = "This is my string!";
const splitString = str.split(" ");console.log(splitString);// ["This", "is", "my", "string!"]
```

我们现在可以通过它的数组位置来访问每个部分..

```
splitSting[0];    // This
splitSting[1];    // is
splitSting[2];    // my
splitSting[3];    // string!
```

我们还可以选择提供第二个参数，在找到一定数量的分隔符匹配后，停止拆分字符串。

让我们看另一个例子，这次使用逗号`‘, ‘`作为分隔符。

```
const shoppingList = 'Bananas, green apples, chocolate, cat food';
const foodItems = shoppingList.split(', ');
console.log(foodItems);// ["Bananas", "green apples", "chocolate", "cat food"] const snacks = shoppingList.split(', ', 2);
console.log(snacks)// ["Bananas", "green apples"]
```

我们的`.split(', '**, 2**)`在第二个逗号后停止拆分，只返回前两个数组条目。

## 修剪()

`trim()`方法从字符串的两端删除任何空白，但重要的是不删除中间的任何地方。

```
const ahhhhWhitespace = "     This is my string!     ";const trimmed = ahhhhWhitespace.trim();console.log(trimmed);// "This is my string!"
```

空白可以是制表符或空格，`trim()`方法可以方便地删除多余的部分。

## 替换()

如果我们希望在字符串中搜索一个值，并用一个新值替换它，我们使用`replace()`。

第一个参数是要查找的值，第二个参数是替换值。

```
const str = "This is my string!";const newString = str.replace("my", "your");console.log(newString);// This is your string!
```

默认情况下，`replace()`方法只替换第一个匹配。要替换**所有匹配**，您需要使用全局标志`g`传入一个正则表达式。如果我们还想忽略大小写，我们可以添加不区分大小写的标志`i`。

```
const str = "GOOGLE is my homepage. But I don't use google Chrome."const newString = str.replace(/google/gi, "Google");console.log(newString);// Google is my homepage. But I don't use Google Chrome.
```

在这里，我们告诉**我们的字符串中的每个**google 实例(不考虑大小写)，用“Google”替换。

# 将字符串转换为数字

在这最后一节，让我们快速看一下将字符串转换成数字的一些方法。

## parseInt()

要将一个字符串转换成一个整数，我们可以使用`parseInt()`。它有两个参数，第一个是我们想要转换的值，第二个称为*基数。*这是数学系统中使用的基数。对于我们的使用，它应该总是`10`，十进制系统。

```
parseInt('64', 10);     // 64parseInt('64.9', 10);   // 64parseInt('64px', 10);   // 64
```

## `parseFloat()`

如果我们希望转换成浮点数(带小数点的数字)，我们可以使用`parseFloat()`。

```
parseFloat('64.9');             // 64.9parseFloat('64bit');            // 64console.log(parseFloat('64'));  // 64
```

## `Number()`

我们也可以用`Number()`方法转换我们的字符串，但是它对可以转换的值有更严格的要求。它只适用于由数字组成的字符串。

```
Number('64');      // 64Number('64.9');    // 64.9Number('64bit');   // NaNNumber('64%');     // NaN
```

`NaN`如果您记得的话，代表的不是数字。

***你准备好让你的 JavaScript 技能更上一层楼了吗？*** *今天就开始用我的新电子书吧！无论你是想学习你的第一行代码，还是想扩展你的知识面并真正学习基础知识..*[*JavaScript 掌握完全指南*](https://gum.co/mastering-javascript) *带你从零到英雄！*

![](img/dde515044536421c6c999650977f80c4.png)

*现已上市！👉*[https://gum.co/mastering-javascript](https://gum.co/mastering-javascript)

# 结论

就是这样！我们已经讲述了创建字符串、字符串连接和语法规则的基础知识。然后，我们继续用许多常见的方法操作字符串，比如格式化、查找、替换和转换值。

我希望这篇文章对你有用！你可以在媒体上关注我。我也在[推特](https://twitter.com/easeoutco)上。欢迎在下面的评论中留下任何问题。我很乐意帮忙！

本系列的下一篇: [JavaScript 基础:掌握 DOM！](/javascript-fundamentals-master-the-dom-part-1-82433084fb40)(第一部分)

# 关于我的一点点..

嘿，我是提姆！👋我是一名开发人员、技术作家和作家。如果你想看我所有的教程，可以在我的个人博客上找到。

我目前正在构建我的[自由职业者完整指南](http://www.easeout.co/freelance)。坏消息是它还不可用！但是如果这是你可能感兴趣的东西，你可以[注册，当它可用时会通知你](https://easeout.eo.page/news)👍

感谢阅读🎉