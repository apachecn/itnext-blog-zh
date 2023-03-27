# Node.js 脚本编写器:顶级异步/等待现在可用

> 原文：<https://itnext.io/node-js-script-writers-top-level-async-await-now-available-a1e86566d5f6?source=collection_archive---------1----------------------->

Async/await 函数是 JavaScript 程序员的福音，简化了异步代码的编写。它将难以编写和调试的末日金字塔变成了干净清晰的代码，结果和错误都在自然的地方。除了一点之外，一切都很好，我们不能在 Node.js 的顶层代码中使用 await 关键字，但是在 Node.js 14.8 中，这个问题已经解决了。

![](img/ee493af9fbc34148cc2a13be724f9d95.png)

编写脚本可能涉及读取或写入文件、检索外部数据、处理该数据并产生结果，该结果可能被写回外部文件或数据库。在 Node.js 中，这需要异步代码。在脚本中，代码位于 JavaScript 文件的顶层，这使得我们无法使用`await`关键字。

考虑:

```
const fs = require('fs').promises; const txt = await fs.readFile('test.txt', 'utf-8');
console.log(txt);
```

我们刚刚描述的脚本可能会对输入文件进行一些处理，然后将输出写到其他地方。但是让我们关注一下读取数据文件的异步行为。

这是一个传统的 CommonJS 风格的 Node.js 程序。程序员对自己说，让我们使用`await`吧，因为它是一个非常有用的关键字。程序员甚至小心翼翼地使用`require('fs').promises`来获得承诺的`fs`包。但是，运行脚本时，他们会看到:

```
$ nvm use 14.7
Now using node v14.7.0 (npm v6.14.7)
$ node m1.js
/Users/David/nodejs/top-level-async-await/m1.js:8
const txt = await fs.readFile('test.txt', 'utf-8');
            ^^^^^SyntaxError: await is only valid in async function
    at wrapSafe (internal/modules/cjs/loader.js:1172:16)
    at Module._compile (internal/modules/cjs/loader.js:1220:27)
    at Object.Module._extensions..js (internal/modules/cjs/loader.js:1277:10)
    at Module.load (internal/modules/cjs/loader.js:1105:32)
    at Function.Module._load (internal/modules/cjs/loader.js:967:14)
    at Function.executeUserEntryPoint [as runMain] (internal/modules/run_main.js:60:12)
    at internal/main/run_main_module.js:17:47
```

在这篇文章中，我将使用`nvm`来管理正在使用的 Node.js 版本。选择 14.7 版本的重要性将在后面变得清楚。

这就是我们过去几年在 Node.js 中所经历的。我们可以在标有`async`关键字的函数中使用`await`关键字，但是我们不能在顶级代码中使用它。换句话说，该脚本必须写成:

```
const fs = require('fs').promises;  
(async () => {
     const txt = await fs.readFile('test.txt', 'utf-8');
     console.log(txt); 
})();
```

这将异步的`await`调用包装在一个轻量级的`async`函数中，该函数被立即调用。这是一种简单的技术，让我们几乎可以在 Node.js 脚本的顶层编写异步/等待代码。重要提示:这个简单的脚本缺少一些重要的错误处理。

我们真正想要的是像这篇文章中的第一个片段那样编写代码。这是我们能得到的最接近的，直到现在。

# Node.js 14.8 中的顶级异步/等待

Node.js 14.8 今天发布，包含了一个非常重要的进步。在 ES6 模块中，我们现在可以在顶级代码中使用`await`关键字，没有任何标志。

这意味着在扩展名为`.mjs`的文件中，我们可以这样写:

```
import { promises as fs } from 'fs'; const txt = await fs.readFile('test.txt', 'utf-8');
console.log(txt);
```

这个`import`语句是我们在 ES6 模块中访问 promised`fs`模块的方式。否则这和上面的例子是一样的。

我们可以这样运行它:

```
$ nvm use 14.8 
Now using node v14.8.0 (npm v6.14.7) 
$ node m1.mjs
Hello, world!
```

这正如我们所希望的那样，在顶层完美地执行。

这就是相同示例在 Node.js 14.7 中的工作方式

```
$ cat test.txt  
Hello, world!
$ nvm use 14.7
Now using node v14.7.0 (npm v6.14.7) 
$ node m1.mjs
file:///Volumes/Extra/nodejs/top-level-async-await/m1.mjs:6 
const txt = await fs.readFile('test.txt', 'utf-8');
            ^^^^^  
SyntaxError: Unexpected reserved word     
    at Loader.moduleStrategy (internal/modules/esm/translators.js:122:18)
    at async link (internal/modules/esm/module_job.js:42:21)
```

