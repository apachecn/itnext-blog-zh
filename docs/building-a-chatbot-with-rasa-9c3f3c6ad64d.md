# 用 Rasa 构建聊天机器人

> 原文：<https://itnext.io/building-a-chatbot-with-rasa-9c3f3c6ad64d?source=collection_archive---------0----------------------->

![](img/cec430cd6ce32198c65993edcd193aaf.png)

## 在 30 分钟内创建您自己的聊天机器人！:)

## 介绍

在开始开发之前，让我们先来详细说明一下需求，以及为什么我要深入研究提到的技术。我想建立一个聊天机器人，它能够学习用户的意图，智能地交互，如果用户要求就执行操作，提供有效的学习机制，最重要的是不使用任何付费服务。有一堆在线服务(如 [wit.ai](https://wit.ai/) )提供不错的 NLU 功能，但要收取流量费。这让我开始寻找一个开源框架，它可以在构建机器人的过程中提供足够的独立性。一个这样的工具是 [Rasa](https://rasa.com/) 。

## Rasa —聊天机器人解决方案

Rasa 提供了一套工具，可以在你的本地桌面上构建一个完整的聊天机器人，而且是完全免费的。他们的旗舰工具是，

*   **拉莎 input】:一种自然语言理解解决方案，它接受用户输入，并试图推断用户意图，提取可用实体。**
*   Rasa 核心(Rasa Core):一个对话管理解决方案试图建立一个概率模型，该模型基于先前的一组用户输入来决定要执行的一组动作。

在这篇文章中，你会发现一些关键词在 Rasa 函数/工具中反复出现，

*   **意图**:将其视为用户输入的目的或目标。如果用户说，“*今天是哪一天*？”，目的是找到一周中的某一天。
*   **实体**:认为是从用户输入中可以提取的有用信息。从前面的例子中，我们知道我们的目标是找到一周中的某一天，但是是哪一天呢？如果我们提取“ *Today* ”作为实体，我们可以执行今天的操作。
*   **动作**:顾名思义，是可以由 bot 执行的操作。它可能是回复一些东西，查询一个数据库，或者任何其他可能的代码。
*   **Stories** :这是用户和机器人之间的一个示例交互，根据捕获的意图和执行的动作来定义。因此，开发人员可以告诉你，如果你得到一个带有/不带有某些实体的用户输入，该怎么做。比如说，如果用户的意图是查找星期几，而实体是今天，则查找今天的星期几并回复。

为了完成这项工作，Rasa 需要一些文件，这些文件存储了构建机器人所需的所有训练和模型信息。简单概述一下，最重要的有:

*   **NLU 训练文件**:它包含一堆用户输入的例子，以及它们到合适的意图的映射和每个例子中存在的实体。你提供的例子越多，你的机器人的 NLU 能力就越强。在这里找到一种创建训练数据的交互方式[。](https://rasahq.github.io/rasa-nlu-trainer/)
*   **故事文件**:里面有一堆可以学习的故事。每个故事都创造了一个互动的概率模型。
*   域文件:这里你列出了所有的意图、实体、动作和类似的信息。您还可以添加示例 bot 回复模板，并将其用作操作。

说够了，让我们开始开发机器人吧！

## 自然语言理解

让我们首先训练机器人理解用户语言，这样它就可以对意图进行分类并提取实体。为此，我们将创建一个示例 NLU 培训文件。

**NLU 培训档案(nlu_train.md):**

这里，意图的名称被定义在' ##intent:'之后，并且该意图的所有样本用户话语在下面被提及。意向通知` *query_days_in_month`* 我们还提到了实体。格式是[word](entity_name)，因此用户话语示例是“一月有多少天”，其中“一月”是名称为“月”的实体。

现在，我们将定义用于训练 NLU 模型的管道，这里我们将使用 spacy 来完成此操作。

**配置文件(nlu_config.yml)**

我们现在准备训练 NLU 模型！让我们用 Python 来做吧。

这将训练 nlu 模型并将其保存在“models/nlu/”中。让我们试着测试这个模型是如何工作的。

输出看起来像这样，

如您所见，该模型能够完美地将用户问题与其意图对应起来(检查' *intent* '下的' *name* '部分)。它说，对于第一个问题，意图是' *query_days_in_month* ，提取的实体是' *January* '(检查' *entities* '下的' *value'* )。第二个问题的输出中有一件很酷的事情，即使我们没有在示例中提供它，它也完全能够猜出意图，甚至提取实体' *march* '。

不错，但聊天机器人只是完成了一半，我们仍然需要建立对话管理。

## 对话管理

首先，让我们定义域文件，该文件包含一些示例模板，我们可以使用这些模板回复用户。

**域文件(domain.yml):**

这里我们列出了意图和实体(都在 nlu 培训文件中)，以及一些模板和动作。模板包含您希望机器人做出的回复，在这种情况下，我希望机器人问候，说再见，并回答用户提出的一个月中的天数问题。现在机器人只会回答这三种月份的答案(机器人不知道闰年，愚蠢的机器人)

现在让我们创建一个用户-机器人交互的例子。

**故事文件(stories.md):**

在这里，我们以故事的形式定义了一个机器人和用户之间的示例交互。事情是这样的，如果用户说了什么，并且它的意图是'*问候*'机器人将执行动作'*发出问候*'。再举一个例子，如果用户的消息有' *query_days_in_month* '意图和'二月'，那么 bot 将执行' *utter_answer_28_days* '动作。

现在让我们来训练对话管理部分，

在这里，您可以更改培训策略，在该策略中，您可以为对话培训定义自己的 LSTM 或 RNN。重要的一点是，' *max_history* '用于定义一个动作如何依赖于前面的问题。如果 max_history 为 1，那么它只记忆个体意图及其相关动作。由于缺乏训练数据(大量的故事)，我现在只是让它记住规则，你可以创建一些样本故事，看看 rasa 对话管理的真正潜力。

现在我们完成了。让我们看看机器人是否能够回复我们并回答我们的问题。

确实如此。

## 结论

将意图-实体的每种组合映射到故事中的动作是非常低效的。想象一下将每个月单独映射到它的答案。闰年呢？嗯，所有这些都可以通过使用自定义操作来处理，其中操作调用一个 python 函数，包含所有信息，如意图和实体。现在，在 python 代码中，您可以执行所有必要的检查并返回消息。

此外，当你给它更多的数据来训练时，Rasa NLU 和核心的真正潜力会大放异彩。或者甚至部署聊天机器人，它从互动中学习。这将使它能够处理尚未输入训练数据的案例。您还可以更改它为模型定型的策略。也许在下一篇文章中会谈到这些。

干杯。

点击查看更多此类文章[。](http://mohitmayank.com)