# 如何使用 Gitleaks 防止推送敏感信息

> 原文：<https://itnext.io/how-to-use-gitleaks-to-prevent-pushing-sensitive-info-cdd0355d5f61?source=collection_archive---------1----------------------->

![](img/9ab95e70c241407552723581c8c948d7.png)

Danaï des 一家注定要永远把水带到一个总是漏水的浴缸里。

我们都在生命中的某一时刻做过。你刚刚在你的公共回购上实现了一个闪亮的新特性，你结束了你的最后一次提交，运行所有的测试和推送。你去厨房拿一杯当之无愧的咖啡，当你正要喝第一口时，你恍然大悟:你已经把你的(密码| API 密钥| API 秘密)推送到你的公共存储库。

您的第一反应是删除项目中的敏感信息，然后再次推送——众所周知，这是没有用的，因为提交仍然在历史中。唯一要做的就是使暴露的敏感信息无效，并尽快发布新的信息，然后修复存储库。假设你很快意识到自己的错误，那么你可能会毫发无损地离开。但是如果你不这样做呢？

互联网上充斥着[关于](https://www.theregister.co.uk/2015/01/06/dev_blunder_shows_github_crawling_with_keyslurping_bots/) [的](https://medium.com/@nagguru/exposing-your-aws-access-keys-on-github-can-be-extremely-costly-a-personal-experience-960be7aad039) [的故事](https://medium.com/@selvaganesh93/what-happens-if-you-accidentally-commit-your-aws-access-token-to-public-github-be50d378b4c7)关于人们错误地提交他们的敏感信息并因此而遭受痛苦。其中大多数涉及恶意的人窃取 S3 访问令牌，并累积数千美元的法案。你没有太多的时间了——在某些情况下，这些所谓的黑客可以在不到 10 分钟的时间内获取你的密钥并滥用它们。但是怎么做呢？

事实证明，如果你知道如何使用 GitHub，扫描敏感信息是相当容易的。你可以直接[搜索在消息](https://github.com/search?q=remove+api+key&type=Commits)中包含“删除 API 密钥”的提交，你会看到成千上万的提交在 diff 中包含敏感信息，一目了然。编写一个抓取这些结果的机器人是微不足道的。还有像 [gitrob](https://github.com/michenriksen/gitrob) 和 [truffleHog](https://github.com/dxa4481/truffleHog) 这样的工具可以帮你做到这一点。我认为可以肯定地说，只要您推送敏感数据，您就应该认为它受到了威胁。

我来给你讲讲 [gitleaks](https://github.com/zricethezav/gitleaks) 。

Gitleaks 是一个扫描本地和远程存储库的工具，可以扫描任何类型的敏感信息。它速度快，易于使用，非常可配置。

要在远程存储库上运行它:

```
gitleaks --repo=https://github.com/path/to/your/repo
```

要在本地运行它:

```
gitleaks --repo-path=path/to/your/repo
```

要查看更多示例，请访问[维基页面](https://github.com/zricethezav/gitleaks/wiki)。出于我们的目的，我们希望在每次推送之前在本地存储库上运行它。由于 gitleaks 实现了非常一致的返回代码(0 表示没有泄漏，1 表示存在泄漏),这实际上非常容易做到；与其用 *git 推*，我们可以只做:

```
gitleaks --repo-path=path/to/your/repo -v && git push
```

对于那些不知道 bash 的人来说， *& &* 让您在前面的命令正确完成的情况下运行一个命令。在我们的例子中， *git push* 只有在没有发现泄漏的情况下才会运行。我们还将 *-v* 放在那里，以便能够看到关于任何当前泄漏的更多信息。

下面是我在一个提交了(假)AWS 密钥的存储库上运行它的一个例子:

如你所见，自从 gitleaks 发现一个漏洞(一个 AWS 键在 *keys.go* )之后， *git push* 就没有运行。我们安全了！

下面是我在移除 AWS 密钥后运行它的一个示例:

因为已经没有敏感信息了，我们可以推送了。

为了尽可能简化这个过程，给我们的新命令起别名 *git push* 是一个好主意，因为每次要推送时都要键入它是一件麻烦的事情。一种方法是创建一个别名，如下所示:

```
alias gl="gitleaks --repo-path=path/to/your/repo -v && git push"
```

这是可行的，但是不是最佳的，因为我们希望我们的命令被嵌入到 git push 中。我们不应该改变我们的工作流程。因此，更好的选择是围绕 *git* 命令创建一个包装函数:

将此放入您的~/bash_profile

现在，这才像话！这将在每次 *git 推送*之前运行 gitleaks，而不会对我们的工作流程造成任何改变。使用这种方法，所有的 *git 推送*选项(比如 *-f* 和*-v】)*也将工作，因为所有的标志和参数都将被传递。我们知道我们的秘密是安全的，所以我们可以快乐地推送。

这就是你想要的——在每次推送之前，一种简单而又没有麻烦的方式来审计你的存储库。如果你和一个团队一起工作，最好在你的 CI 工具中设置这个(你**使用 CI 工具，对吗？).您还可以设置一个 git 挂钩，如果检测到泄漏，它会拒绝提交。编码快乐！**