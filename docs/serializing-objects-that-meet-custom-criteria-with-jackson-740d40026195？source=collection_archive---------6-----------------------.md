# 用 Jackson 序列化符合自定义标准的对象

> 原文：<https://itnext.io/serializing-objects-that-meet-custom-criteria-with-jackson-740d40026195?source=collection_archive---------6----------------------->

![](img/d1e60b70964e2779ea78953e8d28fa54.png)

潘卡杰·帕特尔在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上的照片

在项目开发期间实现的一个有用的特性是序列化满足自定义标准的对象。

例如，我们可以考虑序列化一个对象，只要它没有被隐藏或取消。在这个故事中，我们通过一个**布尔** *隐藏*属性来表示对象的状态。

本文的目的是展示序列化它们是多么容易和可能。

首先我们来介绍一下*Hidable*界面。

使用*@ JsonIgnoreProperties(" hidden ")*注释是为了防止隐藏属性在输出中被序列化。

现在，抽象类 **HidableObject** 实现了 Hidable 接口。

最后，扩展它的具体类: *User* 和 *EmailAddress。*

因此，在我们的模型中，一个用户可能有一个或多个电子邮件地址，但我们将只序列化可见的那些。

## 定制 JSON 序列化程序和模块

为此，我们需要一个定制的 JSON 序列化程序和模块。

JSON 序列化程序

注意，在上面的*序列化*方法中，我们将只序列化*隐藏*属性设置为**假**的对象。

相反，在下面的模块中，当且仅当输入 bean 的类型为 HidableObject 时，我们才实例化我们的自定义序列化程序。否则，只有默认的序列化程序被实例化，使用默认的行为。

更具体地说， *isAssignableFrom* 方法确定由 Hidable 对象表示的类或接口是否与由指定参数表示的类或接口相同，或者是其超类或超接口。

## 杰克逊转炉

此外，为了让 Jackson 知道在序列化阶段要采取的行为，有必要在配置文件中将上面的模块注册到转换器中。

因此，Jackson 能够拦截 POJO 并避免每个隐藏对象的序列化。

## 测试和结论

多亏了 JUnit，测试才得以执行。

和相对输出…

```
{
    "username": "User 1",
    "emailAddresses": [
        {"address":"[e1@mail.com](mailto:e1@mail.com)"},
        {"address":"[e3@mail.com](mailto:e3@mail.com)"}
    ]
}
```

我们可以注意到第二个电子邮件地址 hidden 没有被序列化。

最后，在这个故事中，我们学习了如何序列化满足自定义标准的对象。

你可以从 GitHub 仓库的这里下载项目[。](https://github.com/maunat/JacksonCustomSerialization)