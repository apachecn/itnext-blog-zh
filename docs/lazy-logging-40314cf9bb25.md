# 惰性日志记录

> 原文：<https://itnext.io/lazy-logging-40314cf9bb25?source=collection_archive---------3----------------------->

![](img/997b0efb823cdb4ddfb9e5f8033c1ccf.png)

图片由[凯文·菲利普斯](https://pixabay.com/users/27707-27707/?utm_source=link-attribution&utm_medium=referral&utm_campaign=image&utm_content=951778)来自 [Pixabay](https://pixabay.com/?utm_source=link-attribution&utm_medium=referral&utm_campaign=image&utm_content=951778)

创建日志语句时要小心。即使日志实际上不记录您的输入，也不意味着它不评估 log 语句。

例如:

```
***log***.debug(**"I found {} and {}"**, getone(), gettwo());
```

似乎，很好。字符串中包含两个参数的调试日志。

一切都好吗？

不完全是。

您会看到日志中的两个输入:

*   getone()
*   gettwo()

这两种方法总是被评估。也就是说，即使不使用 log 语句，我们仍然会计算 log 调用的所有输入。

如果这些调用是昂贵的，我们可能会白白浪费大量的 CPU 周期。

典型的解决方法如下所示:

```
**if** (***log***.isDebugEnabled()) {
    ***log***.debug(**"I found {} and {}"**, getone(), gettwo());
}
```

但这相当难看。难道 *log.debug* 的目的不就是为了确保我们的代码只有在启用调试的情况下才会被记录吗？

我们想要的是一个单行调试器语句，并且只有在启用了记录器的情况下才评估输入参数。

为了解决类似的情况，我们将使用 Java 8 的[供应商模式](https://docs.oracle.com/javase/8/docs/api/java/util/function/Supplier.html)。供应商只会在需要时计算价值。

不幸的是，大多数日志框架不支持供应商模式。

至少是直接的。

实际上，记录器并不期望参数是一个字符串。它实际上接受任何对象。记录器将调用对象上的 *toString* 方法，将其翻译成一个字符串。这里的诀窍是，如果需要，记录器将只**调用**toString，即如果启用了日志记录。

所以我们只需要用一个对象包装我们的供应商，该对象在调用 *toString* 时调用供应商。

```
**import** java.util.function.Supplier;

**public class** LazyString {
    **private final** Supplier<?> **stringSupplier**;

    **public static** LazyString lazy(Supplier<?> stringSupplier) {
        **return new** LazyString(stringSupplier);
    }

    **public** LazyString(**final** Supplier<?> stringSupplier) {
        **this**.**stringSupplier** = stringSupplier;
    }

    @Override
    **public** String toString() {
        **return** String.*valueOf*(**stringSupplier**.get());
    }
}
```

我们现在可以更新日志语句，如下所示:

```
**import static** LazyString.*lazy*;***log***.debug(**"I found {} and {}"**, *lazy*(**this**::getone), *lazy*(**this**::gettwo));
```

现在，理想情况下，日志框架将直接支持供应商。但在他们这样做之前，这是一个可以接受的变通办法。

如需完整代码示例，请查看我的 github repo:

[](https://github.com/efenglu/lazyLogger) [## 埃丰卢/拉兹洛格

### 懒惰地评估日志语句。在 GitHub 上创建一个帐户，为 efenglu/lazyLogger 的开发做出贡献。

github.com](https://github.com/efenglu/lazyLogger)