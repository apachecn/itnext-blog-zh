# 在 Android 应用程序中使用语音命令

> 原文：<https://itnext.io/using-voice-commands-in-an-android-app-b0be26d50958?source=collection_archive---------0----------------------->

## 构建一个声控乒乓球记分牌

这个想法是建立一个 Android 应用程序，它只与口头交流而不触摸屏幕。我们先来看看:[Google Now](https://www.google.com/search/about/)&[Google Assistant](https://assistant.google.com/)[语音动作](https://developers.google.com/voice-actions/)[speech recognizer](https://developer.android.com/reference/android/speech/SpeechRecognizer.html)和 [TextToSpeech](https://developer.android.com/reference/android/speech/tts/TextToSpeech.html) 决定哪些适用于我们的 app。

![](img/a902ef29a18ea1a349759621bbfe4c3a.png)

## 用例 App:乒乓球板

这个想法的用例是创建一个乒乓球板应用程序。该应用程序有两个屏幕:第一个屏幕显示可用玩家及其分数的排行榜，第二个屏幕显示当时玩游戏的两名玩家及其游戏分数。

![](img/6a67259c9ea7d472167e4afe501fa5b5.png)

排行榜

![](img/0f61746391ef0c5e159b696b87854a13.png)

游戏屏幕

您可以在下面找到源代码:

[](https://github.com/LINKIT-Group/ping-pong) [## 联络小组/乒乓球

### 乒乓球-一个带有语音识别的乒乓球记分牌应用程序

github.co](https://github.com/LINKIT-Group/ping-pong) 

## Google Now vs Google Assistant:

Google Now 是一个语音激活的个人助理，可以为你搜索网页并执行一些系统定义的任务。它类似于苹果的 Siri，但是 Google Now 是平台无关的，也可以在 iOS 设备上运行。

谷歌助手就像是 Google Now 的升级版兄弟姐妹。它每天利用机器学习和人工智能来了解你，你可以和谷歌助手进行对话。如今，它正一天天地取代谷歌的位置。

激活谷歌助手有两种方式:长按 home 键或者说“OK，Google”。从 Android 6.0 棉花糖开始，系统可能会在我们的应用程序(称为“源应用程序”)顶部为助手打开一个叠加窗口。这让我们有可能从我们的应用程序中收集一些信息，并使用这些信息在我们的应用程序之外采取行动。然而，这并不能帮助我们实现我们想要的。我们想使用语音命令在我们的应用程序中执行动作。(例如*“OK，Google，点为 Efe”*)。您可以在这里找到关于使用助手的更多信息[。](https://developer.android.com/training/articles/assistant.html)

## 语音操作:

谷歌定义了两种类型的语音动作。[系统语音动作](https://developers.google.com/voice-actions/system/)和[自定义语音动作](https://developers.google.com/voice-actions/custom-actions)。起初，听起来自定义语音操作可能会帮助我们，不幸的是，我们看到谷歌不再接受自定义语音操作的请求。另一方面，系统语音操作——如“搜索”、“设置闹钟”、“发起电话”、“拍照”、“打开网址”——对我们的用例应用程序来说不够有趣。

语音动作在这里也分为[系统提供的语音动作和 App 提供的语音动作。在您的清单文件中为您想要启动的活动定义了一个**标签**属性之后，您可以通过说*“OK Google，Start MyApp”*来启动您的应用程序。很高兴知道，但应用程序提供的语音操作不是我们所期待的。同一个](https://developer.android.com/training/wearables/apps/voice.html)[文档页面](https://developer.android.com/training/wearables/apps/voice.html#FreeFormSpeech)也提供了关于自由形式语音输入的信息，这看起来很有希望。

## 自由形式的语音输入:

从 Google [文档](https://developer.android.com/training/wearables/apps/voice.html#FreeFormSpeech)中，我们看到从用户那里获得自由形式语音的一种常见方式是使用`ACTION_RECOGNIZE_SPEECH`动作调用`startActivityForResult`并在`onActivityResult`中接收结果。这种方法显示一个默认视图来获取用户语音输入。但是如果它不能理解用户所说的话，你必须再次触摸屏幕。这与我们只用语言交流的目标相冲突。因此我们决定通过使用`SpeechRecognizer`和`TextToSpeech`类来实现一个类似于 Google Assistant 的通信。

下面是我们将要采取的步骤，以便通过获取用户语音输入来选择两个玩家。

1-问用户“谁是第一个玩家？”

文本到语音

2-在问完我们的问题后，聆听用户输入，选择第一个玩家

3-问用户“谁是第二个玩家？”(与步骤 1 相同)

4-听取用户输入以选择第二个玩家(与步骤 2 相同)

## 最大的挑战:连续语音识别

在选择两个玩家后，应用程序决定谁将开始游戏并说话(例如*“埃菲社将开始游戏”*)。挑战从这一点开始。我们需要不断地听，才能听到像*“埃菲社得分”这样的短语。*然而 Google [明确提到](https://developer.android.com/reference/android/speech/SpeechRecognizer.html)这不是`SpeechRecognizer` api 的本意。

> 这个 API 的实现可能会将音频流式传输到远程服务器来执行语音识别。因此，此 API 不打算用于连续识别，这将消耗大量的电池和带宽。

在我们使用的类似方法的实现过程中，已经报告了几个关于`SpeechRecognizer` api 的错误。此外，重新启动语音识别周期之间的时间间隔阻碍了连续语音识别体验的流动。我们体验到与系统对抗无助于实现我们的目标，并停止了进一步的尝试。

为了完成我们的应用程序，我们决定使用触摸手势(而不是语音识别)来开始游戏，并为玩家增加积分。然而，我们一直通过语音识别来选择玩家，以展示我们在本教程中使用的方法。

## 结论

如果您想继续使用这种方法，请尝试通过直接从设备的麦克风捕获音频并将其传输到语音识别服务来构建自己的实现。也有一些图书馆看起来有希望实现我们想要的。

*   [必应 API](https://docs.microsoft.com/en-us/azure/cognitive-services/speech/getstarted/getstartedjavaandroid)
*   [Java 语音 API](http://www.oracle.com/technetwork/java/jsapifaq-135248.html#what)
*   [CMUSphinx](https://cmusphinx.github.io/wiki/tutorialandroid/)