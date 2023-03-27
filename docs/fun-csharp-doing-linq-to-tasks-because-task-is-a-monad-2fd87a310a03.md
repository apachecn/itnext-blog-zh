# 好玩的 CSharp:给任务做 LINQ，因为任务是单子。

> 原文：<https://itnext.io/fun-csharp-doing-linq-to-tasks-because-task-is-a-monad-2fd87a310a03?source=collection_archive---------0----------------------->

![](img/34a289069acfbc6f55db13915960702c.png)

我已经考虑了一段时间我下一篇文章的主题，我必须承认，即使我有很多想法，我也不知道该用哪一个。前几天看了一个知名的 **C#/。NET** 架构师在一次演示会上公开表示， **C#/。NET** 没有任何类似“单子、部分应用、涂抹等”的花哨东西我感到内心有一团火在燃烧，我知道这将引导我写下一篇文章。

**。NET** 充满了**单子**，一些例子有**IEnumerable<T>，*数组*，**任务**等等。你也可以在 C#中进行局部应用和涂抹(我已经做过很多次了),这并没有什么特别的。如果你没有受过高级思维训练，或者你已经完成了大部分命令式 C#/，你的大脑很难首先从这个角度思考。NET 编程。慢慢地，当然是坏的，微软已经把更多的功能结构带入平台，这让我很高兴；作为一个函数式 JavaScript 爱好者，能够在 **C#** 中做类似的事情是很好的。**

