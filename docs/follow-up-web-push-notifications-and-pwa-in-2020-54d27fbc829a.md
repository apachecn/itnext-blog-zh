# 后续:2020 年的网络推送通知和 PWA

> 原文：<https://itnext.io/follow-up-web-push-notifications-and-pwa-in-2020-54d27fbc829a?source=collection_archive---------2----------------------->

## 这是我一年前的教程“使用 Ionic 和 Angular 的渐进式网络应用中的网络推送通知”的后续

![](img/4c8e0523700f1fb18870d32d79bf971a.png)

Javier Allegue Barros 在 [Unsplash](https://unsplash.com/?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄的照片

我分享[一天一招](https://medium.com/@david.dalbusco/one-trick-a-day-d-34-469a0336a07e)直到原定的 2020 年 4 月 19 日瑞士新冠肺炎隔离期结束。离第一个里程碑还有五天。希望更好的日子就在前面。

如果你在 Twitter 上关注我，你可能已经知道我开发的一个应用程序最近被苹果和谷歌拒绝，因为它不符合他们对当前新冠肺炎疫情的限制性政策。

我写这些文章不是为了分享我对这些公司的看法，而是为了分享我一年前的教程的后续内容:[渐进式网络应用中的网络推送通知](https://medium.com/@david.dalbusco/add-web-push-notifications-to-your-ionic-pwa-358f6ec53c6f)。

事实上，被拒绝的应用程序的一个核心概念依赖于推送通知。由于它是用 [Ionic](https://ionicframework.com) 和 [Angular](https://angular.io) 开发的，我们能够发布一个渐进式的网络应用程序，但这样的功能还能得到很好的支持吗？

# 介绍

我正在写这篇文章**2020 年 4 月 14 日星期二**，这就是为什么它反映了那个特定日期的状态。如果你将来读到这篇文章并注意到改进或变化，请联系我！

今天下午，我在运行 Android v10 的 OnePlus 6 和运行 iOS 13 的 iPhone 6s 上进行了测试。

# 机器人

它就像一个魔咒，句号。我测试网络推送通知时，手机处于空闲模式，处于唤醒状态，并且应用程序处于打开状态。在所有情况下，我都收到了通知。干得好，谷歌👍。

# ios

iOS 上仍然**不支持**网页推送通知。自从我在 2019 年 2 月发布我的教程以来，状态没有改变。在[can use](https://caniuse.com/#search=notification)的帮助下，你可以注意到 iOS Safari 还没有实现通知 API。

# 设置

我在上一篇文章中展示的 [Firebase Cloud Messaging](https://firebase.google.com/docs/cloud-messaging) 设置仍然有效。当然，也许一些截图已经改变或已经实现，但想法是一样的。此外，我已经用完全相同的方式为我的应用程序设置了令牌，一切都很好。

不过，值得注意的一件有趣的事情是，来自[加利罗](https://medium.com/@galilo7g)的[评论](https://medium.com/@galilo7g/good-tutorial-just-mention-the-need-to-specify-the-firebase-version-in-the-service-worker-that-e90d3d8a2231)。根据他/她的经验，服务人员使用的 Firebase 依赖项必须设置为与`package.json`中使用的版本号完全相同的版本号。我没有这个问题，但这可能是值得记住的事情。

# 履行

除了下面的反对意见(可以改进也可以不改进)之外，在我之前的教程中显示的[实现](https://medium.com/@david.dalbusco/deeplinking-in-ionic-apps-with-branch-io-ba1a1c4ed227)仍然有效。它是我在我们的应用程序中实现的，因此我今天在我的 Android 手机上成功测试了它。

也就是说，我认为可能有一种更简单的方法，特别是如果你正在使用 [AngularFire](https://github.com/angular/angularfire) ，在渐进式 Web 应用中实现 Web 推送通知。我没有检查它，但在遵循我的教程之前，它可能值得一个快速的研究，以防你能抽出一些时间😉。

## 贬值

没什么大不了的，但是在看代码的时候，我注意到`await messaging.requestPermission();`被标记为不推荐使用。它可以更新如下:

```
if (Notification.permission !== 'denied') {
    await Notification.requestPermission();
}
```

## 总共

总之，我的增强型 Angular 服务负责注册 Web 推送通知和请求权限。

```
import {Injectable} from '@angular/core';

import {firebase} from '@firebase/app';
import '@firebase/messaging';

import {environment} from '../../../environments/environment';

@Injectable({
    providedIn: 'root'
})
export class FirebaseNotificationsPwaService {

    async init() {
        navigator.serviceWorker.ready.then((registration) => {
            if (!firebase.messaging.isSupported()) {
                return;
            }

            const messaging = firebase.messaging();

            messaging.useServiceWorker(registration);

             messaging
                 .usePublicVapidKey(environment.firebase.vapidKey);

            messaging.onMessage((payload) => {
                // If we want to display 
                // a msg when the app is in foreground
                console.log(payload);
            });

            // Handle token refresh
            messaging.onTokenRefresh(() => {
                messaging.getToken().then(
                    (refreshedToken: string) => {
                    console.log(refreshedToken);
                }).catch((err) => {
                    console.error(err);
                });
            });
        }, (err) => {
            console.error(err);
        });
    }

    async requestPermission() {
        if (!Notification) {
            return;
        }

        if (!firebase.messaging.isSupported()) {
            return;
        }

        try {
            const messaging = firebase.messaging();

            if (Notification.permission !== 'denied') {
                await Notification.requestPermission();
            }

            const token: string = await messaging.getToken();

            // User token
            console.log(token);
        } catch (err) {
            console.error(err);
        }
    }
}
```

# 摘要

希望有一天我们也能在 iOS 设备上发送网页推送通知🤞。

呆在家里，注意安全！

大卫