# 当评估属性时

> 原文：<https://itnext.io/kotlin-when-properties-are-evaluated-a7db5a8e2799?source=collection_archive---------6----------------------->

![](img/9749bb143e27ebb4e9c4fcfa33527753.png)

归属地:【https://www.flickr.com/photos/j_lightning/】T4

Kotlin 给出了在类中声明属性的不同方法。它们的声明方式会影响它们的求值时间。

让我们举一个简单的例子:

```
class Test {
    private fun returnSomething(str: String):String {
        println("evaluating $str")
        return str
    }

    val testGetEquals 
        get() = returnSomething("testGetEquals")  
    val testGetBrackets: String                            
        get() { return returnSomething("testGetBrackets") }
}
```

创建 Test 实例后，调用`testGetEquals`的 getter，每次都会通过`returnSomething`求值。`testGetBrackets`也是如此。

我经常这样写我简单的有值属性:

```
val testEquals = returnSomething("testEquals")
```

但是这将 testEquals 更改为在创建类时只计算一次。如果这是一个 Android 活动或片段，并且对字符串的评估依赖于您拥有的`Context`，这将导致崩溃或不正确的值。

所以如果我想只评估一次房产科特林还有一招。

```
val testLazy by lazy { returnSomething("testLazy") }
```

这也是只评估一次。但是它在第一次被调用时进行了计算。

请点击下面的链接，在科特林游乐场亲自尝试一下

**关于我** 我是一名 Kotlin/Java Android 开发者承包商。我总是很有兴趣听到你的项目和我能做些什么来互相帮助。[www.wedgetech.co.uk](https://www.wedgetech.co.uk/)