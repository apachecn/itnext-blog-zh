# 设计 Kotlin 内存安全模式

> 原文：<https://itnext.io/designing-a-kotlin-memory-safe-mode-c76c06317c3e?source=collection_archive---------2----------------------->

![](img/896e0c30ba33d1b43de921fc309f400a.png)

瑞安·弗兰科拍摄的照片

在我的上一篇文章中，我分享了我对 kot Lin/本机内存模型在运行时如何实施的[挫败感](https://medium.com/@timelzayus/why-the-kotlin-native-memory-model-cannot-hold-ae1631d80cf6)，并提出了语言解决方案的开端，在类型系统中使用 **const** 。

设计一门语言，即使在理论上，也是一个非常有趣的挑战，所以让我们戴上我们的假语言设计师的帽子，看看我们如何为 Kotlin/Native memory 模型提供一个解决方案。

*警告:这是一篇长文，描述了一个不存在的语言特性！不过，我希望如此。这篇文章描述了这个提议背后的原因。*

我将与 rust 语言做一些比较，特别是 Rust 如何处理[所有权](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html)、[生存期](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html)和[并发性](https://doc.rust-lang.org/book/ch16-00-concurrency.html)。您不必阅读这些链接来理解本文，但是如果您想更深入地了解这些概念，我强烈建议您这样做。Rust 是一种令人惊叹的语言，拥有令人惊叹的文档。

当我们想要扩展一种现有的语言时，我们首先必须理解管理这种语言的规则框架。

Rust 有一个非常严格的规则框架:如果没有明确允许，那么就是禁止。默认情况下，对象是不可变的，除非被指定为可变的，除非明确处理所有权，否则不能传递指针，不能有多个可变引用，等等。

另一方面，Java 有一个非常“宽松”的规则框架。如果没有明确禁止，那么它就是允许的。一切都是可变的，每个类都可以扩展，除非显式地是 final，泛型参数是可选的，等等。

Kotlin 介于两者之间:它的设计者一直说它首先是实用主义的，这导致了一些看起来矛盾的决定:

*   默认情况下，类是最终的，但重写只有在显式的情况下才是最终的。
*   对象是可变的，但是集合接口首先是不可变的，然后才是可变的(可变列表扩展列表)。
*   可空性必须显式处理，但所有异常都是隐式的。

因此，我认为，在考虑扩展 Kotlin 语言时，我们必须记住以下“实用”的规则框架:

*   约束不能“妨碍”程序员。如果约束被“变通”的次数多于它被使用的次数，那么它就不应该被强制执行。
*   代码必须简洁明了，这意味着默认行为应该是更频繁的行为，但是行为的任何变化都必须明确定义并且可读。

这两个规则是 Kotlin 执行它最臭名昭著的规则的原因:“默认情况下，一切都是公开的”。

最后，当然:

*   语言的新增加必须是向后兼容的，不能突然改变现有 Kotlin 代码的语义。

我们开始吧。

首先，从著名的 kot Lin/本机内存模型规则“数据要么是可变的，要么是共享的”，我们可以推断出描述完全相同概念的完全相反的规则:“数据要么是线程本地的，要么是不可变的”。

Kotlin/Native 提出了一个冻结 API，它使得一个对象(及其所有子图)不可变。我在以前的文章中已经指出，我认为在运行时实施不变性是一个非常糟糕的想法。那么我们来介绍一下 Kotlin 中的 *const* 关键字。

C++对此有一个有趣的方法:类型注释。在科特林，它看起来像这样:

```
fun description(foo: **const** Foo): String {}
```

*描述*函数声明它不会改变 foo 对象，只会从中读取值。为了加强这一点，编译器将只允许函数访问*常量值获取器*和*常量函数*，在 Kotlin 中类似于:

```
class Foo {
    **const** val answer = 42
    **const** fun getName(): String = *TODO*()
}
```

在这里， *getName* 方法本质上是纯的，编译器将确保它不会使对象变异(没有副作用)。

这种 const 类型注释在 Kotlin 中是一个非常糟糕的想法，原因有很多:

*   这将意味着检查整个标准库代码库，将 *const* 关键字添加到每个纯函数，如 *map* 、 *filter* 等。
*   到处都会有*常数*。绝大多数函数和方法不会改变数据。果然，C++中的 const 关键字无处不在，削弱了可读性。

那么为什么不相反呢？假设一切都是不可变的，除非明确定义为可变的。Rust 用 *mut* 关键字来做这件事。不幸的是，虽然这对喜欢约束的程序员(我很自豪地加入了这个团体)来说很有吸引力，但我们不会改变整个现有 Kotlin 代码库的语义。这将要求每个人仔细检查所有被创建的 Kotlin 代码，以便在任何需要的地方添加 *mut* 关键字。

不会发生的。

不过，这是可能发生的事情:一个*常量类注释*。大概是这样的:

```
**const** class User(val firstName: String, val lastName: String) {
    fun fullName() = "$firstName $lastName"
}
```

*   在编译时，编译器确保 *const* 类不包含 *var* 属性，只包含原语或 *const* 值。
*   在运行时，在 Kotlin/Native 中，默认情况下，所有 const 类的具体化及其子树都是冻结的。

这消除了对冻结 API 的需要。对象要么是常量，要么是冻结的，要么不是。有些人会认为这消除了创建和配置一个对象并在以后冻结它的可能性。我相信这是一个巨大的代码味道(你无法知道对象何时被冻结，并且禁止改变它)，并且会将你重定向到*构建器模式*。还有:大多数垃圾收集者已经成为处理短命对象的专家。

是的，我知道这仍然需要一些标准库的改变:标准类如*字符串*需要用*常量*进行注释。

下一个问题是关于多态性。我们能扩展常量类吗？是否允许*常量接口*？如果是，那么它们的所有实现都需要是 *const* 吗？
这些是非常相关的问题，但与本文的范围无关。本演示不需要回答这些问题。
还有，我不知道；)

附带好处: *const 数据类*可以在初始化时生成并缓存它们的 *toString()* 和 *hashCode()* 值，这将极大地提高使用数据类作为映射键的性能。我的上一个基准测试表明，缓存哈希代码将地图查找速度提高了 3 倍。

让我们回到我们最初的规则:“数据要么是线程本地的，要么是不可变的”。这意味着编译器不应该允许非常数的全局值。这是现有 Kotlin 代码失效的部分。
希望 Kotlin/Native 还处于早期阶段，所以我相信引入一个编译器*安全模式*是可以接受的，在使用 Kotlin/JVM 或 Kotlin/JS 时可以启用也可以不启用，但在任何包含 Kotlin/Native 目标的多平台项目中都会强制启用。

这种“安全模式”将简单地**禁止可变共享全局状态**(但是允许线程本地根变量):

```
class Foo { */*...*/* }
const class Bar { */*...*/* }

val *foo* = Foo() *//* ***Forbidden****, Foo is not const* var *bar* = Bar() *//* ***Forbidden****, var are not allowed* val *global* = Bar() *//* ***Allowed***
@ThreadLocal
var local = Foo() // ***Allowed*** *(because thread-local)* 
fun main() { *TODO*() }
```

> “不要靠分享记忆来交流，要靠交流来分享记忆”。
> —Go 语言设计者。

这已经成为许多现代编程语言和程序员的目标。在某些情况下，线程需要在它们之间传递信息，可以通过改变共享数据(这是不好的，这是全部要点)或者通过从一个线程向另一个线程发送数据来实现。

Rust 有*所有权* & *生存期*的概念，它确保只有一个变量拥有它所包含的数据，并且当它失去对其值的所有权时，变量的生存期就结束了。这使得在线程之间传递可变对象变得可能和安全。这些观念在科特林语中是说不出来的，太晚了。

然而，我们已经引入了常量类的概念。Const classes 对象是直接冻结的，所以它们可以在线程之间共享。对于处理线程消息传递的函数来说，我们需要的只是声明它只接受常数值参数的方法。
泛型可以帮助我们:

```
**const** class ConstChannel<T : **const** Any> { */*...*/* }

fun <**const** T : Any> newConstChannel(): ConstChannel<T> = *TODO*()
```

*常量通道*是一个*常量类*，所以它可以安全地在线程间共享。它允许发送和接收类型为 *T* 的数据，这些数据本身必须为 *const* 。
如何使 *ConstChannel* 成为 *const 类*，同时处理线程间消息传递内部机制(这肯定需要可变性)？继续读！

在 Kotlin 中，lambda 函数可以捕获外部范围的值和变量。本质上，它们实际上是闭包，不应该被称为 lambdas。

如果我们正在使用*执行器模式*(或者 Kotlin/Native worker API)并且想要调度一个 lambda 在执行器上运行，该怎么办？代码很有可能在另一个线程上执行，这意味着这个 lambda 一定不能捕获非常数值。

下面是如何声明 executor 类:

```
**const** class ConstExecutor {
    fun schedule(block: **const** () -> Unit) = TODO()
}
```

块是一个*常量λ*，这意味着它只能捕获常量值。下面的例子说明了它的用法:

```
**const** data class GameState(val level: Int)

val *executor* = ConstExecutor()

fun save(level: Int) {
    val state = GameState(level)
    *executor*.schedule **{** writeToFile(state) **}** }
```

因为 state 的类型是 *GameState* ，并且因为 *GameState* 是 *const 类*，所以 lambda 可以捕获它。
相比之下，这段代码将**不会**编译:

```
fun save(level: Int) {
    var saveLevel = level
    if (level > 10)
        saveLevel -= 1 *// Games become hard and frustrating!
    executor*.schedule **{** writeToFile(GameState(saveLevel))
        *//                    ^^^^^^^^^
        // Cannot capture a mutable value!* **}** }
```

如何使 *ConstExecutor* 成为 *const 类*，并且处理线程间调度内部(这肯定需要可变性)？继续读！

我们需要一个搞砸的方法。或者更具体地说，我们需要一种绕过这些限制的方法。类似于！！运算符，我们可以对编译器说“你不能保证这个值为空，但是我可以，相信我；如果我错了，我接受运行时崩溃的可能性”。

Kotlin/Native 已经用 *Atomic** 类做到了这一点，这些类允许在冻结时进行变异。

我们已经定义了一个 *const lambda* 的可能性，它只能捕获常数值，但是没有什么可以阻止这样的 lambda 返回非常数值。考虑以下定义:

```
**const** class Attachable<T>(creator: **const** () -> T) {
    fun <R : **const** Any?> attach(block: **const** (T) -> R): R
}
```

我们来分析一下。

*   *可附加的*是一个*常量类*，所以它可以在线程间共享。
*   *创建者*构造函数参数是一个*常量 lambda，*，它确保返回一个捕获的常量值(共享安全)或者一个没有其他引用的新的可变数据(因为 lambda 不能捕获可变数据)，这些数据可以在线程之间安全地分离/重新附加。
*   attach 函数执行*块 const lambda* ，在 lambda 执行期间将可变数据附加到当前线程。
    因为它是一个带有*常量返回*的*常量λ*，所以它不能在λ之外转义或“泄漏”引用。
*   如果多个线程同时连接，那么在 Kotlin/native 中运行时会**崩溃，而在 Kotlin/JVM 中不会**(但是可能会出现竞争情况)。

那么，为什么为什么？为什么要为设计一个漂亮的安全系统而头疼，只为提供棒球棒来摧毁它？我们希望确保在运行时没有崩溃，我们希望确保无论目标是什么都有相似的语义，我们在 3 行代码中摧毁了这两个梦。

我们已经把所有我们想避免的放在一个类中。本质上，我们已经将**的不安全性**限制在*的可附加性*中。这就引出了下面一点:*如果你是应用程序开发人员，就不要使用 Attachable*。*可附加的*类只为库开发者存在，为应用开发者提供更高级别的&更安全的工具，如*const channel*&*const executor*。

Rust 与[同步&发送特征](https://doc.rust-lang.org/book/ch16-04-extensible-concurrency-sync-and-send.html)完全相同。只有当你设计一个更高级的工具来正确处理安全问题时，才使用它。请记住，当使用它时，你是在不安全的领土，这里是龙。

有了这个*可附加的*类，创建一个在 Kotlin/JVM 中使用 *ReentrantLock* 和在 Kotlin/Native 中使用 *pthread_mutex* 的多平台互斥体就变得很简单，这导致了死锁的可能性。

互斥并不是一件坏事，它们通常比线程间的消息传递要快得多，并且是低层次、高性能的东西所需要的，例如，Kotlin/Native embedded 开发者可能需要编写的东西。大多数死锁都很容易调试。只需在调试器中暂停应用程序，并查看哪些线程正在等待哪些互斥体。另一方面，竞争条件很难发现和正确调试，因为当它们发生时你不能暂停。你不能可靠地检测它们。这就是为什么我对一个允许死锁的编程规则框架完全满意，但却不可能有竞争条件。

我真诚地希望 Kotlin 语言& kot Lin/Native 编译器能够发展成一个更安全、更好的规则框架，它能够:

*   鼓励安全模式和实践。
*   在编译时强制约束。
*   如果需要，允许不安全的代码。

所有这些 *const* 的东西不会进入 Kotlin 语言(我既不在 Jetbrains 工作，也不在 Kotlin 编译器上工作)，但我真的希望这可以成为“修复”当前情况的一个例子。
科特林多平台&科特林/本土故事都可以改进，我们作为一个社区，可能会成为引发讨论、发起运动、找到解决方案的火花。