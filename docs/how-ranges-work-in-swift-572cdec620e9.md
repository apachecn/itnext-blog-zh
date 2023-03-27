# Swift 中范围的工作方式

> 原文：<https://itnext.io/how-ranges-work-in-swift-572cdec620e9?source=collection_archive---------5----------------------->

![](img/8c295a9344eaf043f378d8b09b0ae26a.png)

照片由[西德尼·塞弗林](https://unsplash.com/@sidseverin?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)在 [Unsplash](https://unsplash.com/s/photos/range?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄

Ranges 家族的类型似乎不会给我们带来惊喜，但是它们隐藏了实现中令人兴奋的地方。

Swift 标准库有一系列我们习惯使用的范围类型，例如:

```
let array: Array<Int> = [0, 1, 2, 3, 4, 5]
array[...3]
array[...]
for i in 1..<4 {
  print(i)
}
```

我们习惯于操作像`[1, 2, 3]`或`[1:2, 3:4]`或`"some string"`这样的文字。我们可以声明我们的范围为 let`r = 2..<4`。它看起来像一个文字，但它是一个二元运算符`..<`，有两个参数作为输入，一个范围作为输出(范围的具体类型取决于运算符——二元运算符`1...10`和`1..<10`，一元运算符`...10`、`..<10`和`1...`)

尽管系列产品的类型有相似之处，但所有产品在实施方面都有显著差异。然而，乍一看，似乎唯一的区别是如何指定范围的边界。

让我们了解一下 Range 是如何工作的。

# **RangeExpression 协议**

Swift 中几乎所有的 Range 类型都有一个共同点:都符合 *RangeExpression* 协议(除了*unboutedrange*)。该协议有两种能力:

*   首先，它可以检查一个元素是否包含在一个范围内(*包含*函数)
*   第二个功能可以将任意量程类型(*partial Range up 到*、 *PartialRangeThrough* 、 *PartialRangeFrom* 、 *ClosedRange* )转换为通用*量程*类型(*相对于*功能)

我们为什么需要它？如你所知，我们可以通过下标来引用集合。例如:

```
let array = [0, 1, 2, 3, 4, 5, 6, 7, 8]
array[1..<3] *// operator 1..<3 returns Range<Int>(lowerBound: 1, upperBound: 3), which we pass as input to the subscript*
array[..<3] // PartialRangeUpTo<Int>
array[...3] // PartialRangeThrough<Int>
array[1...3] // ClosedRange<Int>
array[1...] // PartialRangeFrom<Int>
```

在所有这些例子中，不同的范围类型被传递给下标，所以在每个集合中为每种范围族实现一个下标是不好的。

输入下标接受 *RangeExpression* 协议。在执行下标之前，集合调用协议上的*相对*函数，将结果范围( *PartialRangeUpTo* ， *PartialRangeThrough* ， *PartialRangeFrom* ， *ClosedRange* )转换为通用*范围*类型。

但是有一个特性，我们不能绝对地将任何类型的范围系列转换为*范围*类型。例如，在输出端，我们无法提交`obj1...obj2`并获得通用的*范围* `obj1..<obj3`(其中 *obj3* 是跟随 *obj2* 的对象)，因为我们不知道哪个对象在 *obj2* 之后。但是，如果*绑定的*是*集合*协议中的一个索引，我们仍然可以识别以下元素

```
func relative<C: Collection>(    to collection: C  ) -> Range<Bound> where C.Index == Bound
```

在这种情况下，我们可以调用*collection . index(after:)*，找出下一个元素返回标准*范围*。

# 范围

主范围结构非常简单:两个字段存储在下限和上限结构中。

唯一的最低要求是这些字段必须是可比较的类型。`public struct Range<Bound: Comparable>`。让我提醒你*可比*反过来符合*公平*协议。假设您想要实现符合*可比*的结构。在这种情况下，你必须至少实现`<`操作符(如果你的结构不满足自动合成*等价*的要求，那么你也必须实现`==`操作符)。其余运算符`>`、`<=`、`>=`默认基于`<`运算符实现。因此，您可以创建自己的结构:

```
+---------+---------------+
|  Body   | Diameter (km) |
+---------+---------------+
| Sun     |     1,391,000 |
| Mercury |         4,880 |
| Venus   |        12,104 |
| Earth   |        12,756 |
| Mars    |         6,794 |
| Jupiter |       142,984 |
| Saturn  |       120,536 |
| Uranus  |        51,118 |
| Neptune |        49,532 |
+---------+---------------+**struct** Planet: Comparable {
  **static** **func** < (lhs: Planet, rhs: Planet) -> Bool {
    **return** lhs.diameterKm < rhs.diameterKm
  }
  **let** name: String
  **let** diameterKm: Double
}**let** mercury = Planet(name: "Mercury", diameterKm: 4880)
**let** earth = Planet(name: "Earth", diameterKm: 12756)
**let** neptune = Planet(name: "Neptune", diameterKm: 49532)
**let** jupiter = Planet(name: "Jupiter", diameterKm: 142984)mercury == neptune // false**let** planets = mercury..<neptune
```

我们创造了一个从水星到海王星的范围。地球在这些行星之间，但这个范围不包括海王星和木星。我们只有两颗行星在*范围*内，基于一个简单的比较算子，我们可以找出被检查的对象是否包含在*范围*内:

```
planets.contains(earth) // true
planets.contains(neptune) // false
planets.contains(jupiter) // false
```

我们还可以返回两个范围的交集或检查交叉点。

```
(mercury..<neptune).clamped(to: earth..<jupiter) // earth..<neptune
```

它是*系列*结构的骨架。我们的自定义绑定类型的简单范围仅限于这些函数；没有额外的努力，我们无法迭代或做任何其他事情。Range 的其他特性的实现取决于哪些类型符合 Range 的下限和上限。

# 范围可以是集合

*Range* 可以是一个集合，但前提是 Range 中指定的界限符合 *Strideable* 协议，其中 stride 是一个 *SignedInteger* (下文只提到 *Strideable* ，也暗示了对 *SignedInteger* 的限制)。

如果*系列*的边界符合*可划分的*，那么*系列*具有我们已经熟悉的系列所固有的新特征。例如，可以循环迭代:

```
for i in -1..<8 {
  print(i)
}
```

在这种情况下，我们定义了一个*范围*，但是我们的*范围*也将是一个*序列*，因为它的边界符合*可跨越的*协议。

我们不能为我们的行星写出`for p in mercury…jupiter`,因为在这种情况下，许多函数没有被定义。我们不知道哪个星球跟着哪个星球(我们的星球是不可跨越的)。

符合 *Strideable* 的*范围*有一个类型别名叫做 *CountableRange* 。这样的*范围*可以通过下标进行迭代和访问。常规*范围* ( *绑定*不符合*可绑定*)不具备此功能；它只有两个界限，并且只受 *RangeExpression* 协议能力的限制。

您可以像使用常规集合一样使用 *CountableRange* ，例如`(3..<5)[4]`，在这种情况下，下标必须接收一个值作为参数，该值本身在指定的边界内。因此，表达式`(3..<5)[0]`将不起作用，我们将有一个错误。

# **部分范围到** **和** **部分范围到**

这些结构非常相似，都包含一个 *upperBound* 字段。关于*partialrangeupt 到*，该字段不在范围内，而在*partialrangeup 到*的情况下，该字段在范围内。这两个实体都没有下限。因此，它们遵循与这些结构在任何情况下都不能符合*序列*或*集合*协议的事实相关的特征。因此，您不能迭代它们或通过下标引用它们:

```
**//** this code will not compile **for** n **in** ...10 {
  print(n)
}
**for** n **in** ..<10 {
  print(n)
}
(...10)[9]
(..<10)[9]
```

连这个代码都不行:`(..<4).reversed()`。我们知道我们可以定义反向序列的开始元素，并迭代剩余的元素。尽管如此，由于 *PartialRangeUpTo* 不符合*序列*，该表达式将不被编译。

# 部分距离

这是一个只有下限的系列结构。我们已经有了一个开始元素，但是在这个例子中没有结束元素。因此，如果我们的下限是 *Strideable* ，我们可以实现*序列*协议的功能，但是由于缺少上限，我们将无法在我们的结构上实现*集合*。

让我提醒你，*序列*对元素的数量没有限制，而*集合*中的元素数量必须是有限的。对于来自的*partial range，其中 *Strideable* 的下界是来自*的 type alias*countable partial range，我们可以迭代。*

# 封闭区域

就实现而言，这是系列产品中最不寻常的结构。与常规的*范围*一样，如果边界参数符合*可跨越的*，该结构可以实现*序列*和*集合*协议。似乎与*范围*相比，在*集合*协议的实现上应该有最小的差异。然而，使用索引有很大的不同，只是差别很小。 *ClosedRange* 中集合的索引定义为 enum:

```
public enum Index {
    case pastEnd
    case inRange(Bound)
}
```

该索引是一个枚举！

这种认知差异的原因是什么？*集合*协议有一个 *endIndex* 属性。我强调一下，这是集合的上限，不包含在集合里。对于一个*范围*结构， *endIndex* 很容易被定义为上界( *upperBound* 属性)。在 *ClosedRange* 的权利要求中， *endIndex* 可以定义为跟随上限的索引。

由于上界必须符合*可刻划的*，我们可以称之为`upperBound.advanced(by: 1)`。一切看起来都很好，但是有一个小问题: *Int* 值(符合 *Strideable* )有一个最大值`Int.max`。对于一个定义为`1...Int.max`的 *ClosedRage* ，其 *endIndex* 属性的值应该是`Int.max+1`，但是`Int.max`已经是最大值了。我们可以通过使用 enum 作为索引来解决这个问题，其中 *endIndex* 只是 enum 中的一种情况。处理集合中索引的逻辑是围绕这个枚举实现的。

我们不能像对待常规的*范围*那样对待`(4...10)[5]`。但是我们可以这样做`(4...10)[.inRange(5)]`，此外，在当前的实现中，对于这样的调用，没有包含在 *ClosedRange* 中的检查。`(4...11)[.inRange(3)]`将返回 3(即使范围不包含 3)，而`(4...11)[3]`将不起作用(因为在 closed range index-enum 中)。

原来 *ClosedRange* 是标准库中最奇怪的集合，因为它通过一个下标来引用它的集合；enum 需要使用一个不寻常的结构。

```
// ClosedRange **let** collection1 = 4...100
collection1[.inRange(-100)] // ok: -100
collection1[.inRange(5)] // ok: 5
// collection1[5] // compile error// Range **let** collection2 = 4..<100 
// collection2[.inRange(-100)] // compile error
// collection2[-100] // runtime error
collection2[5] // ok: 5
```

但是您还应该注意，建议使用 *startIndex* 、 *endIndex* 属性和用于计算索引的函数 *index(after:)* 、 *distance(from:to:)* 等来处理集合索引。，在*系列*的协议中定义。它将使您的应用程序在运行时安全，并允许您从所有集合类型的特定索引实现中抽象出来。

```
// a safe way to work with ranges
**let** closedRange = 4...100
closedRange[closedRange.index(after: closedRange.startIndex)]//ok: 5
```

# UnboundedRange

这里有一个更不寻常的系列类型。这是一个定义为`public typealias UnboundedRange = (UnboundedRange_)->()`的 typealias。Enum *UnboundedRange_* 被用作`...`操作符的名称空间，它什么也不做。它允许我们使用这样的`collection[...]`结构。我们可以在解决递归问题的文档中看到这样的例子。

```
func countLetterChanges(_ s1: Substring, _ s2: Substring) -> Int {
  if s1.isEmpty { return s2.count }    
  if s2.isEmpty { return s1.count }       let cost = s1.first == s2.first ? 0 : 1     
  return min(
        countLetterChanges(s1.dropFirst(), s2) + 1,
        countLetterChanges(s1, s2.dropFirst()) + 1,
        countLetterChanges(s1.dropFirst(), s2.dropFirst()) + cost)
}let word1 = "grizzly"
let word2 = "grisly"
let changes = countLetterChanges(word1[...], word2[...])
// changes == 2
```

该类型不符合*range expression*(Ranges 家族类的唯一例外)，因此*集合*和 *MutableCollection* 对于 *UnboundedRange* 有单独的下标实现。

它本质上是一个很小的改动，只需要在集合的下标中指定集合本身的大小范围。

# 摘要

我们探索了所有类型的产品系列。这些类别表示定义范围边界的不同组合。

*   **范围*类型*和*关闭范围*和**类型如果边界工作到*可 Strideable* 就可以符合*集合*(为这种情况定义的类型别名*可计数范围*和*可计数关闭范围*)。但是对于 *CountableClosedRange* 来说，索引通常是通过 enum 实现的，因为否则的话，在实现某些函数时，可能会出现处理类型最大值的问题。
*   **一个来自** 类型的*partial range 如果边界符合 *Strideable* 就可以符合*序列*(这种情况下定义了 type alias*countablepartial range from*)。但是由于缺少上限，我们无法符合这种类型的*集合*。*
*   ***PartialRangeUpTo*和*PartialRangeThrough***类型不能有 *Sequence* 或*集合*实现，因为它们没有定义开始索引。
*   ***UnboundedRange***是一个 hack 类型，由操作符`...`专门用来定义*集合*和*可变集合*上的下标。

您可以在这里看到范围[的实现。](https://github.com/apple/swift/tree/main/stdlib/public/core)