# 理解 JavaScript 闭包的初学者指南

> 原文：<https://itnext.io/a-beginners-guide-to-understanding-javascript-closures-b5eb57fdbeb4?source=collection_archive---------4----------------------->

![](img/ba0fbc1c27e01249b18e6538b846d4e5.png)

闭包是大多数 JavaScript 访谈中会出现的话题之一。刚刚起步的开发人员通常害怕它们(我知道我曾经害怕过)，但是没有理由害怕。就像递归一样，闭包可能会被过度解释，这让它们听起来很复杂，但在本质上，它们再简单不过了。我会用 6 行写一个来证明。

# 一个非常基本的结束

```
function outside() { 
  let x = 2
  return function inside(y) {
    return x * y
  } 
}
```

嘣。这是一个简短的例子，因为闭包只是:

> 一个函数内部的另一个函数:1)使用其父作用域中的变量，2)向外界公开。

“暴露”部分是我们使用闭包的一个重要原因。通过只公开函数，而不公开它使用的变量，我们实质上使它们成为私有的。但是在我们讨论闭包的用途之前，让我们先讨论一下如何创建一个闭包。

# 分解它

在这两个函数中，让我们先看看`outside`。在这个函数的范围内，会发生两件事:我们定义一个变量`x`为 2，我们定义一个名为`inside`的函数，然后返回*定义*。返回它是“奇怪”的部分，因为我们通常不返回函数定义，我们只是调用它们。如果我们调用`outside()`，你可以看到`inside`的定义就是我们得到的:

```
outside()             
//  *f* inside(y) {     
//    return x * y 
//  }
```

你可能以前见过类似的东西；函数定义是当我们忘记用`()`调用函数时的返回值:

```
function example() {
  return "My example value" 
}example
// *f* example() {
//    return "My example value"
// }

// What I MEANT to type was: 
example()
// My example value
```

因此，如果返回一个函数的计算值或其定义之间的唯一区别是`()`，如果我们添加另一个`()`，会发生什么？

```
outside()(4)
// 8
```

我们现在实际上调用了`inside`函数。返回一个函数定义，然后我们简单地调用这个返回值。下面是一些伪代码来说明这种关系:

```
// if: 
outside() is: function inside(y) { 
                return x * y
              } // then that means:
outside()(5) is: inside(5)
```

那个双`()`看起来很奇怪，所以通常人们会这样做:

```
let myClosure = outside() 
myClosure(5)
// 10// remember myClosure is outside(), 
// so then this is still the same thing:

myClosure(5) === outside()(5)
// true 
```

# x 呢？

闭包最好的部分是`x`，因为我们创建了一个变量，程序的其他部分无法触及，但我们仍然可以使用。这就是闭包的强大之处:它让我们创建除了闭包本身之外不能被任何函数修改或访问的值。

这样做的原因是因为作用域可以访问其父级中的值，但不能访问其子级中的值。`x`是在`inside`的父作用域中定义的，所以`inside`仍然可以访问它并乘以`y`。然而，唯一暴露于全局范围的是`inside`、*而不是*变量`x`的定义。这意味着在我们的整个程序中，我们与`x`交互的唯一方式是使用闭包。看看这段**没有**使用闭包的代码:

```
let x = 2 // global scope
function inside(y) {
  return x * y  // reaching up to global scope
}
inside(4)
// 8// but we can change the 'x'
// variable in the global scope and 
// alter inside()'s behaviorx = 5
inside(4)
// 20
```

让我们试着做同样的事情，但是这次用闭包来保护我们的`x`:

```
function outside() {          
  let x = 2                    
  return function inside(y) {  
    return x * y               
  }                    
}let myClosure = outside()myClosure(5)                   
// 10x = 4
// this is not the same 'x' variable: 

myClosure(5)
// 10      // notice how it's still 10 
```

`x`是在我们的`outside`函数中定义的，所以全局范围内的任何东西都无法深入其中并改变它。当我们试图将`x`重新赋值为 4 时，这实际上是一个不同的`x`变量，它现在单独存在于我们的全局范围内，根本不影响`myClosure`的工作方式。

正如你所看到的，闭包的基础很简单:它们只依赖于作用域能够向上延伸，而不能向下延伸。现在你已经知道了基础知识，试着学习一下[如何立即调用函数表达式(iife)](https://blog.mgechev.com/2012/08/29/self-invoking-functions-in-javascript-or-immediately-invoked-function-expression/)使用闭包来创建一些简单而有用的东西:[一个不可触及的计数器](https://www.w3schools.com/js/js_function_closures.asp)。

大家编码快乐，

迈克