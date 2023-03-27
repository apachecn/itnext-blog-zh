# 调试语法错误 Modern Node.js 上 Mocha 中意外的保留字

> 原文：<https://itnext.io/debugging-syntax-error-unexpected-reserved-word-in-mocha-on-modern-node-js-e33bf824a884?source=collection_archive---------6----------------------->

## Mocha 是一个优秀的单元测试框架，我们可以在 Node.js 应用程序中使用。但是，它的不可理解的错误信息会导致事倍功半。

![](img/633de66a364553ba54547896e0f7963e.png)

作者图片

**错误*意外保留字*通常意味着您在非异步函数中使用了 *await* 。但是如果 Mocha 抛出这个错误，引用一个没有此类问题的测试套件文件，该怎么办呢？为了节省毫无结果的时间，请阅读这篇短文。**

错误*意外保留字*意味着 JavaScript 编译器看到了一个*保留字*，这意味着一个对编译器来说特殊的关键字，而它并不是预期的。典型的情况是在一个没有被标记为`async`的函数中使用`await`。

在这种情况下，我发现 Mocha——一个编写单元测试的流行框架——错误地报告了哪个源文件包含了意外的保留字。我的测试套件抛出了那个错误消息，但是 Mocha 识别出了不正确的源文件。

通常一个编译器，或者一个测试框架，报告一个语法错误，会告诉我们实际的代码行，以及有问题的代码。但是，在这种情况下，Mocha 既不报告行号，也不报告受影响的代码，更糟糕的是，它报告错误发生在与实际位置不同的源文件中。

因为 Mocha 错误地报告了意外保留字错误的位置，所以我花了几个小时一遍又一遍地检查不包含这种错误的源文件的语法，但毫无结果。

让我们从通常的情况开始:

```
$ cat unreserved.mjs function a() {
   await console.log(`This will throw an error`); 
} $ node unreserved.mjs 
file:// /home/david/Projects/akasharender/akasharender/test/unreserved.mjs:3
   await console.log(`This will throw an error`);
   ^^^^^ SyntaxError: Unexpected reserved word
    at ESMLoader.moduleStrategy (node:internal/modules/esm/translators:119:18)
    at ESMLoader.moduleProvider (node:internal/modules/esm/loader:483:14)
    at async link (node:internal/modules/esm/module_job:67:21)
```

这是我们通常看到的。我们已经创建了一个 ES6 模块，`unreserved.mjs`，包含一个非异步函数。在这个函数中有一个关键字`await`。这是在错误的上下文中使用了保留字`await`。在这种情况下，错误报告很好地让我们知道了问题是什么，以及问题发生在哪里。

这是期望的行为，编译器很好地让我们知道发生了什么。我们可以直接到受影响的源代码行，在呻吟了“ *Doh* ”之后，立刻知道该怎么做。

考虑一个真实的包，其中已经编写了一个实际的(并且有用的)模块，它包含了那个问题。然后，您要为这个模块编写一个测试套件，因为您遵循了良好的测试优先开发实践。

```
$ cat test-unreserved.mjs import * as unreserved from './unreserved.mjs'; $ npx mocha test-unreserved.mjs 
SyntaxError[ @/home/david/Projects/akasharender/akasharender/test/test-unreserved.mjs ]: Unexpected reserved word
    at ESMLoader.moduleStrategy (node:internal/modules/esm/translators:119:18)
    at ESMLoader.moduleProvider (node:internal/modules/esm/loader:483:14)
```

当然，您的测试套件必须导入应用程序代码来执行测试用例。这意味着导入`unreserved.mjs`模块。因为我们刚刚创建了那个文件，所以我们很容易记住这个错误。在这种情况下，Mocha 并没有给我们一个明确的错误信息。它只是说*意外的保留字*，并且进一步地，它将`test-unreserved.mjs`标识为包含问题。

再一次，相反，考虑这发生在一个真实的包中。测试套件可能有 1500 行代码，被测试的模块也有类似数量的代码。

因为 Mocha 明确地将测试套件识别为包含意外的保留字，所以很容易花几个小时梳理测试套件。你检查了`await`的每一个用法，都是正确的。您仔细检查每个循环、每个函数和每个测试用例。您使用 VS 代码并折叠每个块，因为编辑器正确地折叠了每个块，所以似乎编辑器认为这是正确构造的 JavaScript。然后，您尝试注释掉所有代码，但错误仍然出现。最后，你试着逐个注释掉`import`语句，瞧，错误随着一个特定的`import`语句消失了。

检查该文件，您可能会发现一个使用了`async`关键字的非异步函数。这很容易解决，而且这个错误会消失。可能还有其他问题，您会知道原来的问题已经解决了，因为错误消息发生了变化。

```
$ npx mocha test-cache.mjs SyntaxError[ @/home/david/Projects/akasharender/akasharender/test/test-cache.mjs ]: Unexpected reserved word
    at ESMLoader.moduleStrategy (node:internal/modules/esm/translators:119:18)
    at ESMLoader.moduleProvider (node:internal/modules/esm/loader:483:14) ##### The error changes to this:$ npx mocha test-cache.mjs 
SyntaxError[ @/home/david/Projects/akasharender/akasharender/test/test-cache.mjs ]: Identifier 'setup' has already been declared
    at ESMLoader.moduleStrategy (node:internal/modules/esm/translators:119:18)
    at ESMLoader.moduleProvider (node:internal/modules/esm/loader:483:14)
```

这是发生问题的真实实例。我已经修复了*意外的保留字*问题，却发现了另一个问题——我创建了两个名为`setup`的函数。请注意，Mocha 仍然报告了错误的源文件。但是，很明显问题会出现在另一个源文件中，而不是测试套件中。

# 结论

Mocha 错误地报告了包含代码问题的源文件。

它正确地识别了问题——一些代码错误地使用了保留字。但是错误被报告在错误的文件中，这很容易导致花费数小时仔细检查没有此类错误的代码而毫无结果。

我们的时间是宝贵的，花这么多时间清理一个没有问题的文件是令人沮丧的。

糟糕的摩卡。没有汤给你。

# 关于作者

![](img/638062765439ef21fcd5cec68fdc4507.png)

[**大卫·赫伦**](https://davidherron.com/) :大卫·赫伦是一名作家和软件工程师，专注于技术的明智使用。他对太阳能、风能和电动汽车等清洁能源技术特别感兴趣。David 在硅谷从事了近 30 年的软件工作，从电子邮件系统到视频流，再到 Java 编程语言，他已经出版了几本关于 Node.js 编程和电动汽车的书籍。

*原载于*[*https://techsparx.com*](https://techsparx.com/nodejs/esnext/mocha-reserved-word.html)*。*