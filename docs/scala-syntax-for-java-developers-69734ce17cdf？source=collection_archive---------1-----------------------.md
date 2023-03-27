# 面向 Java 开发人员的 Scala 语法

> 原文：<https://itnext.io/scala-syntax-for-java-developers-69734ce17cdf?source=collection_archive---------1----------------------->

## Scala 和 Java 之间一些常见语法差异的快速指南

![](img/a89b784ea2f39a1f4bcdb0617a6437f8.png)

照片由[安娜斯塔西娅·杰尼娜](https://unsplash.com/@disguise_truth?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄

对 Scala 感兴趣的 Java 开发人员可能会发现语法上的差异令人望而生畏。下面是一些不同于 Java 的 Scala 语法规则的指南。最后，我们将浏览一个全面的 Scala 代码示例。

注意，我们不会在这里探究 Scala 的高级、强大的特性。[For-comprehensive](https://medium.com/wix-engineering/scala-comprehending-the-for-comprehension-67c9f7953655)、[implicit](https://medium.com/@olxc/https-medium-com-olxc-implicits-in-scala-b2dcfccaa9e8)、 [pattern-matching](https://www.tutorialspoint.com/scala/scala_pattern_matching.htm) 以及其他特性将在专门的文章中更好地讨论。相反，本文将帮助那些对探索 Scala 感兴趣，但只是很难理解他们所看到的代码示例的 Java 开发人员。

让我们开始吧。

## 类型声明在变量名之后

在 Java 中，我们先声明类型，然后声明变量名，就像这样:
`**Animal** a = getAnimal();`

在 Scala 中，类型在后面:
`val a: **Animal** = getAnimal();`

## 变量被声明为不可变的(val)或可变的(var)

在 Scala 中，我们以`val`或`var`开始变量声明，而不是像在 Java 中那样以类名开始。

`val`(简称*值*)表示变量不可变(类似 Java 中的`final`):
`val a: Animal = getAnimal(); // `a` *cannot* be subsequently reassigned`

`var`(变量的简称)表示变量是可变的:
`var a: Animal = getAnimal(); // `a` *can* be subsequently reassigned`

Scala 支持不变性，所以大多数情况下，你会看到使用`val`。

## 类型声明通常可以省略

Scala 有非常强的类型推断能力。所以我们很少需要显式声明类型。这就是当声明一个变量时，类放在最后的原因之一；很容易这样简单地省略:
`var a = getAnimal(); // `a` is still an instance of Animal`

## 不需要用分号来表示行尾

分号实际上只在多条语句出现在同一行时才需要:
`var a = getAnimal(); println(a)`

我们可以，而且几乎总是这样做，省略行尾的分号:
`var a = getAnimal()`

## 方法和函数是用“def”关键字声明的

方法和函数(一个*函数*类似于一个*方法*，但不属于任何类)用`def`关键字声明。它们的返回类型在函数名和冒号后声明。他们的身体通常跟在等号后面:

```
def getAnimal(): Animal = {
  // body goes here ...
}
```

## 当调用无参数方法或函数时，可以省略括号

在 Java 中，我们总是在调用方法时提供括号:

```
Animal a = getAnimal();
```

当调用不带参数的 Scala 函数或方法时，可以省略括号:

```
val a = getAnimal()
// or
val a = getAnimal
```

## 方法和函数不需要显式返回任何东西

方法或函数中的最后一个值或表达式成为隐式返回值:

```
def add(x: Int, y: Int): Int = {
  x + y  // the sum of x and y will be implicitly be returned
}
```

因此，尽管 Scala 中存在 *return* 关键字，但它几乎从未被使用过。

## 单位表示无效

如果一个 Java 方法没有返回值，那么它被声明为返回 *void* :

```
public void output(String s) {
  System.out.println(s);
}
```

在 Scala 中，返回类型将被声明为*单元*:

```
def output(s: String): Unit = {
  println(s)
}
```

## 方法和函数很少需要声明它们的返回类型

Scala 的强类型推理意味着我们很少需要声明函数/方法的返回类型:

```
def add(x: Int, y: Int) = {
  x + y  // The compiler infers that an Int will be returned
}
```

请注意，在以下情况下，我们仍然需要声明返回类型:

*   该方法是抽象的
*   我们想返回一个推断返回类型的超类
*   我们只是想更加清楚

```
def getAnimal(): Animal {
  // if we omitted the return type, Scala would infer Dog
  new Dog() 
}
```

## 方法和函数不需要大括号

如果方法或函数的主体不超过一行(或表达式)，则可以省略大括号:

```
def add(x: Int, y: Int) = 
  x + y// ordef add(x: Int, y: Int) = x + y
```

## 可以用命名参数调用方法和函数

在 Java 中，当调用接受多个参数的方法时，我们不能包含参数名。相反，我们只是按照在方法中声明的顺序传递值:

```
int i = add(6, 8);
```

在 Scala 中，我们可以选择将方法/函数称为“Java 方式”,也可以包含全部或部分参数名:

```
val i = add(6, 8)
// or
val i = add(x = 6, y = 8)
// or
val i = add(x = 6, 8)
// note: below will not compile:
val i = add(6, y = 8)
```

## 特征而不是接口

Java 提供接口，Scala 提供特性。像 Java 接口一样，特征可以包括抽象方法。和 Java 接口的默认方法一样，traits 可以提供实现的方法。与接口不同，特征还可以包含变量。

```
trait Shape {
  val point: Point
  def print() // no implementation, so implicitly abstract
}
```

## `延伸'和` with '

和 Java 一样，Scala 类不能扩展多个父类。同样类似于 Java，Scala 类可以实现(或者用 Scala 的说法，*扩展*)多个特征。

当一个 Scala 类扩展一个类或一个特征时，就会使用`extends`关键字。如果这个类扩展了任何额外的特征，那么使用关键字`with`:

```
class Shape { ... }trait Drawable { ... }trait Movable { ... }class Circle extends Shape with Drawable with Movableclass Raster extends Drawable with Movable
```

## 类是用括号中直接列出的成员来声明的

而我们可能在 Java 中定义一个简单的类，如下所示:

```
public class Foo {
  private String a;
  private int b;
  // getters, setters, constructors, etc
}
```

我们通常会在一行中定义一个 Scala 类，就像这样:

```
**class** Foo(a: String, b: Int) { }
```

括号是可选的，除非需要声明额外的成员、方法等:

```
**class** Foo(a: String, b: Int)
```

## 案例类代表专门的数据类

Java 14 引入了记录的概念。如果您熟悉记录，那么您将很快熟悉案例类。否则，case 类就是特殊的数据类。它们的语法类似于常规类:

```
**case class** Foo(a: String, b: Int)
```

然而，编译器会自动生成数据类所需的标准方法，如`equals()`、`hashCode()`和`toString()`方法、getters 等。

它还用`apply()`和`unapply()`方法生成一个“伴随对象”(我们稍后会讨论这些)。

## 泛型由方括号(`[]`)表示，而不是尖括号(`<>`)

在 Java 中，我们用尖括号来表示泛型类型:
`List<String> list;`

在 Scala 中，方括号用来表示泛型类型:
`List[String] list`

## 字符串插值是通过以“s”为前缀的字符串来完成的

在 Java 中，为了快速连接字符串，我们使用加号(+):
`System.out.println("Hello, "+ firstName + " "+ lastName + "!");`

在 Scala 中，我们通过在字符串前面加上 *s* 来执行字符串插值，并通过使用美元符号( *$* ):
`println(s"Hello, $firstName $lastName!")`来引用变量

若要访问嵌套值或表达式，请使用大括号:

```
println(s"Hello, ${name.first} ${name.middle.getOrElse("")} ${name.last}!")
```

## 匿名函数看起来类似于 Java lambdas，但是带有= >符号

Java 在构造 lambda 函数时提供了语法糖:
`list.foreach(item -> System.out.println("* " + item));`

在 Scala 中，我们在匿名函数中使用了`=>`符号:
`list.foreach(item => println(s"* $item"))`

## 下划线充当占位符

Scala 使用下划线作为占位符。您会在一些不同的用例中看到下划线，包括:

***匿名函数:***
而不是在匿名函数中声明变量:
`list.foreach(item => println(item))`

我们可以省略*项*变量，用下划线代替:
`list.foreach(println(_))`

注意，与 Java 类似，我们可以通过提供对`println`函数的引用来简化上面的行(该函数将隐式地将当前变量作为参数):
`list.foreach(println)`

***模式匹配:***
当我们执行模式匹配时，我们试图将一个实例与其类型进行匹配。假设我们有这样一个案例类:

```
**case class** Dog(name: String) extends Animal
```

如果我们有一个动物的实例*和*，我们可以尝试确定它是否是一只有特定名字的狗:

```
a **match** {
  case Dog(name) => println(s"Found a dog named $name");
  ...
}
```

或者，我们可能不关心狗的名字。`Dog`的构造函数仍然接受一个参数，所以我们可以简单地使用下划线作为占位符:

```
a **match** {
  case Dog(_) => println(s"Found some dog");
  ...
}
```

同样，我们可能想知道我们是否找到了一个`Dog`，或者除了`Dog`之外的任何东西。我们再次使用下划线:

```
a **match** {
  case Dog(_) => println(s"Found a dog");
  case _ => println("Found something other than a dog")
}
```

## 伴随对象而不是静态对象

Java 有*静态*关键字，它描述绑定到特定*类*的变量和方法，而不是一个对象的实例。

```
public class UserManager {
  public static void save(User u) {
    //...
  }
}// we can invoke that static method via:UserManager.save(new User("Pat"));
```

Scala 不提供静态关键字*。然而，使用 Scala，我们可以创建对象，这些对象有效地提供了相同的东西:*

```
object User {
  def save(u: User) = {
    // ...
  }
}// we can invoke that object's method via:User.save(new User("Pat"))
```

而且，Scala 允许我们创建“伴侣对象”；也就是说，*对象*对应于同名的*类*:

```
class User(name: String) {
}// This is the companion object to the User class
object User {
  def apply() {
    new User("Noname")
  }
  def apply(name: String) = {
    new User(name)
  }
  def unapply(u: User) = {
    Some(u.name)
  }
  def save(u: User) = {
    // ...
  }
}
```

虽然我们可以在类的伴随对象中定义任何函数，但是`apply()`和`unapply()`方法有特殊的含义。

如果我们要调用对象的名称，后跟带有任意数量参数的括号，那么对象中相应的`apply(…)`方法将被调用。

在上面的例子中，调用`User()`会导致`User.apply()`被调用；调用`User("Chris")`实际上会调用`User.apply("Chris")`。通常，构造并返回伴生类的一个实例。以这种方式，一个伴随对象的`apply()`方法可以——并且通常被——用作工厂方法。

因此，我们通常会看到这样创建的对象:

```
val u = User("Suzie")
```

而不是这个:

```
val u = new User("Suzie")
```

另一方面，`unapply()`用作解构者。每当需要一个类的单个组件时(比如在模式匹配期间)，它的`unapply()`方法就会在后台被调用。

# 一个全面的例子

让我们根据刚才的内容来分析一段 Scala 代码:

```
**case class** Point(x: Int, y: Int)  // 1

**trait** Drawable {  // 2
  **def** draw(): Unit  // 3
}

**abstract class** Shape(**val** center: Point) {  // 4
  **def** area(): Double 
}

**case class** Circle(**override val** center: Point, radius: Int) **extends** Shape(center) **with** Drawable {  // 5
  **override def** draw(): Unit = { // 6
    *println*(**s"⊙ "**)
  }
  **override def** area() = Math.*PI* * (radius * radius) 
  **def** growBy(add: Int) = *Circle*(center, radius + add)  // 7
}

**case class** Square(**override val** center: Point, side: Int) **extends** Shape(center) **with** Drawable {
  **override def** draw(): Unit = {
    *println*(**"◼︎"**)
  }
  **override def** area() = Math.*PI* * (side * side)
}

**object Canvas** {  // 8

  **def** main(args: Array[String]): Unit = {  // 9
    **val** circle1 = Circle(*Point*(50, 25), radius = 10)  // 10
    **val** circle2 = circle1.growBy(5)
    **val** square = Square(*Point*(100, 150), side = 30)

    **val** l: List[Drawable] = *List*(circle1, circle2, square)  // 11
    l.foreach(_.draw())  // 12

    l.map(d => d **match** {  // 13
      **case** Circle(_, r) => **s"Circle of radius $**r**"** // 14 **case** Square(_, s) => **s"Square of side length $**s**"
      case** _ => **"Unknown shape"** // 15}).foreach(*println*)  // 16
  }

}
```

1.  我们定义一个案例类，`Point`。编译器在幕后为我们创建了一堆样板文件，包括一个`Point`伴生类。
2.  我们定义一个特质，`Drawable`，用…
3.  …一个隐式抽象方法，`draw()`。由于该方法是抽象的，我们将返回类型指定为`Unit`(这意味着不会返回任何内容)
4.  我们定义了一个抽象类`Shape`，它声明了一个`val`、`center`和一个隐式抽象方法`area()`
5.  我们定义了一个 case 类`Circle`，它扩展了`Shape`类和`Drawable`特征。
6.  我们使用`override`关键字来实现父类`Shape`的抽象`draw()`方法。这里，我们选择将方法体放在花括号中。
7.  然后我们定义另外两个方法:从父类`Shape`覆盖的`area()`方法，以及返回新`Circle`实例的新`growBy()`方法。在这两个基础中，我们从方法体中省略了花括号。
8.  我们创建一个`Canvas`对象。它的所有功能都可以认为类似于 Java 的静态方法。
9.  我们可以定义一个`main`方法，就像在 Java 中一样。在一个对象中，这个方法是静态的，可以像`Canvas.*main*(*Array*())`一样被调用。
10.  我们使用调用`Circle`伴随对象的`apply()`方法的语法糖创建一个`Circle`实例。回想一下，`Circle`伴随对象及其`apply()`方法是由编译器生成的，因为`Circle`是一个 case 类。
11.  我们创建了一个`Drawable`的`List`，用方括号表示`List`的泛型类型。
12.  我们使用`List`的`foreach`方法来迭代它的`Drawable`方法，调用各自的`draw()`方法。为了简洁起见，我们使用下划线字符来表示当前的`Drawable`。
13.  然后我们使用`List`的`map`方法将每个`Drawable`转换成一个`String`值。
14.  我们使用模式匹配来确定我们正在迭代的`Drawable`的具体类型。在这一行中，我们寻找一个具有任何*中心*值的`Circle`(由下划线字符表示)，并使用 *r* 变量捕获*半径*值。然后我们使用字符串插值来创建一个包含那个 *r* 值的`String`。当匹配下面一行中的`Square`时，我们做类似的事情。
15.  我们通过使用下划线字符来捕获匹配既不是`Circle`也不是`Square`的`Drawable`的情况。
16.  然后我们在`Strings`的结果`List`上调用`foreach`，并引用`println()`函数，该函数被当前正在迭代的`String`隐式调用。

觉得这个故事有用？想多读点？只需[在这里订阅](https://dt-23597.medium.com/subscribe)就可以将我的最新故事直接发送到你的收件箱。

你也可以支持我和我的写作——并获得无限数量的故事——通过今天[成为媒体会员](https://dt-23597.medium.com/membership)。