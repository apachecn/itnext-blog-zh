# 用 MongoDB 改进你的 Twitter 机器人

> 原文：<https://itnext.io/improve-your-twitter-bot-with-mongodb-1f1e51e632d4?source=collection_archive---------8----------------------->

![](img/b56ee0e87209a6423fd6ec6fa65df9fa.png)

米海·沃尔高在 [Unsplash](https://unsplash.com/?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上的照片

在[的另一篇文章](https://medium.com/@mihailgaberov/creating-a-twitter-bot-in-5am-2a42a9920e67)中，我们看到了如何创建一个 twitter 机器人，向他的追随者发送私人消息，包含按预定义标准搜索的推文——我们选择的某些标签。这一次，我们将通过添加额外的功能来改进发送的报告。我们会将结果存储在数据库中，并根据我们存储的内容过滤新收到的推文。因此，我们将只能发送(*几乎** )唯一的结果。

**TL；DR** 这是一步一步教程[的第二部分](https://medium.com/@mihailgaberov/creating-a-twitter-bot-in-5am-2a42a9920e67)向你展示如何创建你自己的 Twitter 机器人，并把它放在 [heroku](http://herokuapp.com) 上。这次我们将通过使用[mongose](https://www.npmjs.com/package/mongoose)和 [MongoDB](https://www.mongodb.com) 来升级我们的代码，以产生更好的结果。此外，我们将通过提取消息内容的组合来扩展我们的*助手*的用法。最后，我们将看看如何使用 [Mlab](https://mlab.com/) 托管服务(免费计划)来托管我们的数据库。

如果你想直接跳到代码，这里是 GitHub repo。

**推理** 那样做的理由是什么？

> 现在有很多关于创建 twitter 机器人的教程和文章，但我找不到任何一个将这个主题扩展到下一个级别的教程和文章——为改进你的机器人处理数据的方式提供方向。

这就是为什么我决定在等式中引入一个新的玩家。一些将帮助我们做更多关于我们的 tweet 报告的事情——数据库。

**数据库** MongoDB 是著名的 [NoSQL](https://en.wikipedia.org/wiki/NoSQL) 数据库，与 [Node.js](https://nodejs.org/en/) 有很大的集成。它快速易用，非常适合我们的目的。在向关注者发送消息之前，我们将使用它来消除提要中的重复内容。

## 逐步地

如果你想坚持到底，以下是你可以采取的步骤:

1.  在 [Mlab](https://mlab.com/signup/) 中注册一个新账户。
2.  将您的 DB_USER 和 DB_PASSWORD 凭据添加到您的。env 文件(与我们在第一部分[中使用 Twitter API 键的方式相同)。](https://medium.com/@mihailgaberov/creating-a-twitter-bot-in-5am-2a42a9920e67)
3.  在项目的根文件夹中创建一个名为“/ *db* 的新目录，并在其中添加一个新文件: **db.js** *。这里是我们放置与数据库连接的逻辑的地方。*
4.  在文件的开头，导入[mongose](http://mongoosejs.com/)——一个 [MongoDB](https://www.mongodb.org/) 对象建模工具，设计用于在异步环境中工作。稍后我们会看到，当使用**mongose**和 **Node.js** 时，使用 MongoDB 是多么容易。

```
const mongoose = require('mongoose')
```

5.下一步是组成我们的连接字符串。为了能够连接和使用我们的数据库，我们需要这个。它应该是这样的:

```
const dbURI = `mongodb://${process.env.DB_USER}:${process.env.DB_PASS}@ds219130.mlab.com:19130/twitter-bot-db`
```

6.这个[文件](https://github.com/mihailgaberov/twitter-bot/blob/v2/db/db.js)的其余部分只是来自*mongose、*的简单逻辑，涉及为可能发生的不同事件设置不同的监听器。其中一个事件是“*连接*”，它告诉我们何时成功连接到数据库。

7.设置您的数据库模型——在同一个文件夹中创建一个名为 **tweets.model.js** 的新文件。这是我们数据库模型背后的逻辑。一个简单的[mongose 模式](http://mongoosejs.com/docs/guide.html)，作为我们数据库文档的蓝图。它只包含四个字段。这个唯一的 ID 参数叫做 i *d_str。*我们将在检查重复时使用它。推文的内容，作者名称*和推文的创建日期*创建于*。所有的字段都定义为 [*字符串和* *必填*](http://mongoosejs.com/docs/schematypes.html) *。**

```
const tweetSchema = new Schema({ 
  id_str: { 
    type: String, 
    unique: true,
    required: true 
  }, 
  status: { 
    type: String, 
    required: true 
  }, 
  author: {
    type: String,
    required: true 
  }
  created_at: {
    type: String,
    required: true
})
```

8.启动和运行数据库的最后一步是创建数据库控制器。该文件将包含与数据库交互的主要逻辑。在这里，我们将把读写记录的方法放入其中。同样在这里，我们将实现仅从每个提要获取唯一 tweets 的功能，并记录它们以供发布使用。首先，在“ */db* ”文件夹中创建一个新文件，将其命名为 **tweets.controller.js** ，并在其中粘贴[此代码](https://github.com/mihailgaberov/twitter-bot/blob/v2/db/tweets.controller.js)。您可能已经看到，在这个文件中，我们导出了一些方法，其中一些被定义为局部常量。遵循最佳实践，我们将在我们的主流程中只使用导出的。

> 通过将其余部分定义为本地，我们限制了对外部世界不必要的暴露。

本地方法，仅在此处使用:

```
const fetchTweets = function () { ... }
const recordTweet = function ({ id_str, text, user }) {...}
```

导出方法，用于 [*bot.js*](https://github.com/mihailgaberov/twitter-bot/blob/v2/bot.js) :

```
module.exports.recordUniqueTweets = function (tweets) {...} module.exports.getUniqueTweets = function (latestTweets) {...}
```

**助手** 在我的[上一篇文章](https://medium.com/@mihailgaberov/creating-a-twitter-bot-in-5am-2a42a9920e67)中我提到了提取或模块化可以在许多其他地方重用的功能有多好。我们用获取当前时间的方法做到了这一点。这次我们将向[**helpers . js**](https://github.com/mihailgaberov/twitter-bot/blob/v2/helpers.js)**，**添加另一个方法，我们将使用它来生成消息内容。如果由于任何原因，我们没有收到任何新的推文，我们将向关注者发送消息。比如这个:

```
*`-— Nothing new this time (14.05.2018 14.02).`.*
```

此外，我们将稍微重构一下获取当前时间的方法。然后我们将能够在编写消息内容时使用它来添加推文创建的日期。

```
const from = this.getTime(tweet.created_at)
```

**更新主流程** 在我们完成与数据库相关的任务后，是时候修改我们的[主应用程序流程](https://github.com/mihailgaberov/twitter-bot/blob/v2/bot.js)并使其开始使用新的设置了。实际上，我们需要做的改变很小。我们需要从" */db* "文件夹中导入 **db.js** 和 **tweets.controller.js** 文件，如下所示:

```
require('./db/db')
const db = require('./db/tweets.controller')
```

然后，在获取 tweets 的第一个调用的回调中，用以下代码替换 *forEach* 循环:

```
 db.getUniqueTweets(data.statuses).then((tweets) => {        
     const content = helpers.composeContent(tweets)
     db.recordUniqueTweets(tweets)
  ...
})
```

仅此而已。👌

你可以在这里找到代码[的最终版本](https://github.com/mihailgaberov/twitter-bot/blob/v2/bot.js)。

**托管在 Mlab 上** 好了，现在我们完成了编码，是时候发布我们更新的应用程序和它使用的数据库了。为此，我们将对数据库使用 [Mlab](https://mlab.com/welcome/) ，对应用程序本身使用 [Herokuapp](http://herokuapp.com/home) 。

如果你在使用 Heroku 部署应用程序时需要帮助，你可以看看本教程的第一部分。只是不要忘记在“设置”标签下的“配置变量”部分添加新的配置变量。

为了开始使用 Mlab 上的数据库，您首先需要登录到您的帐户，并使用提供的控制面板创建一个新的数据库。

![](img/b45159e4ec3da95383b12836da867939.png)

创建后，只需转到它的设置并找到需要添加到应用程序中的连接字符串。

![](img/22938be071033a2920107a90c0e48216.png)

> 如果你已经完成了本教程的编码部分，你将不得不添加一个集合到数据库中，并将其命名为“tweets”，以便与代码保持一致，并最终获得一个工作的应用程序。

一旦正确创建了数据库，连接字符串就在您的代码中就位，应用程序就通过 Heroku 部署好了。🎆 🆒

**结论** 这篇文章有两个目的。一是成为第一篇[文章](https://medium.com/@mihailgaberov/creating-a-twitter-bot-in-5am-2a42a9920e67)的逻辑延续。其次是展示我们如何轻松地使用 noSQL 数据库(如 MongoDB)和 Node.js 以及服务(如 Mlab 和 Heroku)来实现简单 Twitter bot 应用程序的改进版本。值得一提的是，通过这种方式，我们实际上正在创建自己的 twitter 状态数据库，以后可能会用于其他用途。也许是一些统计研究或机器学习的材料，谁知道呢。最棒的是，一切都是自动发生的。

> 总会有选择的。我们只需要想想他们。

注意:*如果不想使用 Mlab 提供的用户界面，进行与数据库的交互，可以使用其他工具，比如*[*studio 3t*](https://studio3t.com)*。*

当然，欢迎任何评论或问题。

**🔥编码快乐！🔥**

*你可以在这里找到我的实现*[*https://github.com/mihailgaberov/twitter-bot*](https://github.com/mihailgaberov/twitter-bot/tree/v2)*。如果你想开始收到和我一样的报告，请在 Twitter @HerrKlinkerhof3 中关注我的机器人。*

我说的是“几乎”，因为在某些情况下，你可能会收到内容相同的推文，但发布者不同。根据已实施的检查，这些是不同的，因为它们具有不同的 id，但仍然…

哦…还有**感谢**阅读。😅