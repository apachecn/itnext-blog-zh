# 如何在 SmartTV (WebOS 和 Tizen)上检查网络连接

> 原文：<https://itnext.io/how-to-check-network-connection-on-smarttv-webos-and-tizen-75256c67584b?source=collection_archive---------5----------------------->

![](img/529c79a6c6ade88f6f03c8fd75b965f7.png)

照片由 [Unsplash](https://unsplash.com/?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上的[延斯·克罗伊特](https://unsplash.com/@jenskreuter?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)拍摄

今天我想告诉你如何在 webOS 和 Tizen 平台的 SmartTV 上实现无连接功能。这听起来很容易，你可以问我，你能告诉我们新的，但相信我，我可以:)

这篇文章是关于如何检测网络连接。

目前，我知道 3 种检查网络连接的方法，每种方法各有利弊，让我们逐一介绍。

## 1)在浏览器中检查网络状态

浏览器具有用于订阅网络状态变化的监听器。我从这个实现开始。

当你在 PC/laptop 浏览器中测试这个特性时，它工作得很好，但是过了一段时间后，我得到通知，它不工作了。为什么？

在电视上，这种实现只有在您从电视上拔下电缆时才有效，但在现实生活中，我们在另一种情况下会出现网络连接错误。测试案例是从路由器上拔下 LAN 电缆，浏览器绝对检测不到这种情况，它说浏览器在线。

转到下一个实现，我们可以检测到这种情况。

## 2)使用原生 SmartTV SDK 检查网络状态

WebOS 和 Tizen 支持使用原生 SDK 检查网络连接。

对于 WebOS，您可以使用 [**连接管理器**](https://webostv.developer.lge.com/api/webos-service-api/connection-manager/) 并使用一次性检查或订阅更改。

对于 Tizen，您可以使用 [**网络 API**](https://developer.samsung.com/smarttv/develop/api-references/samsung-product-api-references/network-api.html) ，也可以使用一次性检查或订阅更改。

使用 SDK 的预期结果是 smartTV 应该了解网络连接何时存在或不存在。

现实看起来是另一回事。

## **蒂泽:**

当您从电视上拔下 LAN 电缆时，它会立即触发网络状态变化，作为浏览器检查。

当您从路由器上拔下 LAN 电缆时，30-40 秒内没有任何反应，只有在这段时间之后，Tizen 才会检测到没有网络。这不仅仅是没有，我们可以利用这一点。

## **网络操作系统:**

从电视上拔下 LAN 电缆会立即触发网络状态变化。

从路由器上拔下 LAN 电缆不会在 30 秒、1 分钟和 1 小时后触发网络状态变化。对我来说是个惊喜。

在那一刻，我开始思考如何在所有平台上实现这个功能，并且有一点延迟。

## 3)通过定期轮询 BE 或第三方端点来检查网络状态

我只看到了一个如何达到目标的解决方案，那就是定期轮询。对于这个 ping，我们可以在后端使用一些健康检查端点或第三方端点(例如 Google)。

如果我们有一个[200，300]之间的响应状态，我们可以说我们有一个连接。

如果我们有一些错误，我们可以说是出了问题，没有网络。

这种实现在 WebOS 和 Tizen 上在所有情况下都能正确工作，延迟为 2 秒。

您可以添加一个计时器来检测是否有问题，并在一段时间后手动中止请求。

这里不能用 [AbortController](https://developer.mozilla.org/en-US/docs/Web/API/AbortController) 和 fetch，因为 Tizen 4 上的 Chromium 不知道这样的接口，这就是我用 XMLHttpRequest 的原因。

在这种情况下，当请求发送在 500 毫秒内没有任何响应时，我在 10-40 分钟后有假阳性结果，电视处理请求的速度比 PC 低，你应该理解这一点。

此外，当服务器当前不可用或负载较高时，您可能会得到一个误报结果，并且您会显示一个无连接屏幕，但在重试后它是可用的。不太人性化。这个问题的一个可能的解决方案是实现一个解决方案框，您可以在其中检查大多数肯定的结果，但是它增加了丢失连接和显示屏幕之间的延迟时间。

## 结论

简单的功能并不总是如此简单，尤其是在使用 SmartTV 时。使用两种方法，SDK 和定期轮询可以覆盖所有情况，并且在丢失连接和 it 检测之间具有低延迟。