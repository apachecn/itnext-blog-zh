# 从一个 Kotlin 开发者的角度来看，我在 Java 中错过了什么

> 原文：<https://itnext.io/miss-java-kotlin-developer-2bc6a8e3539a?source=collection_archive---------0----------------------->

![](img/d3a0d122d0eae62893085d0d83aecad2.png)

近 20 年来，Java 一直是我的谋生手段。几年前，我开始学习科特林语；我从未后悔过。

虽然 Kotlin 编译成 JVM 字节码，但我有时不得不重新编写 Java。每当我这样做的时候，我都无法停止思考为什么我的代码看起来没有 Kotlin 中的好。我错过了一些可以提高代码可读性、表现力和可维护性的特性。

这篇文章并不是要抨击 Java，而是列出一些我想在 Java 中找到的特性。

# 不可变引用

Java 从一开始就有不可变的引用:

*   对于类中的属性
*   对于方法中的参数
*   对于局部变量

```
class Foo { final Object bar = new Object();      // 1 void baz(final Object qux) {          // 2
        final var corge = new Object();   // 3
    }
}
```

1.  无法重新分配`bar`
2.  无法重新分配`qux`
3.  无法重新分配`corge`

不可变引用对于避免讨厌的错误有很大的帮助。有趣的是，使用`final`关键字并不普遍，即使是在广泛使用的项目中。比如 Spring 的`[GenericBean](https://github.com/spring-projects/spring-framework/blob/main/spring-web/src/main/java/org/springframework/web/filter/GenericFilterBean.java)`使用不可变属性，但既不是不可变的方法参数，也不是局部变量；slf4j 的`[DefaultLoggingEventBuilder](https://github.com/qos-ch/slf4j/blob/master/slf4j-api/src/main/java/org/slf4j/spi/DefaultLoggingEventBuilder.java)`三个都不用。

虽然 Java 允许定义不可变的引用，但这不是强制性的。默认情况下，引用是可变的。大多数 Java 代码没有利用不可变引用。

Kotlin 没有给你留下选择的余地:每个属性和局部变量都需要定义为`val`或`var`。另外，不能重新分配方法参数。

Java 关键字`var`则完全不同。首先，它只适用于局部变量。更重要的是，它没有提供其不可变的对应物`val`。您仍然需要添加几乎没人使用的关键字`static`。

# 零安全

在 Java 中，没有办法知道一个变量是否是`null`。明确地说，Java 8 引入了`[Optional](https://docs.oracle.com/javase/8/docs/api/java/util/Optional.html)`类型。从 Java 8 开始，返回一个`Optional`意味着底层值可以是`null`；返回另一种类型意味着它不能。

然而，`Optional`的开发者设计它只是为了返回值。对于方法参数和返回值，语言语法中没有任何内容。为了解决这个问题，一些库提供了编译时注释:

