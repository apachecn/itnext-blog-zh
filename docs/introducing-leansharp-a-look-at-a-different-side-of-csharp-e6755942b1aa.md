# 介绍 LeanSharp:看看 CSharp 的另一面。

> 原文：<https://itnext.io/introducing-leansharp-a-look-at-a-different-side-of-csharp-e6755942b1aa?source=collection_archive---------2----------------------->

![](img/148cb2cc4a6ec9eadbc369d0401cc6ef.png)

在成为面向对象编程(OOP)的粉丝十多年后，我发现了函数式编程，从那以后我看待软件开发的方式就再也不一样了。尽管我尽量不花时间说明一种范式比另一种更好。在这个世界上，事物都是相对的，每个人都有自己看待事物的方式，因此很容易做出相对的假设和陈述。

我主要写函数式 JavaScript，但说到 C#，我喜欢函数式和 OOP 风格的结合。他们可以住在一起，事实上，他们可以成为很好的朋友。在 JavaScript 中使用函数式编程一段时间后(在真实的生产代码中)，我开始在 C#中做同样的事情；在用 C#主要做了一年的函数式/声明式编程之后，我决定创建一个 Nuget 包(和相关的公共 Github 库),它包含了所有的声明式/函数式结构，使我们的团队能够大大减少错误(当然，借助于一个好的测试策略)。在这一年中，我们犯了一些错误，我们学到了很多，我们完善了方法和技术。

