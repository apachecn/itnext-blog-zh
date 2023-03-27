# 任何，单位，虚无和他们所有的朋友

> 原文：<https://itnext.io/any-unit-nothing-and-all-their-friends-e39613b48235?source=collection_archive---------1----------------------->

![](img/3b6bd37457bd0ddd60e3c52c1e627302.png)

# 任何的

这是一个简单的类型，Any 只是“根”类型，所以其他类型都是从它扩展而来的。这就像 Java 中的 Object，实际上，为 Any 类型的值编译的代码是一个对象。

// **科特林**

```
val greeting: Any = "Hi there!"
```

// **Java**

```
private final Object greeting = "Hi there!";
```

# 单位

Kotlin 中的函数返回单元在 Java 中的意思是返回 void。此外，Kotlin 中的函数返回单元不需要显式地返回它。所以这两个函数编译成相同的:

// **科特林**

```
fun returnsUnit(): Unit {
}fun returnsUnitExplicitly(): Unit {
    return Unit
}
```

// **Java**

```
public final void returnsUnit() {
}

public final void returnsUnitExplicitly() {
}
```

## 那么什么是单位呢？

单元被定义为单一实例。因此，该类型只有一个有效值:

```
val *unit*: Unit = Unit
```

从类型层次结构的角度来看，与任何其他类型类似，Unit 是 any 的子级:

```
val *unit*: Any = Unit
```

# 没有任何东西

没有什么是没有任何可能值的类型。这是因为它被定义为具有私有构造函数的常规类。根据文档: *Nothing 没有实例。可以用 Nothing 来表示“一个从来不存在的值”。*

那么如果没有办法构造或获取任何 Nothing 类型的值，它又有什么用呢？

## 键入从不返回(或抛出异常)的函数

```
fun infiniteLoop(): Nothing {
    while (true) {
        *println*("Hi there!")
    }
}
```

或者

```
fun throwException(): Nothing {
    throw IllegalStateException()
}
```

编译器足够聪明，可以在这两种情况下推断出函数永远不会(正确地)返回。这就是为什么任何放在函数调用后不返回任何内容的代码都将被忽略。编译器将对不可访问的代码显示警告。

让我们定义一个可能抛出异常(或返回 null)的新函数:

```
fun mayThrowAnException(*throwException*: Boolean): Nothing? {
    return if (*throwException*) {
        throw IllegalStateException()
    } else {
        *println*("Exception not thrown :)")
        null
    }
}
```

如果我们从我们的主:

```
fun main() {
    val *result* = *mayThrowAnException*(true)
    if (*result* == null) { // **Always true**
        *println*("Ignored code")
    }
}
```

编译器提示我们结果将总是空的。这是为什么呢？因为程序在调用 *mayThrowAnException* 函数后继续运行的唯一方法是返回一个 null(因为否则它将什么都不是)。如果函数没有返回任何东西，是因为它会抛出一个异常，这将导致我们的程序中断。

## 将抛出和返回转换为表达式

你有没有想过为什么这是可能的？

```
val *nullableValue*: String? = null
val *value* = *nullableString* ?: throw IllegalStateException()
```

`nullableString`的类型是字符串？，而`throw IllegalStateException()`什么都不是。碰巧没有什么是每种类型的子类型。这就是该表达式最终计算为字符串的原因。

另一个例子:

```
val *nullableValue*: String? = null
val *value: Int* = *nullableValue*?.*toInt*() ?: return
```

这种情况类似于前一种情况，但是我们没有抛出异常，而是从函数中返回。`return`的类型什么都没有，同上。

最终，该语言将 return 和 throw 都“转换”成表达式(其类型为 Nothing)，这导致了更简洁的编程(而不是成为“异常”语句)。

## 类型层次结构中什么都没有？

我们在文章的开头说过，Any 是在类型系统之上的。没有什么是相反的，是在底部。这是什么意思？这意味着没有什么是每个类型的子类型。这就是上面显示的代码示例有效的原因。让我们回顾一下上一个例子的最后一部分:

```
*nullableValue*?.*toInt*() ?: return
```

`nullableValue?.toInt`的类型是 Int？然后在 nullableValue 为空的情况下，我们从函数中使用 [elvis 算子](https://kotlinlang.org/docs/reference/null-safety.html#elvis-operator)到`return`。而返回表达式的类型是 Nothing。

所以映射到类型，我们会有:

```
Int? ?: Nothing
```

整个表达式的返回类型是两种类型共有的类型。因为没有什么是每个类型的子类型，Int？是他们的共同点。**一般来说，类型 T 和 Nothing 之间的共同类型永远是 T**