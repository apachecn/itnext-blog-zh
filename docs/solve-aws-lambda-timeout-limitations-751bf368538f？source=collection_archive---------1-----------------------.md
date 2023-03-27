# 解决 AWS Lambda 超时限制

> 原文：<https://itnext.io/solve-aws-lambda-timeout-limitations-751bf368538f?source=collection_archive---------1----------------------->

![](img/3c23ed6b8fa58bdae1630cf701c81821.png)

当我一直在起草一个解决 AWS Lambda 超时限制的策略时，我觉得我离解决一个有时让我睡不好觉的问题越来越近了。诚然，有一些已知的模式可以让[使用 SQS](https://www.jeremydaly.com/serverless-microservice-patterns-for-aws/) 达到更高的可扩展性，尽管当 AWS Lambda 被用作核心计算服务时，手头的问题不容易用其中任何一种来解决。

在我关注超时限制问题的同一时期， [AWS 将限制提高到 15 分钟](https://aws.amazon.com/about-aws/whats-new/2018/10/aws-lambda-supports-functions-that-can-run-up-to-15-minutes/)。尽管这肯定是一个有用改变，但我仍然倾向于使用一个没有任何限制的非 lambda 计算来完善我的策略草案。当安全网计算也基于按需求定价模型时，这形成了问题的真正解决方案。实际上，有时不可能预先预测负载，最大限度地提高负载并不是最终的解决方案。

# 显著的改进

以下是我在草案实现和当前实现之间所做的最重要改进的简要概述:

*   [unzip](https://www.npmjs.com/package/unzip) 已经被交换到 [unzipper](https://www.npmjs.com/package/unzipper) 中，理由[充分](https://github.com/EvanOxfeld/node-unzip/issues/120)。API 是一样的。
*   粘合主要和次要 lambda 处理程序的逻辑已更改。我没有使用容易出错的命名约定，而是放置了一个映射“备忘单”。
*   [server less-plugin-lambda-dead-letter](https://github.com/gmetzker/serverless-plugin-lambda-dead-letter)依赖性看起来很有前途，但为了支持 CloudFormation 的原生功能，它被移除了。主要原因:[固有函数的问题](https://github.com/gmetzker/serverless-plugin-lambda-dead-letter/issues/37)。
*   主处理程序中的`event`和`context`已经通过 [ContainerOverride](https://docs.aws.amazon.com/AmazonECS/latest/APIReference/API_ContainerOverride.html) API 移动到环境变量中，因为在执行 [runTask](https://docs.aws.amazon.com/AmazonECS/latest/APIReference/API_RunTask.html) 的过程中，来自原始 JSON 对象的信息被丢弃。就管理环境变量的一致性而言，这也是一种改进。
*   通过相同的 [GetFunction](https://docs.aws.amazon.com/lambda/latest/dg/API_GetFunction.html) 使用来自主处理程序的原始环境变量，该 GetFunction 产生这个[配置](https://docs.aws.amazon.com/lambda/latest/dg/API_EnvironmentResponse.html?shortFooter=true)以及已经获取的主处理程序的源的位置。
*   增加了一个助手来处理不同的`event`结构。看起来 SNS 事件会根据存储桶中管理文件的方式而有所不同。

最终实现的框架可以在这个[库](https://github.com/kalinchernev/immortal-aws-lambda)中看到。

# 履行

那么，经过前面提到的改进后，当前的实现看起来如何呢？

从鸟瞰图来看，结构和主要思想是相同的:

```
**immortal-aws-lambda** 
├── **container** 
│   ├── Dockerfile 
│   ├── package.json 
│   ├── README.md 
│   └── **runner.js** 
└── **serverless** 
    ├── package.json 
    ├── README.md 
    ├── serverless.yml 
    ├── **src** 
    │   ├── **events** 
    │   │   └── onFailure.js 
    │   └── **lib** 
    │       ├── extractors.js 
    │       ├── getHandlerData.js 
    │       └── snsTopicToHandlerMap.js 
    └── webpack.config.js 

5 directories, 12 files
```

# [不朽的 AWS Lambda:容器服务](https://github.com/kalinchernev/immortal-aws-lambda/tree/master/container)

您可能希望首先在 AWS 控制台中创建 ECS 任务，并对无服务器服务进行一些设置。

该服务的唯一目标是将一个`runner.js`脚本放入一个容器中，并从死信队列服务远程运行它。

该脚本的内容实际上非常简单，由 3 个主要步骤组成:

1.  获取 lambda 处理程序的初始源代码
2.  采用同一个处理程序的环境变量
3.  运行处理程序

所有设置都是动态变量。

来源非常简单:

# [不朽的 AWS Lambda:无服务器服务](https://github.com/kalinchernev/immortal-aws-lambda/tree/master/serverless)

这是要部署的无服务器服务。它尽可能简单和独立:

*   提供一个死信队列`LambdaFailureQueue`，当失败时其他人可以将消息推送到这个队列。
*   导出队列的 ARN，以便其他服务能够导入 ARN 的值。([文档](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/outputs-section-structure.html))
*   关于`iamRoleStatements`、`events`订阅和设置的所有其他细节与之前相同。

在`lib`的助手是你的责任，因为`event`结构在你的案例中会有所不同。(最有可能)

尽管如此，拥有此服务的主要目的仍然是运行 ECS 任务来启动容器:

不要忘记从 AWS 控制台获取这些设置，并将它们设置在您的`serverless.yaml`配置文件中。

# 在工作流中集成服务

将其他无服务器服务和处理程序“附加”到此工作流可归结为以下几点:

1.  允许服务将消息推送到 SQS 死信队列:

```
iamRoleStatements:
  # Allow queueing messages to the DLQ https://docs.aws.amazon.com/lambda/latest/dg/dlq.html
  - Effect: 'Allow'
    Action:
      - sqs:SendMessage
    Resource: '*'
```

2.从`Resources`部分添加关于 SQS 队列的 ARN 信息

```
resources:
  Resources:
    fooFunction:
      Type: "AWS::Lambda::Function"
      Properties:
        DeadLetterConfig:
          TargetArn:
            Fn::ImportValue: immortal-aws-lambda:LambdaFailureQueue
```

这是因为无服务器框架还不能正确支持`[onError](https://www.trek10.com/blog/dead-letter-config/)`。感谢 [Siva Kommuri](https://github.com/neowulf) 为[建议了这个变通办法](https://github.com/neowulf)。

现在，当您的服务失败时，错误将被排队到由不朽的 aws lambda 服务提供的死信队列中，不朽的服务将接收此消息，找到正确的处理程序并通过容器服务调用它。

# 最后的想法

我找到这个解决方案的道路并不容易。

所用的工具边缘粗糙。

此外，由于超时、重建容器等原因触发和再现故障的过程。每次迭代都是一个漫长的过程。例如，每次由于缺少字符或拼写错误而失败时，我需要将 lambda 函数的非捆绑包和非优化代码重新部署到云中，以便在 ECS 的日志中获得足够的错误消息用于调试。(疯了！)

顺便说一下，在 Node 中处理流和承诺仍然非常痛苦，很难调试…

因此，我希望拥有这些相互通信的非常薄的变量层将是解决 AWS Lambda 未来几个月超时限制的可行解决方案。

*最初发布于*[*kalinchernev . github . io*](https://kalinchernev.github.io/solve-aws-lambda-timeout-limitations)*。*