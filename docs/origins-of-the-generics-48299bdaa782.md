# 泛型的起源

> 原文：<https://itnext.io/origins-of-the-generics-48299bdaa782?source=collection_archive---------4----------------------->

## 许多编程语言中包含的泛型实际上具有类似面向对象和函数式编程的思想。要解释泛型实际指的是什么，就要查基于数学的起源。

![](img/01ad2850bc8aedbf291b57fe4ef4f27a.png)

照片由[西格蒙德](https://unsplash.com/@sigmund?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄

G enerics 是一种编程类型，最初包含在 2004 年的 Java 编程语言中。它使用“类型”来表示编译时安全的对象。Oracle 暗示了使用泛型的三个主要动机。

## 编译时更强的类型👊

[类型安全是一种属性保证，保证特定指令中的数据在任何条件下都能正确定义和表现](https://medium.com/@thiago_silva/what-is-type-safety-ceb8f14ee6c3)。一旦定义了字符串类型，编程语言就不会改变字符串类型变量的行为。然而，类型不安全的编程语言(如 Javascript、PHP)在操作变量时没有额外的控制。

基本上，会发生两种类型检查。编译器在编译阶段对源代码进行分析。其中一些用它们的类型对变量赋值或函数执行进行验证。这种过程发生在支持静态类型检查的编译器中。[静态类型检查程序有一个快速的执行过程。](https://hackernoon.com/i-finally-understand-static-vs-dynamic-typing-and-you-will-too-ad0c2bd0acc7)因为在运行时没有进行额外的类型检查，但是它们不能确保程序是完全类型安全的。

在某些情况下，静态类型检查是不够的。我们来想一个案例。TypeScript 是基于静态类型检查的语言。typescript 编译器将代码转换成 Javascript 文件。Javascript 没有类型检查功能。最终，javascript 将允许任何类型的输入到任何类型的变量中，即使我们编写了一个类型脚本代码。我们需要运行时的控制。

动态类型检查使语言能够顺利地评估指令。当无效行为可能发生时，程序将抛出一个中断。动态类型的程序稍微慢一点，但是完全保证了类型安全。

## 还原铸造操作👇

bob 叔叔指出，必须避免使用某种编码方式来编写可靠而干净的代码。每一次铸造操作都是一个编码过程。它分散了开发人员对代码块主要动机的注意力。代码读者一直关注实际的业务，而不是实用的东西。

如果没有泛型，我们需要额外的类型转换操作符来确保编译时的类型安全，尤其是在集合类中。

但是，转换运算符在编译时不能完全确保类型安全。一些继承的情况可能会击败编译，但是 JVM 在运行时用 ClassCastException(动态类型检查)捕获。

Java 将类型称为“擦除”关键字，以在编译阶段保持强大的类型安全性，而不管任何运行时问题。

## 让开发人员能够将泛型编程变成⚡️

所有的可能性都应该通过在算法开发中进行大量的分析来揭示。一个算法有一个主要指向解面的作用域。许多分析可能会试图澄清范围。然而，一个好的算法可以用最少的假设来表达。通用编程范式鼓励做出更少的假设来定义算法。

有时候我们只是开始一个算法的具体实现。随着时间的推移，算法范围不会改变，但假设变得更先进。这是泛型编程的第二个主要动机，即在不损失效率的情况下，从我们的算法中获得尽可能多的通用性。

泛型编程是算法实现的基础，无需考虑类型或任何量词。使算法普遍执行，并保持语法纯净，没有重复。

# “仿制药”的现实

数学成分步入舞台。在数理逻辑中有一种形式叫做“量词”来表示类型。随着时间的推移，量词已经有了许多定义。威廉·哈密顿已经达到了目前的定义。

存在两种主要类型的量词。全称量词表示宇宙的所有成员(也称为话语域)。我们用“for every x”语句来解释这个量词。比如说；逻辑陈述“对于 x 中的每个数，x 将等于或大于 2*x”恰好满足宇宙中的所有数。

![](img/1ddbe6075f6c68e705c8297d60105de3.png)

“对于 x 中的每个数字，x 将等于或大于 2x”的逻辑假设

另一方面，有一些量词不是普遍接受的。存在量词保证至少保持一个单一的满意度。它不是全称量词的对比，只是命中一个单独的断言。关键字“for some”用来解释这个量词。比如说；“对于 x 中的某些数，x 等于或小于 x”这句话解释了并不是每个 x 数都验证了 x < x, only negative numbers correct the statement and of course negative number is some values in the number universe.

![](img/d9e2b89cb0d09fb6e66206a3e23209f4.png)

The logical assumption of “For some numbers in x, x³ is equal or lesser than x”

It is actually a difficult issue to express universal quantifiers. Because we are talking about all the values in the universe that correspond to this quantifier. At this point, the concept of variable emerges to tell our quantifier with a correct expression. Considering the universal quantifiers, I will now include a certain variable in the evaluation of a quantifier. As an example, I made the expression “apples are sweet” with the universal quantifier. But I have doubts about the accuracy of this statement. Here I am going to arrange “Red apples are sweet” to make a correct statement. Now my expression is “[有界量词](http://staff.ustc.edu.cn/~xyfeng/teaching/FOPL/lectureNotes/CookFBound89.pdf)的预测，而不是一个全称量词。

# 结论

在这一点上，如果说泛型，我基本上把一个类型定义为 t，这其实对应的是逻辑中的泛量词或者存在量词。

这相当于在编程语言中给出 T，U，C，A，B，D 等值。这里我需要一个“有界”值来用代码实现我的语句。

我在这篇文章中试图解释泛型完全来自数学。在写这篇文章的时候，我一直在质疑函数式编程确实比 OOP 范例有优势。在这一点上，当我了解到几乎所有 OOP 语言中使用的泛型结构实际上来自于一个数学表达式时，我认为 OOP 范例的起源实际上归结于函数以及这些函数所依赖的数学表达式。在我的下一篇文章中，我将尝试从这个角度写一些关于不同编程语言概念的东西。