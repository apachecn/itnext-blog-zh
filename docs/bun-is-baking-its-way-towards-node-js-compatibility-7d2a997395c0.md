# Bun 正在朝着 Node.js 兼容性的方向前进

> 原文：<https://itnext.io/bun-is-baking-its-way-towards-node-js-compatibility-7d2a997395c0?source=collection_archive---------4----------------------->

## 在某些情况下，Bun 比 Node.js 更快，但仍有许多缺失的功能。

![](img/62731a37b5844841f9e8ef357a6f7291.png)

**根据早期的 Bun 测试，从 v0.1.8 开始，它正在快速改进，但仍然缺少重要的功能。**

我有兴趣评估 Bun 与 Node.js 高度兼容，同时提供非常好的性能的说法。如果是真的，这是非常有吸引力的，因为我们所有人都喜欢以更高的性能运行我们的应用程序。

Bun、Deno 和 Node.js 提供了大致相同的东西，一个 JavaScript 执行平台，用于服务器或命令行应用程序中的各种非浏览器场景。因为 Bun 力求 Node.js 兼容，所以可以立即使用 Node.js 包生态系统。相比之下，Deno 不能这样做，使 Bun 比 Deno 更有吸引力。

这里的方法是使用复杂的应用程序来测试两件事:

1.  查看 Bun 是否会执行我的应用程序(AkashaCMS 和相关工具)
2.  它是否会比 Node.js 有更高的性能

用一个复杂的应用程序测试 Bun 的声明，给我们提供了关于何时或是否采用 Bun 的真实数据。如果两个条件都满足，就没有理由不转 Bun 了吧？现在，我们很快就会看到，它还没有完全到位。

