# 作为一名 Flutter 开发人员，我的基本工具

> 原文：<https://itnext.io/my-essential-tools-as-a-flutter-developer-7a055822a777?source=collection_archive---------4----------------------->

![](img/cba9f89c48abaf37a38ade2f6452e74e.png)

作为开发人员，我们的日常工具对于我们的生产力甚至心智健全是必不可少的。对于我们每天做的所有复杂的事情，我们需要我们的工具要么尽可能简单，要么尽可能强大。理想情况下，:D…

作为一名 Flutter 开发人员，我的工具并不复杂，我每天都在使用一些工具，让我的工作、我的激情变得更简单、更高效。

# 1.智能理念

虽然，如果不是大多数与我一起工作的开发人员都使用 VSCode，这是我在近 4 年前留下的一个工具，但它实际上甚至不再安装在我当前的机器上，虽然我确信大多数(如果不是所有)IDEA 为我做的事情都可以在 VSCode 中完成，但我已经过了构建自己的工具的阶段。

对我来说，尤其是在使用 Flutter 时，我更喜欢用户界面、某些工具的可访问性和功能性。我发现它在大型项目中表现更好，全局搜索功能非常棒，虽然 VSCode 确实有这一功能，但它似乎工作得更好，更具扩展性，能够在一个地方对文件、类和函数进行通配符搜索，这简化了在您不能 100%确定其名称时的查找工作。

在提交时，我也可能会非常罗嗦，我喜欢对更改的上下文进行分组，当处理一个功能或一个 bug 时，有时您会更改相当多的内容，但并不是所有的内容都与相同的提交消息相关，我永远不会提交“固定的东西”，我已经看到了这一点…

虽然 VSCode 可以做到这一点，但实现起来要复杂得多，而且更改级别提交，使用 IDEA，我可以浏览文件，取消选择我认为与我尝试进行的提交不相关的更改组或行，最好使用更好的提交描述。

我个人使用 IDEA 的付费版本，与免费社区版本不同，它还支持 JavaScript、HTML 和 CSS 等网络相关语言，这对于我在全力投入 Flutter 之前参与的少数项目来说非常方便。

# 安装的插件

下面是我添加到 IDDEA 的一些插件，让我的生活更轻松。

