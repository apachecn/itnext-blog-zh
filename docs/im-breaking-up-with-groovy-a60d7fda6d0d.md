# 我要和 Groovy 分手

> 原文：<https://itnext.io/im-breaking-up-with-groovy-a60d7fda6d0d?source=collection_archive---------0----------------------->

![](img/f9b29a097ff72f84711ff33562d6dd59.png)

由[布拉克·科斯塔克](https://www.pexels.com/@burakkostak)经由【pexels.com】T2

好像就在昨天，我才被介绍给 groovy。十多年来，我一直是一名相当成功的 java 开发人员，有一天，我的一位同事说:“嘿，看看这个很棒的东西，它像 Java，但更好。”

我几乎立刻就被吸引住了。

我喜欢用 setters、getters、equals、hashCode 和一个有意义的 toString 创建一个简单的 Java bean，而不会用 IDE 生成的代码破坏我的代码库。

```
**class** SimpleBean {
    **int value1** String **value2** }
```

我惊叹于地图构造器的简单性:

```
**new** SimpleBean(
        **value1**: 5,
        **value2**: **"hello"**,
)
```

以及列表和映射初始值设定项:

```
List<String> strings = [**'s1'**, **'s2'**, **'s3'**]
Map<String, Integer> maps = [
        **s1**: 25,
        **s2**: 50,
]
```

和字符串插值，或甜蜜字符串插值:

```
SimpleBean sb = **new** SimpleBean(
        **value1**: 5,
        **value2**: **"hello"**,
)

String message = **"**${sb.value1} **some other string stuff** ${sb.value2}**"**
```

似乎我一天都不能不发现 groovy 的一些令人敬畏的东西。

*   使用字段访问语法调用 get/set
*   [关闭](https://www.tutorialspoint.com/groovy/groovy_closures.htm)
*   UFO，又名宇宙飞船，操作员[(对，就是那个东西)](https://objectpartners.com/2010/02/08/the-groovy-spaceship-operator-explained/)
*   [猫王操作员](http://mrhaki.blogspot.com/2009/08/groovy-goodness-elvis-operator.html)
*   [空安全取消引用](http://mrhaki.blogspot.com/2009/08/groovy-goodness-safe-navigation-to.html)

![](img/4c9985a88b667d5a416283481ab10c33.png)

照片由[freestocks.org](https://www.pexels.com/@freestocks?utm_content=attributionCopyText&utm_medium=referral&utm_source=pexels)从[像素](https://www.pexels.com/photo/wedding-couple-sitting-on-green-grass-in-front-of-body-of-water-at-sunset-70737/?utm_content=attributionCopyText&utm_medium=referral&utm_source=pexels)拍摄

我可以继续下去。

我恋爱了。

我很快向我所有的朋友宣扬 groovy 比他们原始的 Java 优越。我发誓我再也不会用 Java 了。

为什么我要，我已经很棒了。我们是天生的一对。

![](img/e8499ab9795a45ba61eaf5cc678f542f.png)

照片由 [Johannes Plenio](https://www.pexels.com/@jplenio?utm_content=attributionCopyText&utm_medium=referral&utm_source=pexels) 从 [Pexels](https://www.pexels.com/photo/silhouette-photography-of-boat-on-water-during-sunset-1118874/?utm_content=attributionCopyText&utm_medium=referral&utm_source=pexels) 拍摄

麻烦开始于 Java 8 问世的时候。我本应该注意到这些警告信号，但我被对 Groovy 的热爱蒙蔽了双眼。

对于那些不了解情况的人来说，Java 8 可能是自 1.5 以来最大的 Java 版本之一。它引入了许多新的语言结构，包括:

*   [兰达的](https://www.geeksforgeeks.org/lambda-expressions-java-8/)
*   [溪流](https://www.baeldung.com/java-8-streams)
*   [模块](http://tutorials.jenkov.com/java/modules.html)

作为一名优秀的 Java 开发人员，并希望总是尝试新事物，我开始在我的 groovy 代码中使用这些新东西。

![](img/0fca809204a41ae70dd03ee0cc5b5d5f.png)

来自[像素](https://www.pexels.com/photo/food-restaurant-summer-nuts-2424/)的[像素](https://www.pexels.com/@pixabay)

## λ和流

Lambda 和 streams 就像花生酱和巧克力一样。它们让生活变得有价值:

```
List<SimpleBean> myList = **new** ArrayList<>();
myList.stream()
        .filter(b -> b.getValue1() > 5)
        .map(SimpleBean::getValue2)
        .collect(Collectors.*toList*());
```

Groovy 有 lamda，也有 streams。但是我想像那些酷孩子一样使用新的 Java 8，但是是用 Groovy。

```
List<SimpleBean> myList = []
myList.stream()
        .filter({ SimpleBean b -> b.value1 > 5 } **as** Predicate)
        .map({SimpleBean b -> b.value2} **as** Function)
        .collect(Collectors.*toList*())
```

那是什么？？！我简单可爱的小脸蛋怎么了？Groovy 对所有事情都使用闭包！这意味着，当你想使用一个一般类型的接口，比如谓词>或函数,?>时，你需要对你的非类型闭包进行类型转换，这样它才能工作。

## GString vs String

如前所述，早期的字符串插值是可怕的。我喜欢它。但是 groovy 做了一些奇怪的事情。

```
**"**${sb.value1} **some other string stuff** ${sb.value2}**"**
```

这不是字符串。这是个惊喜。在需要呈现之前，它不会计算字符串。理论上，这意味着如果 *sb.value1* 改变，我可以使用相同的 GString 并显示不同的值。实际上，我从来不需要这个功能。实际上，这意味着很多时候我不得不帮助 groovy 和 type 将字符串转换成字符串。？？？

```
**"**${sb.value1} **some other string stuff** ${sb.value2}**" as String**
```

很奇怪吧？

## 类型检查

起初，我觉得自己被证明是正确的。最后，我可以像那些自负的 Python 程序员一样嘲笑 Java 和它愚蠢的类型检查。谁需要它，我现在有 groovy。我只需要 def。

然后我开始得到运行时异常:

*   未知方法
*   没有这样的财产
*   无法在…之间转换

哦，对了。

类型检查会发现所有这些。没关系，我将使用 Groovy 的内置类型检查，并开始用

```
@CompileStatic
```

这应该能解决问题。就这样发生了。但是现在 Groovy 实施了类型检查。我放了些奇怪的东西，比如:

```
.setList([] as List<String>)
```

我可以用 Java 非常简单地做到这一点:

```
.setList(Collections.emptyList())
```

这里的 Java 是类型安全和不可变的。

嗯，关于不变的…

![](img/34961b250a59d64e3c418d414ba04d4d.png)

由[皮克斯贝](https://www.pexels.com/@pixabay)从[Pexels.com](https://www.pexels.com/photo/human-fist-163431/)

## 不变的

大多数人都同意不可变对象由以下部分组成:

*   状态被初始化一次的对象
*   状态不能改变
*   写入时复制

起初，人们会想，哦，这在 groovy 中很容易做到。

```
**class** SimpleBean {
    **final int value1
    final** String **value2** }
```

完成了。

只有，没有。我们需要一个构造函数，因为默认的构造函数不起作用。

```
**class** SimpleBean {
    **final int value1
    final** String **value2** SimpleBean (**int** value, String value2) {
        **this**.**value1** = **value1
        this**.**value1** = value2
    }
}
```

现在好了吗？不完全是。你看，通过提供一个构造函数，你使它不再工作。所以你真的需要提供一个地图构造器…手动的。

```
**class** SimpleBean {
    **final int value1
    final** String **value2** SimpleBean (**int** value, String value2) {
        **this**.**value1** = **value1
        this**.**value1** = value2
    }
    SimpleBean(Map<?,?> values) {
        **this**.**value1** = values[**'value1'**] **as int
        this**.**value2** = values[**'value2'**]
    }
}
```

现在我们终于完成了，对吗？不完全是。虽然这可能行得通，但我们并没有真正以我们应该有的方式实现 map 构造函数，比如防止未知值等。但是在这一点上继续前进，进入一个核心问题。

你看，Groovy 并没有真正的 *final 概念。*就此而言，它其实也没有*私有*的概念。大多数情况下，您可以自由地修改 final 字段，并访问内部私有成员。

来吧，试试看！

这是一种非常 Python 的做事方式。但是我们(Java 开发者)喜欢控制。我们喜欢安全性和封装性。当涉及到不可变对象时，我们需要它！

当然可以，*正确地*在 groovy 中做不可变对象:

```
@Immutable
**class** SimpleBean {
    **int value1** String **value2** }
```

很简单。不知何故，它甚至加强了最终性质和私有变量。哈扎！

只是没有。

如果我想要一个自定义的构造函数呢？

抱歉，不能为不可变对象提供构造函数。

好吧，那默认值呢？

不，也不能这么做。

非空强制呢？

没有。

基本上，如果你想要其中的任何一个，你需要为你的不可变对象创建一个构建器，希望并祈祷有人不要调用那个公共的构建器，它只是坐在那里，等着他们。

![](img/b897b36d7e05a10fa354ec728e941832.png)

照片由 [Pexels](https://www.pexels.com/photo/analogue-art-box-chest-366791/?utm_content=attributionCopyText&utm_medium=referral&utm_source=pexels) 的[大卫·巴图斯](https://www.pexels.com/@david-bartus-43782?utm_content=attributionCopyText&utm_medium=referral&utm_source=pexels)拍摄

## Java 模块

Groovy 在内部严重依赖反射。以至于在 Java 模块发布后的一段时间内，Groovy 无法工作。即使有反射，你也不能访问你不应该访问的东西。公平地说，这破坏了许多其他库，slf4j、CORBA、Spring、Jenkins 等等。任何依赖于具有自由访问权限的全局类路径的东西都会崩溃。

这是我实际上已经习惯并欢迎的事情，因为它让我回到了我在 OSGi 的多级路径时代。但我没想到 Groovy 会如此受欢迎。

## 真实

你要么喜欢它，要么讨厌它。起初我很喜欢它:

```
List items;
if (items) {
   ...
}
```

但是让我们深入研究一下。if 子句中实际发生了什么。Groovy 将尝试将条件子句中的项目转换为 true。它并不总是相同的，也不总是显而易见的。在这种情况下，如果 items 为非 null 且非空，则该子句为 true。相当良性。让我们试试别的:

```
BigDecimal v1 = 0.00
BigDecimal v2 = 0.000v1 == v2
```

这里的等号是对还是错？首先，如果你不知道 Java devs，groovy 中的' == '不是等价检查。正如许多医生会告诉你的那样，这也不是“*等于*”的捷径。不，时髦是卑鄙的。如果可以，它将首先尝试一个“*与*的比较，然后它将依靠“*等于*”。所以上面的 v1 不等于 v2，因为精度不同。但是由于 groovy 使用了 compareTo，所以' == '的值将为 true。现在你可以说 0.00 等于 0.000，在某些情况下，我们想知道精度是否也不同。

这就说到我的重点了。直到 Groovy 在运行时以奇怪的方式中断，有时 Groovy 在做什么并不明显。

![](img/dd93a5f4cb7891f66b58858fba2253d9.png)

来自[像素](https://www.pexels.com/photo/newly-make-high-rise-building-162557/)的[像素](https://www.pexels.com/@pixabay)

## 编译工具

groovyc 没问题，就像 gcc 没问题，javac 没问题一样。意思是一点也不好。使用 Maven 编译 groovy 时，我反复纠结。现在，公平地说，这并不是 groovy 的错，而是 Maven 的错。

有许多方法可以做到这一点。

*   如果你需要 Groovy 1.8，这是镇上唯一的游戏
*   [GMaven](https://groovy.github.io/gmaven/) (已弃用)
*   [GMaven Plus](https://github.com/groovy/GMavenPlus)
*   [Groovy Eclipse Maven 插件](https://github.com/groovy/groovy-eclipse/wiki/Groovy-Eclipse-Maven-plugin)

GMaven Plus 没有集成到普通的 Maven 编译器中，虽然它声称支持联合编译(同时编译 groovy 和 java ),但我发现它并不总是有效。尤其是当 Java 和 Groovy 之间存在某种循环依赖时。

Groovy Eclipse 也好不到哪里去。很难配置。您需要确保对于您打算使用的 groovy 版本，您拥有正确版本的编译器和批处理库。它在联合编译方面做得很好。但是不做 groovy 文档或 groovy 执行。

Gradle 似乎确实比 Maven 更好地处理了 groovy。

## 其他工具

作为第二语言，Groovy 获得了第二工具。Checkstyle 不起作用。Codenarc 还不错，声纳也支持它。IDE 基本上可以正确地使用它。Eclipse 很糟糕，Intellij 工作得相当好。

注意到一个趋势？

## 落后于 Java

作为一种建立在另一种语言之上的语言，从定义上来说，你总是在特性(java 模块)上落后。或者，如果你试图创新和前进，你会冒特性冲突的风险(闭包和 lambda)。如上所述，这两种情况在 Groovy 中都很常见。

## 版本兼容性

Groovy 在这里做了一些好事，在这里也做了一些不太好的事情。我可以向他们推荐，我可以用一段为 groovy 1.8 编译的 groovy 代码在 2.5 中运行，干得好！

但是为什么在微小的版本变化之间会出现问题呢？为什么会改变行为？你为什么撒谎？！？

如果你使用 Groovy，那么 Groovy 发行说明几乎是必读的。

这是 Groovy 中的一个常见主题。他们改变事物。他们以我认为是重大改变的方式改变东西，但是他们没有在他们的版本号中反映出来。

例如，从 Groovy 2.4 到 2.5，他们改变了 CLI 工具。从表面上看，这似乎没什么大不了的，所有的代码仍然可以编译。但是我们有几个工具，从命名的命令行参数，需要一个长的名字和两个破折号

```
--name <value>
```

到只需要一个“-”。

```
-name <value>
```

这破坏了我们的许多自动化工具，我们花了很长时间才弄明白。

![](img/2dc253fda7ff08b6e79f0c840af8e57e.png)

由来自[像素](https://www.pexels.com/photo/sunglasses-woman-girl-faceless-2867/)的[像素](https://www.pexels.com/@pixabay)

## 其他奇怪的狗屎

您知道 Groovy 之前在每个类中插入一个字段作为编译的时间戳吗？我不知道，直到我对我们的库做了 API 分析检查，发现我们类的每个版本都与以前的版本二进制不兼容。

或者如果在子类中重写了超方法，旧版本的 Groovy 不支持调用超方法。

或者 groovy 不支持许多现在标准的 Java 语法，比如:

*   接口中的默认方法(即将推出 Groovy 3)
*   Java 尝试使用资源块(即将推出 Groovy 3)
*   方法引用(即将推出的 Groovy 3)
*   嵌套代码块(即将推出 Groovy 3)

这种事情已经成为一种文化基因在起作用。“哦，是的，那…#太棒了”

![](img/24a3b39d69e0e0d75c6232e6e9f488fc.png)

来自[像素](https://www.pexels.com/photo/person-holding-tape-measure-1385749/?utm_content=attributionCopyText&utm_medium=referral&utm_source=pexels)的[rawpixel.com](https://www.pexels.com/@rawpixel?utm_content=attributionCopyText&utm_medium=referral&utm_source=pexels)的照片

## 膨胀

随着 Groovy 的每一次发布，他们都会添加很多东西。乍一看，这似乎很棒。所有这些新的做事方法。但是 Groovy 试图成为每个人的一切。它给语言增加了许多负担。虽然语法可能更短，但是语法也更复杂，并且总是在变化。

保持一个小的语言特性集是有道理的。做某事没有很多方法。取而代之的是一些正确的方法。以围棋为例。它决不是令人难以置信的功能丰富。恰恰相反。但是你可以在一个下午学会它的语法。

除了语法膨胀，还有库膨胀。作为库开发者，我们需要意识到我们给消费者带来的依赖性。

依赖不是免费的。

*   部署的应用程序规模增加
*   依赖性冲突的风险增加(不在 OSGi :P)

现在 groovy 已经做了不同的事情来解决这些问题。通过隐藏它们的依赖关系，它们消除了类路径冲突问题，但是代码大小加倍了。所以他们重新使用 OSS 库，但是这又引入了冲突。

![](img/2a8f7d68add299da081d02e1607a1ee0.png)

由来自[像素](https://www.pexels.com/photo/man-and-woman-sitting-on-bench-984953/)的[像素](https://www.pexels.com/@pixabay)

# 分手

所以我们站在这里。有一次 Java 开发人员带着 Groovy 跑了。新的热门事物。现在我想要回 Java。Java 8 增加了很多很棒的东西，比如消除 Groovy 闭包需求的流。但是，我如何获得大量我喜欢 groovy 的代码简化功能呢？

## [龙目岛](https://projectlombok.org)

Lombok 是 javac 的扩展。它在编译时插入代码。它插入的代码对其他库没有依赖性。代码很快，并且遵循良好的编程实践。最棒的是，它使阅读 Java 变得非常简单。

是，你没看错 JAVA！你写作。java 文件，并且可以在使用 Lombok 的类文件中编写任何 java 代码。

可变 Bean(Get/Set/Equals/HashCode/ToString)

```
**import** lombok.Data;

@Data
**public class** SimpleBean {
    **int value1**;
    String **value2**;
}
```

不可变 Bean:

```
**import** lombok.Value;

@Value
**public class** SimpleBean {
    **int value1**;
    String **value2**;
}
```

具有构建器模式、默认值和非空强制的不可变 bean:

```
@Value
@Builder
**public class** SimpleBean {
    @Builder.Default
    **int value1**=25;
    @NonNull
    String **value2**;
}
```

我从哪里来。

良好的

1.  我仍然写 Groovy，
2.  我用 Lombok 写 Java

对我来说，groovy 现在比 Java 更好的一点甚至不是真正的 Groovy。

![](img/71d15dd8090a5a0e41d4c01049af26fa.png)

来自[像素](https://www.pexels.com/photo/board-chalk-chalkboard-exam-459793/) (CC0)的[像素](https://www.pexels.com/@pixabay)

## [斯波克](http://spockframework.org)

我们都写测试。但是编写 Junit 测试很烦人，因为我们被 Java 的语言特性所困扰。Spock 是一个很棒的测试框架，它使用 Groovy 的语言特性来行善而不是作恶。我可以快速编写人类可读的测试。我可以快速轻松地模仿、模仿、侦查、注射。当测试失败时，我会得到非常有用的错误信息。

```
**class** SimpleBeanSpec **extends** Specification {
    **def "test simpleBean"**() {
        **given**:
        SimpleBean bean = **new** SimpleBean()

        **when**:
        bean.**value1** = 25

        **then**:
        bean.value1==25
    }
}
```

基本上，如果你没有用 Spock 编写你的测试，你真的应该这么做。因为我只在测试中使用 Spock，所以所有的 groovy 依赖性和我在 groovy 中遇到的许多其他问题都消失了。

# 我想念…

我仍然怀念许多美好的事物。

我想念:

*   易于创建列表和地图
*   字符串插值
*   零解引用

但是我发现我没有足够的想念他们来保证继续奋斗。我知道 Groovy 3 承诺修复我提出的许多问题。但是它也做了很多我不喜欢的事情。Groovy 继续引入更多的语法。Groovy 正在进行其他大的改变，比如改变解析器。最重要的是，他们仍然落后于 Java。

也许有一天，我会像老朋友一样回到 Groovy，或者 Java 最终会添加一些我觉得它缺少的东西。或者也许，只是也许，会有别的事情发生…

[科特林](https://kotlinlang.org)有人吗？