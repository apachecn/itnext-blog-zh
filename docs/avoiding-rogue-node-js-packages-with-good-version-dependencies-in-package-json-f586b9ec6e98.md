# 在 package.json 中避免具有良好版本依赖性的流氓 Node.js 包

> 原文：<https://itnext.io/avoiding-rogue-node-js-packages-with-good-version-dependencies-in-package-json-f586b9ec6e98?source=collection_archive---------2----------------------->

## 不精确的依赖项声明为不安全的代码打开了大门

![](img/1dfba5eb020acf81adc5a3dd77cfe257.png)

图片由作者提供，Node.js 徽标由 Node.js Foundation 提供

到目前为止，Node.js 应用程序中的大范围故障已经多次是由包含恶意软件的 Node.js 包引起的。如果坏代码进入我们的应用程序，会带来什么样的麻烦？这些坏代码拥有与您的应用程序相同的权限。如果坏代码不是立即显而易见的，而是窃取数据的隐形恶意软件，该怎么办？

作为软件专业人员，我们的职责是确保我们的应用程序有尽可能少的问题。我们编写单元测试，我们雇佣一个质量工程师团队来关注产品质量，我们有错误报告系统，这样我们的客户可以通知我们问题，理想情况下，我们负责任地通过修复相关的错误来处理所有这样的报告。

但是，有一天我们输入“ *npm 安装& & npm 更新*”时，我们的应用程序突然不能工作了。如果我们幸运的话，问题会很明显。有时问题是隐藏的，一些第三方软件包涉嫌窃取数据，或运行加密货币挖掘代码。

当前的例子是，在 2022 年 1 月初，一名不满的软件包维护者更新了两个软件包——*colors*和*faker*——以包含恶意软件，导致应用程序明显行为不端，并包含关于亚伦·施瓦茨的消息。开发商之前已经

