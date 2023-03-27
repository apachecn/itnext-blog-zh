# 对 Bun 的性能和与 Node.js 的兼容性进行更深入的测试

> 原文：<https://itnext.io/deeper-testing-of-buns-performance-and-compatibility-against-node-js-9115763965ed?source=collection_archive---------0----------------------->

## Bun 最终可能会使 Node.js 和/或 Deno 变得无关紧要，但现在说它们都是死项目还为时过早

![](img/0ac0273940e54a326df32646af0a2f20.png)

**Bun 是一个新的项目，旨在与 Node.js 兼容，但是有巨大的性能增益。公开发布还不到一个月，人们就声称 Node.js 和 Deno 项目已经死亡。**

“杀死”一个软件平台需要什么？例如，人们仍在使用 COBOL，有多少关于 Perl、PHP 或 Java 死亡的预测？

Bun 项目宣称它将与 Node.js 平台兼容，同时带来巨大的性能优势。如果是真的，这很容易动摇许多软件工程师放弃 Node.js。但这将需要几年时间才能展开。Node.js 处于非常成熟的地位，Bun 项目在完全取代 Node.js 之前还有很多工作要做。

但是，当 Bun 变得足够稳定和成熟，可以运行目前在 Node.js 上运行的复杂应用程序时，会发生什么呢？

我感兴趣的是评估 Bun 是否能像 Node.js 一样运行我的应用程序。

这些主张引起了很多人的注意。我在 YouTube 上看到一群人运行简单的性能测试，然后吹嘘这些测试如何证明 Bun 比 Node.js 快得多。

当然，简单的性能测试证明不了什么。正如我后面所说的，过分看重一个简单的测试是错误的。

这让我尝试在 Bun 上运行一个复杂应用程序的测试。也就是说，我开发了一个静态网站生成器系统(AkashaCMS)，我用它来构建几个网站，包括`techsparx.com`和`greentransportation.info`。AkashaCMS 足够复杂，可以提供一个很好的测试场景。如果 Bun 能够成功运行 AkashaCMS，并且性能更高，这将证明 Bun 团队的说法。这个想法很好，但结果有缺陷。