这篇文章写于 8 月 18 日。Bun 0.1.8/0.1.9 刚刚发布，最近几个版本已经包含了巨大的改进。Bun 团队实现了许多缺失的功能，修复了许多分段错误和其他问题。这是前两篇文章的后续，最近的一篇是——[对 Bun 的性能和与 Node.js 的兼容性进行更深入的测试](https://techsparx.com/nodejs/bun/speed-test.html)。

# 内存数据库查询——但是 Chokidar 不工作

Chokidar 非常流行，它支持动态观察目录树的变化(添加、删除或更改文件)。许多应用程序使用它来支持自动重建，AkashaCMS 就是为了这个目的而使用它的。

AkashaCMS 是一个静态网站生成器，其目的是获取 Markdown、CSS、图像和其他文件，并生成包含 HTML、CSS 等的目录层次结构。它使用 Chokidar 扫描输入目录。然后，它将有关所有文件的数据存储在内存数据库中，在呈现网站时会不断查询该数据库中的信息。

不幸的是，Chokidar 不能在 Bun 上执行，因为`fs.watch`没有实现([参见 Bun 发布队列](https://github.com/oven-sh/bun/issues/832)中的问题 832)。提交的问题包括一个简单的应用程序，当在 Bun 上运行时，错误很明显——不支持: *TypeError: fs.watch 不是一个函数。(在' fs.watch(path，options，handleEvent)'中，' fs.watch '未定义)*

由于无法在 Bun 上使用 Chokidar，我编写了两个测试，加载一个包含大量文件数据的 YAML 文件，然后运行一些查询。一个测试使用 ForerunnerDB，另一个使用 LokiJS，这两个测试都是提供类似 MongoDB 的 API 的内存数据库。您将在以下位置的存储库中找到这些测试:[https://github . com/akashacms/akashacms-perf test/tree/master/bench](https://github.com/akashacms/akashacms-perftest/tree/master/bench)

```
$ node db-forerunner.mjs 
cpu: Intel(R) Core(TM) i7-5600U CPU @ 2.60GHz
runtime: node v18.6.0 (x64-linux)

benchmark                                 time (avg)             (min … max)
----------------------------------------------------------------------------
paths                                  82.39 ms/iter  (74.77 ms … 107.53 ms)
random find                             1.56 ms/iter     (1.03 ms … 7.45 ms)
random siblings                          1.4 ms/iter   (594.58 µs … 5.93 ms)
random indexes                          1.78 ms/iter      (1.1 ms … 8.92 ms)
search layouts using find w/ orderBy   80.23 ms/iter    (74.9 ms … 89.96 ms)
search layouts using find              77.05 ms/iter   (71.96 ms … 91.36 ms)

$ bun db-forerunner.mjs 

error: Cannot find package "child_process" from "/home/david/Projects/akasharender/akashacms-perftest/bench/node_modules/pem/lib/openssl.js"
```

可惜 Bun 还是不支持`child_process`包。此问题阻碍了大量软件包的运行。Bun 团队已经知道了这个问题。

让我们来讨论一下测试场景:

*   `paths` -查询每个索引文件的路径名
*   `random find` -随机选择一个路径名，并获取该文件的数据。
*   `random siblings` -随机选择一个路径名，并检索同一目录中非所选文件的相关数据。
*   `random indexes` -随机选择一个路径名，并检索该目录和每个子目录中文件的相关数据
*   `search layouts` -查找使用给定布局模板的所有文件。一个版本对文件列表进行排序，另一个版本不进行排序。

使用 LokiJS 实现了相同的场景:

```
$ node db-lokijs.mjs 
cpu: Intel(R) Core(TM) i7-5600U CPU @ 2.60GHz
runtime: node v18.6.0 (x64-linux)

benchmark                  time (avg)             (min … max)
-------------------------------------------------------------
paths                    1.13 ms/iter  (720.57 µs … 11.22 ms)
random find            109.44 µs/iter  (74.62 µs … 876.04 µs)
random siblings            92 µs/iter    (42.86 µs … 5.04 ms)
random indexes         135.56 µs/iter    (112.21 µs … 1.4 ms)
search layouts         281.88 µs/iter   (217.32 µs … 1.45 ms)
search layouts sorted    1.33 ms/iter   (870.96 µs … 2.92 ms)

$ bun db-lokijs.mjs 
cpu: Intel(R) Core(TM) i7-5600U CPU @ 2.60GHz
runtime: bun 0.1.8 (x64-linux)

benchmark                  time (avg)             (min … max)
-------------------------------------------------------------
paths                  584.23 µs/iter   (383.01 µs … 2.16 ms)
random find             55.57 µs/iter  (35.71 µs … 794.04 µs)
random siblings        101.52 µs/iter    (41.07 µs … 1.12 ms)
random indexes         122.64 µs/iter   (104.22 µs … 1.34 ms)
search layouts          182.1 µs/iter   (132.24 µs … 1.55 ms)
search layouts sorted    2.55 ms/iter     (1.94 ms … 6.42 ms)
```

在这种情况下，相同的源文件在 Node.js 和 Bun 上执行。

首先，LokiJS 代码比 ForerunnerDB 快得多。如此之快，以至于我已经重写了 AkashaCMS 来使用 LokiJS。使用 ForerunnerDB 实现时，渲染`techsparx.com`网站的时间超过 30 分钟，而使用 LokiJS 时，时间减少到了 5 分钟左右。

其次，请注意，在 Bun 上执行 LokiJS 查询要比在 Node.js 上快得多。这对于未来 Bun 将提供比 Node.js 高得多的性能的前景来说是很好的

# 更多的模板渲染引擎在 Bun 上工作

在[早期测试](https://techsparx.com/nodejs/bun/speed-test.html)中，当在 Mitata 下运行时，几个模板引擎触发了分段错误。我向 Bun 团队报告了这个问题，这些问题和 Bun 中一长串其他 segfault 错误一起得到了修复。我很高兴地告诉大家，我测试过的大多数模板引擎现在都运行正常。

```
$ node render-node.mjs 
cpu: Intel(R) Core(TM) i7-5600U CPU @ 2.60GHz
runtime: node v18.6.0 (x64-linux)

benchmark                       time (avg)             (min … max)
------------------------------------------------------------------
literal                      136.89 ns/iter (119.73 ns … 711.75 ns)
literal list                 437.11 ns/iter   (381.11 ns … 1.02 µs)
ejs-list                      29.48 µs/iter  (24.32 µs … 474.84 µs)
ejs-list-template              3.02 µs/iter     (2.78 µs … 5.39 µs)
ejs-page                      86.84 µs/iter  (74.43 µs … 548.41 µs)
ejs-page-template              6.59 µs/iter     (6.16 µs … 7.52 µs)
handlebars-join                7.32 µs/iter   (4.88 µs … 506.12 µs)
handlebars-list                6.54 µs/iter   (5.06 µs … 640.28 µs)
handlebars-page               16.47 µs/iter    (13.24 µs … 1.17 ms)
liquid-join                   31.73 µs/iter    (17.05 µs … 5.36 ms)
liquid-list                  113.29 µs/iter    (66.86 µs … 5.47 ms)
liquid-page                   199.1 µs/iter   (140.55 µs … 4.12 ms)
nunjucks-join                 45.17 µs/iter    (24.58 µs … 8.53 ms)
nunjucks-list                 81.83 µs/iter    (55.22 µs … 2.29 ms)
nunjucks-list-template         6.52 µs/iter        (6.08 µs … 8 µs)
nunjucks-page                181.09 µs/iter (154.69 µs … 612.43 µs)
nunjucks-page-template        16.51 µs/iter   (13.8 µs … 533.59 µs)
less-css                       2.57 ms/iter     (1.25 ms … 8.26 ms)
markdown-render-simple        28.44 µs/iter    (18.77 µs … 2.03 ms)
markdown-render-test-suite   182.04 µs/iter   (126.57 µs … 1.24 ms)
asciidoctor-render-test-suite 37.49 ms/iter   (26.14 ms … 86.24 ms)
cheerio-simple                  150 µs/iter   (81.09 µs … 11.31 ms)
cheerio-test-suite           351.85 µs/iter   (238.07 µs … 7.83 ms)

$ bun render-node.mjs 
cpu: Intel(R) Core(TM) i7-5600U CPU @ 2.60GHz
runtime: bun 0.1.8 (x64-linux)

benchmark                       time (avg)             (min … max)
------------------------------------------------------------------
literal                     141.16 ns/iter (115.46 ns … 846.57 ns)
literal list                492.97 ns/iter  (434.07 ns … 859.5 ns)
ejs-join                     25.15 µs/iter    (16.64 µs … 5.87 ms)
ejs-list                     42.96 µs/iter    (29.89 µs … 4.14 ms)
ejs-list-template             2.99 µs/iter       (2.8 µs … 3.5 µs)
ejs-page                    123.95 µs/iter    (91.74 µs … 2.66 ms)
ejs-page-template             6.69 µs/iter     (5.75 µs … 1.97 ms)
handlebars-join               4.13 µs/iter     (2.98 µs … 2.49 ms)
handlebars-list               4.59 µs/iter      (3.32 µs … 1.1 ms)
handlebars-page              10.87 µs/iter     (8.14 µs … 1.08 ms)
liquid-join                  36.64 µs/iter    (20.21 µs … 2.48 ms)
liquid-list                 132.03 µs/iter       (85 µs … 1.95 ms)
liquid-page                 230.89 µs/iter   (179.65 µs … 2.46 ms)
nunjucks-join               error: _Loader.call is not a function. (In '_Loader.call(this)', '_Loader.call' is undefined)
...
nunjucks-list               error: _Loader.call is not a function. (In '_Loader.call(this)', '_Loader.call' is undefined)
...
nunjucks-list-template      error: ...
...
nunjucks-page               error: _Loader.call is not a function. (In '_Loader.call(this)', '_Loader.call' is undefined)
...
nunjucks-page-template      error: ...
...
less-css                      2.91 ms/iter     (1.84 ms … 5.87 ms)
markdown-render-simple       38.14 µs/iter    (22.12 µs … 2.16 ms)
markdown-render-test-suite  198.03 µs/iter   (148.47 µs … 1.61 ms)
cheerio-simple               79.21 µs/iter    (51.27 µs … 2.31 ms)
cheerio-test-suite          289.82 µs/iter   (204.11 µs … 1.81 ms)
```

这表明 Bun 现在可以执行每一个模板引擎(我选择测试的那些)而不会触发 segfault。为此向面包队致敬。不幸的是，Nunjucks 引擎不能在 Bun 上工作。我已经向努恩朱克斯小组报告了。

至于性能，有一系列的结果。有些模板引擎在 Bun 上比较快，有些比较慢。引人注目的是，Cheerio 在 Bun 上的速度明显更快。AkashaCMS 广泛使用 Cheerio 进行服务器端 DOM 处理。

每个场景名的第一部分是模板引擎:`literal`使用 JavaScript 文字字符串，`ejs`使用 EJS，`handlebars`使用手柄，`liquid`使用 LiquidJS，`nunjucks`使用 Nunjucks，`less`使用 LESSCSS，`markdown`使用 Markdown-IT，`asciidoctor`使用 AsciiDoctor，`cheerio`使用 Cheerio。

`join`和`list`场景相对简单，如下所示:

```
bench('literal', () => { return `${people.join(', ')}`; });
bench('literal list', () => {
    const ret = `
    <ul>
    ${people.map(person => {
        return `<li>${person}</li>`
    })}
    </ul>
`;
    // console.log(ret);
    return ret;
});
```

你知道在 JavaScript 中可以嵌套模板字符串吗？我没有，但它在这里。

`-page`场景稍微复杂一些，旨在模拟构建一个完整的 HTML 页面。这需要三个单独的模板，其中两个被渲染到第三个中，如下所示:

这在现实世界中是如何应用的，要考虑服务器端呈现网页是如何组织的。要呈现一个主模板的可能性极小。取而代之的是，会有许多小片段，每个片段都是从一个模板中呈现出来的。每个页面都可以由十几个或者数百个单独的模板渲染组合而成。

这些结果表明:

*   文字字符串在 Bun 上比较慢
*   EJS 在面包上比较慢
*   车把在小圆面包上更快
*   液体在面包上速度较慢
*   Nunjucks 不处决 Bun
*   LessCSS 在 Bun 上也差不多
*   Bun 的降价速度较慢
*   AsciiDoctor 不在 Bun 上执行
*   麦片在面包上更快
*   正如所料，更复杂的场景需要更多的计算时间

虽然情况有所改善——Bun 现在执行更多的模板引擎——但模板渲染的性能并没有多少提高。

AsciiDoctor 与 Markdown 相似，都是表示富文本的简单文本格式。Node.js 的 AsciiDoctor 包是从 Ruby 代码交叉编译而来的，并且依赖于 Opal 包。不幸的是，当前版本的 asciidor/Opal 在 Bun 上失败了。但是，这在 Bun 中出现了一个问题，在 0.1.8 中，以下代码失败:

```
import { fileURLToPath, pathToFileURL } from 'node:url';
import * as path from 'path';

console.log(import.meta.url);
console.log(fileURLToPath(import.meta.url));

const __asciidoctorDistDir__ = path.dirname(fileURLToPath(import.meta.url))
console.log(__asciidoctorDistDir__);
```

这种代码模式对于在 Node.js 上模拟 ES6 模块中的`__filename`和`__dirname`变量非常重要。问题是`import.meta.url`中的`file:` URL 无法识别。但是，对 Bun 0.1.9 进行了修复，这个特定的代码现在可以在 Bun 0.1.9 中工作。

对于 EJS 和努恩朱克斯，我还测试了单独编译模板的好处。换句话说，不使用`ejs.render`或`nunjucks.renderString`，而是使用`ejs.compile`和`nunjucks.compile`来产生`template`对象。这些是名称中带有`-template`的测试用例。从这些结果来看，预编译模板可以获得很大的性能提升，但是在预编译模板上 Node.js 和 Bun 之间没有显著的性能差异。

# 用 Bun 复制文件(`fs.copyFile`)快多了

AkashaCMS 实现的另一个场景是将文件从*资产*目录复制到渲染输出目录。这个任务很简单，您生成一个输入文件列表，以及每个文件在输出目录中的位置。然后，您的应用程序使用`fs`包中的函数来复制文件。

您可以使用`fs.createReadStream`和 Streams API 来复制文件，但是这很复杂。在 Node.js 14 中增加了`fs.copyFile`函数，在 Node.js 17 中增加了`fs.cp`函数。这些就简单多了。

因为 Chokidar 不能在 Bun 上工作(见上文),所以我们不能以正常的方式读取资产目录层次。我所做的是生成一个 JavaScript 文件，其中包含一个测试网站中所有资产文件的数组。一个简单的基准测试结果如下:

```
import * as path from 'path';
import { promises as fsp } from 'fs';
import { assets } from './assets.mjs';
import { bench, run } from "mitata";

bench('copy-assets', async () => {
    for (let fileInfo of assets) {
        const outdir = path.join('out', fileInfo.mountPoint);
        await fsp.mkdir(outdir, { recursive: true });
        const outfile = path.join('out', fileInfo.vpath);
        await fsp.copyFile(fileInfo.fspath, outfile);
    }
});

try {
    await run({
        percentiles: false
    });
} catch (err) { console.error(err); }
```

以前这段代码会使用`fs-extra`包中的`mkdirs`和`copy`函数。但是，内置函数可以处理这两个步骤。

发现的另一个问题是 Bun 不支持`fs.cp`功能:

```
copy-assets  error: fsp.cp is not a function. (In 'fsp.cp(fileInfo.fspath, outfile)', 'fsp.cp' is undefined)
```

因此测试使用`fs.copyFile`。

```
$ node copy-files.mjs 
cpu: Intel(R) Core(TM) i7-5600U CPU @ 2.60GHz
runtime: node v18.6.0 (x64-linux)

benchmark        time (avg)             (min … max)
---------------------------------------------------
copy-assets   46.36 ms/iter  (26.71 ms … 149.28 ms)

$ bun copy-files.mjs 
cpu: Intel(R) Core(TM) i7-5600U CPU @ 2.60GHz
runtime: bun 0.1.8 (x64-linux)

benchmark        time (avg)             (min … max)
---------------------------------------------------
copy-assets   10.16 ms/iter    (8.43 ms … 10.88 ms)
```

对于这种复制文件的任务，Bun 要比 Node.js 快得多。

# 丢失的包阻止了许多种类的应用

任何包含使用 Commander 框架的命令行工具的包都会看到以下错误:

```
$ bun node_modules/akasharender/cli.js -- --help

error: Cannot find package "child_process" from "/home/david/Projects/akasharender/akashacms-perftest/node_modules/akasharender/node_modules/commander/index.js"
```

Bun 团队知道`child_process`包不见了，还有其他一些包。这些包被放入 Node.js，因此应该放入 Bun。

例如，`forever-agent`包使用 TLS 包，另一个内置包。当然，任何创建 HTTPS 服务或与之交互的应用程序都将使用 TLS 包。并且，`cacheable-lookup`包加载 DNS 包失败。任何处理使用域名连接到 web 服务的应用程序都必须进行 DNS 查找，并且将被阻止，因为此包不可用。

Bun 团队知道这些问题，我们希望它们能随着时间的推移得到解决。

# 摘要

Bun 正在迅速改进，但是现在还存在明显的功能性缺口。许多非常流行的包因为缺少核心包或函数而无法在 Bun 上执行。

本文是使用 Bun 0.1.8 和 0.1.9 编写的。

我们已经在几个重要场景中证明了 Bun 比 Node.js 更快。在其他场景中，两者的性能大致相当，在某些情况下 Node.js 更有优势。Bun 在一个完整的应用程序上的表现还有待观察，因为许多缺失的功能块阻止了它。

到目前为止，Bun 非常有趣，但是缺少的特性阻止了在 Bun 上运行大多数完整的 Node.js 应用程序。Bun 团队还有很多工作要做。

# 关于作者

![](img/f88392bf042a1dd3054455f7ffb59dea.png)

[**大卫·赫伦**](https://davidherron.com/) :大卫·赫伦是一名作家和软件工程师，专注于技术的明智使用。他对太阳能、风能和电动汽车等清洁能源技术特别感兴趣。David 在硅谷从事了近 30 年的软件工作，从电子邮件系统到视频流，再到 Java 编程语言，他已经出版了几本关于 Node.js 编程和电动汽车的书籍。

*最初发表于*[*https://techsparx.com*](https://techsparx.com/nodejs/bun/test-2022-08-12.html)*。*