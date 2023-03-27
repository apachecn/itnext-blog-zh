# 为未来的自己编写代码

> 原文：<https://itnext.io/writing-code-for-your-future-self-65e66e098303?source=collection_archive---------5----------------------->

![](img/3fa8eb9dd9ba06485773845c836cdad8.png)

照片由[王思然·哈德森](https://unsplash.com/@hudsoncrafted?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 拍摄

我们都写过糟糕的代码。让我们面对现实吧。即使我们尽了最大努力来编写最高效和设计良好的代码，我们都碰巧编写了这种肮脏的方法，我们对此并不感到自豪，并害怕在下个月或明年进行调试。

我不打算谈论那些代码，因为它很容易被发现，也没有必要写关于它的博文。我感兴趣的是代码，当你打完的时候看起来很好，但是有一天会在背后咬你一口。结果是很难维护、扩展甚至理解。经过多年的编码，我已经想出了一套警告，每当我走向死胡同时，它们都会提醒我。

# 意义

代码就是语言。当你使用像 Ruby 这样的编程语言时更是如此，Ruby 被设计成非常具有表现力。但对我来说，当代码变得不清楚它真正做什么时，它就开始变得有味道了。

## 名称

命名至关重要。在我的创建过程中变化最大的一件事可能是我设计的类、方法和变量的名字。人类给他们理解的事物命名。而且一个元素的知识越高，命名就越犀利。在开发过程中重命名我的类通常是一个好的迹象，表明我对它们做什么或代表什么概念有了更好的了解。

## 方法长度

作为有抱负的 OOP 开发人员，我们被教导的最常见的设计模式之一是保持方法短于几行。然而，有一点需要注意的是，代码最终会分散在各处，有时您会与数据的生命周期脱节。对于用于构建数据结构的类来说尤其如此。

例如，这个类构建了一个要发送给 FCM 的`data`散列:

```
module MobilePush
  class FcmNotification < BaseNotification

    def launch
      # Bootstrap stuff... push_service.send_message(project, data)
    end private
      # Lots of methods... def data
       {
          notification: json_notification,
          recipient: recipient_name,
          room: room_name
        }
      end # Lots of other methods... def json_notification
        @notification.to_json
      end def recipient_name
        @recipient.name
      end

      # Maybe a few other methods... def room_name
        @room.name
      end # The rest of the methods... end
end
```

*注意:为了简洁起见，这些例子有点过于简化。但是你明白了。*

我写这个想法*“嘿，真聪明！一切都整理得很好。看起来真的很像干净的代码"*。

然后我和家人一起休息了几天。当我不在的时候，我的一个同事必须处理我的新代码并添加一个新特性。但是他发现这是不必要的抽象，难以维护和扩展。他是对的。所以我把它重构为:

```
module MobilePush
  class FcmNotification < BaseNotification

    def launch
      # Bootstrap stuff... push_service.send_message(project, data)
    end private
      # Lots of methods... def data
       {
          notification: @notification.to_json,
          recipient: @recipient.name,
          room: @room.name
        }
      end # The rest of the methods... end
end
```

好吧，我不得不在班上的两三个地方把`room_name`换成`@room.name`，因为它在其他地方也被使用。但是我也让我的代码不那么抽象了。现在，我的同事一看就知道`data`是什么做的了。

# 故事

写代码时要记住的一件事是，它不仅仅意味着有效和简洁。代码应该被其他人阅读。你的同事，你的团队领导，甚至更重要的是，你的新代码库的负责人。

## 一行程序

我喜欢俏皮话。它们很有趣，直截了当，展示了创作者的技能和智慧。他们对于编程就像妙语或报纸标题对于写作一样。但是谁愿意读一整本这样写的书呢？

尽管我有时会在代码中使用一行程序，但我只有在确信我的继任者——或者未来的我——能够轻松理解它的时候才会这么做。

## 明确一点

我相信大部分代码应该是清晰的，用地标来引导读者，帮助他们理解。

例如，代替这种表达:

```
Device.where('last_seen > ?', 1.hour.ago).where(logging: true).where(notifications: true).last.send_message(topic, payload)
```

我宁愿要么使用类方法，要么使用模型中的作用域，最后得到:

```
device = Device.active_last(1.hour).logging.notifying.last
device.send_message(topic, payload)
```

我不傻。我能理解第一个例子。但是当解析数百行代码来查找 bug 时，我可以查看这些代码并立即判断它的作用。不需要确保我得到了正确的答案。这种方法在输入的当天就完成了一次，我可以放心地依赖它，因为这些方法都经过了测试，而且确实如它所说的那样。

## 有条不紊

我喜欢在课堂上把不同的意图分开。当你讲故事时，你首先用清晰的描述来介绍你的角色。那么只有你在你的故事里给他一个积极的角色。这就是上面的代码示例所做的。我也可以轻松地写下:

```
Device.active_last(1.hour).logging.notifying.last.send_message(topic, payload)
```

但这是混合意图。这一行既是对模型的查询，也是触发一个动作，所以你开始读这一行时会想:"*哦，他正在定义设备"*，结束时会想*"啊。不，他正在向这个装置发送一条信息“*”。你的读者永远不应该被误导，即使是像这样简单的断言。

我试图将查询与操作分开。它们是两种类型的任务，在我的代码中需要两个不同的步骤。

考虑这三句话:

> 酒吧里那个又老又脏的男人正在谈论昨天的足球比赛结果。—凌乱
> 
> 有一个人正在肮脏的旧酒吧里谈论昨天的足球比赛结果。—更清晰
> 
> 在肮脏的旧酒吧里，有一个男人。他正在谈论昨天的足球比赛结果。—清澈透明

我希望我的代码非常清晰。我希望我的代码审查者、我的继任者或我未来在周五下午非常疲惫的自己能够完美地理解我正在做的代码。

# 试验

编写测试帮助我识别依赖性。集成测试通常需要相当多的引导:一个登录的用户，一些相关的实体…这很好。

但是我有时会编写过于依赖其他模型的测试。如果没有 Post 模型就无法测试您的注释模型，那么代码中有些东西耦合得太紧了，需要查询那些实体之间的关系。

断言也引导我走向代码清晰。我发现，当我在测试中难以定义一个期望时，通常意味着这个特定的方法做得不够清楚。它要么需要被分割成更小的方法，要么完全重新考虑。

# 关于评论的一句话

代码需要重构的另一个标志是注释。评论大多指出程序做什么还不清楚。因此，当我在代码中看到注释时，我做的第一件事就是删除它们，并在不阅读它们的情况下找出发生了什么。无论何时出现混乱，这都是一个好迹象，表明它们都可以被一个`TODO: refactor`所取代。

注释通常也是外部依赖性的标志。通常，你会得到这样的东西:

```
# Handle temperature > 21# Deal with legacy customer IDs
```

第一个可能取决于您正在处理的设备，这种依赖性可能由硬件团队来解决。

第二个可以通过求助于一个继承的类来删除，比如`LegacyCustomer`，它可以处理那种特殊的用户情况，并通过一个可预测的简单接口来保持`Customer`类的整洁。

我保留的唯一注释是类声明上面的顶级注释、代码文档、一些重构 TODOs 和关于代码的愚蠢、有趣的 meta。

## 结论

我有时会看着凌乱的代码，想知道，*“这是谁写的垃圾？”*

让我想想…该死的…

*Me* 。

我不喜欢这种感觉。我想为我的代码感到骄傲。我希望其他开发人员，包括我未来的自己，能够立即轻松地理解它，并感谢我为编写有意义、易于调试和维护的代码所做的努力。

希望遵循这些指导方针能帮助我走上这条路。