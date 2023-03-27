# 编写更轻、更快的 JavaScript 函数

> 原文：<https://itnext.io/writing-lighter-faster-functions-c631739af349?source=collection_archive---------3----------------------->

![](img/8c254d2615b6dbf6a2210817713a7e49.png)

[鲁道夫·马克斯](https://unsplash.com/@rodolfomarques?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上的“橙色纸杯蛋糕”

谈到 JavaScript 性能，实际上只有三种改进策略:

1.  少做
2.  少做一点
3.  快点做

本文主要关注第二个策略，减少做事的频率。

在下面的函数`isStooge`中，有一个在函数体`STOOGES`中创建的不变对象。

```
function isStooge(name) {
  const STOOGES = ['Larry', 'Curly', 'Moe'];
  return STOOGES.includes(name);
};isStooge('Moe');
isStooge('Joe');
isStooge('Larry');
isStooge('Barry');
isStooge('Curly');
isStooge('Burly');
```

虽然它是一个小对象，但每次函数运行时都会创建*。这意味着每次调用时，该函数都会为一个三项数组重新分配内存，使用一次，然后将其留给垃圾收集。这不是内存泄漏，但仍然是不必要的开销。对于静态值，最好是声明一次，每次需要时只引用一次。一个简单的方法是在函数之外作为一个全局变量。这通常是不好的建议。这里有一些更好的选择！*

## 在一次关闭中

变量应该在靠近使用点的地方声明，你不能比在闭包里声明得更靠近(但仍然在函数体之外)。变量将只被实例化一次，引用将在函数内部使用。

```
const isStooge = (function() {
  const STOOGES = ['Larry', 'Curly', 'Moe'];
  return (name) => STOOGES.includes(name);
})();isStooge('Moe');
isStooge('Joe');
isStooge('Larry');
isStooge('Barry');
isStooge('Curly');
isStooge('Burly');
```

## 在模块中

如果您可以拥有在闭包中声明变量的所有好处，而不使用复杂的语法，会怎么样？你可以！ES6 模块本质上为其中所有未导出的内容创建了一个闭包。即使您使用 Babel 而不是原生的 ES6 模块，您仍然会得到一个闭包。

```
// stooge.js
const STOOGES = ['Larry', 'Curly', 'Moe'];export function isStooge(name) {
  return STOOGES.includes(name);
};// stoogeChecker.js
import { isStooge } from './stooge';isStooge('Moe');
isStooge('Joe');
isStooge('Larry');
isStooge('Barry');
isStooge('Curly');
isStooge('Burly');
```

## 在函数的一个属性中

如果你的函数只被实例化一次(比如我们的`isStooge`函数)，那么你可以把变量附加到它上面，并在函数内部引用那个属性。这是可能的，因为函数在 JavaScript 中仍然是对象。

```
function isStooge(name) {
  return isStooge.STOOGES.includes(name);
}
isStooge.STOOGES = ['Larry', 'Curly', 'Moe'];isStooge('Moe');
isStooge('Joe');
isStooge('Larry');
isStooge('Barry');
isStooge('Curly');
isStooge('Burly');
```

如果函数本身将被实例化多次，您可以将变量附加到函数的`prototype`以防止它被重新实例化。

## 在类静态属性中

类语法允许声明`static`属性，这些属性可以通过`this`(如果函数也是`static`)或通过类名(下面的`StoogeChecker`)在类的函数中引用。

```
class StoogeChecker {
  static STOOGES = ['Larry', 'Curly', 'Moe']
  static isStooge(name) {
    return this.STOOGES.includes(name);
  }
}StoogeChecker.isStooge('Moe');
StoogeChecker.isStooge('Joe');
StoogeChecker.isStooge('Larry');
StoogeChecker.isStooge('Barry');
StoogeChecker.isStooge('Curly');
StoogeChecker.isStooge('Burly');
```

这个方法与前面的方法有很大的不同，因为变量和函数都是更大的`StoogeChecker`类对象的一部分。不过，在使用 React 组件时，这种模式确实很方便，这也是包含它的原因。还要注意，每次实例化类时，在`constructor`方法中创建的任何变量都将被重新创建。因此，最好将这些变量声明为`static`——盟友。

## 结论

声明不变变量最有用的模式是在 ES6 模块中。语法是如此容易理解和预测，它几乎是令人厌烦的。虽然严格遵循这种模式可能不会带来显著的性能提升，但这仍然是一个很好的实践。