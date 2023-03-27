# 避免在 Kotlin 中输入字符串

> 原文：<https://itnext.io/avoid-stringly-typed-in-kotlin-e802c4d0607c?source=collection_archive---------4----------------------->

![](img/8d65918a13d24a8a6270b5734d706188.png)

几年前，我在科特林开发了一个基于卡蒙达·BPMN 的应用程序来帮助我管理我的会议提交工作流程。它在 Trello 中跟踪我的提交，并在谷歌日历和谷歌表单中同步它们。Google 日历提供了一个 REST API。随着 REST APIs 的发展，到处都充斥着`String`。以下是代码的摘录:

```
fun execute(color: String, availability: String) {
    findCalendarEntry(client, google, execution.conference)?.let {
        it.colorId = color                                      // 1
        it.transparency = availability                          // 2
        client.events()
              .update(google.calendarId, it.id, it).execute()
    }
}
```

1.  设定事件的颜色。有效值为“0”、“1”、…到“11”
2.  设置事件的可用性。有效值为`"transparent"`和`"opaque"`

然而，我的经验告诉我应该支持强类型。我也想避免错别字。我想在这篇文章中列出一些使用`String`的替代方法。

# 常数

书中最古老的技巧，在大多数语言中都可以使用，就是定义常量。在 Java 5 之前，开发人员经常使用这种替代方法*,因为这是唯一可用的方法。它看起来像这样:*

```
*const val Default = "0"
const val Blue = "1"
const val Green = "2"
const val Free = "transparent"
const val Busy = "opaque"*
```

*我们现在可以相应地调用该函数:*

```
*execute(Blue, Busy)*
```

*常量有助于纠正错别字。另一方面，它们不能强制执行强类型:*

```
*execute(Blue, Red)        // 1
execute(Free, Red)        // 2*
```

1.  *传两个颜色，但是编译器没问题*
2.  *颠倒论点；编译器还是好的*

# *键入别名*

*类型别名背后的想法是将现有类型的名称改为更有意义的名称。*

```
*typealias Color = String
typealias Availability = String*
```

*这样，我们可以改变函数的签名:*

```
*fun execute(color: Color, availability: Availability) {
    // ...
}*
```

*不幸的是，类型别名只是装饰性的。不管别名是什么，一辆`String`仍然是一辆`String`。我们仍然可以编写不正确的代码:*

```
*execute(Blue, Red)       // 1
execute(Free, Red)       // 1*
```

1.  *什么都没有改善*

# *列举*

*无论是在 Java 还是 Kotlin 中，枚举都是走向强类型的第一步。我相信大多数开发者都知道它们。让我们将代码改为使用枚举:*

```
*enum class Color(val id: String) {
    Default("0"),
    Blue("1"),
    Green("2"),
}enum class Availability(val value: String) {
    Free("transparent"),
    Busy("opaque"),
}*
```

*我们需要相应地改变函数，包括签名和实现:*

```
*fun execute(color: Color, availability: Availability) {
    findCalendarEntry(client, google, execution.conference)?.let {
        it.colorId = color.id                                   // 1
        it.transparency = availability.value                    // 1
        client.events()
            .update(google.calendarId, it.id, it).execute()
    }
}*
```

1.  *提取由`enum`包装的值*

*枚举的使用强制了强类型:*

```
*execute(Color.Blue, Availability.Busy)          // 1
execute(Color.Blue, Color.Red)                  // 2
execute(Availability.Free, Color.Blue)          // 2*
```

1.  *编制*
2.  *不编译！*

# *内嵌类*

*Kotlin 最近的一个特性完全致力于强类型:内联类。内联类包装单个“原始”值，例如`Int`或`String`。想象一下下面的类:*

```
*data class Person(givenName: String, familyName: String)*
```

*这个类的调用方必须记住第一个参数是名还是姓。Kotlin 已经通过允许命名参数提供了帮助:*

```
*val p = Person(givenName = "John", familyName = "Doe")*
```

*然而，我们可以通过将`String`包装在两个不同的值类型中来改进上面的代码片段，每个角色一个值类型。*

```
*@JvmInline value class GivenName(value: String)
@JvmInline value class FamilyName(value: String)val p = Person(GivenName("John"), FamilyName("Doe"))*
```

*在这一点上，一个人不能把一个名字换成一个姓，反之亦然。同样，我们可以在示例中使用值类，并在伴随对象中定义可能的值。*

```
*@JvmInline
value class Color(val id: String) {
    companion object {
        val Default = Color("0")
        val Blue = Color("1")
        val Green = Color("2")
    }
}@JvmInline
value class Availability(val value: String) {
    companion object {
        val Free = Availability("transparent")
        val Busy = Availability("opaque")
    }
}execute(Color.Blue, Availability.Busy)          // 1
execute(Color.Blue, Color.Red)                  // 2
execute(Availability.Free, Color.Blue)          // 2*
```

1.  *编制*
2.  *不编译！*

# *密封类*

*密封类是强制强类型的另一种可能方式。限制是我们需要在同一个包中定义一个密封类的所有子类。不能有任何第三方继承。实际上，它为您的代码创建了类`open`，为客户端代码创建了类`final`。*

*我们不像在值类中那样定义一个类型和它的几个实例，而是直接定义不同的类型。*

```
*sealed class Color(val id: String) {
    object Default: Color("0")
    object Blue:    Color("1")
    object Green:   Color("2")
}sealed class Availability(val value: String) {
    object Free : Availability("transparent")
    object Busy : Availability("opaque")
}execute(Color.Blue, Availability.Busy)          // 1
execute(Color.Blue, Color.Red)                  // 2
execute(Availability.Free, Color.Blue)          // 2*
```

1.  *编制*
2.  *不编译！*

*注意，我在它们各自的父类中定义了对象。根据您的上下文，您可能希望将它们设置为顶级。*

```
*sealed class Color(val id: String)
object Default: Color("0")
object Blue:    Color("1")
object Green:   Color("2")sealed class Availability(val value: String)
object Free : Availability("transparent")
object Busy : Availability("opaque")execute(Blue, Busy)*
```

# *结论*

*Kotlin 提供了几个选项来加强 API 的强类型:枚举、值类和密封类。*

*虽然大多数开发人员对枚举非常熟悉，但我建议考虑值和密封类，因为它们给表带来了额外的好处。*

***更进一步:***

*   *[枚举类](https://kotlinlang.org/docs/enum-classes.html)*
*   *[内嵌类](https://kotlinlang.org/docs/inline-classes.html)*
*   *[密封类](https://kotlinlang.org/docs/sealed-classes.html)*

**最初发表于 2022 年 2 月 20 日* [*一个 Java 极客*](https://blog.frankel.ch/avoid-stringly-typed-kotlin/)*