# “没有办法防止这种情况”，只有经常发生这种情况的开发社区这样说

> 原文：<https://itnext.io/no-way-to-prevent-this-says-only-development-community-where-this-regularly-happens-8ef59e6836de?source=collection_archive---------0----------------------->

![](img/0af46c18bf16c1d2e342e4d0c8736663.png)

NPM pure script——在 NPM 上的另一场软件包灾难发生后的几个小时内，据报道，唯一一个经常发生这种灾难的社区的开发人员得出结论，没有办法阻止灾难的发生。在这场灾难中，一个单独的开发人员杀死了几十个 CI 版本，并导致数千个其他版本受到严重警告。“这是一个可怕的悲剧，但有时这些事情就是发生了，没有人能阻止它们，”fullstack 开发者 Bob Dynald 在 Reddit 上说，呼应了数千万参与编程社区的个人表达的观点，在过去 9 年中，世界上超过一半最令人愤怒的软件包管理灾难都发生在这个社区，其成员经历意外软件包更新的可能性是其他现有社区的 200 倍。“真可惜，但我们能做什么呢？如果这是他们真正想要的，那么真的没有什么可以阻止这些人破坏和毁掉很多人的一天。”在记者发稿时，世界上唯一一个大型发展社区的居民称他们自己和他们的情况“无助”，在过去的四年里，每个月大约都会发生两次包管理灾难。

*这篇文章的灵感来自洋葱网发布的“*”[没有办法阻止这种事情发生，“只有经常发生这种事情的国家”](https://www.theonion.com/no-way-to-prevent-this-says-only-nation-where-this-r-1835173950)*的文章。可能含有讽刺意味。*

**编辑**:除了我收到的一些针对这个恶搞/讽刺笑话的人身威胁之外，我还被要求就如何解决这个问题提出建议，以下是我个人认为 npm 有问题的不完整列表:

*   npm(客户端和注册管理机构)是一个有缺陷的系统，需要彻底更换，新系统应具备以下功能，以减少此类事件的可能性:
*   没有未包装的包裹
*   范围一对一地绑定到单个用户/组织
*   发布包需要 2FA，包必须通过 GPG 签名
*   除了注册表维护人员之外，任何人都不允许取消发布软件包(Rust 的 cargo 就是这样做的，这是另一种可能的解决方案)
*   模糊依赖版本(^2.0.0 和类似的)在最终版本中不应该被允许。我多次目睹 npm 在新的克隆后运行`npm install`时修改项目的包锁文件，下载比锁文件中指定的版本更新的可传递依赖项版本

关于整个生态系统:TC39 应该考虑给 JS 本身添加一个更好的标准库，这将减少一行程序包的数量。有一个[活动提案](https://github.com/tc39/proposal-javascript-standard-library)，但是遭到了许多 JS 社区成员的蔑视。

如果你对导致这篇文章的事件感兴趣，这里列出了最近和过去的 npm 事件，无论是 npm 的安全措施的错误，糟糕的架构，有趣的轶事，还是在注册表上发布软件包所需的低门槛:

*   [pure script NPM 安装程序中的恶意代码](https://harry.garrood.me/blog/malicious-code-in-purescript-npm-installer/)
*   [npm 6.9.1 因。发布的 tarball 中的 git 文件夹](https://npm.community/t/npm-6-9-1-is-broken-due-to-git-folder-in-published-tarball/8454)
*   [一家大型国际银行意外地向公共 npm 注册中心发布了他们自己的私有包](https://twitter.com/seldo/status/1105153287718723584)
*   [我不知道该说什么:流行软件包的作者把它交给一个未知的开发者，然后他给它添加恶意代码](https://github.com/dominictarr/event-stream/issues/116?source=post_page---------------------------#issuecomment-440927400)
*   [NPM 突然以“呃！418 我是茶壶”的错误。](https://github.com/npm/npm/issues/20791)
*   [NPM v 5 . 7 . 0 中的严重错误破坏了 Linux 服务器](https://github.com/npm/npm/issues/19883)
*   [左键盘](https://www.theregister.co.uk/2016/03/23/npm_left_pad_chaos/)