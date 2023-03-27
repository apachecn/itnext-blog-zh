# 趣味 CSharp:纯、懒、异步管道创建。

> 原文：<https://itnext.io/fun-csharp-pure-lazy-and-async-pipeline-creation-204923eb6e14?source=collection_archive---------1----------------------->

![](img/7e4747411825928bc6ee1cea56fba9f2.png)

几天前，我需要创建一个异步管道，它由相互叠加的动作(方法)组成。我还希望这条管道的构建不会产生任何副作用，包括*异常*、对象状态突变、IO 操作等等。我有函数合成的需求，但也想要一元合成(Kleisli 合成)，并且不想使用*任务。运行*，我不喜欢在 ASP.NET 应用程序中这样做。

在用 IO Monad 编写了 JavaScript 之后，我决定用 C#做一些类似的事情，这满足了我的需求。我想出了一个初步的实现，我把它叫做 *AsyncLazyPipeline* 。我不太擅长给东西命名，所以如果你能想到一个更好的名字，请随时给我你的建议。我做了一些研究，看看是否有我可以使用的东西，我发现的是一个混合了*Lazy<T>和 *Tasks* 的实现，但是一方面使用了 *Task。另一方面，Run* 并没有满足我在上面段落中表达的所有需求。你可以在这里[找到**asynclazypipeline . cs**https://gist . github . com/ericrey 85/da 9671 a 22234 ef 981 e 5 ee 3653 face 4 af # file-asynclazypipeline-cs](https://gist.github.com/ericrey85/da9671a22234ef981e5ee3653face4af#file-asynclazypipeline-cs)。*

为了展示如何使用这个类，我创建了一个 POC，它由一个简单的域类( *Item* )、一个应用服务( *ItemService* )和几个补充类组成，我们将在本文中看到。继续之前，请确保在浏览器中打开 gist 链接。

**AsyncLazyPipeline 成员描述:**

*表达式*:该属性将保存所有传递到管道中的动作(方法)的组合。你可以这样想:给定两个函数/方法 g 和 f，以及一个初始值 x，形式为 *f(g(x))* 的组合意味着执行 *g(x)* ，并将结果传递给 *f* (也执行它)。用 JavaScript 术语来说，它也可能是类似于 *pipe(g，f)(x)* 或 *compose(f，g)(x)* 的东西。这个*表达式*属性将代表那个组合，我所说的管道。一旦你看到代码示例，你就会明白了。

*构造器*:你不需要显式调用它，它正被 *CreatePipeLine* 用来创建管道。

*Flatten:* 执行表达式属性中的委托，给消费者一个*任务< T >* ，表示此时*异步*流水线*。*

*选择*:有两个。它们允许您传递将在当前管道末端通过管道传递的委托，也就是说，将在已经排队的委托之后执行的委托。

*SelectMany* :你还会看到其中的两种方法。其中一个只有一个参数，相当于函数式程序员所知的函数 *bind/chain/flatMap* 。它允许你做一些类似于 *Select* 方法之一的事情，但是它作为参数接收的委托的签名是不同的，这个委托返回一个 *AsyncLazyPipeline* 的实例。最后一个 *SelectMany* ，带两个参数，允许 C#中的 LINQ 查询语法，这在函数式编程世界中称为 Monadic/Kleisli 复合。同样，一旦你看到代码，它会变得更加清晰。

添加两个接口只是为了向您展示管道的创建者( *ItemService* )如何与其他类进行交互。其中一个(*IO operations*)具有返回 *Task < T >* 的方法(在产品代码中，这些方法可能属于某个存储库、外观、IO 执行类等)。另一个接口(*ifinalnotappender*)有一个返回*AsyncLazyPipeline<t source>*的方法，它向您展示了如何进行一元合成。你将在 gist URL 上看到的另一个文件是 *ObjectExtensions* ，它只是将任何对象转换成包装该对象的 *Task < T >* 的一个更短的方法。这将在编写非*异步*委托时派上用场。

我还想提一下，在 *AsyncLazyPipeline.cs* 中还有另一个类， *CreatePipeLine* 。这是一个简单的静态类，作为一个工厂来创建*AsyncLazyPipeline<t source>*实例，而不必处理通用参数 *TSource* 。

