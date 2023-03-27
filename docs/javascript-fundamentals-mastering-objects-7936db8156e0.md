# JavaScript 基础:掌握对象

> 原文：<https://itnext.io/javascript-fundamentals-mastering-objects-7936db8156e0?source=collection_archive---------0----------------------->

![](img/3c6de30143b3219b45ad5cd00a0fccd9.png)

JavaScript 中的*对象*用于存储“键:值”对格式的数据集合。在一个对象中，我们可以有任意数量的变量和/或函数，它们被称为对象属性和方法。

🤓*想与 web dev 保持同步吗？*
🚀想要将最新消息直接发送到您的收件箱吗？
🎉加入一个不断壮大的设计师&开发者社区！

**在这里订阅我的简讯→**[**https://ease out . EO . page**](https://easeout.eo.page/)

# 创建对象

让我们来看一个例子！为了将变量 **car** 初始化为对象**，**我们使用花括号 **{}** :

```
var car = {};
```

我们现在有了一个空对象，只需输入变量名，就可以通过开发人员工具控制台访问它:

```
car// {} [object]
```

空对象并不那么有用，所以让我们用一些数据来更新它:

```
var car = {
  name: 'Tesla',
  model: 'Model 3',
  weight: 1700,
  extras: ['heated seats', 'wood decor', 'tinted glass'],
  details: function() {
    alert('This ' + this.name + ' is a ' + this.model + ' it weighs ' + this.weight + 'kg and includes the following extras: ' + this.extras[0] + ', ' + this.extras[1] + ' and ' + this.extras[2] + '.');
  },
  display: function() {
    alert('This car is a ' + this.name + '.');
  }
};
```

让我们在开发人员工具控制台中访问这些数据:

```
car.name         // Tesla
car.model        // Model 3
car.weight       // 1700
car.extras[1]    // wood decor
car.details()    // This Tesla is a Model 3 it weighs 1700kg and   includes the following extras: heated seats, wood decor and tinted glass.
car.display()    // This car is a Tesla.
```

如您所见，每个名称/值对都必须用逗号分隔，每个名称/值对都用冒号分隔。语法将始终遵循以下模式:

```
var objectName = {
  member1Name: member1Value,
  member2Name: member2Value,
  member3Name: member3Value
};
```

对象成员的值几乎可以是任何值——在我们的 car 对象中，我们有两个字符串、一个数字、一个数组和两个函数。前四项是数据项，称为对象的**属性**。最后两项是允许对象对数据做一些事情的函数，称为对象的**方法**。

这种对象被称为**对象字面量**——我们在创建它的时候，已经逐字写出了对象内容。这是与从类实例化的对象相比较的，我们将在后面看到。

# 点符号

上面，您已经看到了使用**点符号**访问对象的属性和方法。对象名`car`充当**名称空间**——需要首先输入它才能访问对象中的任何内容。然后写一个点，后面跟着要访问的项，可以是简单属性的名称、数组属性的项或对对象方法之一的调用，例如:

```
car.name
car.extras[1]
car.details() 
```

# 删除属性

我们可以使用删除操作符来删除属性，如下所示:

```
car.model
// Tesla 3**delete** car.modelcar.model
// undefined
```

# 方括号

例如，如果我们的对象中有一个多词属性，比如:

```
var user = {
  name: "Sam",
  age: 32,
  "likes potatoes": true  // multiword property name must be quoted
};
```

我们无法使用点符号访问多单词属性:

```
user.likes potatoes     // syntax error!
```

这是因为点要求密钥是有效的变量标识符。那就是:没有空格等限制。

另一种方法是使用方括号，它适用于任何字符串:

```
let user = {};

// set
user["likes potatoes"] = true;

// get
alert(user["likes potatoes"]); // true

// delete
delete user["likes potatoes"];
```

# 更新对象成员

我们可以通过简单地声明您想要用新值设置的属性来更新对象中的值，如下所示:

```
user.age          // 32
user.age = 33     // 33
user.age          // 33
```

您还可以创建对象的全新成员。例如:

```
user.surname = 'Smithessson';// user
{name: "Sam", age: 33, likes potatoes: true, surname: "Smithessson"}
```

# “这个”是什么？

你可能已经注意到在我们前面的例子中使用了单词“this”。请参见以下内容:

```
display: function() {
  alert('This car is a ' + **this.**name + '.');
}
```

`this`关键字指的是正在编写代码的当前对象——所以在这种情况下`this`相当于`car`。

为什么不直接写`car`呢？最佳实践是编写构造良好的面向对象代码，在这样做的时候，`this`的使用非常有用。它将确保当成员的上下文发生变化时使用正确的值(例如，两个不同的`car`对象实例可能有不同的名称，但在显示它们自己的信息时会希望使用它们自己的名称)。

例如:

```
var car1 = {
  name: 'Tesla',
  display: function() {
    alert('This car is a ' + **this.**name + '.');
  }
}var car2 = {
  name: 'Toyota',
  display: function() {
    alert('This car is a ' + **this.**name + '.');
  }
}
```

在这种情况下，`car1.display()`会输出“这辆车是特斯拉。”并且`car2.display()`会输出“这辆车是丰田。”，即使这两种情况下方法的代码完全相同。

因为`this`等于代码所在的对象，所以当您动态生成对象(例如使用构造函数)时`this`变得非常有用，这超出了本文的范围！

# 结论

就是这样！现在，您应该对如何使用 JavaScript 中的对象有了很好的了解——包括创建自己的简单对象，以及访问和操作对象属性。您还将开始看到对象作为存储相关数据和功能的结构是多么有用。

例如，如果您试图将我们的`car`对象中的所有属性和方法作为单独的变量和函数来跟踪，这将是非常低效的，并且我们将冒与具有相同名称的其他变量和函数冲突的风险。对象让我们将信息安全地锁在它们自己的包中。

我希望这篇文章对你有用！你可以在 Medium 上[关注我](https://medium.com/@timothyrobards)。我也上了 [Twitter](https://twitter.com/easeoutco) 。欢迎在下面的评论中留下任何问题。我很乐意帮忙！

***你准备好让你的 JavaScript 技能更上一层楼了吗？*** *今天就开始用我的新电子书吧！无论你是想学习你的第一行代码，还是想扩展你的知识面并真正学习基础知识..*[*JavaScript 掌握完全指南*](https://gum.co/mastering-javascript) *带你从零到英雄！*

![](img/dde515044536421c6c999650977f80c4.png)

*现已上市！👉*[https://gum.co/mastering-javascript](https://gum.co/mastering-javascript)

# 关于我的一点点..

嘿，我是提姆！👋我是一名开发人员、技术作家和作家。如果你想看我所有的教程，可以在我的个人博客上找到。

我目前正在构建我的[自由职业者完整指南](http://www.easeout.co/freelance)。坏消息是它还不可用！但是如果是你感兴趣的东西，你可以[注册，当它可用时会通知](https://easeout.eo.page/news)👍

感谢阅读🎉