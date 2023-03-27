# Typescript 中的安全属性访问器

> 原文：<https://itnext.io/safe-property-accessors-in-typescript-53fd384b83f3?source=collection_archive---------7----------------------->

![](img/c2681d15e9a4ae9c434c2400c30c6a80.png)

魔弦很碍眼。我尤其不喜欢在属性访问器的对象括号符号中看到它们，例如

```
abstract class Main {
  run() {
    const myObject = { a: 2, b: 4 };
    console.log(`Dot Notation (good): ${myObject.a}`);
    console.log(`Bracket Notation (bad): ${myObject['a']}`);
  }
}Main.run();
```

我可以证明使用对象括号符号的唯一用例是用于动态属性访问，但是我讨厌使用这样一个神奇的字符串来访问这样一个属性。没有办法保证你的属性会在源对象上。上面的代码块有两个主要问题:

1.  神奇的字符串在这里是不必要的，因为它使代码更难维护，并且不是自文档化的。
2.  不能保证源对象上存在被访问的属性。

接着是[“鲍勃呢？”接近](https://www.youtube.com/watch?v=fA7LGqwjhYs)，让我们一步一步地开始重构神奇的字符串:

```
abstract class Constants {
    readonly MyProperty: string = 'a';
}abstract class Main {
  run() {
    const myObject = { a: 2, b: 4 };
    console.log(`Better: ${myObject[Constants.MyProperty]}`);
  }
}
Main.run();
```

不再使用神奇的字符串，上面的例子是一个稍微好一点的选择。通过创建单一的事实来源，这是一个更易于维护的解决方案，并且有助于在自文档化代码的道路上走得更远。虽然我们解决了魔术字符串的眼中钉，但我们仍然没有解决我们的“担保财产”问题。

这比婴儿步稍大一点，我们来分解一下:

你应该很熟悉——简单的一个基本类。这其中的`Constants`部分是我在大型 web 应用程序中更好地表示常量的方式。通过创建一个命名空间来封装所有常量，创建一些类来组织字符串，并将 static + readonly 字符串作为常量，您可以拥有一个三级组织模式。然而，请注意`MyProperty`和`MyUnsafeProperty`之间的一个细微差别。一个使用字符串类型，一个使用`keyof`将其强类型化为`MyClass`的一个键。

`keyof`是一个 Typescript 类型的查询，它确保一个字符串作为该对象的属性存在(注意，这也适用于父子关系)。如果`MyProperty`的值在`MyClass`上不作为字段存在，Typescript 会给你一个编译器错误，让你知道你的值在`MyClass`上不作为字段存在。为了更加安全，我们将`_safeNameOf`的第二个参数限制为`T`类型的键。为了防止您已经设法抑制了这个 Typescript 编译错误，我们仍然在`_safeNameOf`中添加了一个额外的检查，以确保这个键确实存在于源代码中。

这是使括号标记属性访问器类型安全的冗长方法，但在一些针对一般对象的旧 Javascript 变通办法中，这是确保类型安全的好方法。