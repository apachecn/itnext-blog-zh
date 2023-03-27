# 将外部 API 数据保存到您自己的 Rails API 中

> 原文：<https://itnext.io/saving-external-api-data-to-your-own-rails-api-fad0fa75066?source=collection_archive---------5----------------------->

如果您试图将外部 API 数据持久化到自己的后端，您可能会发现这篇博客很有帮助。

这是我基于这个项目的演示视频。

如果您正在阅读本文，您可能已经理解了什么是获取请求及其重要性。如果你不知道，你可以在这里停下来，用 fetch 快速阅读[。](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch)

现在，既然您熟悉 fetch 请求，您就知道它们用于在前端 JavaScript 接口上获取、发布、放置和删除数据。有些事情你可能不知道，你在这里的原因是，也有一种方法可以在 Ruby on Rails 应用程序的后端做到这一点。

提示 RestClient 请求！！！

您可能希望利用 RestClient 调用有几个原因。

1.  您正在使用的外部 API 具有旋转数据，并且您想要保存所有这些数据以供回顾(或进行纵向研究)
2.  您对某个外部 API 的 API 调用次数有限，并且您希望限制调用次数
3.  你使用了太多的外部 API 来保持代码的整洁
4.  以上所有以及更多

在我的例子中，我使用 RestClient 将 5 个外部 API 的数据保存到我自己的 Rails API 中。我选择使用 RestClient 的原因如上。

最初，我使用 fetch 请求从前端进行这些调用，但这是一个耗时的过程，它降低了我的应用程序的速度，并使我的代码冗长而混乱。在我的例子中，RestClient 调用将我的所有数据保存在我的数据库中，这使得从我的前端到我自己的个人 Rails API 的一个单一获取请求就可以很容易地到达。

# 项目背景

我决定开展一项研究，对纽约时报、福克斯新闻频道、美国广播公司、英国广播公司和美国有线电视新闻网的文章进行情感分析。使用 React 前端和 Rails 后端，我这样做是因为显然有大量的混乱和与主要新闻站和普通大众的脱节。我想看看当我并排展示数据时，他们文章背后的情绪是否讲述了什么故事。对于这个项目，我使用了 [Indico 情感分析 API](https://indico.io/docs#custom) 。它非常容易使用，我推荐它来获得表面层次的情感。我不太敢说你真的可以相信这些结果，因为它很难理解背景(这是我研究中的一个主要缺陷)。我也开始构建自己的情感分析人工智能，但我的旧 MacBook Pro 花了几个小时来训练我的模型，所以为了节省时间，我放弃了它。

现在，这是我的一个后端 RestClient 文件的样子。我从《纽约时报》API 中提取数据，同时对每篇文章进行情感分析，并将其保存到我自己的数据库中。我已经对代码进行了注释以供参考(所有跟在“#”后面的内容)。如果你想做类似的事情，你应该能够复制一个类似的结构。请特别注意页面顶部需要的宝石、正在解析的数据，以便我能够迭代每个单独的结果或文章，我采取的措施以确保我的数据库中没有重复，以及我如何保存文章(输出可以在页面底部的 GIF 中看到)。

```
require 'rest-client'
require 'indico'
require 'rails/configuration'#New York Times API endpoint
  nyt_url = "[https://api.nytimes.com/svc/topstories/v2/home.json?&api-key=](https://api.nytimes.com/svc/topstories/v2/home.json?&api-key=)"#RestClient get request to the NYT endpoint with my API Key being parsed into a JS object
  data = JSON.parse( RestClient.get("#{nyt_url}#{ENV["NYT_API_KEY"]}") )#Iterating through each result/article of the NYT
  data["results"].each do |article, index|
    #finding existing articles
    existing_article = Article.find_by(headline: article["title"])
    #checking if the articles abstract is there and if the article doesnt exist
    if article["abstract"].length > 0
      if !existing_article
        config = {api_key: "#{ENV["INDICO_API_KEY"]}"}
        #Running sentimental analysis on the article's abstract
        emotion = (Indico.emotion(article["abstract"], config))#Creating a new article in my database and assigning it's properties
        news = Article.new do |key|
          key.headline = article["title"]
          key.news_station = "New York Times"
          key.abstract = article["abstract"]
          key.category = article["section"]
          key.url = article["url"]
          #Some of the articles didnt have pictures so I made a default one if there was no pic
          if article["multimedia"].length === 0
            key.image = "[http://www.nytimes.com/services/mobile/img/ios-newsreader-icon.png](http://www.nytimes.com/services/mobile/img/ios-newsreader-icon.png)"
          else
            key.image = article["multimedia"][0]["url"]
          end
          #Saving sentiments
          key.anger = emotion["anger"]
          key.joy = emotion["joy"]
          key.fear = emotion["fear"]
          key.sadness = emotion["sadness"]
          key.surprise = emotion["surprise"]
          key.date = Time.now
        end
        #Saving articles
        if news.save
          puts "saved nyt"
        else
          puts "not saved"
        end
      end
    end
  end
```

好的，所以代码是有意义的…但是我如何让它把文章拉进我的数据库，我在哪里写这个代码？

有几种不同的方法可以做到这一点。在我的例子中，我决定在服务器初始化时这样做。在做这个项目的时候，我一天要启动我的服务器好几次，所以我会把所有发表的文章都拉进来。然而，我不会说这是最好的方法。它对我有用，因为我选择不将我的项目投入使用；我的数据库太大了，无法在 Heroku 上运行。

如果您想像我这样做，我将把我的 Rails API 的文件结构留在这里。

```
-app
-bin
-*config*
--environments
--*initializers
---<RestClient file as seen above here>
--*locales
-db
-lib
-log
-public
-tmp
-vendor
```

下面是我的 Rails 服务器启动时的样子。正如你所看到的，在初始化服务器时，它为每篇被保存的文章输出“saved <news station="">”。</news>

如果你想把你的项目投入使用，我会建议你不要在初始化时更新。这里有两种方法可以让你在每次上传新文章时或者在设定的时间间隔内更新你的服务器。

## 克朗·乔布斯

[](https://github.com/javan/whenever) [## javan/无论何时

### Ruby 中的任何时候- Cron 作业

github.com](https://github.com/javan/whenever) 

## Heroku 上的计划作业

[](https://devcenter.heroku.com/articles/scheduled-jobs-custom-clock-processes) [## 计划作业和定制时钟流程| Heroku 开发中心

### 使用 Scheduler 插件或通过实现自定义时钟流程，在 Heroku 上计划重复或基于时间的作业。

devcenter.heroku.com](https://devcenter.heroku.com/articles/scheduled-jobs-custom-clock-processes) 

一旦我弄清楚了这个项目的 RestClient 请求，事情就一帆风顺了。祝你好运！