这就是 Node.js 14.8 变化的意义。我们现在可以在顶级 Node.js 代码中使用`await`关键字，给我们自然的异步脚本。我们不再需要面对这个令人不快的错误。

在最后一个节点上，这个特性只有在 Node.js 运行 ES6 模块时才起作用。如果它正在执行一个 CommonJS 模块，就像第一个例子:

```
$ node --version
v14.8.0 
$ node m1.js
/Volumes/Extra/nodejs/top-level-async-await/m1.js:8 
const txt = await fs.readFile('test.txt', 'utf-8');
            ^^^^^  
SyntaxError: await is only valid in async function
     at wrapSafe (internal/modules/cjs/loader.js:1167:16)
     at Module._compile (internal/modules/cjs/loader.js:1215:27)
     at Object.Module._extensions..js (internal/modules/cjs/loader.js:1272:10)
     at Module.load (internal/modules/cjs/loader.js:1100:32)
     at Function.Module._load (internal/modules/cjs/loader.js:962:14)
     at Function.executeUserEntryPoint [as runMain] (internal/modules/run_main.js:72:12)
     at internal/main/run_main_module.js:17:47
```

这是顶部显示的同一个 CommonJS 示例，但这次是用 Node.js 14.8 执行的。两种情况都是一样的错误。

这很好，我们现在可以在 Node.js 脚本中使用异步代码了。但是，让我们尝试一个更全面的例子，而不是这种人为的简单事情。

# **异步下载和处理 Node.js 脚本中的图像**

`node-fetch`包为 Node.js 带来了`fetch`功能，这使它成为从某个地方下载文件的一种极好的方式。当然，下载是作为异步调用来处理的。这个`sharp`包是一个处理图像的优秀工具。让我们一起使用这两者来下载和处理一个图像。

在 Node.js 网站上(在[https://nodejs.org/en/about/resources/](https://nodejs.org/en/about/resources/))有标准 Node.js 徽标的 SVG 版本。正确使用正确的商标图像很重要。让我们编写一个小脚本来下载 SVG，将其大小调整为 300 像素，使其灰度化，并将其写入 JPG。

首先，在一个带有`package.json`的目录中:

```
$ npm install sharp node-fetch --save
```

这将安装所需的软件包。如果你喜欢纱线，就用那个。

然后我们编写一个文件`node-logo.mjs`，包含:

```
import { default as fetch } from 'node-fetch'; 
import { default as sharp } from 'sharp'; 
import { default as fs } from 'fs'; (await fetch('https://nodejs.org/static/images/logos/nodejs-new-pantone-black.svg'))
         .body.pipe(
             sharp().resize(300).grayscale()
                 .toFormat('jpeg')
         )
         .pipe(fs.createWriteStream('node-logo.jpg'));
```

`fetch`函数返回一个承诺，我们使用`await`来处理它的成功或失败。我不明白为什么`node-fetch`文档告诉我们使用`.then`处理程序，而我们可以直接使用`await`。通过这种方式，它产生一个可以通过管道传输到某个地方的 ReadStream。

在管道的中间部分，我们有一个用 Sharp 实现的图像处理器。我们只是告诉它调整图像大小，使其灰度，然后转换为 JPG 格式。

管道的最后一部分使用 WriteStream 将数据发送到命名文件。

在 Node.js 14.7 中，我们不得不将它写在一个异步函数中，如前面所示。在 Node.js 14.8 中，它只是像这样运行:

```
$ node node-logo.mjs
```

它产生了这个:

![](img/a9af298e40f0e9cc99e4d794b27cce77.png)

# 摘要

这一进步将为在 Node.js 中编写简单的脚本提供新的便利。我们可能希望使用 Node.js 完成无数不同的一次性任务。例如，就在几天前，我正在探索与我的静态网站生成器(AkashaCMS)一起使用的确切的图像下载/调整大小/etc 管道。我上周写的例子必须使用异步包装函数，但是今天不再需要了。

这应该消除了 Node.js 采用的一个障碍。我们可以继续使用 Node.js，而不必求助于 Python 或其他语言来编写这样的简单脚本。

*最初发表于*[*【https://techsparx.com】*](https://techsparx.com/nodejs/async/top-level-async.html)*。*