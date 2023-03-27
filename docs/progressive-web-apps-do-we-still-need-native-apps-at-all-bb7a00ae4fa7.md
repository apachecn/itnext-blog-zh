# 渐进式网络应用:我们还需要本地应用吗？

> 原文：<https://itnext.io/progressive-web-apps-do-we-still-need-native-apps-at-all-bb7a00ae4fa7?source=collection_archive---------2----------------------->

## 渐进式网络应用对你来说是一个可行的解决方案

渐进式网络应用( **PWA** )现在是网络上的热门话题。难怪，他们承诺的用户体验不应逊于 Android 和 iOS 的原生应用。同时，应该不再需要为每个平台(尤其是 Android 和 iOS)开发单独的 app，这当然涉及更多的成本和精力。

[谷歌](https://developers.google.com/web/progressive-web-apps/)定义了 pwa 的三个主要特征:它们必须**可靠**(即它们必须即使在糟糕的网络上也能工作)**快速**和**吸引人**(从 UX 的角度来看)。对于用户来说，无论是 PWA 还是原生应用都没有区别。

![](img/a9ca8086c44eaf7381319d7da6e8bb09.png)

PWA 还应该在恶劣的网络条件下可用

PWAs 是移动网站的下一次发展。虽然移动网站能够看起来与本地网站相似，但局限性很快就显现出来:缺乏与硬件相关的功能，可用性差，互联网连接差；许多手机网站看起来像是本地网站的翻版。有了 PWAs，这应该是过去的事了。

## PWA 是如何工作的？

1.  用户访问作为 PWA 的网站(例如，Android 下的 Chrome 或 iOS 下的 Safari)。
2.  要么被访问的网站自动通知用户他可以把它放在他的主屏幕上(Android)，要么他必须手动把它添加到主屏幕上(iOS)。
3.  带有图标和标题的 PWA 现在出现在开始屏幕上。从纯粹的视觉角度来看，用户将无法辨别这不是一个原生应用程序。单击图标将打开 PWA。

## 成功故事和来自大玩家的支持

成功实施 PWA 的一个例子来自社交网络 [*Pinterest*](https://medium.com/@Pinterest_Engineering/a-one-year-pwa-retrospective-f4a2f4129e05) 。到目前为止，用户增长主要来自新的 PWA，它比 Pinterest 的旧移动网站快得多，使用起来也更愉快。 [*优步*](https://eng.uber.com/m-uber/) 也完成了向 PWA 的过渡，即使在慢速 2G 下也能工作。 [*trivago*](https://www.thinkwithgoogle.com/intl/en-154/insights-inspiration/case-studies/trivago-embrace-progressive-web-apps-as-the-future-of-mobile/) 向 PWA 的转换也被证明是成功的，它能够通过 PWA 实现的推送通知显著提高转化率。所有这些真实的例子都有一个共同点，那就是移动网站的性能和 UX 都显著提高了。

其中一个主要的支持者是微软的[](https://preview.pwabuilder.com/)*，它提供了一个有用的在线工具来快速创建 PWA。*谷歌*也是一个强有力的支持者，并提供了一个名为 [*灯塔*](https://developers.google.com/web/tools/lighthouse/) 的工具，集成到谷歌 Chrome 的开发工具中，用于优化 PWAs 的质量，检查[某些质量特征](https://developers.google.com/web/progressive-web-apps/checklist)并提出解决方案。就连*到目前为止在这方面相当勉强的苹果*，现在[从 iOS 11.3](https://medium.com/@firt/progressive-web-apps-on-ios-are-here-d00430dee3a7) 开始支持 PWAs。*

*![](img/db789138e1052ad29a873ea8d4875d76.png)*

*Lighthouse 证明 Pinterest 在满足 PWA 标准方面取得了非常好的成绩。*

*对于尚未发布任何应用的公司，PWA 提供了绕过数字销售平台，向移动网站访问者提供 PWA 的可能性。对于较小的开发团队或较小的开发预算，开发 PWA 比为每个平台开发一个解决方案更具成本效益，也更快。*

*谷歌甚至允许[向 Google Play 商店](https://css-tricks.com/how-to-get-a-progressive-web-app-into-the-google-play-store/)发布 PWA。你只需要使用 Android Studio IDE 创建一个基本的 Android 项目，并设置一个 Google Play 开发者帐户。*

## *为什么我们还不能完全没有原生应用*

*然而，用 Java (Android)或 Swift (iOS)等编程语言开发的原生应用在不久的将来不会从我们的设备中消失。许多 web 应用程序仍然没有优化，无法跟上本地应用程序的步伐。尽管一些原本只在原生应用中可用的功能现在也可以在网络应用中使用(例如推送通知或访问相机)，但一些特定的硬件相关功能(例如访问传感器)仍然不能在原生应用中使用。*

*此外，还不能直接在 Apple App Store 中发布 PWA。对于谷歌和苹果来说，尽管他们(特别是在谷歌大的情况下)支持 PWA，但这应该是一个眼中钉，因为此时 PWA 可以绕过分发平台 Play Store 和 App Store。尤其是苹果，它以严格禁止替代资源而闻名。如果未来的应用程序仅以 PWA 的形式出现，而不再出现在 Google Play 或 App Store 中，那么平台运营商将无法通过在数字商店购买它们来赚取任何钱。*

*![](img/89b8578f4608409e52b807d8997c13c6.png)*

*App Store 和 Google Play 是主要的应用分发平台*

*特别是对于面向消费者的应用程序，提供额外的本地应用程序会很有用。很多用户习惯从 Play Store 或者 App Store 安装应用。网站运营商和操作系统厂商首先要让用户明白，PWAs 可以有不同的用途。例如，当用户访问 PWA 时，iOS 目前不会自动通知用户他们可以安装 PWA，或者该网站根本就是 PWA。*

*网站运营商应注意，并非所有设备都支持 PWAs 的所有功能。Internet Explorer 不支持 PWAs 所必需的服务人员,因为他们提供了 PWAs 所必需的功能。软件不是最新的 iOS 设备根本不支持 PWAs。用户应及时更新浏览器，以便使用 PWAs 的所有功能。*

## *结论*

*从题目来回答问题:看情况。PWAs 是一种简单而有效的方法，可以通过本机应用程序功能来扩展移动网站。特别是对于几乎不需要访问硬件相关功能的应用程序，PWA 可能是值得的，因为不再需要额外的本机应用程序。但是，如果应用程序需要大量的计算能力或对系统和硬件的扩展访问，本机应用程序在这一点上仍然是最佳选择。然而，许多知名公司，如优步和 Pinterest，已经将其移动网站转变为 PWA，同时显著增加了用户数量和用户满意度。这使得 PWA 不仅仅是一个炒作的话题，而是对原生 app 的认真替代和补充。*

*查看 Appscope ，这是一个展示成功的渐进式网络应用的目录，了解更多具体的例子。*

*这篇文章首先用德语发表在 [netkin.de](https://www.netkin.de/progressive-web-apps-brauchen-wir-ueberhaupt-noch-native-apps/) 上，这是一家来自德国科隆的在线营销机构，专门从事 SEO 分析和数字策略。*