**项目服务:**

```
public class ItemService
{
   private IFinalNoteAppender NoteAppender { get; }
   private IOperations Operations { get; }

   public ItemService(IOperations operations, 
                      IFinalNoteAppender noteAppender)
   {
      NoteAppender = noteAppender;
      Operations = operations;
   } public Task<Item> AddNotesToItem()
   {
      var combineNotes = CreatePipeLine
                   .With(Operations.GetFirstFileNote)
                   .Select(Operations.CombineWithSecondFileNote); var appendLastNote =
          from notes in combineNotes
          from finalNote in NoteAppender.AppendFinalText(notes)
          select finalNote; var combinedOperations = appendLastNote
                   .Select(new Item().AddNotes); return combinedOperations.Flatten();
    }
}
```

看看这个类唯一的方法，AddNotesToItem。第一步是从一个文件中获取一个注释，然后将这一步与从第二个文件中获取另一个注释并将这两个注释附加在一起的步骤结合起来。变量 *combineNotes* 的类型为*AsyncLazyPipeline<string>*，这意味着如果您在此时执行管道，您将获得一个*字符串*。接下来，您可以看到我一直在谈论的一元合成(LINQ 查询语法)。这就像你在对着*ienumbiable<T>做，只不过你不是，你是在对着*asynclazy pipeline<T source>*做。*

接下来， *appendLastNote* 表示获取已经附加到彼此的前两个音符并向它们添加最后一个音符的操作。看*落款人的签名。AppendFinalText* :

```
AsyncLazyPipeline<string> AppendFinalText(string value)
```

这意味着管道将在某个时候执行这个方法，并将存储在 notes 中的值传递给这个方法(在 LINQ 查询中)。然后，我们使用 *Select* 方法的版本，它没有采用 *async* 委托，而是采用 sync 委托，来创建一个*项目*，并向其中添加现在已处理的注释。变量 *combinedOperations* 表示当该管道的消费者决定执行它时需要发生的一切。最后一点代码(对 *Flatten* 的调用)确保这个消费者可以访问一个*任务<项目>，而不是 *Func <任务<项目> >* ，尽管如果您更喜欢它(或者更适合您的场景)，您可以返回后者而不是前者。*

假设您有一个控制台应用程序，或者一个 ASP.NET 核心控制器，或者任何其他使用 *ItemService* 的调用者，您可能会有这样的代码(为了演示的目的进行了简化，在生产中应该真正使用构造注入):

```
var service = new ItemService(new Operations(),
                              new FinalNoteAppender());var pipeline = service.AddNotesToItem();
```

完成此操作后，整个管道将已建成，但还没有发生任何事情。没有 IO 从文件中读取注释，没有异常，甚至没有创建*项目*对象。整个管道都已构建，没有对任何系统状态进行更改。

该管道的结构是纯的， *AddNotesToItem* 方法是纯的。你可以想叫多少次就叫多少次，你不会观察到副作用或不同的行为。这对于单元测试来说是一个好消息，对于纯函数/方法擅长的其他事情来说也是如此，我需要一篇文章来讨论这些事情，但是至少我可以说它们是函数式编程的核心及其好处。

现在，当您准备好开始工作，当您准备好释放您的代码并让您的系统在世界上享有它应得的声誉时，您只需:

```
var item = await pipeline;
```

这一行将让管道执行每一个进入它的操作。如果在任何一个被组合的方法中抛出任何异常，它也会爆炸，所以一定要记住这一点。

注意，当使用 *Select* 和第一个 *SelectMany* 方法(带一个参数的方法)时，在流水线执行过程中，只有一个值被传入传入委托。这是您在设计中可能需要考虑的问题。尽管允许一元合成的 *SelectMany* 重载允许您在 LINQ 查询中使用任何*asynclazy pipeline<t source>*返回方法，不管它有多少个参数。

最后一句话；本文介绍的方法还可以为您提供很好的可重用性。asynclazy pipeline<t source>不仅允许您编写方法，还允许您编写*asynclazy pipeline<t source>实例。*这意味着您可以创建子管道，这些子管道可以根据您的意愿进行重用和组合，以形成更复杂的管道。

编码快乐！