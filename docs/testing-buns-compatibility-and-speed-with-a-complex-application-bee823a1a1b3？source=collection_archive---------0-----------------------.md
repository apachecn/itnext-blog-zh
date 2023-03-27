# 测试 Bun 与 Node.js 和一个复杂应用程序的兼容性和速度

> 原文：<https://itnext.io/testing-buns-compatibility-and-speed-with-a-complex-application-bee823a1a1b3?source=collection_archive---------0----------------------->

## 一个新的服务器端 JavaScript 平台宣称 Node.js 的兼容性和巨大的性能提升，但是它符合标准吗？

![](img/fe5a8cd786e95ea777669d84770a5311.png)

作者图片

**最近，一个新的 JavaScript 服务器端运行时 Bun 宣布推出，承诺将大幅提高 Node.js 的速度。与其用一个小应用程序进行测试，不如让我们用一个复杂的应用程序来看看真实世界的性能。**

Bun 网站([https://bun.sh/](https://bun.sh/))让它看起来像是 Node.js 开发的一个非常有趣的选择。它与 Node.js 的想法相同，但承诺有更好的性能。像 Node.js 一样，Bun 从 web 浏览器打包了一个 JavaScript 引擎作为服务器端 JavaScript 平台，并承诺实现 Node.js API 以实现完全兼容性。

它没有使用 Chrome 的 V8 引擎，而是使用了苹果 Safari 浏览器的 JavaScriptCore 引擎。这款发动机备受推崇，被认为比 V8 更快，是第一个宣称的优势。另一个声称的优势是 Bun 系统是用一种新语言 Zig 编写的，而不是用 C++或 Rust。由于一些技术细节，Zig 应该更快。

我发现的另一件有趣的事情是，TypeScript 和 JSX transpilers 都内置于 Bun 中，这应该使运行 TypeScript 代码变得非常容易。

该团队声称与 Node.js 包有 90%的兼容性，包括使用 N-API 来支持执行本机代码 Node.js 包。它还使用 Node.js 的相同的`node_modules`基础设施和包查找算法，这意味着它可以立即与现有的 Node.js 生态系统兼容。

相比之下，Deno(另一个 Node.js 替代方案)与 Node.js 包生态系统不兼容，这给了它很大的劣势。

Node.js 开发人员将如何处理与现有生态系统兼容但速度更快的替代方案？

更新 2:正如我下面提到的，这篇文章错误地没有在 Bun 上测试我的应用程序。相反，它意外地在 Node.js 上运行了测试。

我写了一篇更新的文章:

[](/deeper-testing-of-buns-performance-and-compatibility-against-node-js-9115763965ed) [## 对 Bun 的性能和与 Node.js 的兼容性进行更深入的测试

### Bun 最终可能会使 Node.js 和/或 Deno 变得无关紧要，但现在说它们都是死项目还为时过早

itnext.io](/deeper-testing-of-buns-performance-and-compatibility-against-node-js-9115763965ed) 

更新:看起来我下面的测试是不正确的。在测试中使用的 Bun 版本中，我使用`cli.js`运行我的应用程序，它在文件的前面有一个“shebang ”,也称为`#!/usr/bin/env node. According to a comment below by Jarred Sumner, Bun respected the shebang, and therefore my code was executed by Node.js rather than by Bun.`

在 Bun 0.1.4 中，情况似乎略有变化。这个源文件让我测试 shebang 特性:

```
#!/usr/bin/env nodeconsole.log(process.isBun);
```

`process.isBun`变量可用于测试 Bun 是否运行。根据 Jarred 下面的评论，如果使用 Bun 运行，这将使用 Node.js 执行。但是对于 Bun 0.1.4，它是由 Bun 执行的:

```
david@nuc2:~/ws/techsparx.com$ bun -v
0.1.4
david@nuc2:~/ws/techsparx.com$ node ./shebang.js 
undefined
david@nuc2:~/ws/techsparx.com$ bun ./shebang.js 
1
```

这意味着当给 Bun 一个带有 shebang 的文件时，它将继续执行这个文件，而不是把它交给 Node.js。

shebang 特性从 Berkeley Unix 4.2BSD 开始就存在于 Unix/Linux 生态系统中。当时，我阅读了 4.2BSD 源代码(在 20 世纪 80 年代中期)，并了解到 shebang 是由内核直接处理的。它相当于一个`exec`调用来调用 shebang 中命名的解释器，脚本作为标准输入。解释器应该检测并忽略第一行，并执行它在 stdin 中找到的任何代码。

Bun 关注 shebang 然后执行 Node.js 是不正确的。谢天谢地，这不是它所做的。

顺便说一句，这里有一个你可以用 shebangs 做的有趣且有益的小把戏:

```
david@nuc2:~/ws/techsparx.com$ vi shebang.sed
david@nuc2:~/ws/techsparx.com$ chmod +x shebang.sed 
david@nuc2:~/ws/techsparx.com$ ./shebang.sed 
Hello World
david@nuc2:~/ws/techsparx.com$ cat shebang.sed 
#!/usr/bin/sed 1d
Hello World
```

sed 程序从 20 世纪 80 年代或更早就存在了，是一种在文本流上执行文本操作的方法。本例中的 shebang 命令运行`sed`,删除第一行文本。这是因为`sed`将在`stdin`上接收脚本，通过删除第一行，输出将不包含 shebang。您可以使用这个技巧来创建一个简单的命令，只打印一些文本，比如本地比萨饼店的列表。

所以…在 Bun 0.1.4 中，shebang 不会让 Node.js 运行测试。为了让 Jarred 正确，Bun 0.1.3 和更早的版本必须做到这一点。

我尝试使用 Bun 0.1.4 执行我的代码，遇到了大量的兼容性问题。看起来 Bun 此时无法执行 AkashaCMS。在列出短期任务的 Bun 问题队列中有一个问题，这也是对 Bun 与 Node.js 不兼容的领域的承认。一个特殊的问题是返回 Promise 对象的`fs`包中的函数，这在 AkashaRender 中广泛使用。

这意味着当我运行下面的测试时，它是由 Node.js 运行的，这就是为什么结果数据显示类似的处理时间。

# 简单性能测试的陷阱

YouTube 上已经有几个视频让 Bun 第一次尝试。我看过的每个视频都显示他们运行一些简单的命令，并说天哪，哇，这真快。

有一个众所周知的过分简单的性能测试的谬误。用 Bun 运行一个简单的脚本是否意味着它在实际应用中比 Node.js 快得多？这就是谬误。要验证 Bun 确实更快，需要比几个简单的例子更深入的测试。

我的想法是用一个复杂的案例来尝试 Bun，测试兼容性和性能。

# 我对 Bun 的复杂测试用例

我开发了一个静态网站生成器(AkashaCMS ),用它我建立了几个网站。其中一些相当大。例如，`techsparx.com`有超过 1600 个网页——换句话说就是博客帖子。AkashaCMS 支持多种模板引擎，它做服务器端的类似 jQuery 的 DOM 操作和其他一些事情。

在我的笔记本电脑(一台 Dell Latitude E7250 w/ Core i7 和 16GB 内存)上使用 Node.js，将网站渲染为 HTML 并准备好部署需要大约 30 分钟。如果 Bun 实现了它的声明，那么这个时间应该会减少到 10 分钟？

有两个重要的场景:

1.  Bun 到底能不能执行 AkashaCMS？它能渲染`techsparx.com`网站里的一切吗？
2.  Bun 渲染我的网站能比用 Node.js 快一点吗？

# 第一次观察

我的第一个测试不是运行 AkashaCMS 本身，而是执行一个与 Node.js 包相关的测试套件。我使用 Mocha 构建测试套件，不幸的是，(目前)有一些代码与 Bun 不兼容。一些包在寻找`process`对象中的值来确定兼容性。Bun 没有提供预期的值，Mocha 使用的一些包在各种兼容性测试中崩溃。我在他们的问题队列中报告了这些问题，似乎解决方案已经在进行中。

另一个问题是 Bun 不支持对 Github 上的包的`package.json`依赖。我把 Github 依赖项用于我认为不值得发布到 npm 的包。Bun 问题队列已经注意到了这个问题。如果你和我一样，有一些没有发布到 npm 资源库的包，但是你直接从 Github 加载，那么你现在就不走运了。

我确实发现使用 npm 来安装包，然后使用 Bun 来执行代码，工作得非常完美。Bun 是" *beta* "软件… FWIW。

# 使用 Bun 渲染 AkashaCMS 网站

第一个测试场景 Bun 可以完美地呈现`techsparx.com`网站——几乎没有任何问题。

这个网站确实使用了几个 Github 依赖项。这意味着使用`npm install`来下载依赖项。

通常，我使用`package.json`中的`scripts`条目来驱动构建过程。这是记录此类过程的一种便捷方式，我在之前的文章中讨论过:[如何使用 npm/yarn/Node.js package.json 脚本作为您的构建工具](https://techsparx.com/nodejs/tools/npm-build-scripts.html)

虽然 Bun 可以直接执行`package.json`构建脚本，但是这些脚本运行`akasharender`命令，而后者又是一个使用`node`命令执行的脚本。相反，我手动运行这些命令:

```
$ bun run node_modules/akasharender/cli.js -- copy-assets config.js
$ bun run node_modules/akasharender/cli.js -- render config.js
```

这直接执行`cli.js`。`--`部分是为了确保命令行参数被传递给`akasharender`，而不是被 Bun 插入。第一个命令将资源文件复制到呈现的输出目录，第二个命令将网页和其他文件呈现到该目录中。

更新——以上不再起作用，取而代之的是从命令行中去掉`run`。此外，Bun 在 v0.1.4 中不能执行 AkashaRender。参见上面更长的讨论。

在 Node.js 中，命令行上不需要`run`动词，也不需要`--`标记。否则，就是同一个命令行。

初审成功通过了。它正确地运行了 AkashaCMS 并渲染了`techsparx.com`网站。

# 首次 Bun 性能测试—复制资产文件

为了理解我将给出的性能数字，对 AkashaCMS 有一点了解可能会有所帮助。

一个 AkashaCMS 项目有四种输入目录，*资产*，*文档*和两种模板目录。资产目录中的文件被简单地复制到渲染输出目录，而文档目录中的文件通过渲染过程运行。产生的输出目录包含网站作者想要的 HTML/CSS/JS/Image 文件的任意组合。

因此`copy-assets`本质上就是这些步骤:

1.  在资产目录中查找文件
2.  使用有效的文件复制操作(来自`fs-extra`包的`fs.copy`)

文件复制循环使用`fastq`包运行多达 10 个同步复制操作。不尝试跳过复制已经在输出目录中的文件。

相比之下，`render`命令运行渲染代码、模板引擎和一堆其他东西。

测试系统是一台英特尔 NUC，采用第五代酷睿 i5，16GB 内存，文件存储在硬盘上。

使用`time`命令进行测量。下面的列是 *real* ，表示经过的时间， *user* ，表示用户态代码的 CPU 消耗， *sys* ，表示内核代码的 CPU 消耗。

这些是使用 Bun v0.1.2 运行六次`copy-assets`的时间。

```
real  0m7.326s  user  0m5.241s  sys  0m1.319s 
real  0m4.445s  user  0m5.277s  sys  0m1.101s 
real  0m4.847s  user  0m5.346s  sys  0m1.137s 
real  0m4.874s  user  0m5.345s  sys  0m1.217s
real  0m4.847s  user  0m5.420s  sys  0m1.128s 
real  0m4.867s  user  0m5.472s  sys  0m1.167s
```

这些是使用 Node.js v18.5.0 运行六次`copy-assets`的时间

```
real  0m4.686s  user  0m5.105s  sys  0m1.279s
real  0m4.549s  user  0m5.160s  sys  0m1.224s
real  0m4.659s  user  0m5.246s  sys  0m1.309s
real  0m4.884s  user  0m5.127s  sys  0m1.285s
real  0m4.658s  user  0m5.316s  sys  0m1.148s
real  0m4.968s  user  0m5.174s  sys  0m1.292s
```

换句话说，结果大致相同。

对于`techsparx.com`，有 2000 多份文件要拷贝。

# 第二个 Bun 性能测试——渲染一个充满 HTML 文件的网站

我们刚刚在一个大量复制文件的测试中证明了 Bun 并没有改进 Node.js。下一步是看它如何执行更复杂的任务，将 Markdown 文件呈现为 HTML，然后在 HTML 中处理模板。

这种情况下使用的主要软件包是:

*   使用`markdown-it` v12.x 进行降价渲染
*   使用 Cheerio v1.0.0-rc.10 的服务器端 DOM 处理
*   混合使用 EJS (v3.1.x)和 Nunjucks (v3.2.x)的模板处理

像使用`copy-assets`命令一样，`fastq`用于一次并行执行几个渲染。

使用 Bun v0.1.2 渲染`techsparx.com`的时间

```
real  26m1.263s   user  25m20.973s  sys  0m27.235s
real  25m50.435s  user  25m40.301s  sys  0m24.955s
real  25m53.489s  user  25m58.259s  sys  0m25.791s
```

使用 Node.js v18.5.0 渲染`techsparx.com`的时间

```
real  25m46.665s  user  25m42.305s  sys	0m26.839s
real  26m6.209s   user  27m38.675s  sys	0m25.880s
real  25m42.731s  user  27m29.677s  sys	0m25.977s
```

至于`copy-assets`的情况，两个平台的时间大致相同。

# 摘要

对于这种工作负载，Bun 的性能与 Node.js 大致相同。

由于 Bun 团队对性能提出了很高的要求，我们必须考虑发生了什么。超越，就是认识到 Bun 是 Beta 版，可能有一两个 bug。

显然，这不是最好的性能测试。大约 20 年前，我在 Sun Microsystems 的 Java SE 团队工作，并参与了一个致力于性能增强的小组。他们有一长串针对特定工作负载的测试，其中每个测试侧重于一个特定的性能测量。这样，团队可以说 N 版通过`n%`提高了字符串性能。

虽然运行像 AkashaCMS 这样的应用程序需要使用平台的很大一部分，但这并不是一个清晰的测试场景。正如今天在 YouTube 上发现的过于简单的 Bun 演示不能帮助我们理解 Bun 的表现一样，这个测试也不能。我看到 Node.js 团队中有一个基准测试团队，但是他们的工作空间已经闲置多年了。然而，我猜想他们为此开发了一套正确的测试。

我注意到一个包——[https://github.com/majimboo/node-benchmarks](https://github.com/majimboo/node-benchmarks)——似乎有一套在 Node.js 上执行的基准测试。但是，试图使用 Node.js v18.5.0 安装它却无法安装`microtime`包。修补其`package.json`以使用`microtime` v3.1.x 让测试运行，但我不知道如何解释结果。无论如何，该套件中的测试对于这个目的来说是正确的。

要了解 Bun 是否真的比 Node.js 快，以及在哪些功能领域比 node . js 快或慢，需要适当的比较基准测试。因为两个平台的目标是执行完全相同的代码，所以可以在每个平台上运行相同的基准/性能测试来测量相对性能。

当 Ryan Dahl 在 2009 年首次推出 Node.js 时，他通过展示与其他平台进行比较的性能基准，向全世界推销了它。IIRC:他的测试集中在一个有用的指标上，内存占用和每秒 HTTP 操作的数量。

最后，我对 Bun 印象深刻。这是一个新宣布的项目，旨在复制 Node.js，它已经能够处理有点复杂的应用程序。

# 关于作者

![](img/29c63e0d83fb73abcdbba08b04567a14.png)

[**大卫·赫伦**](https://davidherron.com/) :大卫·赫伦是一名作家和软件工程师，专注于技术的明智使用。他对太阳能、风能和电动汽车等清洁能源技术特别感兴趣。David 在硅谷从事了近 30 年的软件工作，从电子邮件系统到视频流，再到 Java 编程语言，他已经出版了几本关于 Node.js 编程和电动汽车的书籍。

*原载于*[*https://techsparx.com*](https://techsparx.com/nodejs/bun/1st-trial.html)*。*