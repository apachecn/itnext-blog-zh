# JavaScript 基础:理解正则表达式

> 原文：<https://itnext.io/javascript-fundamentals-understanding-regex-fd81891375e0?source=collection_archive---------0----------------------->

![](img/aff9ec72cdeed1a6d778865a5f2dc8d6.png)

JavaScript 正则表达式(或 **Regex** )是一个字符序列，我们可以利用它来有效地处理字符串。使用这种语法，我们可以:

*   **在**中搜索字符串中的文本
*   **替换字符串中的**子字符串
*   **从字符串中提取**信息

追溯到 20 世纪 50 年代，正则表达式被形式化为字符串处理算法中的模式搜索概念。

JavaScript 在语言中直接内置了正则表达式支持。对正则表达式的深刻理解将使你成为一名更有效率的程序员。所以让我们开始吧！

🤓*想了解最新的 web 开发吗？*🚀想要将最新的新闻直接发送到您的收件箱吗？
🎉加入一个不断壮大的设计师&开发者社区！

**在这里订阅我的简讯→**[**https://ease out . EO . page**](https://easeout.eo.page/)

# 一个非常基本的正则表达式模式

这是一个基本模式:

```
var regex = /hello/;console.log(regex.test('hello world'));  
// true
```

我们只是将文字文本与测试字符串进行匹配。我们稍后将详细研究正则表达式测试方法..

# 为什么要使用正则表达式？

如前所述，正则表达式是描述字符串数据模式的一种方式。我们可以使用它们来检查字符串，例如查找电子邮件地址——通过匹配正则表达式定义的模式。

# 创建正则表达式

在 JavaScript 中，我们可以用两种方法创建正则表达式:要么使用 RegExp 构造函数，要么使用正斜杠`/`将 regex 模式括起来。

## 构造函数方法:

语法是这样的:

```
new RegExp(pattern[, flags])
```

比如说:

```
var regexConst = new RegExp('abc');
```

## 字面法:

语法是这样的:

```
/pattern/flags
```

一个例子:

```
var regexLiteral = /abc/;
```

注意:标志是可选的，我们将在本文的后面看到它们！

在需要动态创建正则表达式的情况下，需要使用构造函数方法。

在这两种情况下，结果都将给出一个 regex 对象——它将具有相同的方法和属性，供我们使用。

# 正则表达式方法

当测试我们的正则表达式时，我们通常使用两种方法之一:`RegExp.prototype.test()`或`RegExp.prototype.exec()`。

## 正则表达式.原型.测试()

我们使用这种方法来测试是否找到了匹配。它接受我们用正则表达式测试的字符串，并根据是否找到匹配返回`true` 或`false`。

让我们看一个例子:

```
var regex = /hello/; 
var str = 'hello world';
var result = regex.test(str);console.log(result); 
// returns 'true' as hello is present in our string
```

## 正则表达式.原型.执行()

我们使用这个方法来接收所有匹配组的数组。它接受我们用正则表达式测试的字符串。

一个例子:

```
var regex = /hello/;
var str = 'hello world';
var result = regex.exec(str);
console.log(result);// returns [ 'hello', index: 0, input: 'hello world', groups: undefined ]
```

在这个例子中，`‘hello’`是我们匹配的模式，`index`是正则表达式开始的地方，& `input`是被传递的字符串。

对于本文的其余部分，我们将使用`test()`方法。

# 正则表达式的力量

到目前为止，我们已经看到了如何创建简单的正则表达式模式。这真的只是冰山一角。现在让我们深入语法，看看正则表达式在处理更复杂的任务方面的全部能力！

更复杂任务的一个例子是，如果我们需要匹配多个电子邮件地址。通过使用语法中定义的特殊字符，我们可以做到这一点！

现在让我们看一看，这样我们可以更全面地掌握&因此在我们的程序中使用正则表达式。

# 标志:

在任何正则表达式中，我们可以使用以下标志:

*   `g`:多次匹配模式
*   `i`:使正则表达式不区分大小写
*   `m`:启用多行模式。其中`^`和`$`匹配整个字符串的开始和结束。如果没有这个，多行字符串匹配每一行的开始和结束。
*   `u`:启用对 unicode 的支持
*   `s`:单行*的简称*，它使`.`也匹配新的行字符

标志也可以组合在一个正则表达式中&标志的顺序无关紧要。它们被添加到 regex **文字**中字符串的末尾:

```
/hello/ig.test('HEllo')
// returns true
```

如果使用 RegExp 对象**构造函数**，它们将作为第二个参数添加:

```
new RegExp('hello', 'ig').test('HEllo') 
// returns true
```

# 角色组:

## 字符集

我们使用字符集来匹配单个位置的不同字符。它们将字符串中的任何单个字符与括号内的字符进行匹配:

```
var regex = /[hc]ello/;console.log(regex.test('hello'));
// returns trueconsole.log(regex.test('cello'));
// returns trueconsole.log(regex.test('jello'));
// returns false
```

## **被否定的字符集【^abc】**

它匹配方括号中的**而不是**的任何内容:

```
var regex = /[^hc]ello/;console.log(regex.test('hello'));
// returns falseconsole.log(regex.test('cello'));
// returns falseconsole.log(regex.test('jello'));
// returns true
```

## **范围[a-z]**

如果我们想在一个位置匹配字母表中的所有字母，我们可以使用范围。比如:**【A-j】**会匹配从 A 到 j 的所有字母，我们也可以使用类似**【0–9】**这样的数字或者类似**【A-Z】**这样的大写字母:

```
var regex = /[a-z]ello/;console.log(regex.test('hello'));
// returns trueconsole.log(regex.test('cello'));
// returns trueconsole.log(regex.test('jello'));
// returns true
```

如果在我们测试的范围内至少存在一个字符，它将返回 true:

```
/[a-z]/.test('a')  // true
/[a-z]/.test('1')  // false
/[a-z]/.test('A')  // false (as our range is in lower case)
/[a-c]/.test('d')  // false
/[a-c]/.test('cd') // true (as 'c' is in the range)
```

也可以使用`-`组合范围:

```
/[A-Z-0-9]//[A-Z-0-9]/.test('a') // false
/[A-Z-0-9]/.test('1') // true
/[A-Z-0-9]/.test('A') // true
```

**多个范围项目匹配**

我们可以检查一个字符串是包含一个字符还是只包含一个字符。以`^`开始正则表达式，以`$`结束:

```
/^[A-Z]$/.test('A')  // true
/^[A-Z]$/.test('AB') // false
/^[A-Z]$/.test('Ab') // false
/^[A-Z-0-9]$/.test('1')  // true
/^[A-Z-0-9]$/.test('A1') // false
```

## **元字符**

元字符是具有特殊含义的字符。让我们来看看其中的一些:

*   `\d`:匹配任意数字，为`[0-9]`
*   `\D`:匹配任何不是数字的字符，有效`[^0-9]`
*   `\w`:匹配任意字母数字字符(加下划线)，相当于`[A-Za-z_0-9]`
*   `\W`:匹配任何非字母数字字符，除了`[^A-Za-z_0-9]`
*   `\s`:匹配任何空白字符:空格、制表符、换行符和 Unicode 空格
*   `\S`:匹配任何不是空格的字符
*   `\0`:匹配空值
*   `\n`:匹配换行符
*   `\t`:匹配一个制表符
*   `\uXXXX`:用代码 XXXX 匹配一个 [unicode](https://flaviocopes.com/unicode/) 字符(需要`u`标志)
*   `**.**`:匹配任何不是换行符的字符(例如`\n`)(除非你使用`s`标志，稍后解释)
*   `[^]`:匹配任何字符，包括换行符。它在多行字符串上非常有用

## **量词**

量词是在正则表达式中具有唯一意义的符号。

让我们看看他们的行动:

*   `+` 匹配前面的表达式 1 次或多次:

```
var regex = /\d+/;console.log(regex.test('1'));
// trueconsole.log(regex.test('1122'));
// true
```

*   `*` 与前面的表达式匹配 0 次或更多次:

```
var regex = /hi*d/;console.log(regex.test('hd'));
// trueconsole.log(regex.test('hid'));
// true
```

*   `?` 匹配前面的表达式 0 或 1 次，即前面的模式是可选的:

```
var regex = /hii?d/;console.log(regex.test('hid'));
// trueconsole.log(regex.test('hiid'));
// trueconsole.log(regex.test('hiiid'));
// false
```

*   `^` 匹配字符串的开头，后面的正则表达式应该在测试字符串的开头:

```
var regex = /^h/;console.log(regex.test('hi'));
// trueconsole.log(regex.test('bye'));
// false
```

*   `$`匹配字符串的结尾，它前面的正则表达式应该在测试字符串的结尾:

```
var regex = /.com$/;console.log(regex.test('test@email.com'));
// trueconsole.log(regex.test('test@email'));
// false
```

*   `{N}`与*完全匹配*前面正则表达式的 N 次出现:

```
var regex = /hi{2}d/;console.log(regex.test('hiid'));
// trueconsole.log(regex.test('hid'));
// false
```

*   `{N,}`至少与前面正则表达式的匹配 N 次。

```
var regex = /hi{2,}d/;console.log(regex.test('hiid'));
// trueconsole.log(regex.test('hiiid'));
// trueconsole.log(regex.test('hiiiid'));
// true
```

*   `{N,M}`匹配前面正则表达式的*至少* N 次和*最多* M 次(当 M > N 时)。

```
var regex = /hi{1,2}d/;console.log(regex.test('hid'));
// trueconsole.log(regex.test('hiid'));
// trueconsole.log(regex.test('hiiid'));
// false
```

*   `X|Y`交替匹配*X 或 Y:*

```
*var regex = /(red|yellow) bike/;console.log(regex.test('red bike'));
// trueconsole.log(regex.test('yellow bike'));
// trueconsole.log(regex.test('brown bike'));
// false*
```

**注意:*使用任何特殊字符作为表达式的一部分，例如，如果你想匹配文字`+`或`.`，那么你需要用反斜杠`\`对它们进行转义。像这样:*

```
*var regex = /a+b/;  
// this doesn't workvar regex = /a\+b/; 
// this works!console.log(regex.test('a+b')); 
// true*
```

# *查看正则表达式*

*有了这些新鲜的概念，让我们来回顾一下我们所学的！*

## *匹配任意 10 位数字:*

```
*var regex = /^\d{10}$/;console.log(regex.test('4658264822'));
// true*
```

*所以`\d`匹配任何数字字符。`{10}`匹配前面的表达式，在本例中`\d` **正好是**的 10 倍。因此，如果测试字符串包含少于或多于 10 个数字，结果将为假。*

## ***用以下格式匹配日期:***

*`**DD-MM-YYYY**` **或** `**DD-MM-YY**`*

```
*var regex = /^(\d{1,2}-){2}\d{2}(\d{2})?$/;console.log(regex.test('01-01-2000'));
// trueconsole.log(regex.test('01-01-00'));
// trueconsole.log(regex.test('01-01-200'));
// false*
```

*这里我们将整个表达式包装在`^` 和`$`中，这样匹配就跨越了整个字符串。`(`是第一个子表达式的开始。`\d{1,2}`匹配最少*1 位数字和最多*2 位数字。`-`匹配文字连字符。`)`是第一个子表达式的结尾。***

*然后`{2}`与第一个子表达式*精确匹配* 2 次。`\d{2}`与*完全匹配*的 2 位数。`(\d{2})?`与*完全匹配*的 2 位数。然而，这是可选的，所以任何一年包含 2 位数或 4 位数。*

****你准备好让你的 JavaScript 技能更上一层楼了吗？*** *今天就开始用我的新电子书吧！无论你是想学习你的第一行代码，还是想扩展你的知识面并真正学习基础知识..*[*《JavaScript 精通完全指南》*](https://gum.co/mastering-javascript) *带你从零到英雄！**

*![](img/dde515044536421c6c999650977f80c4.png)*

**现已上市！👉*[https://gum.co/mastering-javascript](https://gum.co/mastering-javascript)*

# *结论*

*我们走吧！我们研究了正则表达式，从最基础的一直到更高级的实现。包括文字和构造方法、测试方法、标志和字符语法。*

*正则表达式确实可以相当复杂！然而，花时间学习语法将极大地帮助您更容易地识别正则表达式模式。你获得的任何新的信心肯定会让你准备好征服你在编码之旅中遇到的下一个障碍！*

*我希望这篇文章对你有用！你可以在媒体上关注我。我也在[推特](https://twitter.com/easeoutco)上。欢迎在下面的评论中留下任何问题。我很乐意帮忙！*

# *关于我的一点点..*

*嘿，我是提姆！👋我是一名开发人员、技术作家和作家。如果你想看我所有的教程，可以在[我的个人博客](http://www.easeout.co)上找到。*

*我目前正在构建我的[自由职业者完整指南](http://www.easeout.co/freelance)。坏消息是它还不可用！但是如果这是你可能感兴趣的东西，你可以[注册，当它可用的时候会通知你](https://easeout.eo.page/news)👍*

*感谢阅读🎉*