# 如何用热门词检测轻松指挥你的 app？

> 原文：<https://itnext.io/how-to-easily-command-your-app-with-hotword-detection-1001e582c46e?source=collection_archive---------0----------------------->

## 创建自己的热门词汇，类似于“好的，谷歌”，“嘿 Siri”

我在最后的教程中建议了一些实现连续语音识别的库。我们的目标是开发一款只支持语音控制的安卓应用。我们首先研究了类似于[Google Now](https://www.google.com/search/about/)&[Google Assistant](https://assistant.google.com/)[语音操作](https://developers.google.com/voice-actions/)[speech recognizer](https://developer.android.com/reference/android/speech/SpeechRecognizer.html)和 [TextToSpeech](https://developer.android.com/reference/android/speech/tts/TextToSpeech.html) 的本地方法。我们还体验到，Google Assistant - `"Ok, Google"`，不允许我们在前台保持应用程序的同时采取进一步的自定义操作。

解决方案是创建我们自己的热门词汇来触发应用程序采取进一步的行动，比如增加我们的用例中某个玩家的分数:乒乓球板应用程序。有两个特色的第三方库，看起来有希望拥有自己的热门词汇。 [PocketSphinx](https://cmusphinx.github.io/wiki/tutorialandroid/) 和 [Snowboy](https://snowboy.kitt.ai/) 。

![](img/f7e056f44bbfbc486898d55f6d3f1a8c.png)

使用热门词汇检测的乒乓球游戏的用户界面。

# PocketSphinx:

我们从下载示例 github 项目开始。它首先检测热门词汇`"oh mighty computer"`，然后检测其他热门词汇，如`"digits", "forecast", "phones"`。结果并不令人满意，我们无法继续实现这个库。

[](https://github.com/cmusphinx/pocketsphinx-android-demo) [## cmusphinx/pocket sphinx-Android-演示

### 这是一个 pocketsphinx 在 android 上的演示

github.com](https://github.com/cmusphinx/pocketsphinx-android-demo) 

# 雪男孩:

他们的网站用一个教程视频欢迎你，这个视频解释了这个库的功能，这正是我们正在寻找的。这是一个可定制的热门词汇检测引擎，由深度神经网络驱动。它是轻量级和嵌入式的。您不需要互联网连接，并且它与示例语言模型配合得很好。

> 您可以查看他们的文档，了解如何将这个库包含到您自己的应用程序中。

[](https://github.com/Kitt-AI/snowboy) [## kitt-AI/雪球

### 基于 snowboy - DNN 的热门词和唤醒词检测工具包

github.com](https://github.com/Kitt-AI/snowboy) 

## **个人型号:**

通过他们的[网站](https://snowboy.kitt.ai/dashboard)或者[热门词 API](https://snowboy.kitt.ai/api/v1/train/) ，你可以创建你的个人模型，。pmdl 文件。不好的是这种个人模式只对你的声音有效。

## **通用型号:**

如果你想拥有一个可以被任何人触发的热门词汇，你需要和尽可能多的人一起训练你的语言模型。为你的热门词汇创建一个通用模型。umdl)并不是免费的。您应该联系图书馆所有者了解价格。有两个用于测试的示例通用模型，`Alexa`和`Snowboy`，我们将在本教程中使用它们。

# 决定我们自己的声音动作实现

我们有两种方法可供选择。更简单和更准确的方法是使用两个热词，如`Alexa`和`Snowboy`，来确定谁在我们的应用程序中得分。如果我们需要更多的语音命令，这将不是一个合理的实现。在我们的例子中，我们不需要复杂的语音命令，因此这种方法非常适合。

第二种方法是只使用一个热门词汇，比如`Alexa`，然后对命令的其余部分使用语音识别。我们尝试微软 Bing API 在热门词检测后识别命令，如`Alexa, red`或`Alexa, blue`

我们还使用一个标志来保持屏幕打开，以便在游戏过程中进行连续的语音识别。

```
getWindow().addFlags(WindowManager.LayoutParams.***FLAG_KEEP_SCREEN_ON)***
```

## **仅使用带有两个热门词汇的 Snowboy 库:**

![](img/04a80720bd473120b0fd5edada779619.png)

我们首先用两个不同的模型初始化我们的热门词检测器类`SnowboyDetect`。

我们将使用 native `AudioRecord`类实现连续监听来检测热门词汇。

通过下面的代码块，我们对每个语音数据块运行热门词汇检测。该库能够识别热门词汇，即使它们由于缓冲区大小而被拆分。

**用 Snowboy 检测热门词后使用 Bing API 进行语音识别:**

![](img/af9a30ba8fe35ee6eb6d0efb085429ab.png)

对于我们的用例来说，这种方法的缺点是我们必须为区分两个静态关键字的每个调用付费。虽然使用两个热门词汇对我们来说已经足够了，但我们希望用动态命令来改进我们的实现。因此我们需要声音识别。没有免费的服务可以将语音转换成文本，但有许多付费的好服务，如[亚马逊 Lex](https://aws.amazon.com/documentation/lex/) 、[谷歌语音 API](https://cloud.google.com/speech/?gclid=Cj0KCQjw5arMBRDzARIsAAqmJeyjkv2jJGsBi4j7LsCirwy4GJsNUTkLTBQjdeaWokEyCsJSlzuRKvoaAmWzEALw_wcB&dclid=CIjsk7uZytUCFQ2rdwodUXIEsA) 、[微软必应 API](https://docs.microsoft.com/en-us/azure/cognitive-services/speech/getstarted/getstartedjavaandroid) 。我们需要一个获取音频数据并返回不同可信度文本的服务。对于我们的用例，Bing API 是众多满足我们需求的服务中最容易实现的。

这个 api 有两种不同的语音识别方法。你可以使用麦克风或发送波形文件。我们方法中最大的挑战是将 Snowboy 实现与 Bing API 相结合。因为我们已经为 Snowboy 库使用了麦克风，所以我们选择发送 wave 文件选项。我们期待像`Alexa, red`或`Alexa, blue`这样的命令。在检测到 hotword 后，我们录制下面的语音一段时间，并将其作为 wave 文件发送到 Bing API，并获得文本结果。根据结果，我们增加相关玩家的分数。

## 结论

热门词检测是实现连续语音识别目标的基础部分。目前，这是触发您的应用程序收听语音操作的最符合逻辑的方式。使用两个热门词汇的方法最适合我们。它很流畅，而且相当准确。唯一的问题是当环境中有太多噪音时识别语音命令。

通过服务结合实现本地热门词汇检测和语音识别也很有前景。但是它不如第一种方法精确。有时，检测到热门词汇后录制的音频数据可能会出现静音检测问题。这可能与 Bing API 有关。此外，决定何时开始和停止记录发送到语音识别服务的语音数据可能很棘手。

我们已经开始在办公室使用我们的乒乓球应用程序来记录我们的分数，现在我们面临成为最佳球员的挑战。

如果你喜欢你刚刚读到的，别忘了阅读关于这个项目的其他文章[](/using-voice-commands-in-an-android-app-b0be26d50958)**。觉得这是一个很酷的项目，喜欢文章？通过点击鼓掌图标来给予一些掌声👏🏽。**