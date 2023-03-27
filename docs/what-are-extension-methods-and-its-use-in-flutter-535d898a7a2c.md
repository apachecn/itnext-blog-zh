# 什么是延伸方法及其在颤振中的应用？

> 原文：<https://itnext.io/what-are-extension-methods-and-its-use-in-flutter-535d898a7a2c?source=collection_archive---------3----------------------->

这篇文章将帮助你理解什么是扩展方法和它的用途。

## 扩展函数可以添加到已经编译好的类中。

因此，如果你想扩展一个已经添加到你的应用程序中的库，你可以利用扩展方法。

# 观看视频教程

让我们来看一个场景，我们想要大写一个字符串的第一个字母，我们通常在一些实用函数的帮助下做到这一点。效用函数看起来像这样

```
class Utils {
  static String firstLetterToUpperCase(String string) {
    if (null != string) {
      return string[0].toUpperCase() + string.substring(1);
    }
    return null;
  }
}
```

现在让我们看看如何在*扩展函数*的帮助下实现上述功能。

这里我们将编写一个*字符串*类的扩展。

```
extension ExtendedString on String {
  // int
  get firstLetterToUpperCase {
    if (null != this) {
      return this[0].toUpperCase() + this.substring(1);
    }
    return null;
  }
}
```

这里的“this”表示调用此方法的字符串。

**例如**

```
Text('hello world'.firstLetterToUpperCase()),
```

# 扩展小部件

我们已经扩展了“String”类，现在让我们看看如何扩展小部件。

让我们用下面的代码创建一个文本小部件

```
Container(
    padding: const EdgeInsets.all(16),
    margin: const EdgeInsets.all(16),
    color: Colors.green,
    child: Text(
        'Hello World',
        style: TextStyle(
        fontSize: 50.0,
        color: Colors.white,
        ),
    ),
),
```

在扩展函数的帮助下，我们可以像这样重写上面的代码…

```
extension ExtendedText on Text {
  //
  Container addBox() {
    return Container(
      padding: const EdgeInsets.all(16),
      margin: const EdgeInsets.all(16),
      color: Colors.green,
      child: this,
    );
}Text setBigWhiteText() {
    if (this is Text) {
      Text t = this;
      return Text(
        t.data,
        style: TextStyle(
          fontSize: 50.0,
          color: Colors.white,
        ),
      );
    }
    return null;
  }
}
```

我们将像下面这样使用上面的函数

```
Text('Hello World').setBigWhiteText().addBox(),
```

请记住，因为我们已经在文本上扩展了，所以这个函数不能应用于其他小部件。

为了使它适用于其他小部件，我们必须像这样修改代码

```
extension ExtendedText on Widget
```

ExtendedText name 可以用来导入其他类的扩展，就像这样

```
import 'your_class.dart' show ExtendedText;
```

如果您不想导入扩展，那么使用

```
import 'your_class.dart' hide ExtendedText;
```

还有一件事，如果你在 *ExtendedText* 前面加上前缀“_”，那么这个扩展将成为声明它的类的私有扩展。

你可以看我的 [*视频 tutoria*](https://youtu.be/2zCBn1m-6Gc) *l* 看看上面的一切都在行动。

更多教程请访问我的 [*博客*](https://www.coderzheaven.com/) 。

如果你觉得这篇文章有用，请鼓掌。

感谢阅读。