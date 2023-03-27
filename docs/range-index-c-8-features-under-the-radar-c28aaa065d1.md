# 范围和索引—雷达下的 C#8 特性

> 原文：<https://itnext.io/range-index-c-8-features-under-the-radar-c28aaa065d1?source=collection_archive---------0----------------------->

![](img/12dace7d6046c029338d7318655231cd.png)

最近，我发了几篇关于 C#8 中嵌入的一些奇特特性的帖子，但是，我觉得有一个特性非常有用，但是经常被忽略——Range & Index。

虽然是许多其他集合的基本构建块，但使用简单数组通常仅限于简单的索引访问。因为数组也实现了 IEnumerable，并且能够使用 Linq，所以使用起来仍然感觉很舒服，但是如果没有它，除了这个语法之外就没有什么了。

```
int[] arr = new int[]{1};Console.WriteLine(arr[0]); //1
```

然而，随着范围和指数的引入，这种情况发生了巨大变化。让我们来看看。

# 索引

索引是一个非常简单的结构，起初并不清楚为什么它不是一个简单的 int。访问带索引的数组将返回

```
Index index = 1;
int[] arr = new int[]{1, 5};
Console.WriteLine(arr[index]); // 5
```

然而，Index 的特别之处在于它还允许你定义一个从数组的**端**开始的索引！这是通过使用帽子运算符 **^** 来实现的，它使得索引读作“获取第 n 个索引，从末尾开始”。不过有一个奇怪的问题——与前面不同，从后面数从 1 开始。这是政府有意识的决定。不过是网队。

```
Index index = **^**1;
int[] arr = new int[]{1, 5};
Console.WriteLine(arr[index]); // 5
```

足够简单，事实上比通常的 *arr 有用得多。长度-1* 组合。为了方便起见，编译器会将语法变成这样:

```
Index index = 1;
=> Index index = (Index)1;Index reverse = ^1;
=> Index reverse = new Index(1, true);
```

注意这里的 *true* 指的是索引构造器的 fromEnd *参数*。

# 范围

索引很棒，但是范围才是新工具真正开始发光的地方！

范围又是一个结构，所以你可以赋值甚至传递它——你看到它不仅仅是语法上的糖。

这里有一个简单的例子来说明一个系列的作用:

```
Range range = 1..3;
int[] arr = new int[]{1, 2, 3, 4, 5};
int[] result = arr[range]; // result now contains: [2, 3]
```

这已经非常有用了，但是您甚至可以更进一步，将它与 hat 语法混合使用，将范围限制在从末尾开始的特定索引。不过这需要一点时间来适应，因为在这种情况下，与索引不同， *^0* 是完全合法的。该范围将评估到，因此它将不会尝试访问实际的 *^0* 位置，这将超出界限(本质上类似于访问 *arr[arr。计数]* )。

```
Range range = 2..^0;
int[] arr = new int[]{1, 2, 3, 4, 5};
int[] result = arr[range]; // result now contains: [3, 4, 5]
```

您甚至根本不必指定开始或结束，甚至可以省略两者:

```
int[] arr = new int[]{1, 2, 3, 4, 5};
int[] result = arr[1..]; // result now contains: [2, 3, 4, 5]
int[] result2 = arr[..^1]; // result now contains: [1, 2, 3, 4]
int[] result3 = arr[..]; // result now contains all: [1, 2, 3, 4, 5]
```

太棒了。请注意，所有这一切只可能发生在选定的类上，主要是数组、Span <t>和 string(因为它是 char[])。</t>

# 有用的用例

虽然不是最闪亮的特性，但它有很好的用例来大大提高代码的可读性。

我最喜欢的用例可能是子字符串。假设我们有一个字符串，想组成一个从 0 到任意位置的子串。

```
var baseString = "Hello";
var randomEnd = new Random().Next(1, 4);Old:
var substringOld = baseString.Substring(0, , baseString.Length - randomEnd);New:
var substringNew = baseString[..^randomEnd];
```

很方便，对吧？

另一个超级有用的用例是使用*跨度< T >。*这是一个非常自然的搭配，因为 span 代表一个连续的内存区域，访问这个内存的一个索引或一个特定的片是一个非常常见的用例。

```
var baseString = "Hello";
var baseSpan = baseString.AsSpan();
Console.WriteLine(baseSpan[1..3].ToString()); // el
```

和现代的许多功能一样。Net 中，对索引和范围的支持是通过 ducktyping 实现的。这里有一个很好的总结:[https://docs . Microsoft . com/en-us/dot net/cs harp/language-reference/proposals/cs harp-8.0/ranges # implicit-index-support](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/proposals/csharp-8.0/ranges#implicit-index-support)。

例如，如果您想要一个实现隐式索引支持的自定义类，可以这样做:

```
var customBuffer = new CustomBuffer();
Console.WriteLine(customBuffer[^3]); // 7public class CustomBuffer
{
    private int[] _buffer = new int[]{1,2,3,4,5,6,7,8,9};

    public int Length => _buffer.Length;

    public int this [int index] => _buffer[index];
}
```

除了 ducktyping 之外，您还可以添加显式支持，当然是通过添加一个索引或范围作为方法参数，或者作为一个直接的索引器。请注意，在这种情况下，我们移除了隐式支持，因为该类不再具有长度属性，并且索引器被更改为接受 Index 而不是 int:

```
var customBuffer = new CustomBuffer();
Console.WriteLine(customBuffer[^3]); // 7public class CustomBuffer
{
    private int[] _buffer = new int[]{1,2,3,4,5,6,7,8,9};

    public int this [Index index] => _buffer[index];
}
```

所以，总的来说，这是一个非常好的特性，即使更大的 C#8 特性，比如可空性，有点抢了风头。