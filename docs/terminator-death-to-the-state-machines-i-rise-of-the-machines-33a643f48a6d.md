# 《终结者:机器之死 I——机器的崛起》

> 原文：<https://itnext.io/terminator-death-to-the-state-machines-i-rise-of-the-machines-33a643f48a6d?source=collection_archive---------7----------------------->

[*点击这里在 LinkedIn* 上分享这篇文章](https://www.linkedin.com/cws/share?url=https%3A%2F%2Fitnext.io%2Fterminator-death-to-the-state-machines-i-rise-of-the-machines-33a643f48a6d)

我原本以为这是第一次[奇异果红宝石](https://kiwi.ruby.nz/)大会的轻松(但严肃)的演讲。不幸的是(对我来说),还有很多更好的演讲/话题。在任何情况下，我觉得我仍然应该分享我们与状态机的经验，所以我们在这里。

我喜欢这个主题的乐趣，这意味着是一个小时的谈话，所以这可能会跨越多个职位。剧透:这是一个让你跟随我的温和尝试，它将以第二部的悬念结束！ ***更新*** *:* [*第二部是 up*](https://medium.com/@corrodedlotus/terminator-death-to-the-state-machines-ii-salvation-d46bd273e7f2) *！*

![](img/664c7dac40dfd125ae209c89e7503bc6.png)

这可能是天网！照片由[帕特里克·grądys](https://unsplash.com/@patrykgradyscom?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄

# 机器接管之前的生活

曾经有一段时间，国家机器和开发者和平共处。我使用 Ruby 已经十年了，我已经使用了许多可用的状态机插件中的三个。在我处理的几乎所有大型项目中，我们都以这样或那样的方式使用了状态机，所以我可以有把握地说，我在这些 abom——嗯，事情上有很多经验。这三个状态机在我心中占有特殊的位置: [transitions](https://github.com/troessner/transitions) 、 [acts-as-state-machine(亲切地称为 aasm)](https://github.com/aasm/aasm) ，以及[我们遗留代码库中使用的状态机:state_machine。](https://github.com/pluginaweek/state_machine)

当你第一次使用一个状态机 gem 时，它给了你很多 bat 的特性，使得开发速度更快，而且感觉非常好。需要你每个州的范围吗？不要担心！我使用的三个状态机都有自动作用域。[见](https://github.com/troessner/transitions#automatic-scope-generation) [为](https://github.com/aasm/aasm#automatic-scopes) [自己](https://github.com/pluginaweek/state_machine#activerecord)！

事件过渡怎么样？当然可以。任何自我尊重的状态机都可以转换到任何状态，并有一个 DSL:

```
# transitions
event :move_cart do
  transitions to: :pick_line_items, from: :picking_line_items
end# aasm
event :run do
  transitions from: :sleeping, to: :running
end# state_machine
event :park do
  transition [:idling, :first_gear] => :parked
end
```

很时髦，对吧？对吗？那检查州政府呢？所有这些都包括了你。通常他们有一个`state?`方法，所以如果你有一个叫做`running`的状态，那么你也会有一个叫做`running?`的方法，如果当前状态是`running`，那么这个方法就会返回(哎呀，谢谢，队长明显！)

![](img/7cf47054128548e0c0ef94f59c930e0a.png)

对我来说他看起来像个上尉。你怎么想呢?照片由[塞维多夫·热莫夫斯基](https://unsplash.com/@yuriyr?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄

将它添加到一个绿地项目中是很容易的，每个人都很高兴。每个人都使用国家机器，一切都很好。

直到革命。

未完待续…开个玩笑！我们说到哪了？啊是的。革命。

# 机器的崛起

我的大多数项目平均持续一至三年。有些是重叠的，有些是我在移交旧项目后被分配的项目。那些年，我目睹了国家机器的崛起。他们让你依赖他们，直到你碰到边缘情况。或者说是一种限制。或者是虫子。如果您的错误立即得到解决，祝您好运，但是我们知道有些问题太具体了，有时不值得为 0.0001%的问题进行修复。

我应该预见到的。或者也许我有，但是我已经太依赖了，以至于我被蒙蔽了。也许它也已经在那里，所以删除它将是一种浪费。不管是什么原因，我一直看到相同的模式:使用状态机宝石，一旦你需要一个特定的功能，或者一旦所有其他功能、状态和转换组合起来达到爆炸点的临界质量，看着一切都颠倒过来。

[状态机](https://github.com/state-machines/state_machines#example)中的例子说明了一切:

![](img/4dc4c4a9d85d77059d64ab45625aae99.png)

一个机器失控的例子。

这应该是状态机 gem 所能做的所有特性的一个人为例子。它也可以反映你的代码在你安装后的十个月内会是什么样子。

当然，如果你在它还是新的时候就开始使用它，这是非常明显的。一切都会。这是一个对话的例子，一个开发人员在开始的时候做所有的事情，一个初级/新员工问了一些事情:

```
Original dev: “Ah yes, as long as a car is not parked, stalled or idling, moving? should return true.”
New dev: “Okay, but what if it’s parked?”
Original dev: “It should return false”
New dev: "But moving? is only defined in all the states EXCEPT parked, stalled and idling"
Original dev: "... um. Ah...y-yes. No. Maybe? I'll need to check the docs."
```

是的——使用其中的一些工具，您实际上可以只为特定的状态定义一个方法！太酷了，我是说这很危险吧？很显然。当我浏览一个模型的方法，发现类似于`moving?`的东西并调用它时，我被咬了无数次，只是因为它在我面前爆炸——因为对象处于另一种状态，它没有被定义。太神奇了。

# 观察者:(邪恶的)伪装的机器

当 [Rails 4 被释放](http://weblog.rubyonrails.org/2013/2/25/Rails-4-0-beta1/)时，观察者被杀死。我的意思是弃用。如果你以前玩过[星际争霸](https://starcraft.com/en-us/)，观察者是看不见的小机器，显示隐形单位。除了探索地图之外，它们主要用于发现这些隐形单位。Rails 以前也有这种机器，如果它看到有事情发生，就会运行一些东西。就像试镜一样。

随着公司和开发者采用/升级到 Rails 4，越来越多的人回避观察者。有很多更好的方法来实现像观察者这样的东西，比如回调或者在事件发生后显式地做一些事情。延期工作/ Resque / Sidekiq 也有帮助。所以我们不需要观察者，就像我们不需要呼吸一样，对吗？当然可以。此外，状态机也支持`after_transitions`,不管怎样，它的工作方式是一样的。

等等，什么？

没错。状态机能够通过`before/after/during`转换来模仿观察者的功能。改变到特定状态后需要发生什么吗？不要害怕，`after_transitions` 都在这里！

以前观察器的问题是很难测试，如果你没有系统的先验知识，它会产生一些真正意想不到的结果(*让我们回到我们最初/新的开发者的故事中去一会儿*

```
New dev (ND): What is this thing smoking? I just set it to move but now it has set the speed to 20.
Original dev (OD): That's because it's in the after_commit transition.
ND: Oh. Wait, how was I supposed to know or even ask about that?
OD: You don't. You'll be with us most of the time anyway so just shoot us any question you'd like or when you see something unexpected.
```

这就是为什么观察者不被认可的主要原因——他们导致了许多意想不到的行为。

# 开发商的没落

(国家)机器的权力已经上升。它们已经变得笨拙、不可预测，并且它们已经遍布整个代码库。测试变得很慢，代码行增加了十倍，没有人想再碰任何与状态相关的东西。我们输了。一切都会燃烧。

*明日未完待续！敬请关注，看我们如何反击！*

*更新:* [*上集*](https://medium.com/@corrodedlotus/terminator-death-to-the-state-machines-ii-salvation-d46bd273e7f2) *！*