# 有趣的 CSharp:以安全优雅的方式处理空值。

> 原文：<https://itnext.io/fun-csharp-dealing-with-null-values-in-a-safe-and-elegant-way-4ffdf86a38c8?source=collection_archive---------0----------------------->

![](img/9ffaf33cf5c4df675df17bf2c61f061d.png)

图片来源:Chainarong praserthai | Getty Images

我在几乎所有 C#代码库中看到的一个经常出现的问题是，开发人员隐式或显式处理空值的方式很糟糕。当您创建一个可能隐式或显式返回空值的方法时，您会给自己带来很多麻烦。其中一些以数十或数百个空检查的形式出现，这产生了大量样板代码和维护问题。尽管有一种更糟糕的情况，忘记检查 NULL 将使您成为臭名昭著的 NULL 引用异常的牺牲品。

让我们考虑下面这段代码:

```
public Product GetByName(string productName)
=> Products.FirstOrDefault(product => product.Name == productName);
```

您以前见过这种类型的代码。如果没有给定名称的产品会发生什么？你是对的，返回 NULL 值。这有非常明显的问题，还有一个不那么明显，但同样糟糕。现在，该方法的所有消费者都需要防范可能的空引用异常，忘记这样做可能会对您、您的代码和您的客户带来灾难性的后果。不太明显的问题是，当查看方法的签名时，没有办法判断它是否可能返回 NULL，因此开发人员需要转到实现并分析代码，以尝试推断这是否是一种可能的情况。

当你看到一个真正的代码库时，问题就变得更大了，在那里你有很长的方法调用链，并且*方法 1* 调用*方法 2* ，而这个又依次调用*方法 3* 和*方法 4* 并且……你明白了。总之，没有任何关于这个潜在灾难性问题的警告，如果我们想知道它是否是一个问题(或者无论如何只是一直保持警惕)，我们需要手动调查许多行代码，我们不是被迫处理它(由开发人员来明确检查和处理空值)。

我们都知道当开发人员可以自由决定我们是否要做某事时会发生什么，这就是为什么很多时候，强类型函数式编程语言会迫使开发人员处理这种类型的事情，而不是留给他们的心情、记忆或天性。

你见过多少次类似这样的代码:

```
var product = productRepository.GetByName("Galaxy Watch");if (product != null)
{
   // ...
}
else
{
   // ...
}
```

你有没有想过，必须有一种方法来删除这样的样板代码，并阻止它用重复的逻辑来扰乱我们的代码库，这些逻辑只是为了一次又一次地做同样的事情？如果有一种方法可以做到这一点，同时还能清楚地传达一个值可能不存在或找不到，同时迫使开发人员处理这种情况，而不是让您决定或记住任何事情，会怎么样？

OOP 社区熟悉一种叫做空对象模式的东西。在使用这种模式十多年后，我清楚地意识到它不够灵活，而且会产生大量代码(因为您通常会为每个要处理潜在空值的类创建一个空对象实现)。我还注意到，这导致开发人员仍然使用 if/else 语句来查看给定的实例是 NULL 对象实例还是真实的实例。长话短说，空对象模式不适合我。

果然，函数式编程社区已经有了解决方案，我们 OOP 开发者所要做的就是了解它是如何工作的，并使用它。F#有一个类型叫做 Option([https://fsharpforfunandprofit.com/posts/the-option-type/](https://fsharpforfunandprofit.com/posts/the-option-type/))，允许你处理这个问题。C#开发人员没有现成的这种类型，但是你可以在网上找到很多实现，大多数都叫 Maybe(而不是 Option)。接下来让我们看看如何使用这种方法。