结果，**lean sharp**[https://github.com/ericrey85/LeanSharp/](https://github.com/ericrey85/LeanSharp/)，它也作为一个同名的 Nuget 包存在。这篇文章假装是一篇非常入门的文章，在那里我可以与其他 C#开发人员分享，这些东西不是在理论上，而是在实践中，允许我们创建更好、更健壮的代码。我并不是说这种风格适合所有人，或者比你现在的风格更好，我只是认为它可能会向你展示你不知道的 C#的一面，并且有一天，你可能会从中受益。

我将使用 Github 资源库上的*入门*会话来指导示例和解释。如果您从手机导航到 Github，请确保点击 README.md 文件，否则您将无法看到该部分。

**入门:**

安装 LeanSharp Nuget 包:

```
Install-Package LeanSharp
```

现在，确保在要开始使用的 C#文件中添加一个名称空间引用:

```
using LeanSharp;
```

**面向铁路的编程(ROP):**

ROP 是一种处理错误的函数式方法。它从一个*结果*开始，在任何给定的时间，它要么有一些数据(成功)，要么有一个错误(失败)。这种方法在函数式编程领域通常被称为要么单子，可能是右单子(成功)或左单子(错误)。这种方法，除了别的以外，允许我们摆脱异常的副作用。我相信 ROP 这个术语是由 Scott Wlaschin 创造的，他因 https://fsharpforfunandprofit.com/的网站[和他关于 F#和函数式编程的演讲而闻名，尽管我第一次听说它是在 https://www.youtube.com/watch?v=uM906cqdFWE](https://fsharpforfunandprofit.com/)的[。](https://www.youtube.com/watch?v=uM906cqdFWE)

在 **LeanSharp** ，中，与 ROP 相关的初始代码取自[https://github.com/habaneroofdoom/AltNetRop](https://github.com/habaneroofdoom/AltNetRop)，然后基于现实世界的使用，它成长(现在包括更多的方法和一个声明性的 try/catch 方法)到它当前的状态。

说够了，让我们看一些代码！

```
var firstResult = Result<int, string>.Succeeded(2);var secondResult = firstResult.Map(two => two + 3); // Success(5)var finalResult = secondResult
     .Bind(five => Result<int, string>.Succeeded(five + 5));// Success (10)
```

如果您发现自己用相同的泛型参数重复结果~~,那么您可以将一个静态 using 语句添加到。cs 文件，然后您就不必再重复它了。~~

```
using static LeanSharp.Result<int, string>;
```

然后:

```
Succeeded(2).Map(two => two + 3).Bind(five => Succeeded(five + 5))
```

如何将可能抛出异常的方法转换成不会抛出异常的方法？

```
public async Task<Result<Customer, Exception>> Insert(Customer customer)
{
  try
  {
    // Try to asynchronously insert the Customer into the DB.
    // ...
    return Result<Customer, Exception>.Succeeded(customer);
  }
  catch (Exception ex)
  {
    return Result<Customer, Exception>.Failed(ex);
  }
}
```

使用 **LeanSharp** 您可以更进一步，将上面的 try/catch 语句转换为等价的声明性表达式，其中删除了样板 catch 逻辑和显式成功创建:

```
public async Task<Result<Customer, Exception>> Insert(
 Customer customer)
=> await Try.ExpressionAsync(async () =>
{
  // Try to asynchronously insert the Customer into the DB.
  // ...
  return customer;
});
```

在获得结果(从任务内部)之后，可以链接像 Map 和 Bind 这样的方法，并且只有当结果包含成功时，传递给这些方法的委托才会运行。

由于我们向 Result 添加了所有必要的方法，所以您也可以使用 LINQ 查询语法(不要忘记静态 using 语句来编译该代码):

```
var result = from s1 in Succeeded(1)
             from s2 in Succeeded(2)
             from s3 in Succeeded(3)
             select s1 + s2 + s3; // Success(6)
```

# 避免处理空值，可能的单子:

**LeanSharp** 中的 Maybe monad 摘自[https://gist . github . com/johnazariah/d 95 c 03 e 2c 56579 c 11272 a 647 Bab 4 BC 38](https://gist.github.com/johnazariah/d95c03e2c56579c11272a647bab4bc38)，可以随意导航到那里查看各种示例和精彩的解释。

让我们看看如何使用它:

```
public Maybe<Order> GetOrderById(int id)
{
  var order = GetOrderFromDB(id);
  return Maybe<Order>.Some(order);
}
```

在调用 GetOrderById 时不必处理 null(或者忘记它并以 NullReferenceException 结束)，您可以在 Maybe 上执行安全操作，它们是安全的，因为如果没有找到订单，操作将不会执行，并且 Maybe 封装并集中了这样做的逻辑。此外，对于这个签名，GetOrderById 是诚实的，它非常明确地表明订单可能找不到，这是一条非常重要的信息。

```
/* 
AssingOrderToCustomer returns a newly created Customer with the 
given order assgined to it.
What if the order was not found? Leave that to the Maybe Monad. 
*/

var customerId = GetOrderById(5)
  .Map(order => AssingOrderToCustomer(customer, order));
  .GetOrElse(customer => customer.Id, 0);
```

# 声明性处置(制成表达式):

```
var result = await Dispose.UsingAsync(() => new DisposableInstance, async disposableInstance =>
{
  // Add in here whatever you need in the body of the dipose.
  // ...
  return disposableInstance.DoTheWork();
});
```

总的来说，尽量支持表达式而不是语句。表达式是可重用的，它们可以从一个方法中返回或者传入一个方法。语句不能做到这些。这是一项如此强大的技术，以至于微软在 C# 8 中添加了使用表达式的*来处理一次性对象。并且他们用*开关*做了同样的表情。*

在函数式编程的核心，你会发现映射函数。可以把它们想象成一个函数/方法，对输入数据进行一些转换，产生一些输出。一些代码？当然可以！但是首先，让我们看看在 C#中这样做的传统和必要的方式:

```
var customer = GetCustomerById(id);
var billModel = new BillModel
{
  CustomerId = customer.Id,
  CustomerName = customer.FullName,
  StreetName = customer.StreetName
};
```

我说的那个地图怎么样？给你:

```
var billModel = GetCustomerById(id).MapTo(c => new BillModel
{
  CustomerId = c.Id,
  CustomerName = c.FullName,
  StreetName = customer.StreetName
});
```

MapTo 的命名是为了避免与 Result 类中的 Map 扩展方法和 Maybe 类中的 instance 方法发生名称冲突。

# 管道(可组合管道):

当我试图找到一种方法来编写同步和异步代码，同时支持延迟执行时，我想到了*管道*类。我在我的文章[https://it next . io/fun-cs harp-pure-lazy-and-async-pipeline-creation-204923 eb6e 14](/fun-csharp-pure-lazy-and-async-pipeline-creation-204923eb6e14)中谈到了它们。请随意浏览该文章，以了解更多关于管道的信息。让我们看一些例子。

用方法组成管道。

```
int AddFour(int number) => number + 4;var pipeline = CreatePipeline.With(() => 5).Select(AddFour);var task = pipeline.Flatten(); // Task(9)
```

用另一个管道组成管道:

```
var initialPipeline = CreatePipeline.With(() => 5);var finalPipeline = initialPipeline.SelectMany(
  five => CreatePipeline.With(() => five + 4));var task = finalPipeline.Flatten(); // Task(9)
```

又是你好一元作文！

```
var firstPipeline = CreatePipeline.With(() => 5);
var secondPipeline = CreatePipeline.With(() => 6);
var thirdPipeline = CreatePipeline.With(() => 9);var finalPipeline = from firstValue in firstPipeline
                    from secondValue in secondPipeline
                    from thirdValue in thirdPipeline
                    select firstValue + secondValue + thirdValue;var task = finalPipeline.Flatten();  // Task(20)
```

要查看使用管道的更真实的示例，可以查看[https://gist . github . com/ericrey 85/da 9671 a 22234 ef 981 e 5 ee 3653 face 4 af](https://gist.github.com/ericrey85/da9671a22234ef981e5ee3653face4af)。尽管请注意，我已经将名称从 *AzyncLazyPipeline* 更改为简单的 *Pipeline* 。

# 安全管道(可组合的无异常管道)

在写了一年 ROP 代码后，我发现明显的缺点是为了等待链式任务(由异步方法产生)而产生了大量连续的 await。为了让开发者免于此，我创建了*Pipeline；*但是有了它们，如果我想同时做 ROP，就需要同时处理两个单子(管道和结果)。在看到多少次我用类似于*管道<结果<t 成功，t 失败> >* 的东西来创建一个没有异常的管道后，我决定创建安全管道。

SafePipeline 和 Pipeline 一样，是一个 Monad，也是一个 Monad Transformer，但它为您提供了一个封装 ROP 的管道，并知道如何根据底层值的成功或失败来采取行动。让我们看看用常规方法或另一个 SafePipeline 构建 SafePipeline 的几种方法。

```
async Task<Result<int, string>> GetFirstValue(int number) => await Result<int, string>.Succeeded(number + 4).AsTask();async Task<Result<int, string>> GetSecondValue(int number) => await Result<int, string>.Succeeded(number + 5).AsTask();async Task<int> GetThirdValue(int number)
=> await (number + 4).AsTask();int GetFourthValue(int number) => number + 5;SafePipeline<int, Exception> GetFithValue(int number) => 
CreateSafePipeline.TryWith(() => number + 6);var firstPipe = CreateSafePipeline.With(() => GetFirstValue(5));
var secondPipe = firstPipe.Select(GetSecondValue);
var thirdPipe = secondPipe.Select(GetThirdValue);
var fourthPipe = secondPipe.Select(GetFourthValue);
var fithPipe = fourthPipe.SelectMany(
  value => GetFithValue(value).ToStringFailure());var finalPipeline = from firstValue in secondPipe
                    from secondValue in thirdPipe
                    from thirdValue in fithPipe
                    select firstValue + secondValue + thirdValue;var result = await finalPipeline.Flatten(); // Success (57)
```

抛出异常的操作现在可以安全使用了，如果出现异常，它会被封装在一个失败的结果中。

```
int InvalidDivideByZeroOperation(int number) => number / 0;async Task<int> GetValue(int number) => await (number + 4).AsTask();var firstPipeline = CreateSafePipeline.TryWith(
  () => InvalidDivideByZeroOperation(5));var finalPipeline = firstPipeline.Select(GetValue);var result = await finalPipeline.Flatten(); // Failure (exception)
```

需要注意的一点是，SafePipeline 是纯确定性的，不管调用多少次，只要传递相同的参数，就会得到相同的响应(在软件开发中有巨大的好处)，异常不会破坏这一点。

你可以用 **LeanSharp** 做很多事情。我希望这篇介绍是有帮助的，我也希望即使你不喜欢你在这篇文章中看到的思维转变，你至少喜欢 C#的另一面，在那里声明性的方法被选择而不是命令性的方法，尽管没有说它们不能共存。在 Github 存储库中，您将能够找到超过 100 个单元测试，可以帮助您理解 Nuget 包中的类/方法是如何工作的。

编码快乐！