[写](http://web.archive.org/web/20210704022108/https://github.com/Marak/faker.js/issues/1046)关于"*财富 500 强*"从开源程序员那里拿走免费工作，并且不支持他们。有问题的软件包被广泛使用，行为不端的更新代码引起了广泛的问题。

这不是第一次通过 npm 注册表中的包传播广泛的问题。而且，问题不仅限于 Node.js 包。例如，由于 Log4J 库包含安全漏洞，我们的 Java 开发人员面临着一个普遍的问题。在过去的一年中，白宫至少两次警告开源软件(

[2021 年 6 月](https://www.reuters.com/technology/white-house-warns-companies-step-up-cybersecurity-2021-06-03/))，并已开始与软件行业合作改善开源软件的安全性([2022 年 1 月](https://www.bleepingcomputer.com/news/security/white-house-reminds-tech-giants-open-source-is-a-national-security-issue/))。

Node.js 生态系统中的几个例子是:

*   【2022 年 1 月*colors*和 *faker* 的开发者将明显的恶意软件引入这些软件包，如上所述
*   【2021 年 12 月—Discord 使用的一个恶意 npm 包正在窃取 Discord 用户的凭据。
*   [2021 年 10 月](https://thehackernews.com/2021/10/malicious-npm-packages-caught-running.html)发现三个恶意 npm 包正在运行加密货币挖掘代码。
*   [2021 年 9 月](https://thehackernews.com/2021/09/critical-bug-reported-in-npm-package.html)一个广泛使用的软件包中报告了一个严重的安全漏洞。
*   [2016 年 3 月](https://blog.npmjs.org/post/141577284765/kik-left-pad-and-npm)一个包开发者陷入了一场关于他的一个包的名称纠纷，一气之下从 npm 库中删除了他所有的包。其中之一， *left-pad* ，被广泛使用，导致了各种各样的应用程序中断。

这清楚地表明，npm 包存储库对来自有意或无意地具有严重问题或恶意软件的包的问题是开放的。问题是——我们如何避免这些问题？

每种情况下的一个常见问题是`package.json`中列出的包依赖关系。为了方便起见,`dependencies`字段为我们提供了允许使用一系列包版本的选项。假设似乎是大多数包开发者是诚实的，如果版本 N 没问题，那么版本 N+1 也没问题。

使用*颜色*和*伪造者*以及*左键盘*的经验表明，这不是一个安全的假设。

仔细测试你的应用程序是很重要的，在某种程度上也可以测试第三方软件包。但是，重要的是不仅要测试它，还要注意包依赖关系中的版本号。

原则是——在用版本 N 测试你的应用程序后，那是你测试用的版本，你不允许应用程序用在不同的版本上，直到你用那个版本测试。

`package.json`中的`dependencies`字段是我们声明包依赖的方式。这是将代码自动组装到工作项目中的一个很好的方法，依赖项字段给了我们很大的灵活性。例如，我们可以直接从 HTTP URL 加载包，完全绕过 npm 这样的中央存储库。

但是考虑这些 npm 包依赖版本说明符:`>version`、`>=version`、`~version`、`^version`、`1.2.x`、`*`和空字符串

这种类型的依赖声明允许安装您没有测试过的包版本。假设您根据依赖关系的`version`版本测试了您的应用程序。这是不是意味着`version + 1`也可以工作，你可以允许你的应用程序使用它，即使你还没有对那个版本进行测试？

这些说明符是一种便利，在大多数包中，破坏性的改变或恶意软件的引入不会发生。我们可以指定`1.2.x`，并且知道我们的应用程序将自动更新到`1.2`系列发布中的最新版本。在大多数情况下，这很有效，我们可以继续生活。直到有一天一个心怀不满的开发者发布了包含恶意软件的版本`1.2.42`。

任何不精确的包依赖都容易出现这种问题。

解决方法是在依赖项中明确使用精确的版本号。这意味着使用`version`而不是类似`>version`的东西。

我想我们对此都有责任，我确定。下面是我创建的应用程序之一的`dependencies`:

```
"dependencies": {
    "archiver": "^5.3.x",
    "cheerio": "1.0.0-rc.10",
    "chokidar": "^3.5.2",
    "commander": "^8.3.x",
    "globfs": "^0.3.3",
    "js-yaml": "^4.1.x",
    "mime": "^3.x",
    "sprintf-js": "^1.1.2",
    "text-statistics": "0.1.1",
    "unzipper": "^0.10.x",
    "uuid": "^8.3.x",
    "xmldom": "^0.6.x"
}
```

很明显，我刚刚给了自己一个任务来清理这个。这是 10 个使用不精确版本号的依赖项。这些包裹中的任何一个都可能成为问题。对于每一个版本，可能会发布一个包含重大变化的更新，但我不会看到，因为我没有对该版本进行测试。

以这种方式声明依赖关系，我们试图避免什么？这是为了降低我们的管理费用。这是 10 个包，我必须以某种方式跟踪它们的状态，然后不时更新。每次都必须检查每个包的更新。更重要的是，我们如何自动得到安全漏洞的警告？

一些可用的工具包括

*   `npm audit`命令查询已知安全漏洞的数据库——例如`xmldom`有一个已知的漏洞，但没有已知的修复
*   `[npm-check-updates](https://www.npmjs.com/package/npm-check-updates)`列出有可用更新的依赖项
*   `[npm-check](https://www.npmjs.com/package/npm-check)`还列出了更新可用的依赖项。这不仅扫描了`package.json`，还扫描了`require`和`import`语句中引用的源代码。
*   `npm list --depth 0`命令列出了您的应用程序直接依赖的包
*   `npm outdated`命令列出了有可用更新的软件包
*   `npm cache clean --force`清除 npm 缓存，这在进行更新时很有用

我们只能控制`package.json`中列出的直接依赖项。这里提出的问题可能来自嵌套依赖，其中我们依赖的一个包依赖于一个损坏的包。这意味着让中间包的维护者对更新他们的依赖感兴趣。

# 关于作者

[***David Herron***](https://davidherron.com)*:David Herron 是一名作家和软件工程师，专注于技术的明智使用。他对太阳能、风能和电动汽车等清洁能源技术特别感兴趣。David 在硅谷从事了近 30 年的软件工作，从电子邮件系统到视频流，再到 Java 编程语言，他已经出版了几本关于 Node.js 编程和电动汽车的书籍。*在`https://davidherron.com`了解更多关于大卫·赫伦的信息

*原载于【https://techsparx.com】[](https://techsparx.com/nodejs/install/pinned-versions.html)**。***