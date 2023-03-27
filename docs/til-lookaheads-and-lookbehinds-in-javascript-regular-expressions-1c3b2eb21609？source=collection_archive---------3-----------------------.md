# TIL—JavaScript 正则表达式中的 lookaheads(和 lookbehinds)

> 原文：<https://itnext.io/til-lookaheads-and-lookbehinds-in-javascript-regular-expressions-1c3b2eb21609?source=collection_archive---------3----------------------->

![](img/71d782bec9f38bd6bfe832f4b3747932.png)

正则表达式本身就是一个挑战。对我来说，我总是要花几分钟时间才能理解一个特定的正则表达式是做什么的，但毫无疑问，它们是有用的。

今天，我刚刚喝完周日早上的咖啡，看了幻灯片[](https://slidr.io/mathiasbynens/what-s-new-in-es2018#1)[bene dikt Meurer](https://twitter.com/bmeurer?lang=de)和 [Mathias Bynens](https://twitter.com/mathias) 的《ES2018 中的新内容】。

这些幻灯片中有很多有用的信息，除了新的语言特性，如异步迭代、对象扩展属性和正则表达式中的[命名捕获组](https://github.com/tc39/proposal-regexp-named-groups)(🎉)还介绍了正则表达式中的 lookaheads(以及即将出现的 lookbehinds)。

我偶尔会遇到 JavaScript 正则表达式中的前视头，我不得不承认我从来没有使用过它们，但是现在语言中也会出现后视头，所以我决定阅读一些文档，最终了解这些前视头是什么。

# JavaScript 中的前瞻

使用 lookaheads，您可以定义只有在被另一个模式跟随或不被另一个模式跟随时才匹配的模式。

[关于正则表达式的 MDN 文章](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_Expressions)描述了正则表达式中两种不同类型的前视。

正面和负面展望:

*   `x(?=y)`–正向前瞻(匹配后跟“y”的“x”)
*   `x(?!y)`–否定前瞻(当后面没有‘y’时匹配‘x’)

# JavaScript 中的捕获组——看起来相似的同伴

哦，好吧…`x(?=y)`–如果你问我的话，这是一个复杂的语法。最初让我困惑的是，我通常在 JavaScript 表达式中使用`()`来表示捕获的组。

让我们看一个捕获组的示例:

```
const regex = /\w+\s(\w+)\s\w+/;regex.exec('eins zwei drei'); 
// ['eins zwei drei', 'zwei'] 
//                      /\ 
//                      || 
//                 captured group 
//                 defined with
//                     (\w+)
```

您在上面看到的是一个正则表达式，它捕获一个被一个空格和另一个单词包围的单词(在本例中是`zwei`)。

# 前瞻不像被捕获的组

所以让我们来看一个典型的例子，当你阅读 JavaScript 正则表达式中的前视时，你会发现这个例子。

```
const regex = /Max(?= Mustermann)/;regex.exec('Max Mustermann')
// ['Max']regex.exec('Max Müller')
// null
```

每当`Max`后面跟有`Mustermann`时，这个例子就匹配它，否则就不匹配，并返回`null`。对我来说有趣的是，它只匹配`Max`，而不匹配前瞻中定义的模式。在使用正则表达式一段时间后，这看起来很奇怪，但是当你想到这一点时，这就是前瞻的意义。

“Max Mustermann”的例子在我看来是没有用的，所以让我们用一个真实的用例来深入研究正面和负面的前瞻。

# 积极前瞻

让我们假设你有一长串的减价商品，其中包括一系列人和他们的食物偏好。当一切都是一长串的时候，你怎么知道哪些人是素食主义者呢？

```
const people = `
 - Bob (vegetarian)
 - Billa (vegan)
 - Francis
 - Elli (vegetarian)
 - Fred (vegan)
`; const regex = /-\s(\w+?)\s(?=\(vegan\))/g;
//                |----| |-----------|
//                  /          \ 
//            more than one     \
//           word character     positive lookahead
//            but as few as     => followed by "(vegan)"
//              possible let result = regex.exec(people);
while(result) {
  console.log(result[1]);
  result = regex.exec(people);
} // Result: 
// Billa
// Fred
```

让我们快速浏览一下正则表达式，并试着用单词表达出来。

```
const regex = /-\s(\w+?)\s(?=\(vegan\))/g;
```

好吧…让我们开始吧！

> *匹配任何破折号，后跟一个空格字符、一个以上但尽可能少的单词字符(A-Za-z0–9 _)、一个空格和“(纯素食)”模式*

# 消极/否定的前瞻

另一方面，你怎么知道谁不是素食主义者？

```
const people = `
 - Bob (vegetarian)
 - Billa (vegan)
 - Francis
 - Elli (vegetarian)
 - Fred (vegan)
`;const regex = /-\s(\w+)\s(?!\(vegan\))/g 
//                |---| |-----------|
//                  /         \
//           more than one     \
//          word character    negative lookahead
//           but as few as    => not followed by "(vegan)"
//              possiblelet result = regex.exec(people);while(result) {
  console.log(result[1]);
  result = regex.exec(people);
} // Result:
// Bob
// Francis
// Elli
```

让我们快速浏览一下正则表达式，并试着用单词表达它。

```
const regex = /-\s(\w+)\s(?!\(vegan\))/g
```

> *匹配任何破折号，后跟一个空格字符，后跟一个以上但尽可能少的单词字符(A-Za-z0–9 _)，后跟一个空格字符(包括换行符)，后面没有“(纯素食)”模式*

# 前视者很快就会有后视者的陪伴

[Lookbehinds](https://github.com/tc39/proposal-regexp-lookbehind) 将以相同的方式工作，但对于匹配模式之前的模式(lookaheads 考虑匹配部分之后的模式)和[在今天的 Chrome 中已经得到支持](http://kangax.github.io/compat-table/es2016plus/#test-RegExp_Lookbehind_Assertions)。他们也可以作为正面回顾`x(?<=y)`和负面回顾`x(?<!y)`。

当我们翻转例子中的字符串时，仍然使用 lookbehinds 以同样的方式工作。:)

```
const people = `
 - (vegetarian) Bob
 - (vegan) Billa
 - Francis
 - (vegetarian) Elli
 - (vegan) Fred
`; const regex = /(?<=\(vegan\))\s(\w+)/g;
//             |------------|  |---| 
//                   /           \__ 
//         positive lookbehind      \ 
//         => following "(vegan)"   more than one
//                                  word character
//                                  but as few as possiblelet result = regex.exec(people);while(result) {
  console.log(result[1]);
  result = regex.exec(people);
} // Result:
// Billa
// Fred
```

*旁注:对于摆弄正则表达式，我通常推荐*[*RegExr*](http://regexr.com/)*，但是还不支持 lookbehinds。*

如果你对更前沿的特性感兴趣，看看 Mathias 和 Benedikt 的关于 JavaScript 新特性的幻灯片会有更多令人兴奋的东西出现。

*最初发表于*[*【www.stefanjudis.com】*](https://www.stefanjudis.com/today-i-learned/the-complicated-syntax-of-lookaheads-in-javascript-regular-expressions/)*。*