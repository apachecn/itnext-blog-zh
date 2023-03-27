# 将您的应用迁移到 Android Oreo——第 2 部分广播接收器

> 原文：<https://itnext.io/migrate-your-app-to-android-oreo-part-2-broadcastreceivers-3f1f12f764fc?source=collection_archive---------2----------------------->

在上一篇文章中，我解释了如果你想在 Android Oreo 中调用“startService()”你必须做的一些改变。

在这篇文章中，我将尝试解释如果你想使用 **BroadcastReceivers** 你必须完成的一系列改变。但是，当你瞄准 Android Oreo 时，不要忘记记住所有受影响的 Android 部分:

1.  [服务—背景限制](/migrate-your-app-to-android-oreo-e0fa794b1d21)
2.  **广播接收机**
3.  [通知](/migrate-your-app-to-android-oreo-part-3-notifications-b59d7c6313f9)
4.  运行时权限
5.  警告
6.  Firebase 云消息传递(截止日期:2019 年 4 月)

![](img/7f4ac5a1f362b822c76ac4dd0af3ee98.png)

# **安卓奥利奥中的广播接收器**

本质上，Android Oreo 中 BroadcastReceivers 的问题是，所有我们在 AndroidManifest.xml 中声明并侦听隐式意图的 BroadcastReceivers 将不再接收这些广播(有一些情况是允许的)。

![](img/319b0c099d8ea2d37401b98befd02237.png)

首先，什么是隐含意图？

在 Android 中，我们有两种意图去做一个动作:

*   *隐含意图*:不要命名一个特定的组件，而是声明一个要执行的通用动作。隐含意图的一个例子可能是当我们想要打开一个 url 时，在这种情况下，我们做类似这样的事情:

```
val browserIntent = Intent(Intent.ACTION_VIEW, URL);
startActivity(browserIntent);
```

*   *明确意图*:指定哪个组件将要采取动作，用户不能什么都不做。例如，当我们发起一项活动时，我们会制作这样的东西:

```
val intent = Intent(context, ActivityB::class).
```

因此，如果我们在 AndroidManifest.xml 中声明了一个 BroadcastReceiver，并且正在侦听隐式意图(不在 actions 白名单中),那么它将不会接收广播。

这些是允许的隐含意图:

[https://developer . Android . com/guide/components/broadcast-exceptions](https://developer.android.com/guide/components/broadcast-exceptions)

因此，如果我们要声明一个 broadcastReceiver，并且这是在侦听一个自定义操作，我们必须声明将捕获该操作的类:

```
<!-- BROADCAST RECEIVER -->
<receiver android:name="your.package.BroadCastReceiver">
    <intent-filter>
        <action android:name="your.action" />
    </intent-filter>
</receiver>
```

有关广播接收器的更多信息:

[https://developer.android.com/guide/components/broadcasts](https://developer.android.com/guide/components/broadcasts)