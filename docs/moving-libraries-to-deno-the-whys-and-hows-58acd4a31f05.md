# 将图书馆迁移到 Deno:为什么和如何

> 原文：<https://itnext.io/moving-libraries-to-deno-the-whys-and-hows-58acd4a31f05?source=collection_archive---------0----------------------->

![](img/c3511bcf88878ab330b8827aa9aa33ea.png)

粗略地看一下关于 Deno [](#dd75)的众多介绍性文章，你会发现它是 Node.js 的替代品，避开了 NPM，内置了 TypeScript，并通过权限标志实现了安全性。潜在的信息经常暗示与现有 Node.js 生态系统的不兼容性，强调“让我们从头开始”的方法，完全没有抓住要点。坦率地说，我认为这些甚至都不是 Deno 的主要特性，将它与现有的 Node.js 生态系统相提并论从根本上说是被误导了。

自从 Deno 问世以来，我一直关注着它，并于去年在其中重写了我的同构 JavaScript 库 [](#1563) [](#8b63)[⁴](http://20e5) 。根据这一经验，我想让您相信 Deno 不仅是一个有前途的平台，而且已经是一个有能力编写面向浏览器、Deno 和 Node.js 的同构 JavaScript 的环境。

*免责声明*:我与 Deno 公司没有关联，虽然我接受贿赂，但我没有从他们那里接受任何贿赂(至今)。观点是我自己的，不代表其他人的观点。

# Deno 的案例

在浏览器和服务器之间共享代码，即所谓的*同构 JavaScript* ，一直是 Node.js 带来的主要优势之一。它减少了代码量，但更重要的是，它减少了我们在处理多种语言时必须进行的上下文切换。不幸的是，由于历史原因，我们现在有不同的 API 在 Node.js 和浏览器中执行类似的任务。和解的过程是缓慢而痛苦的。另一方面，Deno 依赖于标准的 web APIs，潜在地消除了上下文切换。你甚至可以在 MDN 中查看 Deno 是如何支持每个 web API 的。

遵守标准 API 变得更加重要，因为 web 不再严格区分客户端和服务器:我们现在有服务工作人员，他们实际上是生活在客户端机器上的“服务器”，而边缘计算工作人员(如 Cloudflare 工作人员)位于两者之间，他们都实现标准 web APIs 的子集。

如果你在过去的十年中听说过 JavaScript，你可能会熟悉*the fatigue*——一个末日场景，一个流氓`node_modules`文件夹消耗了网络上所有可用的空间...当然，我开玩笑，但是 jest，流行的测试运行程序，引入了大约 300 个依赖项，其他工具和框架也紧随其后。数量本身并不是问题，但是，每个这样的工具都会成倍地增加工具链的复杂性，使其更难设置和维护。最重要的是，每当 NPM 出现一些不知名的字符串填充库丑闻时，你肯定会在你的依赖项中找到罪魁祸首，通常是通过开发工具。

Deno 用几种方法解决了这个问题。首先，Deno 将有问题的文件夹重命名为`cache`,极大地抑制了其潜在的破坏性。其次，Deno 引入了标准库 [⁵](#20b96) ，认为 lodash 符合 node.js apis。最后，Deno 有内置的工具链...

# Deno 工具链

开箱即用，Deno 带有一个格式化程序、一个 linter、一个测试运行程序、一个覆盖报告程序、一个 bundler、一个 doc 生成器，Ryan 知道还有什么，所有这些都只需一个控制台命令就能完成。这是开发者体验的巨大进步。为了产生高质量的类型脚本代码，你不再需要安装更漂亮的，ESLint，Jest，TS 编译器，Webpack，并花一天的时间试图通过各种插件，预置和仪式化的舞蹈让它们一起工作。

这些工具本身大部分是用 Rust 编写的，并且通常比它们的 JavaScript 对应物快一个数量级。有些是从零开始写的，比如`deno_lint` [⁶](#edc3) ，有些是从现有的 Rust 项目中采用的，比如格式化程序使用`dprint` [⁷](http://98c4) ，各种编译捆绑工具依赖`swc` [⁸](#7286) 。也就是说，Deno 仍然包括用于类型检查的`tsc`,仅仅是因为缺少替代方案。

Deno 工具链的一个不被重视的特性是它所采用的固执己见、零配置的方法。它采用事实上的行业标准，并促使用户遵循这些标准。因此，举例来说，格式化程序和 linter 的配置选项被有意地限制，尽管如果必要的话，总是有一种选择退出的方法。

实际上，确保项目中的代码质量可以归结为一些没有外部依赖性的一行程序:

```
# check formatting
deno fmt --check
# run the linter
deno lint
# run tests collecting coverage
deno test -A --coverage=cov
# check coverage
deno coverage cov
```

不可否认的是，这些工具可能不像它们流行的同类产品那样功能丰富，有时它们还很粗糙。linter 可能缺少自定义 ESLint 规则集中的一两条规则。formatter 虽然与 Prettier 不相上下，但在大多数情况下，有时会让您感到惊讶，例如，现在它有一种在字符串模板中格式化 HTML 的特殊方法，如 Lit 和 HyperHTML 所使用的方法。

# 交叉编译和分发

用 Deno 编写的代码可以被编译并捆绑到内置的捆绑器中，供浏览器或 Node.js 使用。然而，如果您决定使用为 Deno 编写的 TypeScript 代码，就像在 Typescript 编译器(`tsc`)中一样，例如在我们通常的 Node.js 工具链中，您将遇到一个关于文件导入的小而棘手的问题。Deno(像浏览器一样)完全按照您的要求导入，它需要导入文件的完整路径(扩展名和全部),无论是 URL 还是本地路径:

```
import {...} from "https://deno.land/std@0.123.0/testing/asserts.ts"
import {...} from "./mod.ts"
```

同时，`tsc`更喜欢 magic，并将在导入中标记`.ts`扩展的使用。细节就省了吧，不过就是那种`tsc`不对，Deno 对的情况。这个问题已经向 Typescript 团队提出了无数次，并在⁹进行了长时间的讨论。

在为 Node.js 工具链分发 Deno 代码时，已经有了解决这个问题和其他问题的变通方法。最近，Deno 团队推出了`dnt`[⁰](#8391)——一个从 Deno 代码创建 NPM 包的构建工具。这是一个雄心勃勃的工具，它解决了导入问题，将您可能有的第三方依赖项映射到它们的 NPM 包，为 Node.js 中没有的 Deno 或特定于 web 的 API 添加了垫片，等等。

在实践中，您可以从 Deno 工具中获得一个单一的 TypeScript(和/或 JavaScript)代码库，然后可以轻松地将其分发给浏览器和 NPM。它只需要一个使用`dnt`的小构建脚本，如果您也想自动发布到 NPM，可能还需要您的 CI 脚本中的几行额外代码。

还应该提到的是，在撰写本文时，Deno 团队正忙于改进与现有 Node.js 包的兼容性，以便 Deno 可以无缝地使用那些依赖于特定于节点的 API 的包。同构 NPM 包已经通过 cdn 广泛使用，如 esm.sh、Skypack 等。

# 约定

Deno 中有一些常见的实践，在编写库或简单地浏览生态系统时，您可能会发现这些实践很有帮助。Deno 并不强制执行这些规则，它们最多被认为是约定，您可以随意丢弃其中的任何一个或全部:

*   Deno 库的主入口点通常称为`mod.ts`，而`index.ts`则留给应用程序。
*   第三方依赖项被收集到一个名为`deps.ts`或`dev_deps.ts`的文件中，用于开发人员专用的依赖项。这种方式可以集中更新版本，就像在 NPM 的`package.json`一样。
*   snake_case 用于文件名，测试文件以`_test.ts`结尾。
*   库的文件夹结构是扁平的，大多数导出的文件位于主文件夹和嵌套文件夹中，用于支持和辅助元素，如用于 API 文档的`docs`，用于用法示例的`examples`等。由于 Deno 使用 URL 进行导入，因此您的依赖项的导入路径会更短。

总的来说，我在 Deno 中移植和维护库的经验比以前的工具链有了很大的改进。事情并不总是美好的:在 VSCode 中的调试偶尔会中断，在测试 API 被彻底检查时，覆盖范围一度不精确，导入一些特定于节点的库仍然很棘手。然而，当某个东西坏了，很容易找到原因，或者至少是归咎于谁。而对于我们通常收集的 Node.js 工具，这通常会变成对十几个存储库的徒劳追逐。Deno 团队也积极响应并修复和改进平台。

[**1**]:[Deno——JavaScript 和类型脚本的现代运行时](https://deno.land/)

[ **2** ]: [结构化:高性能 JavaScript 应用程序的数据结构。](https://github.com/zandaqo/structurae)

【**3**】:[compago:现代 web 应用的极简框架。](https://github.com/zandaqo/compago)

[**4**:[object 64:将 MongoDB ObjectIDs 转换成 URL 友好的 base64。](https://github.com/zandaqo/objectid64)

[**5**:[Deno 标准库](https://github.com/denoland/deno_std)

[ **6** ]: [deno_lint:用 Rust 编写的 JavaScript 和打字稿的高速 linter](https://github.com/denoland/deno_lint)

【**7**】:[d print:用 Rust 编写的可插拔、可配置的代码格式化平台。](https://github.com/dprint/dprint)

[**8**:[SWC:基于 Rust 的 Web 平台](https://github.com/swc-project/swc)

[**9**:[允许自愿。导入路径的 ts 后缀问题#37582 microsoft/TypeScript](https://github.com/microsoft/TypeScript/issues/37582)

[ **10** ]: [dnt: Deno 到 npm 包构建工具。](https://github.com/denoland/dnt)

[**11**:[改善节点兼容模式问题#12577 denoland/deno](https://github.com/denoland/deno/issues/12577)