*   [JSR 305](https://github.com/amaembo/jsr-305/tree/master/ri/src/main/java/javax/annotation) 带注释`@Nonnull`和`@Nullable`
*   [弹簧](https://github.com/spring-projects/spring-framework/tree/main/spring-core/src/main/java/org/springframework/lang)、`@NonNull`和`@Nullable`
*   [喷射脑](https://github.com/JetBrains/java-annotations/tree/master/common/src/main/java/org/jetbrains/annotations)、`@NotNull`和`@Nullable`
*   [Findbugs](https://github.com/stephenc/findbugs-annotations/tree/master/src/main/java/edu/umd/cs/findbugs/annotations) 、`@NonNull`和`@Nullable`
*   [日食](https://github.com/eclipse/aspectj.eclipse.jdt.core/tree/main/org.eclipse.jdt.annotation/src/org/eclipse/jdt/annotation)、`@NonNull`和`@Nullable`
*   以及[检查器框架](https://github.com/typetools/checker-framework/tree/master/checker-qual/src/main/java/org/checkerframework/checker/nullness/qual)、`@NonNull`和`@Nullable`

显然，一些库专注于特定的 ide。此外，库之间很难兼容。有这么多可用的库，以至于 StackOverflow 上有人问用哪一个。由此产生的活动很能说明问题。

最后，使用可空性库是可以选择的。另一方面，Kotlin 要求每个类型要么可以为空，要么不可以为空。

```
val nonNullable: String = computeNonNullableString()
val nullable: String? = computeNullableString()
```

# 扩展功能

在 Java 中，通过子类化来扩展一个类:

```
class Foo {}
class Bar extends Bar {}
```

子类化有两个主要问题。第一个问题是一些类不允许这样做:它们用关键字`final`标记。几个广泛使用的 JDK 类是`final`、*，例如*、`String`。第二个问题是，如果一个方法在我们的控制之外返回一个类型，我们就会被这个类型所束缚，不管它是否包含我们想要的行为。

为了解决上述问题，Java 开发人员发明了实用程序类的概念，通常将类型`XYZ`命名为`XYZUtils`。实用程序类是一组带有`private`构造函数的`static`方法，因此它不能被实例化。这是一个美化了的名称空间，因为 Java 不允许类之外的方法。

这样，如果某个方法在某个类型中不存在，实用程序类可以提供一个方法，该方法将该类型作为参数并执行所需的行为。

```
class StringUtils {                                          // 1 private StringUtils() {}                                 // 2 static String capitalize(String string) {                // 3
        return string.substring(0, 1).toUpperCase()
            + string.substring(1);                           // 4
    }
}String string = randomString();                              // 5
String capitalizedString = StringUtils.capitalize(string);   // 6
```

1.  实用程序类
2.  防止此类型的新对象的实例化
3.  `static`方法
4.  不考虑边角情况的简单大写
5.  `String`类型不提供大写功能
6.  使用一个实用程序类来考虑这种行为

注意，在早期，开发人员在项目内部创建了这样的类。如今，这个生态系统提供开源库，比如 Apache Commons Lang 或 Guava。不要多此一举！

Kotlin 提供了[扩展函数](https://kotlinlang.org/docs/extensions.html#extension-functions)来解决同样的问题。

> *Kotlin 提供了用新功能扩展类或接口的能力，而不必继承类或使用设计模式，如* Decorator *。我们可以通过称为*扩展*的特殊声明来实现它。*
> 
> 例如，你可以从第三方库中为一个类或一个接口编写新的函数，但你不能修改它。这样的函数可以像它们是原始类的方法一样以通常的方式被调用。这种机制称为扩展功能*。*
> 
> *要声明一个扩展函数，在它的名字前面加上一个*接收方*类型，它指的是被扩展的类型。*

使用扩展函数，可以将上面的代码重写为:

```
fun String.capitalize2(): String {                           // 1-2
    return substring(0, 1).uppercase() + substring(1);
}val string = randomString()
val capitalizedString = string.capitalize2()                 // 3
```

1.  自由浮动函数，不需要类包装
2.  `capitalize()`已经存在于科特林的 stdlib 中
3.  调用扩展函数，好像它属于`String`类型

注意扩展函数是“静态”解析的。它们并没有真正将新的行为附加到现有的类型上；他们假装这样做。生成的字节码非常类似于(如果不是相同的话)一个 Java 静态方法。然而，语法要清晰得多，并且允许函数链接，这在 Java 的方法中是不可能的。

# 具体化的泛型

Java 的第 5 版带来了泛型。然而，语言设计者热衷于保持向后兼容性:Java 5 *字节码*需要与 Java 5 *之前的字节码*完美地交互。这就是为什么泛型类型没有被写入生成的*字节码*中:它被称为*类型擦除*。相反的是具体化的泛型，泛型类型将被写在字节码*中。*

泛型类型仅仅是编译时的问题，会产生一些问题。例如，以下方法签名产生相同的*字节码*，因此，代码无效:

```
class Bag {
    int compute(List<Foo> persons) {}
    int compute(List<Bar> persons) {}
}
```

另一个问题是如何从值容器中获取类型化的值。这里有一个春天的例子:

```
public interface BeanFactory {
    <T> T getBean(Class<T> requiredType);
}
```

开发人员添加了一个`Class<T>`参数，以便能够知道方法体中的类型。如果 Java 已经具体化了泛型，那就没有必要了:

```
public interface BeanFactory {
    <T> T getBean();
}
```

想象一下如果科特林具体化了泛型。我们可以改变上面的设计:

```
interface BeanFactory {
    fun <T> getBean(): T
}
```

并调用该函数:

```
val factory = getBeanFactory()
val anyBean = getBean<Any>()               // 1
```

1.  具体化的泛型！

Kotlin 仍然需要遵守 JVM 规范，并与 Java 编译器生成的*字节码*兼容。它可以通过一个叫做内联的技巧来工作:编译器用函数体替换内联的方法调用。

下面是让它工作的科特林代码:

```
inline fun <reified T : Any> BeanFactory.getBean(): T = getBean(T::class.java)
```

# 结论

在这篇文章中，我描述了我在 Java 中错过的四个 Kotlin 特性:不可变引用、空安全、扩展函数和具体化泛型。虽然 Kotlin 提供了其他很棒的特性，但这四个特性足以对 Java 进行大部分改进。

例如，有了扩展函数和具体化的泛型，再加上一点语法上的好处，就可以很容易地设计 DSL，比如 Kotlin Routes 和 Beans DSL:

```
beans {
  bean {
    router {
      GET("/hello") { ServerResponse.ok().body("Hello world!") }
    }
  }
}
```

毫无疑问:我知道 Java 作为一门语言有更多的惰性需要改进，而 Kotlin 天生更灵活。但是，竞争是好的，两者可以互相学习。

同时，我只在必要时才写 Java，因为 Kotlin 已经成为我在 JVM 上选择的语言。

**更进一步:**

*   [变量](https://kotlinlang.org/docs/basic-syntax.html#variables)
*   [无效安全](https://kotlinlang.org/docs/null-safety.html)
*   [扩展功能](https://kotlinlang.org/docs/extensions.html#extension-functions)
*   [具体化类型参数](https://kotlinlang.org/docs/inline-functions.html#reified-type-parameters)

*原载于* [*一个 Java 怪胎*](https://blog.frankel.ch/miss-in-java-kotlin-developer/)*2022 年 6 月 12 日*