如果想查看维基百科对 **Monad** 的定义，可以随意访问:[https://en . Wikipedia . org/wiki/Monad _(functional _ programming)](https://en.wikipedia.org/wiki/Monad_(functional_programming))。每当我谈论函数式编程概念时，我都尽量不要太专业，也不要对定义吹毛求疵，我尽量像在和一个五岁的孩子谈论编程一样解释事情。所以，如果这个孩子问我:什么是**单子**？我可能会说:“它就像一个芒果”。你可以买一个芒果，放在袋子里，放在你的车里，开回家，洗干净，但是在你把皮剥掉，接触到“肉”之前，它并没有太大的用处。以类似的方式，您可以传递*数组*、*任务*、*字典*，但是最终，为了让它们有用，您需要访问它们的底层内容(您需要剥芒果皮)。所有这些类型还为您提供了与其底层内容进行交互的方法。

例如，以**IEnumerable<T>为例，它充当了 **T.** 类型的一些数据的“容器”。此外，您并不真正需要使用命令式 **for** 和/或 **foreach** 循环来遍历底层数据。您可以使用所有的*扩展方法* **。NET** 为您提供的，以便与它的内容进行交互。因此，用非常简单的术语来说，一个**单子**，是一个包装器类型，它有内置的功能让你处理它的内容。老实说，为了让某样东西成为单子，它必须遵循一定的法则(如果没有这些法则，会有多有趣)，但是讨论这个问题超出了本文的范围。**

回到芒果的类比，例如，当你*等待*它或者访问它的*结果*属性时，你“剥离”一个**任务**。为了更准确地进行类比，你可以考虑洋葱而不是芒果，因为**单子**可以有几个包装层，就像*任务*的*数组*的*数组*。

**C#/。NET** 内置了对***，**一元合成的支持，也许我不会试图定义这个术语，或者把你带到一个令人困惑的链接，我会用代码向你展示我的意思。*

*让我们创建这个简单的*类:**

```
*public class TaskCreator
{
  public async *Task*<*int*> CreateTask1() => await Task.FromResult(4); public async *Task*<*int*> CreateTask2() => await Task.FromResult(6); public async *Task*<*string*> CreateTask3()
  => await Task.FromResult("The result is: ");
}*
```

*它包含 3 个**任务**——返回方法，在实际的产品代码中，这些方法还可以处理文件系统、数据库、网络调用等等。接下来，我将在**任务** *类*中添加三个*扩展方法*。*

```
*public static class TaskExtensions
{
  public static async Task<B> Bind<A, B>(
     this Task<A> @this, Func<A, Task<B>> fn)
  => await fn(await @this); public static async Task<B> Map<A, B>(
     this Task<A> @this, Func<A, B> fn)
  => await Task.FromResult(fn(await @this)); public static Task<C> SelectMany<A, B, C>(
     this Task<A> @this, Func<A, Task<B>> fn, Func<A, B, C> select)
  => @this.Bind(a => fn(a).Map(b => select(a, b)));
}*
```

*我们增加了*地图*、*绑定*和*选择多项。Map* 是一个非常方便的方法，你可以修改它以用于任何类型的对象，而不仅仅是**单子**，它类似于“我将从一个对象 *a 开始，*我将对它应用你传递给我的函数，然后我将给你对象 *b* ”。当你看到一个**单子**和一个*映射函数*(同样遵循一定的数学法则)时，你就在一个**仿函数**面前。*绑定*方法(也称为*链*)类似于*映射*，但是可以看到，在***Func<>****中传递的是一个**任务<>**而不是仅仅一个**。*选择多*就是**。NET** 对*一元合成*的支持，事不宜迟，我就给你看看那是什么样子(剧透提醒:这种表情你已经很熟悉了)。****

```
****public async *Task*<*string*> CoolMonadicComposition()
{
  var taskCreator = new *TaskCreator*(); var result = from a in taskCreator.CreateTask1()
               from b in taskCreator.CreateTask2()
               from c in taskCreator.CreateTask3()
               select c + (b + a);

  return await result;
}****
```

****如果你认为 *LINQ 查询语法*是 **IEnumerable/IQueryable** 独有的，你会大吃一惊。您可以在 **C#/中的任何**单子**中使用这个*单子组合*语法。网**。实际上我是从[https://gist . github . com/johnazariah/d 95 c 03 e 2c 56579 c 11272 a 647 Bab 4 BC 38](https://gist.github.com/johnazariah/d95c03e2c56579c11272a647bab4bc38)得到这个想法的，它是**也许是**单子的一部分。我现在已经用几个**做了类似的事情。NET** **单子**，内置的和自定义的，包括著名的 *Result < TSuccess，TFailure >* **单子**用在 **C#** (我很喜欢的一种编码风格)中面向铁路的编程。****

****在上面最后这段代码中需要注意的一点是，如果属于 *TaskCreator* 的三个**Task**-return 方法中的任何一个抛出**异常**，它将在您*等待* *结果*时传播，请记住这一点。****

****花点时间去理解三个*扩展方法*里面发生了什么，如果你一开始不能理解，坚持做下去直到你理解为止，当这种情况发生时，我向你保证这将是非常有益的。****

## *****更新:*****

****为了帮助您理解我们对 *SelectMany* 的实现，我想向您展示一下如果我们没有 *LINQ 查询语法，我们如何编写同样的*一元复合*会很好。*为了完整起见，我将包括第一种方法，但将其注释掉。正如您将从下面的代码中推断的那样，您可以在您的*一元合成*中拥有任意多的*一元表达式*。****

```
***// var result = from a in taskCreator.CreateTask1()
//              from b in taskCreator.CreateTask2()
//              from c in taskCreator.CreateTask3()
//              select c + (a + b);var result = taskCreator
 .CreateTask1()
 .SelectMany(_ => taskCreator.CreateTask2(), (a, b) => new { a, b })
 .SelectMany(_ => taskCreator.CreateTask3(),
            (param1, d) => d + (param1.a + param1.b));return await result;***
```

***顺便说一下，在幕后， **C#** 编译器将上面注释掉的 *LINQ 查询*(这个语法只是语法糖)转换成它下面的代码(显式使用 *SelectMany* )。如果你在你的 *LINQ 查询*中使用了比我们上面使用的更多的表达式，编译器将开始在结果代码*中使用*透明标识符*。****

***编码快乐！***