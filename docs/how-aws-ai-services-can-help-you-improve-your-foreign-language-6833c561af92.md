# AWS 人工智能服务如何帮助您提高外语水平

> 原文：<https://itnext.io/how-aws-ai-services-can-help-you-improve-your-foreign-language-6833c561af92?source=collection_archive---------8----------------------->

![](img/060c16198ce67924939e47caf1cb2468.png)

照片由[元素 5 数码](https://unsplash.com/@element5digital)在 [Unsplash](https://unsplash.com/photos/qTqPAJHJ4Gs) 上拍摄

AWS 提供几种人工智能(AI)服务。有了人工智能服务，你可以实现一些有用的人工智能东西:图像和视频分析、文档分析、文本到语音或语音到文本的翻译，等等。然而，这些 AWS 服务不仅可以用于企业应用程序，还可以用于您的自我开发应用程序。

我们现在对三种人工智能服务感兴趣:

*   Polly —文本到语音服务。它通过文本产生音频。
*   [转录](https://aws.amazon.com/transcribe/) —语音转文字服务。它可以识别语音到文本。
*   [翻译](https://aws.amazon.com/translate/) —翻译服务。它将文本从一种语言翻译成另一种语言。

# 技能

应用这些服务，我们能够实现一个应用程序来提高我们的外语技能。

让我们将 AWS 人工智能服务映射到语言技能:

*   试听技巧— `Polly`允许通过文本生成音频，帮助您听到未知单词和短语的发音。
*   发音技能— `Transcribe`允许将您的语音识别为文本，这将帮助您训练新单词和短语的发音。
*   词汇量— `Translate`允许将单词、短语或整个文本从您的语言翻译成外语，反之亦然，这将帮助您理解未知的单词和短语。

它没有涵盖所有的技能，但我们可以通过这种方式发展其中的一些技能。

**注意** : AWS AI 服务有不同的支持语言集，所以有可能其中一个不支持你的语言。此外，这些服务支持有限数量的语言。

# 演示应用程序

我编写了一个演示应用程序来展示这个概念的证据。位于[我的 GitHub 库](https://github.com/Jaitl/aws-lambda-telegram-lang-demo)的演示应用。

我使用 Telegram Bot API 作为应用程序的 UI。但是，您可以选择任何您想要的 UI，CLI、Web、本机 OS 表单等等。

电报提供了几种适合我们的消息类型:

*   语音消息——我们可以下载这个消息作为一个`OGG OPUS`文件。AWS 转录服务接受`OGG OPUS`格式作为语音识别的输入。因此，我们将能够发送电报语音信息的权利，在转录服务没有转换。
*   音频消息—我们可以将 MP3 文件作为音频消息发送。AWS Polly 服务返回一个 MP3 文件，因此我们将能够将 MP3 文件发送到 Telegram，无需转换。

我利用这两种消息类型来实现演示应用程序的两个主要特性:

*   用户发送语音消息作为响应，他接收到消息的文本。
*   用户发送文本消息作为响应，他接收音频消息和来自文本的翻译。

## 语音消息识别

正如我上面所说，识别语音信息，我们称之为 AWS 转录服务。转录服务支持流式传输，因此我们将电报语音消息流式传输到这一服务。转录流需要三个实用程序类`[AudioStreamPublisher](https://github.com/Jaitl/aws-lambda-telegram-lang-demo/blob/main/src/main/kotlin/com/github/jaitl/aws/telegram/english/aws/steamming/AudioStreamPublisher.kt)`、`[SubscriptionImpl](https://github.com/Jaitl/aws-lambda-telegram-lang-demo/blob/main/src/main/kotlin/com/github/jaitl/aws/telegram/english/aws/steamming/SubscriptionImpl.kt)`和`[StreamResponseHandler](https://github.com/Jaitl/aws-lambda-telegram-lang-demo/blob/main/src/main/kotlin/com/github/jaitl/aws/telegram/english/aws/steamming/StreamResponseHandler.kt)`。这些文件不包含业务逻辑，所以这里不会显示它们的代码。

下面的函数接受二进制流作为输入，创建一个请求来转录流并返回一个识别的文本。

下面的函数以流的形式接收电报语音消息文件，用流调用我们的`transcribe`函数，接收语音消息的识别文本，并将文本返回给用户。

我们不将电报语音消息文件下载到我们的文件系统，因为我们从文件中获取流，并将该流直接发送到转录服务。

## 文本翻译

将文本从一种语言翻译成另一种语言，我们称之为 AWS 翻译服务。下面的`translate`函数接受文本，将文本发送给服务，并返回翻译后的文本。

## 音频消息生成

从文本生成音频文件，我们称之为 AWS Polly 服务。下面的`synthesizeSpeech`函数接受文本作为输入，调用服务并返回一个包含生成的 MP3 文件的字节数组。

下面的函数接受来自 Telegram 的文本消息，调用我们的`translate`函数翻译文本，调用我们的`synthesizeSpeech`函数生成 MP3 文件，然后将结果返回给用户。

在这种情况下，我们也不使用我们的文件系统，因为我们将 MP3 文件保存到 RAM，然后将文件从 RAM 直接发送到 Telegram。

我只向您展示了我的演示项目代码的重要部分。完整代码位于[我的 GitHub 库](https://github.com/Jaitl/aws-lambda-telegram-lang-demo)中。请随意从存储库中读取代码并运行它。

# 结论

有了这三个 AWS 人工智能服务:Polly、transcriptor 和 Translate，我们就实现了一个帮助我们提高一些外语技能的工具。然而，本文中提出的概念是进一步特性的唯一核心。

基于这一核心，您可以实现其他功能，例如:

*   将原文/翻译文本和音频保存为[抽认卡](https://en.wikipedia.org/wiki/Flashcard)
*   用一套保存的抽认卡练习
*   收集关于锻炼结果的统计数据
*   基于收集的统计数据实施[间隔重复](https://en.wikipedia.org/wiki/Spaced_repetition)

诸如此类。你会给这个核心添加什么，只取决于你的要求和想象。

感谢阅读。我希望这有所帮助。如果你有任何问题，请随时回复。

# 资源

*   [演示应用库](https://github.com/Jaitl/aws-lambda-telegram-lang-demo)