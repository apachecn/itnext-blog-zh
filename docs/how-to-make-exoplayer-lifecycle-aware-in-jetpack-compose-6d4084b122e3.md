# 如何在 Jetpack Compose 中实现 ExoPlayer 生命周期感知

> 原文：<https://itnext.io/how-to-make-exoplayer-lifecycle-aware-in-jetpack-compose-6d4084b122e3?source=collection_archive---------0----------------------->

![](img/61b354bc44a45cb039d20064a9f4c761.png)

托马斯·威廉在 Unsplash 的照片

在我的上一篇文章[在 Android Jetpack Compose 上实现视频播放](/implementing-video-playback-on-android-jetpack-compose-f73b437560ea)中，我解释了如何在 Android 上的 Jetpack Compose 中使用 ExoPlayer 播放视频。

[](/implementing-video-playback-on-android-jetpack-compose-f73b437560ea) [## 在 Android Jetpack Compose 上实现视频播放

### 编辑描述

itnext.io](/implementing-video-playback-on-android-jetpack-compose-f73b437560ea) 

一位读者观察到以下行为:当把示例应用程序放到背景中时，它仍在播放。他问了一个问题，如何让 ExoPlayer 知道生命周期，以便在应用程序暂停时视频停止。

让我们回顾一下上一篇文章中的代码，它是一个在`AndroidView`中保存`ExoPlayer`的`@Composable`:

```
@Composable
fun VideoView(videoUri: String) {
    val context = LocalContext.current

    val exoPlayer = ExoPlayer.Builder(LocalContext.current)
        .build()
        .also { exoPlayer ->
            val mediaItem = MediaItem.Builder()
                .setUri(videoUri)
                .build()
            exoPlayer.setMediaItem(mediaItem)
            exoPlayer.prepare()
        }

    DisposableEffect(
        AndroidView(factory = {
            StyledPlayerView(context).apply {
                player = exoPlayer
            }
        })
    ) {
        onDispose { exoPlayer.release() }
    }
}
```

我们需要的是`LocalLifeCycleOwner`的当前实例，它“告诉”我们正在发生的生命周期事件。这是完整的解决方案。之后我会一步步解释细节。

```
@Composable
fun VideoView(videoUri: String) {
    val context = LocalContext.current

    val exoPlayer = ExoPlayer.Builder(LocalContext.current)
        .build()
        .also { exoPlayer ->
            val mediaItem = MediaItem.Builder()
                .setUri(videoUri)
                .build()
            exoPlayer.setMediaItem(mediaItem)
            exoPlayer.prepare()
        }

    val lifecycleOwner = rememberUpdatedState(LocalLifecycleOwner.current)

    DisposableEffect(
        AndroidView(factory = {
            StyledPlayerView(context).apply {
                player = exoPlayer
            }
        })
    ) {
        val observer = LifecycleEventObserver { owner, event ->
            when (event) {
                Lifecycle.Event.ON_PAUSE -> {
                    exoPlayer.pause()
                }
                Lifecycle.Event.ON_RESUME -> {
                    exoPlayer.play()
                }
            }
        }
        val lifecycle = lifecycleOwner.value.lifecycle
        lifecycle.addObserver(observer)

        onDispose {
            exoPlayer.release()
            lifecycle.removeObserver(observer)
        }
    }
}
```

我来给你解释一下。

首先我们得到了`LifecycleOwner`的当前实例

```
val lifecycleOwner = rememberUpdatedState(LocalLifecycleOwner.current)
```

在`DisposableEffect`的范围内，我们首先创建一个`LifecycleEventObserver`。

```
val observer = LifecycleEventObserver { owner, event ->
    when (event) {
        Lifecycle.Event.ON_PAUSE -> {
            exoPlayer.pause()
        }
        Lifecycle.Event.ON_RESUME -> {
            exoPlayer.play()
        }
    }
}
```

当应用暂停时，当它进入后台时，播放器也暂停。当应用程序恢复时，回到前台，播放器继续播放视频。

然后，我们将这个观察者添加到生命周期实例中。

```
val lifecycle = lifecycleOwner.value.lifecycle
lifecycle.addObserver(observer)
```

我们需要再添加一行来进行适当的清理。当应用程序被关闭时，观察者需要从生命周期中移除

```
lifecycle.removeObserver(observer)
```

差不多就是这样！希望这将有助于你的项目！

感谢您的阅读！

*   如果你喜欢这个，请[跟我上 Medium](https://twissmueller.medium.com/)
*   给我买杯咖啡让我继续前进
*   支持我和其他媒体作者[在这里注册](https://twissmueller.medium.com/membership)

[](https://twissmueller.medium.com/membership) [## 通过我的推荐链接加入媒体

### 阅读托拜厄斯·维斯缪勒(Tobias Wissmueller)的每一个故事(以及媒体上成千上万的其他作家)。您的会员费直接…

twissmueller.medium.com](https://twissmueller.medium.com/membership) 

## 资源

*   [**Jetpack compose——当应用程序返回前台时我如何刷新屏幕**](https://stackoverflow.com/a/66807899/1065468)