如果你想跟着做，继续安装 **LeanSharp** Nuget 包(你可以在[https://github.com/ericrey85/LeanSharp/](https://github.com/ericrey85/LeanSharp/)找到 Github 库)

```
Install-Package LeanSharp
```

继续将这个 using 语句添加到。cs 文件，您将在其中使用 Maybe 类:

```
using LeanSharp.Extensions;
```

现在，让我们重构前面的例子，使用 Maybe(称为 Maybe 单子)。

```
public Maybe<Product> GetByName(string productName)
=> Products
     .FirstOrDefault(product => product.Name == productName)
     .ToMaybe();
```

第一件值得注意的事情是，通过查看该方法的签名(或它实现的接口)，可以非常清楚地看到可能存在找不到产品的场景(不需要调查来推断这一点)。它不仅向开发人员传达了这一点，而且还迫使她/他处理这一事实。现在让我们来看看使用这种方法的一种可能方式。

```
var productPrice = product.GetOrElse(p => p.Price, -1);
```

这个 Maybe 类包含许多方法(GetOrElse 是其中之一),这些方法允许您安全地声明在值存在或缺失的情况下您想要做的事情。在上面的代码中，如果找到了产品，变量 *productPrice* 将以实际的产品价格结束，如果没有找到，则以值-1 结束(不要花太多时间从业务逻辑的角度考虑我们是否想要这样做，这不是重点)。

正如你所看到的，在 Maybe 类中的某个地方有逻辑保护你不受空值的影响，你不需要在一千个不同的地方重复你自己(DRY 原则)来做这件事。您也不会忘记它，因为方法的签名迫使您处理 Maybe 类型。

现在，让我们看一个处理集合的例子。我一直看到许多开发人员在需要检查给定集合(数组、列表等)是否包含任何元素时，重复同样的逻辑:

```
var products = productRepository.GetByCategory("Electronics");if (product != null && products.Any())
{
   // ...
}
```

我一次又一次地看到这种重复的逻辑。C#开发人员更了解它的“现代”特性，通常会将这种条件逻辑简化为:

```
if (products?.Any() == true)
```

在我看来，这更好，但是您仍然必须记住在任何需要这种类型的逻辑的时候这样做，并且您也将有不习惯使用这种语法的团队成员使用第一种语法(这将产生不一致)。通过使用 **LeanSharp** ，您可以安全地检查一个集合是否有元素，甚至不用担心或花时间去试图弄清楚是否首先需要检查 NULL。看看下面的例子:

```
if (products.SafeAny())
```

如果你需要使用谓词，你可以这样做:

```
if (products.SafeAny(p => p.Price > 100))
```

**LeanSharp** 的目的是帮助创建一种通用且一致的 C#编程方式，这也允许我们提高开发人员的生产力(通过创建更易于维护的代码)并极大地减少您的 bug 数量(通过解决已知的不良编码实践并实施良好的实践)。

如果你注意到，Maybe monad 被用在一个场景中，在这个场景中，找不到产品是完全有效的。如果在您的场景中，找不到值或没有值表示出现了“意外”问题，该怎么办？您通常会看到这些场景被编码成这样:

```
// Assume the variable newCount exists in this context.if (product == null)
{
  throw new InvalidOperationException("The product was not found.");
}
else
{
   product.UpdateInventory(newCount);
}
```

大多数时候，我喜欢以不同的方式处理这个问题，使用被称为面向铁路的编程(ROP)的东西——【https://fsharpforfunandprofit.com/rop/。这个话题值得有自己的文章(说实话更像是自己的书)。这个想法也来自于函数式编程，函数式编程中有一种叫做“要么单子”的东西(因为你可以有一个左边的错误，或者右边的实际结果)。

**LeanSharp** 包含 C#的 ROP 实现，它是一个通用结果< TSuccess，TFailure >类，包含扩展方法，允许您编写代码，同时防止您在失败时执行不应该执行的逻辑。上述案例可以重构为:

```
// Assume the variable newCount exists in this context.product  
  .IfNull("The product was not found.")
  .Map(p => p.UpdateInventory(newCount));
```

在上面的代码中，如果产品实例为空，则不会调用 *UpdateInventory* 方法。 *IfNull* 方法将产品实例“提升”到结果<产品、字符串>实例中，然后您可以使用流畅的类似 API 的风格来链接方法调用(以安全的方式)，这只不过是一种函数/方法组合的方式。

我在这里的意图不是告诉你，你在这篇文章中看到的想法代表了做事的最佳方式，或者它们比你目前正在做的更好。我只是想和你分享我作为一名 C#开发人员所做的一些事情，这些事情在大多数时候是 C#中传统做事方式的更好替代。几十年来，我写了很多代码，犯了很多错误，但我相信其中的一些想法可以解决你当前 C#代码库中可能存在的一些问题。

我希望你喜欢这篇文章，如果你很想知道你可以用 **LeanSharp** 做的所有事情，去看看它的 GitHub 库，看看你可以使用的所有类/方法。如果您需要帮助理解某个类/方法允许您做什么以及如何使用它，我建议您查看单元测试(目前超过一百个)。

**更新 :**

一些 Reddit 用户对这篇文章发表了评论，提到要使用 C# 8 的可空引用类型。通过这样做，你仍然会遇到本文中提到的大部分问题，见下面我在 Reddit 上的部分回复。

*邀你阅读*[*https://ericbackhage . net/c/nullable-reference-types-comparated-to-the-option-monad/*](https://ericbackhage.net/c/nullable-reference-types-compared-to-the-option-monad/)*。*

*长话短说，可空引用类型并不消除您到处检查空值的需要，只有当值来自使用可空引用类型的方法时，如果您不检查空值，C#编译器才会给出警告。因此，作为开发人员，您仍然没有被迫处理 null，您仍然有在整个代码库中检查 null 的繁琐任务和维护问题。*