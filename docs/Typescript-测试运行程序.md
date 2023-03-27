# 用 80 行代码编写您自己的 Javascript/Typescript 测试运行程序>

> 原文：<https://itnext.io/write-your-own-javascript-typescript-tests-runner-in-80-lines-of-code-8eb00eeb2e63?source=collection_archive---------3----------------------->

![](img/33afa3604c7f55cd40b77f53442d8afb.png)

声音图像来自网络

在 javascript echosystem 中有许多测试运行程序，Jest、Karma、艾娃、Mocha 等等。Deno 有一个内置的，在 echosystem 中内置 test 和 fmt 是更好更成熟的 IMO。

那么这个帖子的动机是什么呢？基本上是为了表明在当前的 nodejs echosystem 中设计一个高性能的跑步者并不难。

最初我想用 Rust 写一个，所以我问自己，我的基本技术要求是什么？

1.  我需要**测试** **源代码**包括它的依赖项，所以我需要一个捆绑器，为此我可以使用 [SWC](https://github.com/swc-project/swc) 。

> SWC 是一个可扩展的基于 Rust 的平台，用于下一代快速开发工具。它被 Next.js、Parcel 和 Deno 等工具以及 Vercel、字节跳动、腾讯、Shopify 等公司使用。

**2。**如果我要支持 Typescript，我需要一个将转换成 javascript 的前一步**——也可以使用 [SWC](https://github.com/swc-project/swc) 。**

**3。**我需要**在 javascript 运行时上执行代码**——嗯，我可以尝试使用来自 deno 项目的[rusty _ V8](https://github.com/denoland/rusty_v8)crate(crate = lib，类似于 npm 的节点模块)，该 API 看起来非常好——创建一个脚本并在 [isolate](https://v8docs.nodesource.com/node-0.8/d5/dda/classv8_1_1_isolate.html) 上运行它(如果我目前理解的话)。

**4。我需要设计一个测试结构和套件本身的 API，这部分必须在 Javascript 中，因为这是用户将使用的东西，用户是 Javascript 开发人员。**

当我想到 4 个要求时，我明白用 Rust 写一个并不值得，也不会以 80 行结束:P

## Run-Chewy CLI

长话短说，我决定在 nodejs 中编写一个小小的 runner:[run-chewy](https://github.com/LironHazan/run-chewy)这样我就可以更好地理解 SWC 核心包并使用工人线程。SWC 公开了一个易于使用的 javascript API，顺便说一句，这个插件系统可以添加你自己的访问者，也很不错..

**切入点:**

测试运行器是一个 **CLI 工具**，为了简单起见，我将传递给它一个参数，要求用户指定他是否使用 Typescript。
另一个选择是像 chewy.json 这样的配置文件

项目入口点是触发 run()方法的 **cli.js** 文件。

```
const yargs = require('yargs/yargs')
const { hideBin } = require('yargs/helpers')
const argv = yargs(hideBin(***process***.argv)).argv
const ChewyRunner = require('./lib/runner');

if (argv.isTS) {
    ***console***.log("running")
    ChewyRunner.*run*({isTS: argv.isTS});
} else {
    ***console***.log("supply the isTS arg to decide if running js or ts tests")
}
```

**亚军:**

对跑者最基本的技术要求是:
1。我们需要运行测试文件的列表，该列表应该从我们正在运行的当前文件夹中获取。
2。我们需要迭代列表并解析每个文件，以获得完整的源代码(包括依赖项)。
3。如果是 Typescript，我们需要在执行之前将源代码转换成 javascript。
4。我们需要并行执行这些代码。
5。我们需要打印测试结果/打印错误，以防任何运行时故障。

**免责声明** :
以上代码在不到 60 行的代码中涵盖了我们定义的基本需求，但是**缺少了** the:
1。对于每个测试，我们都启动一个新的工作线程，而不是通过使用池来限制创建。
2。run 方法是异步的，但是我没有捕获 worker 错误事件并拒绝。
3。在这个实现中不支持 es 模块。
4。我没有使用 Typescript 来编写 runner :P

在我看来，实现中有趣的部分是使用 swc javascript API。
了解其配置的一个好地方是:

[](https://swc.rs/playground) [## SWC SWC 游乐场

### SWC 是一个可扩展的基于 Rust 的平台，用于下一代快速开发工具。它被工具使用，例如…

西南铁路公司](https://swc.rs/playground) 

**API:** API 应该包括一个分组块，用于运行几个作为“套件”的测试，以及套件的描述，对于测试块也是如此，并且应该支持异步功能。
这个块应该是热切的并立即运行，类似于我们熟悉的常见测试框架的“描述”结构。
API 应该公开比较器(不是必须的，因为我们可以使用外部库)。

```
Chewy.suite('suite', async () => {
    ***console***.log('suite cb');
    Chewy.test('test', () => {
        Chewy.assertEq([[[1, 2, 3]], 4, 5], [[[1, 2, '3']], 4, 5]);
    })
})
```

**免责声明** :
上面的代码在大约 20 行中涵盖了我们为 API 定义的基本要求，但是有一些缺失的特性，比如可能更好的错误处理、before/teardown 挂钩、跳过测试/套件实现以及可能的 mocking utils..

综上所述——用 nodejs 编写一个简单的几乎没有依赖的运行器是非常容易和有趣的，我没有发布到 npm，所以不要在家里使用它:P

目前就这些:)
感谢您的阅读，欢迎您开始[回购](https://github.com/LironHazan/run-chewy)，祝您编码愉快！

Liron \m/\m/