我写了一篇[的文章，当时我相信我已经成功地使用 Bun](https://techsparx.com/nodejs/bun/1st-trial.html) (也在[介质](/testing-buns-compatibility-and-speed-with-a-complex-application-bee823a1a1b3)上)渲染了我的网站，但看起来我却意外地运行了 Node.js，我的测试是无效的。

本文旨在重温我尝试做的事情，并展示一些更仔细构建的性能测试。一路上，我发现了 Bun 中的一些 bug，这些 bug 已经被归档在 Bun 发布队列中。

目前 Bun 中的错误和不完整的特性阻止了使用它来运行 AkashaCMS。但是我已经可以用 Bun 执行它的一部分了。正如我们将看到的，这显示了某些方面的性能提升。

# 什么是 Bun，而不是 Node.js

Node.js 是 JavaScript 应用程序的服务器端执行平台。它在 2009 年出现，并声称单线程事件驱动架构如何能够提供比通常复杂的基于线程的架构更好的系统性能。

Node.js 是在浏览器外运行的 JavaScript。自 2009 年以来，一个大型的工具和框架生态系统已经围绕 Node.js 发展起来，涵盖了各种软件开发工具、web 应用程序框架、数据库 ORM 层，甚至 GUI 应用程序工具包。

Bun 像 Node.js，但又不一样。其中 Node.js 基于 Chrome 的 V8 引擎，用 C++编写，Bun 基于 Safari 的 JavaScriptCore，用 Zig 编写。在听说 Bun 之前，你可能从来没有听说过 Zig，我也没有，但是 Zig 声称它比 C++等其他系统编程语言有很多好处。另外，Bun 旨在实现 Node.js 用例，即支持在 web 浏览器之外运行现代 JavaScript。

这并不是第一次尝试在不同的 JavaScript 引擎上运行 Node.js。几年前，有人试图在 ChakraCore ( [GitHub](https://github.com/nodejs/node-chakracore) )上运行 Node.js，但当微软从 Edge 上放弃 ChakraCore 后，这一尝试被放弃了。

小圆面包的卖点是:

1.  与 Node.js 兼容，包括直接使用 Node.js 的包(甚至是本地代码包)
2.  巨大的性能好处:a)是用 Zig 写的，而不是 C++，有一些好处；b)它建立在 JavaScriptCore 引擎之上。该引擎应该比 Node.js 和 Deno 的核心 V8 引擎更快。
3.  它支持直接执行类型脚本和 JSX 代码。

如果 Bun 团队能够完全实现这些卖点，我想 Node.js/Deno 社区中的许多人会转而使用 Bun。

但是，从实际情况来看，Node.js 已经经历了 12 年的完善、改进和错误修复。正如我们将会看到的，Bun 有很多工作要做。

任何有经验的软件工程师在他们的职业生涯中都可能使用过许多不同的编程工具和平台。我们不断评估使用哪些工具，我们中的大多数人都足够聪明，能够看到 Bun 团队的声明，并知道他们做出了一些非常大的声明。我当然希望那些声明能够实现，但是在这个阶段

# Bun 可以杀死 Node.js 和/或 Deno

已经有人声称 Bun 会扼杀 Deno 和 Node.js 两个项目。基本原理就是我刚才提到的:如果 Bun 能够以高度的兼容性实现所有 Node.js，同时保持巨大的性能优势，那么他们显然有一个赢家。

因为 Bun 采用了`node_modules`基础设施，所以它比 Deno 有很大的优势。Deno 很难利用成千上万的软件包。这是我们共同建立的宝贵资源，Bun 将能够充分利用它。

例如，可以使用`npm install`建立一个`node_modules`目录，然后立即与 Bun 一起使用。目标是用同样的`package.json`用`bun install`做同样的事情，但是性能更好。

Bun 不会马上杀 Node.js。正如我们在下面看到的，有许多缺失的特性和许多错误需要修复。此外，为了使 Bun 成为一个自我维持的项目，必须开发所有的流程和后勤支持。

Node.js 开发人员将如何处理与现有生态系统兼容但速度更快的替代方案？

# 简单性能测试的陷阱

YouTube 上已经有几个视频让 Bun 第一次尝试。我看过的每个视频都显示他们运行一些简单的命令，并说天哪，哇，这真快。

有一个众所周知的过分简单的性能测试的谬误。用 Bun 运行一个简单的脚本是否意味着它在实际应用中比 Node.js 快得多？这就是谬误。要验证 Bun 确实更快，需要比几个简单的例子更深入的测试。

# Bun 的现有性能测试

Bun 源代码树包括一套基准测试。要运行这些测试:

```
$ git clone https://github.com/oven-sh/bun.git 
$ cd bun/bench
$ bun install
$ bun run ffi
$ bun run log
$ bun run gzip
$ bun run async
$ bun run sqlite
```

我没有足够的空间来展示所有的结果。但是，让我们看看 SQLite 测试:

```
david@davidpc:~/Projects/bun/bun/bench/sqlite$ bun run bench
$ bun run bench:bun && bun run bench:node && bun run bench:deno
$ $BUN bun.js
[0.02ms] ".env"
cpu: Intel(R) Core(TM) i7-5600U CPU @ 2.60GHz
runtime: bun 0.1.4 (x64-linux)

benchmark                        time (avg)             (min … max)
-------------------------------------------------------------------
SELECT * FROM "Order"         43.62 ms/iter   (40.67 ms … 47.89 ms)
SELECT * FROM "Product"      121.84 µs/iter  (87.83 µs … 928.85 µs)
SELECT * FROM "OrderDetail"  499.15 ms/iter  (470.1 ms … 620.22 ms)
$ $NODE node.mjs
cpu: Intel(R) Core(TM) i7-5600U CPU @ 2.60GHz
runtime: node v18.6.0 (x64-linux)

benchmark                        time (avg)             (min … max)
-------------------------------------------------------------------
SELECT * FROM "Order"        108.33 ms/iter (106.17 ms … 113.98 ms)
SELECT * FROM "Product"       318.2 µs/iter (285.53 µs … 775.32 µs)
SELECT * FROM "OrderDetail"     2.13 s/iter       (2.02 s … 2.37 s)
$ $DENO run -A --unstable deno.js
cpu: Intel(R) Core(TM) i7-5600U CPU @ 2.60GHz
runtime: deno 1.23.4 (x86_64-unknown-linux-gnu)

benchmark                        time (avg)             (min … max)
-------------------------------------------------------------------
SELECT * FROM "Order"         274.7 ms/iter (263.29 ms … 342.62 ms)
SELECT * FROM "Product"      490.34 µs/iter   (377.47 µs … 7.49 ms)
SELECT * FROM "OrderDetail"      1.6 s/iter       (1.43 s … 2.12 s)
```

这个特定的基准对 SQLite 数据库进行`SELECT`查询。对于 Node.js，测试使用`better-sqlite`，Deno 测试使用`"https://deno.land/x/sqlite/mod.ts"`。相比之下，Bun 使用直接集成在 Bun 源代码中的 SQLite 实现。

这些是令人印象深刻的性能差异。

# Bun 的不完整性阻碍了更深入的测试

我的目标是运行更大的应用程序来评估 Bun 的兼容性和性能。在我的例子中，这个应用程序是 AkashaCMS，我为`techsparx.com`、`greentransportation.info`和一些其他网站使用的静态网站生成器。AkashaCMS 使用 Cheerio 进行服务器端 DOM 处理，Cheerio 使用各种模板引擎(主要是 EJS 和 Nunjucks)等等。换句话说，它将为 Bun 与 Node.js 的兼容性提供一个很好的测试案例。

但是，存在许多兼容性问题:

```
$ bun ./node_modules/akasharender/cli.js copy-assets config.js
error: Cannot find package "child_process" from "/home/david/ws/techsparx.com/node_modules/akasharender/node_modules/commander/index.js"
```

`akasharender`命令使用 Commander 来解析参数，它不能工作，因为`child_process`包不存在。Commander 是一个非常流行的用于在 Node.js 中编写 CLI 工具的包，缺少`child_process`内置包会阻止任何围绕 Commander 构建的工具运行。

FWIW，Bun 发布队列包含 [Buns 路线图](https://github.com/oven-sh/bun/issues/159)列出了一堆还没有实现的东西。

我曾希望简单地使用 AkashaCMS 来评估 Bun。既然做不到这一点，下一个最好的解决方案就是选择 AkashaCMS 的部分来进行评估。

# 模板引擎的文本处理、性能以及 Bun 和 Node.js 之间的兼容性

这里讨论的代码在:[https://github . com/akashacms/akashacms-perf test/tree/master/bench](https://github.com/akashacms/akashacms-perftest/tree/master/bench)

`akashacms/akashacms-perftest`用于 AkashaCMS 的性能测试。在`bench`目录中，我打算为 AkashaCMS 的某些特性创建一些类似基准的测试。

例如，考虑:

```
import { bench, run } from "mitata";

let people = ['geddy', 'neil', 'alex'];

// TEMPLATE STRINGS

bench('literal', () => { return `${people.join(', ')}`; });

// EJS

import * as ejs from 'ejs';

bench('ejs-join', () => {
    ejs.render('<%= people.join(", "); %>', { people: people });
});
bench('ejs-list', () => {
    ejs.render(`
    <ul>
    <% people.forEach(function (person) {
        %><li><%= person %></li><%
    }) %>
    </ul>
`, { people: people });
});
```

Bun 项目使用 Mitata 基准执行框架。为了帮助比较 Bun 测试的结果，我也将使用 Mitata。

第一个测试是测试模板字符串中的文本替换。

第二个是使用 EJS 模板引擎的几个场景。在这两种情况下，它都接受一个值数组，并以几种方式格式化该数组。我已经为多个模板引擎实现了类似的代码，从下面的结果可以明显看出。

另外两项测试是:

*   `markdown-render`使用 MarkdownIT 包处理降价。
*   `cheerio`使用`cheerio`进行服务器端 DOM 处理，这是 AkashaCMS 的一个主要特性。

结果显示了这些性能差异:

```
$ npm run bench

> bench@1.0.0 bench
> npm-run-all render:node render:bun

> bench@1.0.0 render:node
> node render-node.mjs

cpu: Intel(R) Core(TM) i7-5600U CPU @ 2.60GHz
runtime: node v18.6.0 (x64-linux)

benchmark            time (avg)             (min … max)
-------------------------------------------------------
literal          133.73 ns/iter (119.76 ns … 661.32 ns)
ejs-join          18.15 µs/iter  (14.89 µs … 583.95 µs)
ejs-list           30.6 µs/iter  (25.98 µs … 398.45 µs)
handlebars-join    5.95 µs/iter   (4.78 µs … 371.31 µs)
handlebars-list    5.97 µs/iter   (4.76 µs … 411.31 µs)
liquid-join       30.26 µs/iter    (18.04 µs … 3.66 ms)
liquid-list       91.91 µs/iter     (64.02 µs … 1.3 ms)
nunjucks-join     46.25 µs/iter    (26.71 µs … 1.09 ms)
nunjucks-list     93.28 µs/iter    (65.62 µs … 1.22 ms)
markdown-render   38.78 µs/iter  (28.93 µs … 603.53 µs)
cheerio          130.17 µs/iter    (78.42 µs … 5.04 ms)

> bench@1.0.0 render:bun
> bun render-bun.js

cpu: Intel(R) Core(TM) i7-5600U CPU @ 2.60GHz
runtime: bun 0.1.4 (x64-linux)

benchmark                 time (avg)             (min … max)
------------------------------------------------------------
literal               129.64 ns/iter (107.87 ns … 534.11 ns)
handlebars-join-once    4.34 µs/iter     (3.07 µs … 1.17 ms)
handlebars-list-once    4.66 µs/iter     (3.44 µs … 1.22 ms)
liquid-join            38.63 µs/iter    (21.23 µs … 2.63 ms)
liquid-list           125.84 µs/iter    (87.41 µs … 2.12 ms)
cheerio                68.81 µs/iter    (43.25 µs … 2.09 ms) 
```

从中可以得出两点:

1.  我无法在 Node.js 和 Bun 上实现所有场景
2.  Cheerio 测试有显著的性能提升，而其他测试的性能提升不太显著

为什么全套场景不能在两者上实现？在 Mitata 下，某些场景存在分段错误。已在 Bun 的问题队列中提交了一个问题，描述了该问题:[https://github.com/oven-sh/bun/issues/811](https://github.com/oven-sh/bun/issues/811)

对于某些模板引擎，Mitata 加上该模板引擎的组合导致了分段错误。一个只运行模板引擎而没有正确执行 Mitata 代码的脚本。

就性能而言，Bun 显示了在两种平台上都能工作的场景的性能增益。啦啦队表现的提高非常有趣。

# Chokidar 暴露了问题

Chokidar 是一个流行的包，用于扫描目录树和动态地注意变化。在 AkashaCMS 中，它用于通知文件何时被更改以及进行自动重建。它起着核心作用，我想知道在 Node.js 和 Bun 之间扫描给定目录的执行时间是否有任何差异。

测试在这里:[https://github . com/akashacms/stacked-directory/tree/master/example/ad hoc](https://github.com/akashacms/stacked-directories/tree/master/example/adhoc)

测试案例是这样的:

```
import { inspect } from 'util';
import { default as chokidar } from 'chokidar';

let watcher;

const start = new Date();
let count = 0;

try {
    await new Promise((resolve, reject) => {
        try {
            watcher = chokidar.watch(process.argv[2]);
            watcher
            .on('error', async (error) => {
                console.error(error);
                reject(error);
            })
            .on('add', (fpath, stats) => {
                // console.log(`add ${fpath} ${inspect(stats)}`);
                count++;
            })
            .on('change', (fpath, stats) => {
                // console.log(`change ${fpath} ${inspect(stats)}`);
            })
            .on('ready', async () => {
                // console.log(`ready`);
                await close();

                const finish = new Date();

                console.log(`time ${(finish - start) / 1000} seconds - ${count} files`);

                resolve();
            });
        } catch (err) { reject(err); }
    });

} catch (errr) { console.error(errr); }

async function close() {
    await watcher.close();
    watcher = undefined;
}
```

它使用 Chokidar 扫描命令行上传递的目录。在我的测试中，我让它扫描了`node_modules`目录，以确保有很多文件需要扫描。Chokidar 根据它正在扫描的文件系统中发生的情况发出几个事件。当它完成初始扫描时，发出`ready`事件。我们使用该事件来关闭 Chokidar 实例并计算时间。

这意味着它可以作为 Node.js 和 Bun 之间的性能比较，但是正如我们将看到的，Bun 不能执行 Chokidar。

使用 Node.js:

```
$ node choke.mjs ../../node_modules/
time 0.763 seconds - 3498 files
```

但是，对于 Bun，测试并不成功:

```
$ bun choke.mjs ../../node_modules/ 
241 |     ); 
242 |   } 
243 |  
244 |   filterPath(entry) { 
245 |     const {stats} = entry; 
246 |     if (stats && stats.isSymbolicLink()) return this.filterDir(entry);
                      ^  TypeError: stats.isSymbolicLink is not a function. (In 'stats.isSymbolicLink()', 'stats.isSymbolicLink' is undefined)
       at filterPath (/home/david/Projects/akasharender/stacked-directories/node_modules/chokidar/index.js:246:17)
       at /home/david/Projects/akasharender/stacked-directories/node_modules/readdirp/index.js:141:79
```

这意味着`fs.stats`返回的 Stats 对象不包含`isSymbolicLink`函数。这个函数从 Node.js v10.10.0 就有了，当然应该有。

问题已经被修复了——我的错误报告[https://github.com/oven-sh/bun/issues/797](https://github.com/oven-sh/bun/issues/797)——注意在底部有一个链接指向更多 Stats 对象方法的问题跟踪实现。截至本文撰写之时，Bun 0.1.5 已经发布，并且`isSymbolicLink`功能已经存在。

很遗憾，Chokidar 出现了一个新错误。

```
114 |         sysPath.resolve(path, evPath), KEY_LISTENERS, sysPath.join(path, evPath) 
115 |       ); 
116 |     } 
117 |   }; 
118 |   try { 
119 |     return fs.watch(path, options, handleEvent);
                ^ TypeError: fs.watch is not a function. (In 'fs.watch(path, options, handleEvent)', 'fs.watch' is undefined)
       at createFsWatchInstance (/home/david/Projects/akasharender/stacked-directories/node_modules/chokidar/lib/nodefs-handler.js:119:11)
       at /home/david/Projects/akasharender/stacked-directories/node_modules/chokidar/lib/nodefs-handler.js:166:14
       at _watchWithNodeFs (/home/david/Projects/akasharender/stacked-directories/node_modules/chokidar/lib/nodefs-handler.js:331:13)
       at _handleFile (/home/david/Projects/akasharender/stacked-directories/node_modules/chokidar/lib/nodefs-handler.js:395:17)
       at /home/david/Projects/akasharender/stacked-directories/node_modules/chokidar/lib/nodefs-handler.js:637:15
```

的确，我们可以证实它并不如此存在:

```
import * as fs from 'fs';
console.log(fs.watch);
```

在 Node.js 上它打印一个函数对象，但是在 Bun 上它当前打印`undefined`。发布日期:[https://github.com/oven-sh/bun/issues/832](https://github.com/oven-sh/bun/issues/832)

# 摘要

烤箱中的一个面包(优先级列表中的[)是这样描述的:](https://github.com/oven-sh/bun/issues/798)

> *提高 Node.js 兼容性。快递需要工作。像 chalk，debug，discord.js 这样的流行包需要工作。需要实现 child_process 和 net。我们需要更多的测试。这些包应该通过实现较低级别的 Node.js APIs 来支持，而不是通过侵入兼容层来支持。从长远来看，Bun 可能希望用本机代码实现 Express，但今天我们只让原语工作良好。*

是的，上面清楚地表明了 Node.js 兼容性目前是缺乏的。但是，这只是修复错误或添加缺失功能的问题。正如优先级列表所说，让更多的人参与项目将有助于解决这些问题。

如果 Bun 项目允许我提几个建议:

*   客观准确的兼容性测试列表。换句话说，最好有一个核心 Node.js 模块的列表，以及当前的实现状态。要获得灵感，看看[https://node.green/](https://node.green/)
*   应该在数据库中跟踪性能测量，这样我们就可以随时观察改进情况。
*   应该有比现在更多的性能测量。(我应该考虑贡献我上面描述的场景)

第一个建议，Node.js API 兼容性测试来自于我在 Sun Microsystems 的 Java SE 团队工作了 10 多年的经验。([见我那篇关于我从 Java 过渡到 Node.js 的文章](https://blog.sourcerer.io/why-is-a-java-guy-so-excited-about-node-js-and-javascript-7cfc423efb44))有多个组织为多个平台创建的多个 Java 实现。多个 Java 实现之间的兼容性由相应的 TCK 来保证。我不记得缩写是什么意思了，但它是一个兼容性测试，或者更准确地说，是对规范的符合性测试。

Bun 团队正着手创建 Node.js API 的第二个实现。这使它处于与独立开发的 Java 实现相同的角色。

作为 Node.js 或 Bun 的潜在客户，我们需要了解两者之间的兼容程度。我们的工作可能是维护一家价值数十亿美元的企业的网站，在两者之间做出糟糕的选择可能会扼杀这家企业。

在 Java 生态系统中，一致性/兼容性测试是建立可信度的一大步。通过这些测试是一件大事，并允许一个项目或产品自称为 Java 兼容的。

Node.js 没有这样的测试套件，也没有 Node.js API 的正式规范。`https://nodejs.org/api/index.html`的 API 文档很好，但不是正式的规范。在 Java 生态系统中，一致性测试是通过仔细解析规范中的细节来编写的。

随着时间的推移，可能会有其他尝试来构建 Node.js API 的独立实现。像今天的 Bun 团队一样，这些团队将面临同样的问题，即确定与 Node.js 的兼容程度。

理论上，团队需要梳理 Node.js API 文档。一致性测试可以被开发，理想地作为一个独立的项目，可以被 Node.js 和 Bun 团队使用。

然而，这需要大量的工作，谁有资金支付这样的测试套件？

目前，Bun 团队将创建他们认为适合 Bun 的任何测试。如果我在 Bun 团队工作，我会寻找一种方法来利用 Node.js 团队开发的测试。我甚至可能在一个单独的存储库中这样做。Bun 和 Node.js 团队的目标是共同合作开发 Node.js API 的测试套件。

我想到的一个想法是，有人可以开发一个网站或 GitHub 存储库，项目所有者可以通过它来声明“*与 Bun* 一起工作”……越来越多的“*与 Bun* 一起工作”包应该有助于证明 Bun 是一个不错的选择。“【T4 作品与包子”倡议的发起者会用什么标准来验证真实性？

可能我跑题太远了。

这篇文章证明了两件事:

1.  试图在 Bun 上运行重要的应用程序还为时过早，因为它缺少许多功能。
2.  根据功能的不同，性能会有所提高。

底线是如果 Node.js 社区参与进来，Bun 将会成功。在推荐将 Bun 用于重要应用程序之前，还有许多工作要做。性能提升非常有希望。

# 关于作者

![](img/b0ae04524ec14860d9209ec45070aa7c.png)

[**大卫·赫伦**](https://davidherron.com/) :大卫·赫伦是一名作家和软件工程师，专注于技术的明智使用。他对太阳能、风能和电动汽车等清洁能源技术特别感兴趣。David 在硅谷从事了近 30 年的软件工作，从电子邮件系统到视频流，再到 Java 编程语言，他已经出版了几本关于 Node.js 编程和电动汽车的书籍。

*原载于*[*https://techsparx.com*](https://techsparx.com/nodejs/bun/speed-test.html)*。*