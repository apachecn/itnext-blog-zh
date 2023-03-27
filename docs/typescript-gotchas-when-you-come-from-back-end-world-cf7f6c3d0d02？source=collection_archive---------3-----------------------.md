# 当你来自后端世界时

> 原文：<https://itnext.io/typescript-gotchas-when-you-come-from-back-end-world-cf7f6c3d0d02?source=collection_archive---------3----------------------->

## 以打字打的文件

Typescript 发展如此之快，以至于无法避免。无论是在 Angular、React、Vue 应用程序中使用，还是在后端使用，它都极大地增强了 Javascript。

![](img/097ee77725a14e56f4a05adf09409071.png)

[猎人哈利](https://unsplash.com/@hnhmarketing?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上的照片

它提供了许多来自 OOP(面向对象编程)的好处，但是它也有一些缺点，你可以在 Eric Elliott 的文章中找到:

[](https://medium.com/javascript-scene/the-typescript-tax-132ff4cb175b) [## 打字税

### 成本与收益分析

medium.com](https://medium.com/javascript-scene/the-typescript-tax-132ff4cb175b) 

两年多前我开始使用 Typescript。我想，和许多后端开发人员一样，我不想错过这个浪潮。

本文收集了 Java 或 C#等后端技术中使用的所有常见行为、数据结构和编码技巧。当然，我会尽量让这篇文章保持最新。请随意评论(评论、错误或提示)，我会提到你。

# 重载方法

重载一个方法意味着相同的名字影响到几个参数不同的函数。

这在 C#中相当容易:

```
public string SayHello() {
  return "Hello World!";
}public string SayHello(string firstName, string lastName) {
  return $"Hello {firstName} {lastName}!";
}
```

但是 Typescript 呢？这不是小事，我们必须检查参数:

```
sayHello(): string;
sayHello(firstName: string, lastName: string): string {
  if(firstName && lastName) { 
    return `Hello ${firstName} ${lastName}`;
  } else {
    return 'Hello World!';
  }
}
```

在提升阶段，扫描所有的函数名。由于函数参数与区分两种方法无关(由于 Javascript 中的动态类型)，所以不能使用它们。因此，我们需要检查参数(无论是否提供)及其类型。

**提示:**你可以通过`typeof`(原语)或者`instanceof`(复杂类型)关键字来检查参数类型。您也可以使用[型防护装置](https://www.typescriptlang.org/docs/handbook/advanced-types.html#user-defined-type-guards)。

# 当界面是空的时

事实上，当接口从 Typescript 转换到 Javascript 时，它们就消失了。什么都没留下。即使只有 void，它们也有很多用处:构造你的代码，扩展类，对象，…

在 C#中用 I 作为接口的前缀是一个常见的习惯，例如`IDisposable`。在 Typescript 中，这不是强制性的。

多亏了接口，你可以扩展类、对象和其他接口。关于…

# …扩展现有对象

有时，您只需要向现有对象添加方法。这是扩展该对象功能的一种简单方法。

在 C#中:

```
public static string AppendSmiley(this string message) {
  return $"{message} :-)";
}string msg = "Home, sweet home!".AppendSmiley();
// Now msg equals "Home, sweet home! :-)"
```

在 Typescript 中，我们只需要记住两个接口可以拥有相同的名称:

```
declare global {
  interface String {
    sayHello(): String;
  }
}
String.prototype.sayHello = function(): string {
  return 'Hello World!';
}
export{};
```

# 映射数据

在某些算法中关联数据更有效。您需要创建一个键值对。这样，您可以通过一个键确保唯一性，并改进一组数据中的查找。

在 C#中，字典 <tkey tvalue="">( [doc](https://docs.microsoft.com/fr-fr/dotnet/api/system.collections.generic.dictionary-2?view=netframework-4.8) )对象存在:</tkey>

```
var dico = new Dictionary<int, string>();
dico.Add(1, "One");
dico.Add(2, "Two");
dico.Add(3, "Three");
dico.ContainsKey(2); // true
```

Typescript 提供了一组实用程序类型，包括[记录](https://www.typescriptlang.org/docs/handbook/utility-types.html#recordkt)类型，其行为与 C#中的 Dictionary 完全相同:

```
// Extract from doc**interface** PageInfo {
    title: string;
}

**type** Page = 'home' | 'about' | 'contact';

**const** x: Record<Page, PageInfo> = {
    about: { title: 'about' },
    contact: { title: 'contact' },
    home: { title: 'home' },
}; 
```

您将在以下位置找到所有可用的实用程序类型:

 [## 实用程序类型

### 在上面的例子中，makeObject 的参数中的 methods 对象有一个上下文类型，包括 ThisType 和…

www.typescriptlang.org](https://www.typescriptlang.org/docs/handbook/utility-types.html) 

# 在文字类型中省略类型

最近，我不得不在我的应用程序中处理过滤器。其中一个标准是 readonly，然后我想从`Filter`类型中提取可编辑的键。

在这个要点中，我们有一个格式属性为 readonly 的`Filter`类。如果我们想让所有可编辑的过滤键在下面的方法中使用它们:`updateFilters(key: EditableFilters, newValue: unknown)`，我们需要“省略”format 属性。

的确，`keyof`会返回所有的属性。根据上下文，这个技巧非常有用，但它只在较新版本的 Typescript (3.5 以上)中有效。如果您想将其用于旧版本，可以使用变通方法:

`Pick<T, Exclude<keyof T, K>>`

正如开头所建议的，请随意发表评论，以使本文更好。你可以在我的个人资料页面找到其他关于 Typescript 的文章。