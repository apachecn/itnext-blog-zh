# 可通过 Swift 中的动态密钥解码

> 原文：<https://itnext.io/decodable-with-dynamic-keys-in-swift-78f4293f5654?source=collection_archive---------1----------------------->

自 Swift 4.0 以来，可解码和可编码协议是我最喜欢的功能。你可以在这里查看我第一篇关于 Swift 中的 Codable enum 的文章:[https://blog . untitled kingdom . com/Codable-enums-in-Swift-3a B3 dacf 30 ce？gi=4c150f2a90ff](https://blog.untitledkingdom.com/codable-enums-in-swift-3ab3dacf30ce?gi=4c150f2a90ff) 。在本文中，我将介绍如何以一种非常简便的方式处理内部带有动态键的 JSON 响应。

![](img/7ffb4f5509a9c83e5e9224d6c64ef2bb.png)

# 情况

在我们的应用程序中，我们需要处理用户 JSON 表示，它可以通过两种不同的方式发送:

```
{"name": "Jon","surname": "Doe","age": 42,"isAdmin": true}
```

或者

```
{"name": "Jan","surname": "Kowalski","age": 42,"isViewer": true}
```

它可以是一些遗留的 API 契约，它可以是来自第三方设备(如 BLE 设备)的一些响应，或者可能只是一些类似字符串的通信方式，无论如何，我们需要处理它，并且我们需要知道给定用户的角色。

让我们从定义用户模型开始

# JSONSerialization-way:

它更像 Obj-C 代码，我们从将 JSON 字符串对象解析到 Swift 字典(类型为[String: Any])开始，然后根据字典中的值动态提取适当的角色和所有其他属性，如姓名和年龄。当然，这是可行的，但也有一些缺点。

优点:

*   易于阅读和理解
*   易于实施

**缺点**:

*   重用性差
*   不适用于通用解决方案(编码、映射)

# 救援解码！

在这里，我们将所有的键提取到 DynamicKey 对象中，然后我们在容器中匹配所有的键，寻找“isAdmin”或“isViewer”键。如果我们找到了给定的键，我们需要检查这个键的布尔值，然后我们可以存储适当的用户角色。

对于所有其他键，我们使用传统的编码键枚举编码解决方案。

**DynamicKey** 实现应该是这样的:

**优点:**

*   通用和可重用(可解码)
*   可编码很容易被采用
*   您可以使用不同的序列化类型(JSON、XML)

**缺点:**

*   更难阅读和理解

# 结论:

在这篇文章中，我们再次看到可解码(和可编码)的协议比我们想象的更强大。有时，值得考虑实现“init(from: decoder)”实现，而不是采用映射或 JSONSerialization 路径。

这么快乐的代码！🚀