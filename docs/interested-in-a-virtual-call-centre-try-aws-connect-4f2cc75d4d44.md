# 对虚拟呼叫中心感兴趣吗？尝试 AWS 连接

> 原文：<https://itnext.io/interested-in-a-virtual-call-centre-try-aws-connect-4f2cc75d4d44?source=collection_archive---------5----------------------->

## 通过 AWS Connect 创建基于虚拟云的呼叫中心。

![](img/a1062d66392b6f4c1ce11d0ca679b7ec.png)

来源:https://aws.amazon.com/connect/

最近，我发现自己花了一些时间在一些不太知名的 AWS 服务上，我想引起人们对这些服务的关注。

其中之一 AWS Connect 被证明是一个有趣的用例。

随着远程工作需求的不断增长，在新冠肺炎疫情爆发期间，远程办公的使用量有所增加。它允许公司创建一个基于虚拟云的呼叫中心，使员工能够在任何有电脑的地方接听电话。

## 基于云的呼叫中心？

AWS Connect 将自己标榜为*“全渠道云联络中心”*，但这到底意味着什么呢？

AWS Connect 是一种构建和管理完全无服务器呼叫中心的通用方式，允许分布在世界各地的团队远程工作。

它可以用作管理代理和连接客户的简单方法，也可以用作构建复杂路由系统的方法，该系统可以使用多个客户输入和分叉路径将客户路由到不同的代理团队。

![](img/02a85011da379b4361b5331069176b60.png)

来源:[https://www . pexels . com/photo/woman-weaking-earpier-use-white-laptop-computer-210647/](https://www.pexels.com/photo/woman-wearing-earpiece-using-white-laptop-computer-210647/)

AWS Connect 与传统呼叫中心的区别在于，它能够在几分钟内创建和扩展呼叫中心，并实现远程工作，因为它依赖于网络界面，而不是传统的手机。

AWS Connect 为以下各项提供了一体化服务:

*   获取公共电话号码
*   创建分布式团队
*   创建运营时间
*   创建简单到复杂的呼叫路由
*   AWS 上呼叫日志的安全数据存储和加密
*   与 AWS 数据库集成，实现自动日志和统计
*   为代理提供一个简单的用户界面来应答呼叫，而不需要物理手机

## 有多简单？

AWS Connect 是那些设计良好的产品之一，可以非常简单，也可以非常复杂，这取决于您的设计和您使用的组件。

它被设计为任何人都可以使用，并且不需要开发人员的配置经验，尽管通过 AWS KMS 了解一些 S3 桶和数据加密是有益的。

![](img/7d18f483f2c898850a07ec7275ded53a.png)

来源:[https://unsplash.com/photos/BeVGrXEktIk](https://unsplash.com/photos/BeVGrXEktIk)

在没有任何经验的情况下，在我第一次尝试使用 AWS Connect 时，我能够在 15 分钟内建立并运行一个完整的无服务器呼叫中心，包括:

*   公共拨入号码
*   通话记录的安全加密数据存储
*   具有不同管理权限的经理和代理的资历角色
*   关于呼叫指标和统计的报告
*   代理的工作时间
*   一种呼叫路由，利用键盘输入和队列检查，在没有代理可用时让客户等待
*   3 种不同的用户类型(管理员、经理、代理)，可以通过 PC 和耳机登录和接听来电

## 复杂性，如果你想要的话

另一方面，AWS Connect 支持大量定制和支持服务。

AWS Connect 可以集成到 AWS Lambda 和 AWS Lex 中，这意味着可以编写脚本来实现以下一些功能:

*   语音到文本翻译—为代理提供呼叫摘要
*   与 AWS 数据库解决方案集成—提供可查询的统计数据和呼叫指标
*   语言检测—允许在通话中识别和标记关键词和短语，以帮助了解总体客户满意度

![](img/1ecc8618fb94079ee67970b177b72224.png)

来源:[http://anthillonline . com/WP-content/uploads/2018/07/chatbot . jpg](http://anthillonline.com/wp-content/uploads/2018/07/chatbot.jpg)

AWS Connect 是一个不可思议的多功能和可扩展平台，允许公司建立一个定制和灵活的虚拟呼叫中心。使他们能够适应规模、灵活性和分布的压力，以克服传统呼叫中心的障碍和僵化结构。

这只是对 AWS connect 的一个高度概括，在可预见的未来，我将会整理出一个更具技术性的指南，所以请关注这个空间。

更多关于 AWS 和无服务器的信息，请随时查看我的其他博客和我的网站， [Manta 创新](https://manta-innovations.co.uk/)。