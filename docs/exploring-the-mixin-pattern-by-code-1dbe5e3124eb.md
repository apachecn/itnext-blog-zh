# 探索 Mixin 模式→按代码

> 原文：<https://itnext.io/exploring-the-mixin-pattern-by-code-1dbe5e3124eb?source=collection_archive---------5----------------------->

![](img/199cc1b2eb9c58ebb6c8714c298e542b.png)

mixin 的概念已经在面向对象的语言中使用了很多年，比如 JAVA 和 C#，流行的 css 预处理程序 Sass 和 vue.js 框架也通过让开发人员使用一个显式的 API 来支持这种模式，那么 Mixin 是什么呢？我想看一些例子！

根据维基百科的定义:

> Mixin 编程是[软件开发](https://en.wikipedia.org/wiki/Software_development)的一种风格，其中功能单元在一个类中创建，然后与其他类混合。

换句话说:Mixin 让我们有了一种行为构成。

当我需要一个导航菜单组件的引用时，我真的对 mixin 实现产生了怀疑。

M [aterial2](https://github.com/angular/material2) 通常是我设计 infra 组件时的第一个参考，我在查看: [MatTabNav](https://github.com/angular/material2/blob/4f755cf10c383b7e26190d5d1f36caa2a08b137f/src/lib/tabs/tab-nav-bar/tab-nav-bar.ts#L75) 类代码时注意到了 **_MatTabNavMixinBase** 类，所以我在探索该主题时继续深入研究代码。

让我们继续阅读维基百科的定义:

> mixin 类充当父类，包含所需的功能。一个[子类](https://en.wikipedia.org/wiki/Subclass_(computer_science))可以继承或简单地重用这个功能，但不是作为一种专门化的手段。通常，mixin 会将期望的功能导出到一个[子类](https://en.wikipedia.org/wiki/Subclass_(computer_science))，而不会创建一个严格的、单一的“是一个”关系。

我不能真正完全理解一个概念，直到我开始编码，所以下面是我对 mixin 的实现，很大程度上受有角材料的启发:

让我们看看我的实现是否符合维基百科的定义:

下面的函数创建了 **mixin 类**:

```
**const** *mixinRedAlert* = <T **extends** Constructor<**any**>>(base: T) : T  => {
   **return class extends** base {

      **constructor**(...args: **any**[]) {
         **super**(...args);
      }

      **public** addRedClassToEl(element, c) {
         element.**classList**.add(c)
      }
      **public** removeRedClassToEl(element, c) {
         element.**classList**.remove(c)
      }
   }
}
```

正如我们所看到的，返回的类将包含混合功能。

接下来，我将使用***mixinRedAlert***函数 util 来创建 mixin 类，该类也将具有 **classB** 功能。

```
**class** classB {
   **constructor**() {}

   **private thing** = **'something'**;

   **public** doSomething() {
      **return this**.**thing**;
   }
}**const** MixedSomething: GeneralCtor & **typeof** classB = mixinRedAlert(classB);
```

接下来，我将使用 MixedSomething 类作为 **classA** 父类，我们可以看到它重用了 **classB** doSomething 功能，而实际上并不是 **classB 的类型！**

```
**class** classA **extends** MixedSomething {
   **constructor**() {
      **super**();
   }

   **public** useMixinFn() {
      **return this**.doSomething();
   }
}
```

正如我们所见，classA 实例可以访问所有涉及类的公共方法:

```
**const** instance = **new** classA;

instance.addRedClassToEl(firstDiv, **'alert'**);
instance.addRedClassToEl(secDiv, **'border'**);
instance.doSomething(); instance.useMixinFn();
```

一般来说，理解与 Javascript 相关的两个事实是很重要的:

1.  class 关键字只是语法上的修饰，本质上它只是一个构造函数和原型链接的使用。
2.  Mixin 类不会持有 classB 功能的实际“副本”,它持有指向其父类(原型)的链接，父类持有对其父类的引用，依此类推，直到指向 null。(继续读取 [mdn](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Inheritance_and_the_prototype_chain) 上的原型链)。

此外，这里有一个使用 Angular 的 mixin 示例，它只是用于概念演示——我不建议在实际应用中使用它(UI 操作应该通过指令来完成..)但是继续玩代码活模式:)。

因此，在这篇文章的结尾，我想说:保持务实，每项工作都有合适的工具，想出最简单的可重用解决方案，并学习更多的技术/模式，以便将来你可以“拿出来”)。

参考一篇我非常喜欢的关于这个话题的帖子:

[](https://medium.com/javascript-scene/functional-mixins-composing-software-ffb66d5e731c) [## 功能混合蛋白

### 编写软件

medium.com](https://medium.com/javascript-scene/functional-mixins-composing-software-ffb66d5e731c) 

干杯，

利伦。