# 深入研究 Dart 构造函数

> 原文：<https://itnext.io/deep-dive-in-dart-constructors-51e4c006fb8f?source=collection_archive---------0----------------------->

![](img/bcdca16946154c5f3f3ad740151cbe33.png)

## 你会发现所有关于构造函数的东西

## 首先

# 什么是构造函数？

> 构造函数是一种用于初始化对象的特殊方法。当创建类的对象时，调用构造函数。

## 基本上

```
final ehe = **MyClass();** // Creates an instance**class MyClass** **{**
  **MyClass();** // Fires immediately when created (this guy is cons.)
**}**
```

## 构造函数中只有一个规则！

也就是；

> 与其类名同名！！

## 好的，我们知道了！但是..。

# 我们到底有什么样的构造函数类型？

## 默认构造函数—类()

```
// Default Constructor
// Does nothing specialclass User {
  String name = 'ehe';
  **User();**
}///////////////////
// Constructor with parameters
// Gets parameters and assigns to them to their variables class User {
  String name;
  **User(this.name);**
}///////////////////
// Constructor with the initial method
// Fires the codes immediatelyclass User {
  String name;
  User(this.name) **{
    // do some magic
  }**
}/////////////////
// Constructor with assertion
// Asserts some rules to parametersclass User {
  String name;
  User(this.name) **: assert(name.length > 3);**
}////////////////
// Constructor with initializer
// You can also initialize the values with customizations tho!class User {
  static String uppercase(String e) => e.toUpperCase();
  String name;
  User(name) **: name = yell(name);** static String yell(String e) => e.toUpperCase();
}/////////////////////
// Constructor with super()
// you can override the values it extends**abstract** class **Person {**
  String id;
  **Person(this.id);**
}class User **extends Person {**
  String name;
  User(this.name,**String id**) **: super(id);**
}/////////////////////
// Constructor with this()
// you can redirect the values as wellclass User {
  String name;
  int salary; **User(this.name, this.salary);** **User.worker(String name) : this(name, 10);
  User.boss(String name) : this(name, 9999999);**
}
```

## 私有构造函数-类。_()

你可以使用 _ 创建私有构造函数，但是这样做有什么好处呢？

我们来看一个例子！

```
class Print {
  static void log(String message) => print(message);
}Print.log('ehe');// You want to write an util like this but there is a problem with that, because you can also create an instance which is something we don't wantPrint(); // it's absolutely unnecassary in this case// How to prevent that? Answer is private constructors!class Print {
  **Print._(); // This one will prevent creating instance**
  static void log(String message) => print(message);
}Print(); // This will give compile time error nowYour instance is safe now!
```

所以基本上你可以阻止创建一个实例！

## 命名构造函数 Class.named()

您可以在一个`class`中创建不同类型的实例

比如说；

```
class User {
  String name;
  int salary;
  **User.worker**(this.name) : **salary = 10**;
  **User.boss**(this.name) : **salary = 99999999**;
}
```

## 私有命名构造函数-类。_ 已命名()

你可以很容易地清洗你的实例！

```
class User {
  String name;
  int salary;
  **User.worker**(this.name) : **salary = 10**;
  **User.boss**(this.name) : **salary = 99999999**;
  **User._mafia**(this.name) : **salary = 9999999999999**;
}
```

玩笑归玩笑，这还是很有帮助的。

例如，您可以使用私有构造函数创建单例！

```
class User {
  **User._privateConstructor();**
  **static final User instance = User._privateConstructor();** 
}
```

## 注意

在一些项目中可以看到`_internal`键。没什么特别的。 **_internal** 构造只是一个名字，通常被赋予类的私有构造函数(名字不一定是**)。_internal** 你可以使用任何**类创建私有构造函数。_someName** 建设)。

## Const 构造函数— const 类()

你可以使用`const constructor!`让你的类不可变

> const 构造函数是一种优化！编译器使对象不可变，为所有的`Text('Hi!')`对象分配相同的内存部分。—弗兰克·特拉塞

```
**const** user = User('ehe');class User {
  final String name;
  **const** User(this.name);
}
```

## 工厂构造函数—工厂类 class()

我们说过构造函数不允许返回。你猜怎么着？

**工厂构造者**可以！

还有工厂构造函数能做什么？

您根本不需要创建新的实例！您可以调用另一个构造函数或子类，甚至可以从缓存中返回一个实例！！

最后，对工厂的小警告！

您不能调用超类构造函数(`super()`)

## 简单的例子

```
class User {
  final String name;
  User(this.name); **factory User.fromJson**(Map<String, dynamic> json) {
    **return** **User(json["name"]);**
  }
}// Singleton Example
class User {
  User._internal(); static final User _singleton = Singleton._internal();

  **factory User() => _singleton;**
}
```

# 参考

[](https://flutterigniter.com/deconstructing-dart-constructors/) [## 解构 Dart 构造函数

### 曾经对 Dart 构造函数中的神秘语法感到困惑吗？冒号，命名参数，断言，工厂…读这个…

flutterigniter.com](https://flutterigniter.com/deconstructing-dart-constructors/) [](https://dash-overflow.net/articles/factory/) [## “工厂构造函数”和“静态方法”的区别

### 如果您以前使用过 Dart，那么您可能听说过 factory 关键字。也有可能你已经看过…

dash-overflow.net](https://dash-overflow.net/articles/factory/) [](https://www.freecodecamp.org/news/constructors-in-dart/) [## Dart 中的构造函数——用例及示例

### 我们大多数人都熟悉构造函数的概念。它们允许我们创建类的不同实例…

www.freecodecamp.org](https://www.freecodecamp.org/news/constructors-in-dart/) [](https://stackoverflow.com/questions/52299304/dart-advantage-of-a-factory-constructor-identifier) [## 工厂构造器标识符的优势

### 工厂构造函数的用途之一是，我们可以决定创建哪个实例，在运行时移动所有的逻辑…

stackoverflow.com](https://stackoverflow.com/questions/52299304/dart-advantage-of-a-factory-constructor-identifier) [](https://dart.dev/guides/language/language-tour#factory-constructors) [## Dart 语言之旅

### 本页将向您展示如何使用 Dart 的每个主要功能，从变量和运算符到类和库，以及…

dart.dev](https://dart.dev/guides/language/language-tour#factory-constructors) 

## 感谢您的阅读！

我试图创造尽可能简单的例子。希望你喜欢。

如果你喜欢这篇文章，请点击👏按钮(你知道你可以升到 50 吗？)