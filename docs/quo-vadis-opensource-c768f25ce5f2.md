# Quo Vadis，开源？

> 原文：<https://itnext.io/quo-vadis-opensource-c768f25ce5f2?source=collection_archive---------2----------------------->

## 社区里有很多工作要做。你准备好迎接挑战了吗？

![](img/5df2a9171ef7672ecbcb15b513caca5f.png)

信用:[洛伦佐·埃雷拉](https://unsplash.com/@lorenzoherrera)

T 他的文章只是收集了我对开源运动的想法，二十多年来，我一直是开源运动的骄傲参与者和传播者。你可能不同意，感到不安，或者对这篇文章持保留态度，但是我确信我们观察到了 IT 社区内的整个一代人和态度的转变，这对它不利。

# 这一切是如何开始的

开源运动和这个术语本身是由 Paolo Alto 的一群传奇人物在 90 年代末创造的，起因是 Netscape 发布了它的 Navigator 浏览器源代码。它起源于自由软件(自由是指修改和交换的自由，而不是货币自由)运动，该运动始于 80 年代，由理查德·斯托尔曼发起的 GNU 项目。不久之后，Bruce Perens 和 Eric Raymond 成立了一个名为[开源倡议](https://en.wikipedia.org/wiki/Open_Source_Initiative)的组织，负责开源解决方案的宣传和推广。

我在 14 岁的时候就爱上了开源的想法，完全被集体创造、修改和贡献的可能性迷住了，投入个人的时间和努力来改善自己和他人的生活。向他人学习，在 IRC 上讨论问题，在笔记本上做计划是现实，因为甚至不存在容易访问和贡献的软件。自从 SVN ( git 的爷爷)被发明出来已经有 20 年了，那时我们仅有的变化历史是一个 FTP 服务器和一个 *changelog* 文件。

对于协作来说，那真是一个艰难的时期——今天，你在一个想法和实际开始工作之间点击几下鼠标的距离。

**点击** —这里是 Github 仓库，有项目、问题和 CI/CD。
**单击** —有空闲时间可以与团队成员讨论一切，召开会议，并获得世界上所有关于项目进度的更新。
**点击**——你最终会拥有一个免费的 AWS 或 GoogleCloud 账户，为你的项目注入活力，并获得额外的积分和免费等级。

一些我们做梦都想不到的事情在当时变成了日常现实，经常得不到重视，被视为理所当然。

# 开源项目质量

这里有许多我每天都在使用的项目。更重要的是，我会浏览这些信息，以便在“我正在做的事情”中做出明智的决定。我已经学会了仔细检查提交频率的艰难方法，当涉及到提出的问题时，维护人员的反应如何，他们有多热衷于接受来自第三方贡献者的拉请求。没有人想把自己的辛勤工作建立在一个最有可能被放弃的项目上，对吗？去年我每天都在使用 Python、Go 和 JS，然而——*我并不称自己为开发人员*——我对它们的了解足以完成工作，理解代码、最佳实践，并且，总的来说——让事情运转起来。我过去曾与 PHP(是的，我现在还能看到这些微笑)和 Ruby 一起工作过，这让我对这些语言背后的思想和可能的解决方案有了一个相对全面的了解。

我不热衷于用“*这种语言比*那种语言更好”来挑起任何激烈的战争，整个比较尽可能客观，没有额外的偏见——就像你可以通过旅行来开阔你的视野一样——你可以检查其他语言如何处理你可能遇到的问题，成为语言不可知论者，并确保你会根据各种因素的判断选择最佳语言来做这项工作。

## 判决传奇

*   **易用性:**该语言从零开始学习(不是掌握)并开始使用的难易程度。
*   雇佣容易:如果你开始做这个项目，也许将来你需要雇佣一些人来帮助你。这一评级是基于进行的面试——申请人的数量与能够胜任该工作的人的数量。
*   **开源代码质量:**它甚至不掉毛吗？代码如何遵循最佳标准和实践？容易修改甚至阅读吗？通过添加新功能来扩展怎么样？下次更新库/包时，出现问题的可能性有多大？
*   **开源维护:**项目作者接受潜在批评、回复或承认问题并对新的拉请求做出反应的可能性有多大。
*   **文档:**文档有多全面？它涵盖了所有的功能和选项吗？新增功能后是否更新？

# 让(友好的)战斗开始吧

![](img/ac482837c1ed0b3337dd186ab196826e.png)

信用:[静止不动](https://unsplash.com/@stillnes_in_motion)

## ⭐ **节点 JS &一般 js**

我绝对喜欢它，因为我的私人项目开发从后端切换到前端。在 [hipertracker](https://medium.com/u/2124d7e93ea4?source=post_page-----c768f25ce5f2--------------------------------) 的推荐下，我选择了 [Vue.js](https://medium.com/u/9b930cf6db26?source=post_page-----c768f25ce5f2--------------------------------) ，就像每个新手一样，我爱上了热情过度的 *yarn install。*一段时间后，我注意到我安装的大多数包都没有得到维护，甚至被它们的创建者抛弃了，带来了几十甚至几百个问题(并提出了拉取请求),但维护者方面没有明显的动机来审查或合并补丁和新特性。文档中谈到它们更多的是关于'*检查我们的代码来找出'*而不是一种清晰易懂的使用方式。
我强烈认为，这种情况是由成为 JS 开发人员的简单入门级要求造成的，再加上将代码推向世界的可能性，这是一种危险的混合物。越来越受欢迎的 TypeScript 肯定会提高代码和项目的质量，尽管这可能需要一段时间。

```
┌──────────────────────────┬───────┐
│ Ease of use              │   4/5 │
│ Ease to hire             │   5/5 │
│ OpenSource code quality  │   3/5 │
│ OpenSource maintenance   │   2/5 │
│ Documentation            │   2/5 │
└──────────────────────────┴───────┘
```

## ⭐ **巨蟒**

我还没有听说过一个开发人员对 Python 一无所知。它无处不在——从简单的系统脚本，到用 pySpark 处理大数据，再到人工智能建模。最近 2.7 的寿终正寝和向 3.x 的强制迁移无疑改善了整个代码库，我真诚地希望项目所有者能跟上。

```
┌──────────────────────────┬───────┐
│ Ease of use              │   4/5 │
│ Ease to hire             │   5/5 │
│ OpenSource code quality  │   4/5 │
│ OpenSource maintenance   │   3/5 │
│ Documentation            │   4/5 │
└──────────────────────────┴───────┘
```

## ⭐ **走**

市场上一个相对较新的玩家，它成功地偷走了数百万寻找快速、可伸缩的重复构建解决方案的开发人员的心(毕竟，你得到的只是一个二进制文件)。它最大的优点之一是内置的文档服务器，我个人使用和滥用它，希望有人足够关心它的功能并写一些评论。

```
┌──────────────────────────┬───────┐
│ Ease of use              │   3/5 │
│ Ease to hire             │ 3.5/5 │
│ OpenSource code quality  │   4/5 │
│ OpenSource maintenance   │   3/5 │
│ Documentation            │ 4.5/5 │
└──────────────────────────┴───────┘
```

![](img/f33dfe8706dad691927ef9873bf1551b.png)

凯文·Ku

# 封闭的开源社区

他这篇文章的目的绝不是抨击语言本身，因为我非常热衷于开发上述任何一种语言。在任何应用程序开发周期中，您都需要依赖文档和社区本身。不幸的是，许多库的创建者要么把他们的代码推一次，然后把它留在原处，要么把任何错误报告当作人身攻击。

> 我将会给你一个我已经使用了一年多的 ***GENIUS*** 项目的例子——由[克里斯蒂安·阿尔佛尼](https://medium.com/u/d4ec01100893?source=post_page-----c768f25ce5f2--------------------------------)开发的 Overmind.js，它允许我以一种非常简单且符合逻辑的方式处理应用程序状态。在专门的演示会议上，我向前端开发人员展示了我的代码库，之后我甚至向他们推荐了这款软件。不幸的是，它以“缺乏维护基础”为由被拒绝，因为当时它被放弃了几个月，一系列问题不断增加。经检查，似乎作者要么休息了很长时间，要么放弃了它，所以我决定打开一个[问题，请求项目维护](https://github.com/cerebral/overmind/issues/469)。在这个问题本身下一段时间没有答案后，我决定在 Twitter 上 ping 作者，因为他在那里相对活跃——只是遭到了第三方(而不是作者本人)的攻击，告诉我不要问。幸运的是，Christian 很快介入，并解释说他将很快带着项目本身回来。

很少有什么事情没有带走我对主宰本身的爱。然而，在另一个项目中，我对它的依赖性提出了一个严重的问题——问号，从那时起，我就把它作为挑选关键任务库的经验法则。

1.  一个人的维护团队是不够的。从来没有也永远不会有。太多的事情可能会出错(或者像在克里斯蒂安的生活中一样——再一次过于顺利——*祝贺你！❤)，他们中的大多数人把其他项目置于严重的风险之中。*
2.  拿你的或者别人的代码(*原文如此！*)太过个人化会阻止社区贡献和提出问题，并阻止使用你的产品。
3.  提交的频率，以及 PRs 合并到主分支的平均时间，整体通信。
4.  在你的软件中发现 bug 不是犯罪，也绝对不是人身攻击，不应该这样对待。

我完全理解这个项目可能会变成一个婴儿——毕竟，花了那么多的时间和夜晚来克服令人沮丧的错误。最重要的是，处理不满意的用户可能是一个令人伤脑筋的经历。我去过那里，我好几次设法以暴躁的方式回应用户。如果你想知道发生了什么——我的用户离开了，发现一个不同的项目在类似地工作。从那以后，我找到了 [BeastByte](https://medium.com/u/a365ee91b890?source=post_page-----c768f25ce5f2--------------------------------) ，他开始负责社区管理并每天帮助用户，由于时区差异，这一点非常有效(他住在委内瑞拉，会说英语和西班牙语！).用户现在知道他们可以在白天或晚上的任何时候问问题。总会有人支持他们的。

# 保持开源的开放

M 开源项目的维护是一件非常痛苦的事情。然而，如果在我们的*日常工作*中以同样的专业精神对待，它可能会变成一种祝福，帮助你飙升的人气和提升你的技能，这迟早会变成工作机会，对双方都有利。我目睹过用他们的公共贡献的很好的投资组合来雇佣人(呸，甚至是 ***我雇佣了他们*** ),我知道相当多的地方重视这样的经历。每个企业都在寻找的不仅仅是做这项工作的人，而是一个优秀的团队成员。如果你能展示你解决问题的方法、项目管理和优秀的沟通技巧——同时因为你喜欢而免费这样做——你就自动加入了市场上最热门的资产，每个合理的公司都想为之奋斗。

关于项目本身——我们大多数书呆子都是内向的人。由于分离和缺乏一般的人际交往，我们有时会把事情看得很私人，有时甚至有点过分。这不会让我们成为糟糕的开发人员，因为我们倾向于关注代码和一般结果，而不是证明另一方谁是正确的，为什么是我。然而，这会使我们成为糟糕的项目维护者，排斥那些希望花费时间和精力为项目本身增加实际价值的新人才和热心的贡献者。

![](img/9640b1c4cd0dd9f16417007b3e684ad4.png)

致谢:[约书亚·阿拉贡](https://unsplash.com/@goshua13)

我喜欢保持开源项目实际开放，本着最初运动的真正精神，当人们分享知识时，当总是有人热衷于解释和教学时，或者当我们都在一起工作时，带着进步的想法，不带个人偏见和敌意时，任何可以贡献的人都受到欢迎。通常有一群人(被戏称为“*互联网的长者*”)负责项目维护，支持贡献者并审查他们的工作，通过 IRC 或 USENET 与每个人交流，并允许双方建立长期友谊和分享额外的想法——这显然导致了我们今天都在使用的更伟大的项目。

# 企业要树立榜样

答在女王陛下身边服务了几年后，我接触了各种技术和多个项目，我了解到分享是好的，甚至是我想要的。英国政府内部的许多项目都是公开的，来自第三方的拉动请求在《公务员法》的精神——正直、客观和公正中被接受。英国政府真正为企业铺平了道路，展示了在一个每个人的意见都很重要，所有努力都集中在共同利益上的完美世界里，事情是如何完成的。
有很多私营公司加入了开源运动，这对他们两个都有好处——他们的产品得到免费的贡献，这通常是在企业或订阅的基础上提供的(当然还有附加功能)。简单的天才方法将“*分享的精神带入商业化的世界，目前被 [nats.io](https://nats.io/) 、 [Hasura](https://medium.com/u/5f5557965017?source=post_page-----c768f25ce5f2--------------------------------) 、 [Portainer.io](https://medium.com/u/4f16ad1b3349?source=post_page-----c768f25ce5f2--------------------------------) 、[micro mu](https://micro.mu/)和许多其他公司使用，这些公司比那些把一切都锁起来的公司做得更好，最重要的是——在竞争对手甚至能够向他们发送报价之前，偷走他们未来客户的心。*

# 有什么可做的

我想用一系列步骤来结束这篇文章，这些步骤可能在字里行间被遗漏了。

*   保持你的代码开放，即使它不是最高质量的——有可能有人会开始使用它并提出改进建议，这将允许你和/或你的工程师学习并打开他们的思维。
*   不要针对第三方提出的问题。相反，你应该感到高兴，因为世界上有人足够欣赏你的工作并使用它，并花时间让你注意到错误，而不是换一个不同的产品。
*   为了让一切都与最初的目标保持一致——考虑为新版本创建一个包含一系列功能的公共路线图——有可能会有外部人员挑选出来帮助您工作和出主意。
*   围绕你的项目创建一个社区，确保每个人都受到欢迎，所有的意见都被听到，所有的问题都得到回答(*提示*:“Google it”不是对技术问题的回答)。
*   为其他项目做贡献，参与讨论，不要害怕提出问题——尽管如此，保持各种积极和尊重的交流，就像你对同事做的那样。

~ ***互联网应该是免费的，还有你的大部分代码。**** ~
*确实适用于业务关键型软件和逻辑。