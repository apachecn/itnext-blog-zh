# 使用 WebAssembly 和模块联合部署下一代无服务器功能

> 原文：<https://itnext.io/deploying-next-gen-serverless-functions-with-webassembly-and-module-federation-202d2bca497e?source=collection_archive---------5----------------------->

## 不需要部署自动化。第 1 部分，共 3 部分。

![](img/6575ac75c1dd729a113e3763814e10a3.png)

因为 WebAssembly (WASM)是容器的替代方案，提供了比 OS 级隔离更多的不受信任代码的保护，以大多数流行语言的编译目标的中间格式分发，并达到接近本机的速度，所以它是无服务器运行时的理想选择。WasmEdge 是目前最快的 WASM 运行时和堆栈机器，实现了不到 30 毫秒的冷启动。

但是我们如何使用 WASM 开发和部署无服务器功能呢？第一个问题的答案可以在大量关于 C、C++、C#、Go、Rust 工具链的文献中找到……不胜枚举。然而，第二个问题的答案有点黯淡，取决于当前的容器技术，例如与 K8s 和其他容器工具的集成。

让我提供一个替代方案。根本不要安装软件。

这意味着什么呢？这意味着在运行时通过网络(例如从 CDN)传输软件，而不是在服务器上安装软件。这种类似浏览器的分发技术由 [*联合应用*](https://medium.com/codex/federated-applications-ba396d6925b1?sk=c198b9af988537c754edb5cf283adf9e) 在后端使用。联合应用程序组件包括:

*   由许多不同的开发团队构建:没有单一的代码库，
*   设计为在运行时从多个 repos 和多个网络位置加载，
*   不是安装在服务器上，而是根据需要通过网络传输，
*   不需要部署自动化
*   高度可组合，支持动态功能和零停机时间的服务组件运行时绑定，
*   在任何环境中以相同的方式安装，不受供应商或计算交付方式的影响。
*   可以在单个进程中一起运行
*   可以随时重新部署，而无需重新启动流程或中断流程中运行的其他组件

你可以在这里看到这种技术[的一个例子，当 Github 发布新版本时，它直接从 Github 传输代码。目前，它支持用 Javascript 编写的组件的所有上述特性。](http://github.com/module-federation/microlib)

WebAssembly 支持正在开发中，在无服务器的情况下，需要对实现进行更改，即通过在嵌入式 JS 引擎(例如 secondstate.io 的 QuickJS)中执行或通过移植到 Rust 或 AssemblyScript 来支持 WASM 运行时。也就是说，基本支持*是*可用的。

当加载一个模块时，系统生成一组 API，这些 API 调用针对该模块的专用数据源的 CRUD 操作，该数据源也是自动生成的。适配器被加载并绑定到模块在运行时公开的端口，允许它们相互通信或与任何其他本地或远程服务通信。模块还可以通过管道连接在一起，形成本地或分布式控制流，自动回滚以补偿下游事务失败…等等。

那么，在无服务器部署解决方案中，所有这些是如何结合在一起的呢？我们会在[第二部](https://trmidboe.medium.com/deploying-next-gen-serverless-functions-with-webassembly-and-module-federation-1471b94e425c)中看到。敬请关注。感谢阅读。欢迎评论和提问。