*   [阻塞](https://plugins.jetbrains.com/plugin/12129-bloc)
*   [颤振增强套件](https://plugins.jetbrains.com/plugin/12693-flutter-enhancement-suite)
*   [颤振片段](https://plugins.jetbrains.com/plugin/12348-flutter-snippets)
*   [GitToolBox](https://plugins.jetbrains.com/plugin/7499-gittoolbox)
*   [彩虹括号](https://plugins.jetbrains.com/plugin/10080-rainbow-brackets)
*   [救人动作](https://plugins.jetbrains.com/plugin/7642-save-actions)

[**链接**](https://www.jetbrains.com/idea/download/)

# 2.阿尔弗烈德

Alfred 只会让 Mac 用户受益，它是 spotlight 的替代品，它几乎支持 spotlight 所做的一切，但可以通过自定义工作流程和配置进行扩展。

Alfred 的功能非常强大，从在机器上启动应用程序到进行在线网络搜索，甚至是计算和转换。

我可能从 Alfred 中使用最多的是我添加的一些自定义网络搜索，如`ppac`是 pub package 的缩写，我可以在其后键入任何 dart/flutter 插件的名称，直接进入页面，或`pdev`是对 pub.dev 的搜索。

在此之后，将是“[计算任何东西](https://github.com/biati-digital/alfred-calculate-anything)”工作流，它使用自然语言过程来计算几乎任何东西，从简单的数学到货币和单位转换。

只需键入“50 美元欧元”就会为您进行货币转换，您也可以设置一个基础货币，这将是默认的，所以如果您的基础货币是“欧元”，那么您只需键入“50 美元”。

[Github Repos](https://github.com/edgarjs/alfred-github-repos) 是我经常使用的另一个工具，它根据你的前缀命令搜索 Github 或你自己的 Repos，以快速启动选定的结果。

其他有用的东西是 [Kill Process](https://github.com/nathangreenstein/alfred-process-killer) ，用于通过搜索终止一个正在运行的进程。

这些只是我使用的一些工作流和现有工作流的一小部分，如果没有你需要的东西，你可以自己编写，因为大多数语言都可以创建工作流。

# 3.ZSH +哦，我的天

很肯定，现在每个人都知道 zsh，但它肯定不会遗漏任何提高开发人员效率和生产力的工具列表，作为 bash 的增强功能，结合 oh-my-zsh 的可扩展性，有很多方法可以提高生产力，如果没有其他原因，内置别名可以用插件启用，虽然编写自己的别名很简单，但设置它们更容易，而且有了这么多插件，很容易挑选出可以增强工作流的插件。([插件](https://github.com/ohmyzsh/ohmyzsh/wiki/Plugins))

最重要的是，[主题化](https://github.com/ohmyzsh/ohmyzsh/wiki/Themes)只是给你的工作环境增加了一点个性化，用颜色来增加它的味道，通常在版本控制的文件夹中添加活动分支。

我已经用了 4 到 5 年了，我不认为自己会回头，它简单而强大。

正如他们所说

> *哦，我的 Zsh 不会让你成为 10 倍的开发者……但你可能会觉得自己是。*

[有兴趣可以链接](https://ohmyz.sh)和我的 [zshrc](https://gist.github.com/RemeJuan/ba8dc0fbcea4d3709b1ef7640d58c572) 文件。它会让您对一些可用的特性有更多的了解。

# 4.火花

Spark 可能是我一生中使用过的最好、最简单的邮件应用程序之一，虽然我确实经常更喜欢使用网络用户界面来管理电子邮件，但对于单个应用程序处理多个邮件帐户的便利性，我非常肯定，如果不是我们所有人都至少有两个电子邮件帐户，个人和工作/职业帐户。

作为一个邮件应用程序，它并不引人注目，但我认为这正是它的伟大之处，他们添加了一些有用的功能，但尽可能保持简单和用户友好，他们没有尝试在这一点上重新发明轮子。

它将您所有的邮件帐户和日历加载到统一的视图中，根据我的经验，他们会为组织提供智能、准确的建议，告诉您要将邮件归档到哪里。

Spark 还使用 SSO，当你最初注册为你创建一个 spark 帐户时，这在你有多台设备时很方便，因为你只需要在其中一台设备上进行设置，你的帐户就会同步到你的其他设备上。

它们还支持智能通知，为收到的电子邮件分配重要性，以确定是否需要提醒你，或者只是增加应用程序徽章上的数字，就像 Gmail 优先邮件一样。我发现的唯一有点奇怪的事情，可能是由于设计的原因，是不同设备的重要性分类不同，所以你可能会在 iPad 上收到通知，而不是在手机上收到通知。

你也可以在每个设备的账户级别上禁用通知，这很方便，对我来说，工作邮件只会在我的 Mac 上通知，而不会在我的手机或 iPad 上通知。

我的基本工具库实际上很小，显然，现在我的 Mac 上还有许多其他工具，但这是对我的日常生活影响最大的 4 个工具。

我希望你觉得这篇文章信息丰富，或者有趣。如果您有任何问题或意见，请随意。

我希望您对此感兴趣，如果您有任何问题、评论或改进，请随时发表评论。享受你的颤振发展之旅:D

如果你喜欢，一颗心会很棒，如果你真的喜欢，一杯[咖啡](https://www.buymeacoffee.com/remelehane)会很棒。

感谢阅读。

[](https://medium.com/geekculture/unit-testing-datetime-now-with-the-help-of-dart-extensions-d2b0c9f991bf) [## 借助 Dart 扩展对 DateTime.now()进行单元测试

### 当需要使用 DateTime.now()时，快速演练 Flutter 中的单元测试

medium.com](https://medium.com/geekculture/unit-testing-datetime-now-with-the-help-of-dart-extensions-d2b0c9f991bf) [](/flutter-web-should-i-use-it-part-1-seo-842d87ff9d28) [## Flutter Web:我应该使用它吗？(第 1 部分—搜索引擎优化)

### 有很多关于网页抖动和它的缺点的讨论，在这一部分我们将讨论 SEO 和网页清理器。

itnext.io](/flutter-web-should-i-use-it-part-1-seo-842d87ff9d28) 

*原载于 2021 年 7 月 25 日*[*https://remelehane . dev*](https://remelehane.dev/posts/my-essential-tools-as-a-flutter-developer/)*。*