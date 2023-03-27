# 🎉热控制器简介:零配置、热重装控制器库

> 原文：<https://itnext.io/introducing-hot-controller-a-zero-config-hot-reloading-controller-library-1d99c5390853?source=collection_archive---------6----------------------->

今天我真的很兴奋终于宣布热控制器。零配置，热重装控制器库。
[在 GitHub 上查看](https://github.com/hot-controller/hot-controller)

![](img/abda5f1f4e59091e9a3ef40f766ba442.png)

# 特征

*   🔧零配置需要开始。
*   🔥控制器上的热模块更换
*   🍾`async` / `await`支持
*   ⚙️作为中间件与`express`一起使用或独立使用。

# 问题是

多年来一直在开发 node.js 应用程序。开始使用一个简单的 hello-world-server 非常容易和快速，但是它缺乏结构和一些特性，比如 async/await(是的，你可以很容易地使用 async/await，但是你仍然需要为每个 async-route 编写一些不必要的代码)。让我们同意，每次你改变路线时都必须重启服务器，这有时很烦人。

# 解决方案

要么像大多数人一样“git 克隆”一个启动程序/样板文件。但是 9/10 的时候，你从来没有得到你想要的结构，最终删除了一堆文件和代码行和注释。

为了避免这一点，我建立了热控制器。
Hot-controller 是一个零配置或最低配置的控制器库，内置了对热模块替换(无需在更改时重启服务器)和异步/等待的支持。Hot-controller 可以在 express-application 中用作中间件，也可以使用 CLI 独立使用。离你的第一个控制器不到一分钟了。

## 示例控制器

# 尝试一下

运行" **npm i -g 热控制器**"

这个包带有一个内置的 CLI。运行“**热控制器初始化**”开始。该命令创建一个名为“控制器”的文件夹，并在其中放置一个简单的示例控制器。准备就绪后，运行“ **hot-controller** ”启动开发服务器，激活 HMR。

在 [README.md](https://github.com/hot-controller/hot-controller) 中了解更多信息。

让我知道你的想法。我感谢任何反馈和贡献。你可以通过推特( [@philipodev](https://twitter.com/philipodev) )联系到我。