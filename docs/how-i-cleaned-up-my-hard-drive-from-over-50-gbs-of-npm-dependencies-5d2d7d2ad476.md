# 我如何从超过 50gb 的 npm 依赖项中清理硬盘。

> 原文：<https://itnext.io/how-i-cleaned-up-my-hard-drive-from-over-50-gbs-of-npm-dependencies-5d2d7d2ad476?source=collection_archive---------0----------------------->

![](img/f2961c3ac359fce68decee35159d5e03.png)

照片由[金加洛帕廷](https://unsplash.com/@locked_in_the_lens?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 拍摄

**在我们开始免责声明之前**。

这个帖子的技巧和精彩来自另一个帖子，你可以在这里找到。虽然最初的想法是我的，但我在这里的意图是围绕我的工作流程用一些额外的背景和历史来包装它。进一步添加一些替代方案，其他方式如何处理这个问题。不过，所有的功劳都应该归于作者( [**马克·皮耶扎克**](https://medium.com/@MarkPieszak?source=post_page-----f3954843aeda-----------------------------------) )，他也帮了我很多。谢谢你，马芮克，如果你正在读这封信。

每天，我都在处理许多节点事务:客户项目、开源包、企业解决方案——一天中从一个节点切换到另一个节点很多次。正如您可能猜到的那样，我需要安装并保留所有这些节点依赖项——放在`node_modules`文件夹中。

我的硬盘只有 250 GBs 的可用空间(MacBook Pro)。其中一些空间由操作系统填充，一些由应用程序填充，一些由文档、照片、音乐、视频等常规数据填充。

有一天，我用完了可用空间，因此我的应用程序无法正常工作——没有可用空间来存储内存、缓存等等。为什么？

所以，我有这个想法，也许是因为我的应用程序/项目太大了。我已经开始寻求解决方案，并且找到了(上面的文章)，并且使用/尝试了一些命令。

首先，检查`node_modules`文件夹分配了多少空间。为此，请转到您的开发文件夹。对我来说，它就在我的用户根目录中的`Dev`。然后使用这个命令。

```
mac: find . -name “node_modules” -type d -prune | xargs du -chswin: FOR /d /r . %d in (node_modules) DO @IF EXIST "%d" echo %d"
```

这可能需要一些时间，但是作为一个输出，你会得到这个。

```
---
list of all folders containing node_modules with space usage
---55G total
```

什么？🤯天啊。这是巨大的。

那我现在能做什么？嗯，我可以跳转到一些旧的、暂时不用的项目，然后删除`node_modules`文件夹。但是，如果我将它们全部删除，当我需要重新安装时，我会这样做。

好吧，怎么去掉？使用这个命令，但是**注意这个过程是不可逆的，你需要非常小心**。

```
mac: find . -name "node_modules" -type d -prune -exec rm -rf '{}' +win: FOR /d /r . %d in (node_modules) DO @IF EXIST "%d" rm -rf "%d"
```

🧨 💥 💣**我刚刚清理了硬盘上的 50 多 GB！** 🧨💥 💣

很好。现在，当我返回到一些旧项目时——如果需要的话——我会再次安装所有的依赖项。通过使用[纱线](https://yarnpkg.com)的替代方案，我可以比使用 NPM 更快地实现这一点。

出于好奇，我在我的个人文件夹里检查了同样的东西。结果是，我仍然保留着 Atom IDE 依赖项，我已经好几年没用了。垃圾。

最后，我有一句话给你。

> 保存在磁盘上的每个字节都可以用于库代码之外的其他用途。

# 可供选择的事物

有一些其他的选择可能会把你从这个问题中解救出来。

[](https://pnpm.io) **PNPM 现在还不太受欢迎，但`pnpm`可能会成为`npm`的终极替代品。在幕后，它使用一个中心位置来安装所有的依赖项/包，如果你在一堆项目中使用`Vue.js`，那么`pnpm`将只使用一次。可以想象，依赖项所需的空间会小得多。此外，一般的安装过程会快得多。**

**[**纱**](https://yarnpkg.com)T22`yarn`的第一个问题是它不会为你节省硬盘空间。但是与`npm`相比，它将大大加快你的安装过程，因为它是异步工作的。但是，还有一个额外的特性可以使用:`yarn` [自动清洗](https://classic.yarnpkg.com/en/docs/cli/autoclean)。这个函数将创建一个特殊的文件，包含所有可以删除的不必要的文件。不必要的文件？是的，不幸的是，软件包创建者倾向于不使用`.npmignore`文件，而将软件包本身不需要的所有文件留在里面。**

**那么怎么用呢？首先，初始化干净的配置。**

```
yarn autoclean --init
```

**然后，一个特殊的`.yarnclean` [文件](https://gist.github.com/lukasborawski/f043cc43b36cf07ffed996548b63adb1)将被创建在您的项目根文件夹中。在这个文件中，你会发现类似于`tests, docs, configs, etc.`的东西。你可以根据自己的需要编辑这个列表，也许你愿意保留一些东西🤷。接下来，您可以启动这个命令。**

```
yarn autoclean --force
```

**Yarn 将搜索所有这些文件并删除它们。终于节省了一些磁盘空间。该清理的平均 MB 大约为 25–30mb。这是对的吗？**

**[**UNPKG**](https://unpkg.com)
多一个，不同种类的替代可能是`unpkg`。它不是软件包管理器，而更像是软件包 CDN。因此，该服务正在跟踪所有的`npm`包，并保持它们未被打包。因此，通过一个简单的 [URL](http://unpkg.com/:package@:version/:file) 模式，您可以获得包源代码并在您的项目中使用它，而无需安装它。有趣的方法，对于一些应用程序来说，这可能非常合法。**

**我认为 NPM 有更严格的替代方案，但大多数类似的工具更像 monorepo，微前端管理器，而不是软件包本身。但是您仍然可以检查[位](https://bit.dev)或[拉什](https://bit.dev)作为参考。**

****就这样。希望你喜欢它，它会有所帮助。干杯。****