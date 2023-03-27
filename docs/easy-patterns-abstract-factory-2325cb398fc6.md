# 简单模式:抽象工厂

> 原文：<https://itnext.io/easy-patterns-abstract-factory-2325cb398fc6?source=collection_archive---------4----------------------->

![](img/2ec951759b34191811bcbc08932ea0d6.png)

本文是 easy patterns 系列描述的延续，介绍了抽象工厂模式，它解决了在整个应用程序中实例化特定外观类的问题。

## 创作模式:

> [**简易工厂**](/easy-patterns-simple-factory-b946a086fd7e)
> 
> [**工厂法**](/easy-patterns-factory-method-5f27385ac5c)
> 
> [**建造者**](/easy-patterns-builder-d85655bcf8aa)
> 
> [**单个**](/easy-patterns-singleton-283356fb29bf)
> 
> [](/easy-patterns-abstract-factory-2325cb398fc6)***(本文)***
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

**这种模式也称为套件。**

**抽象工厂模式描述了一种将单个 fabricss 组合在一起的方法，这些 fabric 是按照一些通用标准组合在一起的。**

**这种模式包括四个主要角色:**

*   ****AbstractFactory —** 为创建产品对象的操作声明一个接口。**
*   ****具体工厂** —实现创建具体产品对象的操作**
*   ****产品** —定义由相应的具体工厂创建的产品对象**
*   ****客户端** —只使用由 AbstractFactory 和 Product 类声明的接口。**

**这个想法很简单。AbstractFactory 将产品对象的创建委托给它的 ConcreteFactory 子类。为了创建单独的产品对象，客户必须使用单独的混凝土工厂。**

# **使用示例**

**抽象工厂类通常用[工厂方法模式](/easy-patterns-factory-method-5f27385ac5c)来实现。ConcreteFactory 通常是一个 [Singleton](/easy-patterns-singleton-283356fb29bf) (从 ConcreteFactory 应该创建一个具体的类家族的角度来看，这是合理的)。**

**例如，我们正在创建几个产品:代码片段和代码解析器。每个代码片段应该对应于特定的代码解析器。这就是为什么创建 CodeProcessorFactories 来一前一后地创建代码片段和代码解析器是有意义的。**

# **利润**

**这种模式隔离了具体的类。它帮助您控制应用程序创建的对象的类别。AbstractFactory 接口定义了客户端操作实例的方式。**

**这使得更改应用程序使用的具体工厂变得很容易。它可以简单地通过改变具体工厂来使用不同的产品配置。**

**它促进了产品之间的一致性。当一个系列中的产品设计为协同工作时，一个应用程序一次只能使用一个系列中的对象是很重要的。**

# **薄弱的地方**

**扩展抽象工厂来生产新产品并不容易。AbstractFactory 接口修复了可以创建的产品集。支持新类型的产品需要扩展 AbstractFactory 接口，这也涉及到改变它的所有子类。**

# **结论**

**如果您觉得这篇文章有帮助，请点击👏按钮并在下面随意评论！**