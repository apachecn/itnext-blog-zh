# 简单模式:简单工厂

> 原文：<https://itnext.io/easy-patterns-simple-factory-b946a086fd7e?source=collection_archive---------5----------------------->

本文是 easy patterns description 系列的延续，描述了以简单快捷的方式创建连续实例的 creational pattern Simple Factory。

目前，您可以找到此类模式的文章:

## 创作模式:

> [](/easy-patterns-simple-factory-b946a086fd7e)***(本文)***
> 
> **[**工厂法**](/easy-patterns-factory-method-5f27385ac5c)**
> 
> **[**构建器**](/easy-patterns-builder-d85655bcf8aa)**
> 
> **[**单个**](/easy-patterns-singleton-283356fb29bf)**
> 
> **[**抽象工厂**](/easy-patterns-abstract-factory-2325cb398fc6)**
> 
> **[**原型**](/easy-patterns-prototype-e03ec6962f89)**

## **结构模式:**

> **[**适配器**](/easy-patterns-adapter-9b5806cb346f)**
> 
> **[**装饰者**](/easy-patterns-decorator-eaa96c0550ea)**
> 
> **[**桥梁**](/easy-patterns-bridge-28d50dc25f9f)**
> 
> **[**复合**](/easy-patterns-composite-8b28aa1f158)**
> 
> **[**立面**](/easy-patterns-facade-8cb185f4f44f)**
> 
> **[**飞锤**](/easy-patterns-flyweight-dab4c018f7f5)**
> 
> **[**代理**](/easy-patterns-proxy-45fc3a648020)**

## **行为模式:**

> **[**来访者**](/easy-patterns-visitor-b8ef57eb957)**
> 
> **[**调解员**](/easy-patterns-mediator-e0bf18fefdf9)**
> 
> **[**观察者**](/easy-patterns-observer-63c832d41ffd)**
> 
> **[**纪念品**](/easy-patterns-memento-ce966cec7478)**
> 
> **[**迭代器**](/easy-patterns-iterator-f5c0dd85957)**
> 
> **[**责任链**](/easy-patterns-chain-of-responsibility-9a84307ad837)**
> 
> **[**策略**](/easy-patterns-strategy-ecb6f6fc0ef3)**
> 
> **[**状态**](/easy-patterns-state-ec87a1a487b4)**

# **主要本质**

**简单工厂模式是最简单的类，它有创建其他类实例的方法。例如，如果您需要为某个特定的类创建大量的实例，使用为您返回实例的方法要容易得多。这是主要的想法。工厂帮助我们将所有的对象创建放在一个地方，避免将`new`键值分散到整个代码库中。**

# **使用示例**

**在这个例子中，我们创建了两个类:一个用于 box，一个用于 tape。我们正在创造礼品盒包装。一旦我们需要正方形的盒子(宽度===高度)，用方法创建工厂类就更容易了，这个方法通过在数组中定义大小在迭代中为我们返回它们。就像我们后来用胶带粘盒子一样。在为 box 和 tape 返回新对象的方法中，我们包含了额外的业务逻辑，它在控制台中通知我们哪个对象是在运行时创建的(参见示例代码的底部)。**

**因此，当您有一个应该添加到创建类实例中的业务逻辑时，使用简单工厂而不是将这个逻辑分散到代码中会更容易。**

# **薄弱的地方**

**不幸的是，简单性给我们带来了一些限制——无法控制创作过程。您可以通过的参数有限。如果你需要做的不仅仅是简单的创建，你会喜欢更高级的模式:[工厂方法](/easy-patterns-factory-method-5f27385ac5c)，[抽象工厂](/easy-patterns-abstract-factory-2325cb398fc6)，构建器等等。**

# **结论**

**如果您觉得这篇文章有帮助，请点击👏按钮并在下面随意评论！**