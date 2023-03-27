# 什么是 WebAssembly？

> 原文：<https://itnext.io/what-is-webassembly-e1a06f856974?source=collection_archive---------3----------------------->

## 它是什么，为什么它对 web 开发的未来很重要

![](img/e75aee38b6a76f32dfe9a1696ea6a23c.png)

低水平、高性能—图片来自 [Pixabay](https://pixabay.com/?utm_source=link-attribution&utm_medium=referral&utm_campaign=image&utm_content=1236578) 的 [MustangJoe](https://pixabay.com/users/MustangJoe-2162920/?utm_source=link-attribution&utm_medium=referral&utm_campaign=image&utm_content=1236578)

## 什么是 WebAssembly？

自 2015 年 [WebAssembly 社区团体](https://www.w3.org/community/webassembly/)成立以来，WebAssembly 逐渐流行起来，但它到底是什么呢？

正如作者在[https://webassembly.org](https://webassembly.org)所定义的:

> “WebAssembly(缩写为 *Wasm* )是基于堆栈的虚拟机的二进制指令格式。Wasm 被设计为 C/C++/Rust 等高级语言编译的可移植目标，支持客户端和服务器应用程序在 web 上的部署。”

WebAssembly 提供了一个基于精简堆栈的虚拟机，通过利用快速加载的二进制格式(也可以转换为文本格式以便调试)，允许 web 应用程序以接近本机的速度运行。

这是一种完全不同的 web 前端软件开发方法，与通常使用的大量 JavaScript 库形成鲜明对比，这些 JavaScript 库为一些问题提供了多层兼容性解决方案，而这些问题可能在五年或十年内都不存在。[四大浏览器加上 node 已经采用了它](https://webassembly.org/roadmap/)，这是朝着最终实现跨浏览器兼容性迈出的巨大一步，高性能 web 应用是默认而不是例外。

## 为什么 WebAssembly 很重要？

如果我们回顾一下 JavaScript 的历史，它最初被称为 *Mocha* ，最初被认为是一种完整的 web 应用程序语言，而不仅仅是用于前端 UI。随着 Node 的广泛采用，这一点用了近 20 年时间才完全生效，当时 Node 对几乎所有人来说都是一个新概念。

其原因主要是由市场营销推动的，因为 Sun 将 JavaScript 吹捧为 Java 的伴侣语言，这似乎经常在某些大型企业文化中引起共鸣，这些企业文化将 Java 作为其主要应用程序语言，并将 web 前端视为只是从应用程序中获取数据的地方。然而，并不是所有东西都是简单的 CRUD 企业应用程序。如果你只有一把锤子，所有的东西看起来都像钉子。嗯，有时候你不需要锤子，你可能需要一把锯子或者数控激光。

WebAssembly 只是第二种被 web 浏览器自然理解的语言，第一种语言已经陷入了无休止的标准遵从性问题、严重的性能问题、关于如何实现解决方案的相互冲突的概念以及庞大笨重的框架的浪潮中，从长远来看，这些框架通常[造成的问题比它们解决的问题](https://medium.com/@kennethreilly/what-is-technical-debt-4c087d30a056)更多。因此，在经历了 25 年的良好运行之后，是时候至少让 T2 的另一种语言尝试一下了。

## 架构概述

WebAssembly 是一个 [*虚拟指令集架构*](http://webassembly.github.io/spec/core/intro/introduction.html#scope) (虚拟 is a)，它有效地允许熟练的开发人员构建快速加载的模块，运行速度几乎与编译的 C 或 C++一样快，就好像这些功能直接编译到 web 浏览器本身一样。WebAssembly 文件有两种不同的格式，可以相互转换:

*   ***。wat*** 文件:人类可读的 [S 表达式](https://en.wikipedia.org/wiki/S-expression)语法文件
*   ***。wasm*** 文件:机器可读的编译二进制文件

编写 Web 程序集文本( ***)。wat*** )文件手工处理当然是一种选择，但不是唯一的一种。幸运的是，有许多方法可以生成和使用 WebAssembly 文件。以下是其中的几个例子:

*   一个预编译的 *C++到 WebAssembly* 的工具链，[在这里可用](https://webassembly.org/getting-started/developers-guide/)
*   一个类型脚本到 web 汇编编译器，[汇编脚本](https://github.com/AssemblyScript/assemblyscript)
*   在线 web 组装记事本， [WasmExplorer](https://mbebenita.github.io/WasmExplorer/)
*   在线 IDE， [WebAssembly Studio](https://webassembly.studio/)
*   [许多其他编译器选项](https://github.com/appcypher/awesome-wasm-langs)

随着 WebAssembly 被 Chrome、Edge、Firefox、WebKit 和 Node 的作者广泛采用而变得越来越受欢迎，将会有更多的组件可用。很容易看出这项技术不会很快消失，并且很可能会对前端开发和整个 web 技术产生非常大的影响。

## 绩效基准

就像任何一种酷的新技术一样，在我们兴奋不已、盲目跟风之前，有必要问一个古老的问题:*为什么这是一个好主意，我应该为此烦恼吗？*

关于 web 应用程序性能问题的信息和讨论并不缺乏，尤其是当涉及到单页面 web 应用程序和具有大量附加依赖的大型前端界面时。在软件行业中，有很多关注点集中在解决性能问题的概念上，实际上，这些问题是由于选择使用某种框架 X 或技术 Y 来制作 web 应用程序而自我诱发的，因为其他人都在这样做，这是很酷的事情，对吗？

让我们来看看来自[的一些基准测试，它是一个优秀的在线工具](https://wasmboy.app/benchmark/)，这是一篇关于使用仿真器对 WebAssembly 进行基准测试的文章的主题:

![](img/2b8b76980b2936ad51bfc5e1d0b13ef4.png)

蓝色的 WebAssembly:顶部图形越低越好，底部越高越好

上面两张图是 WasmBoy 基准测试的结果，使用游戏“Back to Color”运行，这是一个演示游戏，带有各种音频和图形事件，旨在展示 GameBoy Color 的功能。基准测试是在 2017 年的 13 英寸 MacBook Pro 上的 Safari 中进行的。

[编译成 WebAssembly 的 AssemblyScript](https://github.com/AssemblyScript/assemblyscript) 显示为蓝色，竞争者的 TypeScript 直接编译成 ES6(黄色)，闭包编译器(绿色)。该测试非常有用，因为模拟器的 TypeScript 逻辑基本上是相同的，这允许我们测试每个编译器目标之间的性能差异。

请注意，这些指标是比较同类产品(与其他高效工具)，这意味着竞争对手仍然是游戏模拟器*和* *的高性能 ES6 实现，而不是您的典型网站 JavaScript* 。编译后的 WebAssembly 应用程序和典型的笨重的框架应用程序之间的速度差异可能会大得多。

在上面的图表中，显示了每帧的*运行时间*，这是绘制每帧所需的总时间(越低意味着越快)。WebAssembly 的这一时间远远低于任何一家竞争对手。

下图显示了平均帧吞吐量，即每秒帧数。这个度量显示了前两千帧左右的不同介绍场景以不同的方式对模拟器的每个实现造成的负担。平均而言，WebAssembly 版本比其他版本具有更高的吞吐量，尤其是对于介绍动画。

## 后续步骤

WebAssembly 在弥合 web 应用程序的客户端和服务器组件之间的差距方面显示出了很大的潜力，这在我们进入分布式计算和开放 web 标准的时代时尤为重要。随着越来越多的本地数据和能源变得可用，利用现代个人计算设备不可思议的能力将是朝着更加可访问、高效和有趣的未来的正确方向迈出的重要一步。

虽然这项技术可能不会为每个可能考虑使用它的人提供即时回报，但对于那些有理由尽早采用的人来说，它有巨大的优势，可以立即开始获得回报。尤其是在 [AssemblyScript](https://github.com/AssemblyScript/assemblyscript) 的情况下，因为它允许前端开发人员利用他们可能比 C++或 Rust 更熟悉的现有语言。

例如，使用 AssemblyScript，前端开发人员可以将所有性能关键的功能(如搜索算法或游戏人工智能的紧密循环)迁移到超快速编译的二进制格式中，其运行速度几乎与原生应用程序一样快(可能更快，具体取决于程序员)。

## 结论

有关 WebAssembly 的更多信息，请查看以下资源:

*   [WebAssembly 社区组](https://webassembly.org)
*   [这篇关于 freeCodeCamp 的伟大文章](https://medium.freecodecamp.org/get-started-with-webassembly-using-only-14-lines-of-javascript-b37b6aaca1e4)
*   [使用 WebAssembly API，来自 Mozilla Dev Network](https://developer.mozilla.org/en-US/docs/WebAssembly/Using_the_JavaScript_API)

![](img/caa823c9dc011234f08655014384f838.png)

我希望您喜欢这篇关于 WebAssembly 的文章，这是构建 web 应用程序的一种强大的革命性的新方法。

感谢阅读！

> 肯尼斯·雷利( [8_bit_hacker](https://twitter.com/8_bit_hacker) )是 [LevelUP](https://lvl-up.tech/) 的 CTO