# JAVA 中的比较器和比较器

> 原文：<https://itnext.io/comparable-and-comparator-in-java-bbb8e89f1fec?source=collection_archive---------2----------------------->

![](img/f36e7a9fde82e74e5a882a31246fc0b9.png)

迭戈·杜阿尔特·塞雷塞达在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的“圣地亚哥的一长排橙色出租自行车”

> [*点击这里在 LinkedIn 上分享这篇文章*](https://www.linkedin.com/cws/share?url=https%3A%2F%2Fitnext.io%2Fcomparable-and-comparator-in-java-bbb8e89f1fec%3Futm_source%3Dmedium_sharelink%26utm_medium%3Dsocial%26utm_campaign%3Dbuffer)

当构建程序来解决特定问题时，我们会遇到许多创建自己的类(自定义类)的情况。例如，一个表示新闻文章的类可以在抓取新闻网站时保存信息。

无论我使用什么编程语言，都有许多有用的内置实用程序来扩展我自己的代码。最有用的内置函数工具之一是对数组或列表进行排序。然而，我们需要付出一点努力来实现适用于自定义类的排序功能。因为内置的排序工具不知道有一个新创建的类，我们应该让它们知道有一个。

Comparable 和 Comparator 是为了给 JAVA 中的自定义类提供排序能力而引入的。在这篇文章中，我想谈谈他们。

# 可比较的

Comparable 是 java.lang 包中包含的接口之一。意思是不需要你自己包含，在 JAVA 里是很原始的。即使你不知道，JAVA 提供的一些基本类已经自带了。

Comparable interface 的目的是让新的类实例能够将其自身与属于同一类类型的其他实例进行比较。你可以在这里找到官方文档([https://docs . Oracle . com/javase/7/docs/API/Java/lang/comparable . html](https://docs.oracle.com/javase/7/docs/api/java/lang/Comparable.html))。如果你仔细研究过，在使用它之前，你应该只记得两件事。

*   **类型参数(你想比较哪个类？)**
*   **compareTo 法(如何比较？)**

类型参数几乎总是与实现 Comparable 的类的类类型相同，并且类型参数指示我们想要与哪个类进行比较。然后，我们实际上让自定义类知道如何将自己与其他类进行比较，这种能力可以通过实现 compareTo 方法来提供。

compareTo 方法接受一个自定义类希望与之进行比较的参数变量，compareTo 方法返回一个整数值，以指示自定义类应该放在对手实例的前面、后面还是相等位置。因此，返回值应该分为三个值域。

*   **0(相等)**
*   **+(后)**
*   **-【前】**

上面的示例代码显示了 Person 类如何通过名为“age”的变量进行自我比较。当当前实例的年龄为 10 岁而对手的年龄为 11 岁时，compareTo 方法返回-1。这意味着当前实例应该放在对手之前，当我们把它们放在一起的时候。

```
List<Person> people = fetchMembers();
Collections.sort(people);
```

在定义了具有可比接口的自定义类之后，我们可以通过将列表传递给 Collections.sort 方法([https://docs . Oracle . com/javase/7/docs/API/Java/util/collections . html # sort(Java . util . list)](https://docs.oracle.com/javase/7/docs/api/java/util/Collections.html#sort(java.util.List))作为参数，轻松地对包含该类实例的列表/数组进行排序。如果存储在给定列表中的实例的类没有实现可比较的接口，Collections.sort 方法将抛出一个错误。

Collections.sort 方法不返回任何内容，但是传递给它的列表将会对其元素进行排序。

# 可比的问题

对于我们的代码来说，类似的接口带来了多少便利，这看起来很不错。然而，这种方法有一个问题。OOP(面向对象编程)方法也许能够解释这个问题。在 OOP 范例中，最好尽可能将代码解耦。

对于上一节中的例子，我们让 Person 类实现 Comparable 接口，将它自己与名为“age”的变量进行比较。如果我们想用不同的变量比如“名字”来比较呢？我们每次都必须修改 compareTo 方法内部的代码。Person 类可以看作是一个模型类，它可以用在很多地方。这意味着 Person 类也可以被其他开发者的代码引用！

这是不好的，因为如果我们在修改代码时犯了语法错误，离开办公室一段时间，许多同事将不得不等待，甚至不知道发生了什么。

然后，我们可以对自己说，如果我们能够将人员类别与比较功能分开，那就更好了。

# 比较仪

下面的代码是以非常标准的形式编写的，在下一节中可以找到更实用的代码实现。所以，你不用一直提心吊胆的写这么繁琐的代码。

Comparator 是 java.util 包中包含的一个接口，用于解决上一节中讨论的问题。我们可以从自定义类中提取比较功能，并在单独的类中定义该功能。

下面的代码示例定义了实现比较器接口的 PersonAgeComparator 类。如您所见，比较器界面看起来与可比界面非常相似。区别在于方法名(comareTo → compare)和参数数量。正如方法名的变化所表明的，Comparator 看起来更像 Comparable 的通用接口。正如你所猜测的，这个方法采用两个参数的原因是它不知道要比较哪一个。

```
List<Person> people = fetchMembers();
Collections.sort(people, new PersonAgeComparator());
```

存在覆盖的 collections . sort([https://docs . Oracle . com/javase/7/docs/API/Java/util/collections . html # sort(Java . util . list，%20java.util.Comparator)](https://docs.oracle.com/javase/7/docs/api/java/util/Collections.html#sort(java.util.List,%20java.util.Comparator)) 方法，该方法采用实现比较器接口的类实例的附加第二个参数。其他一切都和前面的例子一样。

# 匿名比较器

如果比较标准在整个项目中非常流行，那么用比较器编写类的标准方法会很有用。然而，一些开发人员希望为临时目的定义这样的功能。然后，为了提高他们的工作效率，一直定义单独的类可能是一项非常繁琐的工作。

对于这种情况，我们可以使用被称为“匿名类”的语法技巧。下面的代码示例展示了如何实现这一点。关于匿名类的更多详细信息，请通读这个([https://docs . Oracle . com/javase/tutorial/Java/javaOO/Anonymous classes . html](https://docs.oracle.com/javase/tutorial/java/javaOO/anonymousclasses.html))。

# 带有 Lambda 表达式的比较器

匿名类看起来对提高我们的工作效率很有用，但是我们可以通过 JAVA8 中新引入的 Lambda 表达式进一步提高。如果你了解函数式编程语言，这并不是什么新鲜事。Lambda expression 只是一个附加特性，反映了函数式编程范式的日益流行。Lambda expression 的官方教程可以在这里找到([http://www . Oracle . com/web folder/tech network/tutorials/OBE/Java/Lambda-quick start/index . html](http://www.oracle.com/webfolder/technetwork/tutorials/obe/java/Lambda-QuickStart/index.html))。

使用 Lambda 表达式和 List 类中的新方法“sort ”,我们可以用一行代码对列表进行排序。您可能需要一些时间来熟悉这种风格的代码，但是您会惊讶于它能提高多少代码可读性。

上面的示例代码展示了三种不同的方式来编写 Lambda 表达式。