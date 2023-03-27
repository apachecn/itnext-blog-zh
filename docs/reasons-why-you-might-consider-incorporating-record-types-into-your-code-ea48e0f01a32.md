# 您可能想考虑将记录类型合并到代码中的原因

> 原文：<https://itnext.io/reasons-why-you-might-consider-incorporating-record-types-into-your-code-ea48e0f01a32?source=collection_archive---------2----------------------->

![](img/a7016a720af4064a89cc74d123384d32.png)

C#9 中最大的特点可能是引入了*记录*类型。

一个*记录*实现了许多有用的功能——它是一个引用类型，但同时拥有许多开发人员使用值类型的特性，例如，当两个*记录*的所有字段都被认为相等时，它们之间的相等性被给出。此外，与一个完全成熟的类相比，这是一种定义托管一组属性的引用类型的非常简洁的方法。

然而，虽然很多人都知道记录，但并不总是清楚它们如何以及在哪里有用。在这篇文章中，我想回顾一下其中的一些，以启发你适应这种新的语言特性

# 少点 POCO 样板！

使用*记录*的一个很好的方法是将其用作 POCO(普通旧 CLR 对象)。开发人员经常创建小类，这些小类除了以尽可能简单的方式存储数据之外没有任何责任。请看下面的例子:

```
void Main()
{
    var container = new MyContainer("Hello", 5);
}public class MyContainer
{
    public string Name { get; set; }

    public int Counter { get; set; }

    public MyContainer(string name, int counter)
    {
        Name = name;
        Counter = counter;
    }
}
```

这是非常基本的，每个开发人员都知道反复编写相同的样板文件有多烦人。然而，现在您可以大幅减少样板文件:

```
void Main()
{
    var container = new MyContainer("Hello", 5);
}public record MyContainer(string Name, int Counter);
```

厉害！好多了！注意，与常规的现成类相比，检查这个对象的相等性和 hashcodes 的行为会有很大不同，但是除非您依赖常规的相等性行为，否则这是减少为相同输出编写的代码的一个很好的方法。

如果你想详细看看编译器在幕后为你创建了什么，可以看看[https://sharp lab . io/# v2:cylg 1 apgaggazajwkygmd 2d hwlie 8 bkqgwafajyb 2 alnagjkidovanhfaiwamcaykrwabkanxa = =](https://sharplab.io/#v2:CYLg1APgAgzABAJwKYGMD2DhwLIE8BKqGwAFAJYB2ALnAGJkIDOVANHFAIwAMcAykRWABKANxA==)。编译器为你构建了很多功能。

当然，您总是可以通过附加属性(这将是可选的，因为它们不会出现在构造函数中，除非您定义一个自定义属性)或其他方法在*记录*的主体中扩展*记录*，因此您可以扩展更多。同样，*记录*可以像普通类一样利用继承，但是规则与普通*类*不同。

POCO 的功能使得*记录*特别适用于为您的控制器端点创建小的 UIModels，而您并不真的想在上面写出来。

# 记录支持简单的不可变编码风格！

不变性是越来越多的人喜欢使用的概念，因为它使得大量代码的行为更加可预测和可靠。就对象而言，不变性主要旨在使对象本身不能改变——当您希望对象改变时，您只需创建一个新的对象。这对于参考问题也有很大的帮助。如果你需要改变一些东西，你得到一个新的参考，你可以很容易地比较这个。

*记录*类型用心支持不可变的编码模式。有几个概念清楚地表明了这一点:

默认情况下，编译器在*记录*类型中生成的属性(您在*记录*的大括号中声明的属性)是 init 专用的。这意味着，它们可以在对象创建期间被分配，但不能在之后被重新分配。因此，以下代码是无效的:

```
void Main()
{
    var container = new MyContainer("Hello", 5);
    container.Name = "Bye"; // This is forbidden
}public record MyContainer(string Name, int Counter);
```

表明*记录*类型包含不变性的另一个重要特性是带有关键字的*。虽然我刚才提到不可能修改一个*记录*类型的默认属性，但是您可以简单地创建一个带有修改属性的**新的**对象。请参见以下语法:*

```
void Main()
{
    var container = new MyContainer("Hello", 5);
    var newContainer = container with { Name = "Bye" };
    Console.WriteLine(ReferenceEquals(container, newContainer)); 
    // Output: false
}public record MyContainer(string Name, int Counter);
```

基本上，带有关键字的*所做的是获取一个对象的现有属性，将其与您指定的想要修改的属性合并，并使用结果创建一个新的*记录*。最棒的是，随着应用到对象的更改，您还创建了一个新对象。正如不变性原则所规定的那样。*

现在想象用一个普通的班级做这个。代码要多得多。对于相同的行为，您必须实现代码来克隆对象，然后在其上设置新的属性，以获得我们刚刚用一行代码实现的相同行为。

# 引用类型的值语义！

你可能认为*记录*类型有用的最后一点是它们在值类型语义和引用类型之间架起了一座桥梁。如果您没有意识到，值类型被认为是相等的，如果它们的内容是相等的:

```
void Main()
{
    var myStructOne = new MyStruct { Number = 5 };
    var myStructTwo = new MyStruct { Number = 6 };
    Console.WriteLine(myStructOne.Equals(myStructTwo)); // False
    myStructTwo.Number = 5;
    Console.WriteLine(myStructOne.Equals(myStructTwo)); // True
}public struct MyStruct
{
    public int Number { get; set; }
}
```

现在，如果你想在一个引用类型中实现这种行为，你必须特别重写 *==* 或者*。等于(…)* 。猜猜看——这正是*记录*类型已经为您提供的现成内容。编译器只是为您创建了所有这些。

这有什么用处呢？

整个设置使得使用值类型比较行为变得很容易，但同时不必使用值类型语义的其余部分，比如在将对象传递给方法时复制整个对象。

虽然这不是开创性的，毕竟，我们总是能够自己创造这一点，这绝对是一个非常有用的东西。

这些是我最喜欢的关于*记录*类型的 3 件事，也许你也觉得你越来越想采用这个特性，因为上面提到的品质。