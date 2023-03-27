# Kotlin 集合中未处理的异常

> 原文：<https://itnext.io/unhandled-exceptions-in-kotlin-collections-d3d9459f15de?source=collection_archive---------3----------------------->

## 你为什么不早点警告我？

K

不幸的是，**这些方法中的一些会抛出异常**。如果您的程序还没有处理这些异常，这是非常不可取的，会导致您的程序在运行时崩溃。恶心。

我个人遇到过运行时引发的类似`java.lang.**IndexOutOfBoundsException**`或`java.util.**NoSuchElementException**` 的异常，那是因为使用了`List::first`。(请注意,“不安全”在其他语言中有不同的含义。)

正是因为这个原因，我们应该**更喜欢返回集合中元素的可空类型**的方法。以下是可以抛出的方法列表(左)及其首选的更安全的对应方法(右):

*   `[**component1**](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/component1.html)`、`[**component2**](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/component2.html)`、…——(无)
*   `[**elementAt**](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/element-at.html)` — `[elementAtOrNull](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/element-at-or-null.html)`，`[elementAtOrElse](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/element-at-or-else.html)`
*   `[**first**](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/first.html)`——`[firstOrNull](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/first-or-null.html)`
*   `**get**`——`[getOrElse](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/get-or-else.html)`，`[getOrNull](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/get-or-null.html)`。注意这只适用于[列表界面](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-list/)。对于[映射接口](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-map/),`[get](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/get.html)`函数已经返回了一个可空类型🤔。
*   `[...]`——(同上)
*   `[**last**](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/last.html)` — `[lastOrNull](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/last-or-null.html)`
*   `[**random**](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/random.html)`——`[randomOrNull](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/random-or-null.html)`
*   `[**removeAt**](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-mutable-list/remove-at.html#kotlin.collections.MutableList$removeAt(kotlin.Int))`——(无)
*   `[**removeFirst**](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/remove-first.html)`——`[removeFirstOrNull](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/remove-first-or-null.html)`
*   `[**removeLast**](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/remove-last.html)` — `[removeLastOrNull](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/remove-last-or-null.html)`
*   `[**single**](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/single.html)`——`[singleOrNull](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/single-or-null.html)`

旁注——集合方法不是唯一可以无声地抛出异常的方法。还有其他像`String::toInt`这样的，但那是以后的事了。

那么为什么编译器没有向我发出这些方法会抛出的警告，而我却没有捕捉到它们呢？为什么`Map::get`方法返回一个可空的而不是`List::get`？

TypeScript 中的情况类似，运行时的 transpiled JavaScript 仍然会抛出`Cannot read properties of undefined 'foo'`，因为编译器没有警告一些早期的`undefined`返回值。另一方面，在 Rust 中，块上的新 kid 访问 vector 的第一个和最后一个元素，返回一个可空值: [Vec::first](https://doc.rust-lang.org/std/vec/struct.Vec.html#method.first) 和 [Vec::last](https://doc.rust-lang.org/std/vec/struct.Vec.html#method.last) 。麻烦，但是安全。

我希望团队在未来的版本中删除上面列出的方法(我在最近的调查中提到了这一点)。但是现在，**更喜欢使用** `**xxxOrNull**` **的变体**。当他们在代码中写`.first()`来访问列表的第一个元素时，不要相信你自己或你的同事😉。

在撰写本文时，最新的 Kotlin 版本是 1.7.10。

我发表关于人工智能、机器学习、编程语言、Web 框架、生产力和学习的文章。

*如果你喜欢阅读更多关于 web 框架的内容，你可以通过我的推荐链接* [*订阅*](https://remykarem.medium.com/subscribe) *随时接收更新或者* [*注册*](https://remykarem.medium.com/membership) *！请注意，您的会员费的一部分将作为介绍费分摊给我。*