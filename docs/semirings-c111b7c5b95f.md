# 半环

> 原文：<https://itnext.io/semirings-c111b7c5b95f?source=collection_archive---------6----------------------->

如果你不熟悉[半群&幺半群](/semigroups-faf7a70da96a)请先阅读我之前的文章。

一个**半群**有一个追加`<>`运算。

一个**半环**有两个追加操作`plus` ( `+`)和`times` ( `*`，以及两个各自的标识元素`zero`和`one`。

![](img/5d16d653b20d2b250864f4368bced03d.png)

## 规则

*   加法**必须**返回相同的值，而不管值的顺序。这就是所谓的*交换幺半群*。
*   与`zero`相乘**必须**返回`zero`。这就是通常所说的*歼灭*。
*   就结合性而言，标准规则适用于+和*

基于这个描述，我们可以编写一个类似这样的协议:

```
**protocol** Semiring {
    **static** **func** + (lhs: Self, rhs: Self) -> Self
    **static** **func** * (lhs: Self, rhs: Self) -> Self
    **static** **var** zero: Self { **get** }
    **static** **var** one: Self { **get** }
}
```

现在让我们看看这如何与`Int`类型一起工作。

```
**extension** Int: Semiring {
    **static** **let** zero = 0
    **static** **let** one = 1
}
```

在这种情况下，我们可以免费获得`+`和`*`行为。

我承认还不是很令人兴奋，我们还没有做任何标准 Swift 框架没有提供的事情。

我们还可以用半环协议扩展 Bool，这对我们下面的例子很有用…

```
**extension** Bool: Semiring {
    **static** **func** + (lhs: Bool, rhs: Bool) -> Bool {
        **return** lhs || rhs
    } **static** **func** * (lhs: Bool, rhs: Bool) -> Bool {
        **return** lhs && rhs
    } **static** **let** zero = **false
    static** **let** one = **true** }
```

这个有点不一样。我们可以声明:

*   `+`与`||`相同，因为(`true || false` = `true`)和(`false || true = true` ) —通过第一个规则
*   `*`与`&&` ( `true && false` = `false`)相同——通过第二个规则

所以现在我们有能力用`+`和`*`组合布尔值。

仍然不是很令人兴奋，所以让我们创建一个包装 lambda 的结构。

下面是实现过程:

```
**struct** Function<A, B> {
    **let** execute: (A) -> B
}
```

现在我们有能力把一个`Function`变成一个`Semiring`这样:

```
**extension** Function: Semiring where B: Semiring {
    **static** **func** + (lhs: Function, rhs: Function) -> Function {
        **return** Function { lhs.execute($0) + rhs.execute($0) }
    } **static** **func** * (lhs: Function, rhs: Function) -> Function {
        **return** Function { lhs.execute($0) * rhs.execute($0) }
    } **static** **var** zero: Function {
        **return** Function { **_** **in** S.zero }
    } **static** **var** one: Function {
        **return** Function { **_** **in** S.one }
    }
}
```

上面的操作符基本上是取两个`Function`对象，对每个对象应用(`A`)值以返回(`B1`和`B2`)，然后在两者之间应用操作符(`B1 +/* B2`)。

为了返回`zero`和`one`值，我们可以只返回我们的`S`值定义的值。简单。

太好了。这为许多强大的东西提供了基础，当与我们上面的*布尔*示例结合时，例如我们可以创建*谓词*。其中`S`是布尔值，`A`是我们要验证的表达式。

# 谓词—一个实际的例子:

为什么？我们已经有预测了。是的，这是真的，但是它不是*编译时安全的*，我们通常传入一个`String`来验证它可能对它所应用的对象没有意义。

在*编译时*比在*运行时*更容易发现问题，所以让我们尽可能支持*编译时*安全。

我们可以这样定义我们的`Predicate`:

`**typealias** Predicate<A> = Function<A, Bool>`

我们也可以将取反运算符定义为:

```
**prefix** **func** ! <A> (p: Predicate<A>) -> Predicate<A> {
    **return** .init { !p.execute($0) }
}
```

现在我们已经讨论了积极和消极的情况，让我们创建一些函数…

```
**func** equalTo<T: Equatable>(**_** value: T) -> Predicate<T> {
    **return** Predicate<T> { $0 == value }
}**func** greaterThan<T: Comparable>(**_** value: T) -> Predicate<T> {
    **return** Predicate<T> { $0 > value }
}**func** lessThan<T: Comparable>(**_** value: T) -> Predicate<T> {
    **return** Predicate<T> { $0 < value }
}
```

谓词与我们的`+`和`*`操作符结合是没有意义的，但是您会期望它们分别与`||`和`&&`结合。因此，让我们来定义这些运算符重载:

```
**func** || <A> (lhs: Predicate<A>, rhs: Predicate<A>) -> Predicate<A> {
    **return** lhs + rhs
}**func** && <A> (lhs: Predicate<A>, rhs: Predicate<A>) -> Predicate<A> {
    **return** lhs * rhs
}
```

酷，现在我们已经有了创建一些强大的谓词组合的框架。然而，目前我们的谓词是倒着读的，我们可以这样称呼它们:

```
lessThan(5).execute(0) // **true**
```

让我们颠倒一下这个操作，通过在`Comparable`上定义一个扩展，它使用一个`Predicate`来使它更具可读性。

```
**extension** Comparable {
    **func** `is`(**_** predicate: Predicate<Self>) -> Bool {
        **return** predicate.execute(**self**)
    }
}
```

> 注意:Comparable 扩展了 Equatable，所以这是我们在下面的例子中唯一需要的函数。

假设我们想检查一个值是否在 1 和 5 之间，但不等于 3。我们可以使用现有的函数定义如下:

```
**func** between<T: Comparable>(_ a: T, and b: T) -> Predicate<T> {
    **return** greaterThan(a) && lessThan(b)
}**let** valid = between(1, and: 5) && !equalTo(3)1.is(valid)     *//* ***false*** *-- outside of range* 2.is(valid)     *//* ***true*** *-- in range and not equalTo 3*
3.is(valid)     *//* ***false*** *-- is equalTo* 4.is(valid)     *//* ***true*** *-- in range and not equalTo 3*
5.is(valid)     *//* ***false*** *-- outside of range* 5.is(!valid) *//* ***true*** *-- inverse of previous*
```

我们还可以扩展 Sequence 来返回一个过滤后的数组，如下所示:

```
**extension** Sequence {
    **func** filtered(by predicate: Predicate<Element>) -> [Element] {
        **return** filter(predicate.execute)
    }
}[0, 2, 3, 6].filtered(by: valid) *// [2]*
```

> 我听到你说…所以呢？我可以用闭包做同样的事情…

虽然这是真的，但还是有一些*微妙的*差异使得`Predicates`(以及推而广之`Semirings`)更加**强大**:

*   我们可以很容易地 ***组合*** 它们，没有任何令人困惑的函数签名。上面的`valid`变量是这种可读性的一个很好的例子。
*   我们可以在继承结构中的任何一点扩展*它们，我们可以扩展`Semiring`、`Function`或`Predicate`来实现更多的方法，这些方法将向下流到我们的子类型。*

*让我们通过实现一个简单的`apply`函数来探索第二点:*

```
***extension** Function {
    **func** apply(**_** f: **@escaping** (A) -> A) -> Function<A, B> {
        **return** .init { **self**.execute(f($0)) }
    }
}*
```

*现在我们可以颠倒我们之前的例子来处理负数:*

```
***let** validNegative: Predicate<Int> = valid.apply(-)(-2).is(validNegative) *//* ***true****
```

*或者在运行谓词之前将数字翻倍或减半:*

```
***let** validIfHalved = valid.apply { $0 / 2 }4.is(validIfHalved) *// validates 2***let** validIfDoubled = valid.apply { $0 * 2 }2.is(validIfDoubled) *// validates 4**
```

# *单元测试——使用谓词*

*现在我们可以再次扩展`Comparable`，并为`XCTAssertTrue`添加一个包装器。*

```
***extension** Comparable {
    **func** assert(**_** predicate: Predicate<Self>, file: StaticString = **#file**, line: UInt = **#line**) {
        XCTAssertTrue(**self**.is(predicate), file: file, line: line)
    }
}
**extension** Sequence **where** Element: Comparable {
    **func** assert(**_** predicate: Predicate<Element>, file: StaticString = **#file**, line: UInt = **#line**) {
        **for** element **in** **self** {
            element.assert(predicate, file: file, line: line)
        }
    }
}*
```

*现在我们可以编写表达性的单元测试来断言我们的结果。*

```
***func** testMinimumIsValid() {
    2.assert(valid) *// runs single test using comparable method*
}**func** testNumbersAreValid() {
    [2, 4].assert(valid) *// runs 2 tests using sequence method*
}**func** testNumbersAreInvalid() {
    [1, 3, 5].assert(!valid) *// runs 3 tests using sequence method* }*
```

*当然，我们可以用这些扩展测试任何`Comparable`或`Equatable`类型，这也让我们可以替换`XCTAssertEqual`和`XCTAssertNotEqual`。*

*只需很少的努力，我们也可以扩展这个功能来处理`nil`类型，以取代`XCTAssertNil`和`XCTAssertNotNil`。*

```
***func** isNil<T>() -> Predicate<T?> {
    **return** Predicate { $0 == **nil** }
}**func** isNotNil<T>() -> Predicate<T?> {
    **return** Predicate { $0 != **nil** }
}**extension** Optional {
    **func** assert(**_** predicate: Predicate<Wrapped?>, file: StaticString = **#file**, line: UInt = **#line**) {
        XCTAssertTrue(predicate.execute(**self**), file: file, line: line)
    }
}**let** optional: Int? = 1
optional.assert(!isNil())
optional.assert(isNotNil())*
```

# *结论*

*仅仅是半环力量的一个例子，它将帮助你编写更有表现力的代码。它展示了定义`Predicates`然后用一种简单而富于表现力的方式测试它们，同时拥有*编译时类型安全*是多么容易。*

*半环还有很多其他的应用，例如，你也可以返回`Sets`或者像另一个 iOS 例子一样——动画。*

***Gist:**
[https://Gist . github . com/CJ nevin/92a 253 b 59 ff 07 ff 00 B3 e 85 F2 de 284 DFB](https://gist.github.com/cjnevin/92a253b59ff07ff00b3e85f2de284dfb)*

***参考资料:** Brandon Kase:[https://github . com/bkase/slides/blob/master/animations-semi rings/slides . MD](https://github.com/bkase/slides/blob/master/animations-semirings/slides.md)*

*布兰登·威廉姆斯:[https://www . fewbutripe . com/swift/math/algebra/semi ring/2017/08/01/semi rings-and-predicates . html](https://www.fewbutripe.com/swift/math/algebra/semiring/2017/08/01/semirings-and-predicates.html)*