# 科特林:构造函数

> 原文：<https://itnext.io/kotlin-constructors-6233e810a5bd?source=collection_archive---------2----------------------->

## 通过实例学习

![](img/52ba5f4a2157e3e489fc9ecdf17f6cd3.png)

兰迪·法特在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 拍摄的照片

像许多其他使用 Android 的开发人员一样，我开始用 Java 编写应用程序，后来随着它被 Google 和 Android 开发人员社区更广泛地接受，我转移到了 Kotlin。

当从 Java 过渡到 Kotlin 时，我遇到的第一个差异是构造函数的定义方式。

在 Java 中，你有零个或多个显式定义的构造函数。如果没有定义构造函数，则会为您创建一个默认的无参数构造函数。构造函数在 Java 类的主体中定义，每个定义的构造函数都通过重载其参数列表来与其他构造函数区分开来。

相反，在 Kotlin 中，你有所谓的主构造函数，它可以在类的签名中随意定义。除了主构造函数之外，您还可以定义零个或多个辅助构造函数。

下面是 Kotlin 中一个简单的主构造函数的例子:

```
**class** Dog **constructor**(**val name**: String) {}
```

这里有几件事需要注意:

*   在定义主体之前，**构造函数** 关键字的使用以及它如何出现在类的签名中。这个关键字在这个例子中是可选的，只有在使用注释(如`@Inject`)或可见性修饰符(如使构造函数私有)时才需要
*   主构造函数不能包含任何代码。通常出现在该构造函数中的初始化代码应该驻留在一个或多个初始化( **init** )块中。

如果提供了这些信息，上面的例子可以重写以省略**构造函数**关键字，结果将是相同的:

```
**class** Dog (**val name**: String)
```

主构造函数的一个例子是一个你不希望从类代码之外实例化的类，其中**构造函数**关键字确实是必需的。例如，您可能希望从工厂方法中创建 **Dog** 类的所有新实例:

```
**class** Dog **private constructor**(**val name**: String) {

    **companion object** {
        **fun** newDog(name: String) = Dog(name)
    }
}
```

与 Java 相比，Kotlin 中构造函数工作方式的另一个区别是它们是如何被调用的，换句话说就是如何创建新的实例。在 Kotlin 中没有新的关键字。相反，类的新实例也同样被创建:

```
**val dog01** = Dog(name = **"Sparky"**)
```

在 Kotlin 中，如果您没有显式定义主构造函数，那么将会为您创建一个。例如，下面的代码是完全可以接受的:

```
**class** EmptyDog
```

通过调用生成的无参数主构造函数，可以创建 **EmptyDog** 的新实例，如下所示:

```
**val emptyDog** = EmptyDog()
```

如果您希望在实例化您的类时运行某种初始化类型的代码，那么您可以使用一个或多个初始化块来实现这一点:

```
**class** Dog **constructor**(**val name**: String) {

    **init** {
        *println*(**"Registering $name with the AKC"**)
        registerDogWithAKC()
    }

    **private fun** registerDogWithAKC() {
        *// TODO: perform registration tasks* }
}
```

关于初始化块的一些快速注释:

*   初始化块与主构造函数相关联
*   无论是否显式定义主构造函数，每个定义的初始化块都将在类被实例化时运行
*   如果定义了一个以上的初始化块，那么它们将按照在类体中出现的顺序执行
*   出现在主构造函数中的字段可以从初始化块中引用(在上面的例子中，可以看到初始化块中引用了 **name** 字段)
*   如果已经定义了任何二级构造函数，那么请注意，所定义的初始化块将在任何二级构造函数体执行之前执行

说到二级构造函数，现在来说说那些:)

由于 Java 中缺少默认参数，你会经常看到一种被称为*伸缩构造函数*的反模式，其中构造函数被一次又一次地重载，在编辑器或类图中给它一种伸缩的外观。下面是这种反模式的一个简短示例:

```
**private** String **name**;
**private** String **breed**;
**private boolean registered**;

**public** Dog() {
    **this**(**"Scruffy"**);
}
**public** Dog(String name) {
    **this**(name, **"Terrier"**);
}
**public** Dog(String name, String breed) {
    **this**(name, breed, **false**);
}
**public** Dog(String name, String breed, **boolean** registered) {
    **this**.**name** = name;
    **this**.**breed** = breed;
    **this**.**registered** = registered;
}
```

这种反模式可以通过使用[构建器模式](https://en.wikipedia.org/wiki/Builder_pattern)来避免，但是在 Kotlin 中这几乎是不必要的，因为使用 Kotlin 您可以声明构造器参数的默认值:

```
**class** Dog (**val name**: String, 
           **val breed**: String = **"Terrier"**, 
           **val registered**: Boolean = **false**)
```

尽管如此，在很多情况下，您可能会发现需要定义第二个(或第二个)构造函数。

例如，假设一个复制构造函数将现有的 **Dog** 实例的值复制到一个新的 **Dog** 实例中:

```
**class** Dog (**val name**: String) {
    **constructor**(dog: Dog) : **this**(dog.**name**)
}

**val dog01** = Dog(name = **"Sparky"**)
**val dog02** = Dog(**dog01**)
```

关于二级构造函数的几点注意事项:

*   它们必须以关键字**构造函数**为前缀
*   它们必须直接调用主构造函数，或者通过另一个辅助构造函数间接调用。如上例所示，调用主构造函数是用 **this** 关键字完成的。
*   初始值设定项块总是在任何二级构造函数体之前执行
*   次级构造函数中指定的参数不会成为该类的属性或字段。它们不能以 **var** 或 **val 为前缀。**换句话说，你需要将那些传入的值赋给字段，或者在二级构造函数体中对它们做些什么。

**结论** 如果你来自 Java 背景，那么你可能会发现 Kotlin 中的构造函数一开始可能会令人生畏。希望这篇文章能帮助你走出学习曲线。一如既往，如果你想了解更多，请查看官方 Kotlin 文档并练习、练习、再练习。

编码快乐！托马斯·桑德兰

[](https://www.linkedin.com/in/thomas-sunderland/) [## 托马斯·桑德兰-安卓开发者| LinkedIn

### 加入 LinkedIn 我叫 Thomas Sunderland，是一名软件工程师，专注于原生 Android 开发。我…

www.linkedin.com](https://www.linkedin.com/in/thomas-sunderland/)