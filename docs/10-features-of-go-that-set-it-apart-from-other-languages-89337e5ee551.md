# Go 区别于其他语言的 10 个特征。

> 原文：<https://itnext.io/10-features-of-go-that-set-it-apart-from-other-languages-89337e5ee551?source=collection_archive---------0----------------------->

![](img/1256041fbb00119ae9a9b560cf799d68.png)

作为编程语言，go 还相当年轻。它于 2009 年 11 月 10 日首次发布。它的创造者[罗伯特·格里斯默](https://en.wikipedia.org/wiki/Robert_Griesemer)
[罗布·派克](https://en.wikipedia.org/wiki/Rob_Pike)和[肯·汤普森](https://en.wikipedia.org/wiki/Ken_Thompson)在谷歌工作，大规模扩展的挑战激发他们将 Go 设计为一个快速高效的编程解决方案，用于大型代码库的项目，由多个开发人员管理，具有严格的性能要求，并跨越多个网络&处理核心。

Go 的创始人在创造他们的新语言时，也抓住机会学习其他编程语言的优点、缺点和疏忽。结果是一种干净、清晰、实用的语言，只有相对较少的命令和功能。

在本文中，我将解析 Go 区别于其他语言的 10 个特性(根据我个人的观察)。

# 1.Go 总是在构建中包含二进制文件

Go 运行时提供诸如内存分配、垃圾收集、并发支持和联网等服务。它被编译成每个 Go 二进制文件。这与许多其他语言不同，其中许多语言使用虚拟机，需要与程序一起安装才能正常工作。

将运行时直接包含在二进制文件中使得发布和运行 Go 程序变得非常容易，并且避免了运行时和程序之间的不兼容问题。Python、Ruby 和 JavaScript 等语言的虚拟机也没有针对垃圾收集和内存分配进行优化，这解释了 Go 相对于其他类似语言的优越速度。举个例子，Go 在[栈](https://en.wikipedia.org/wiki/Stack-based_memory_allocation)上存储尽可能多的数据，在那里数据按顺序排列，访问速度比[堆](https://www.educba.com/what-is-heap-memory/)快得多。稍后将详细介绍。

关于 Go 的静态二进制文件的最后一点是，因为没有运行所需的外部依赖，**它们启动起来非常快**。如果你使用像[谷歌应用引擎](https://cloud.google.com/appengine)这样的服务，这是很有用的，这是一个运行在谷歌云上的平台即服务，它可以将你的应用程序缩减到零个实例，以节省云成本。当一个新的请求进来时，App Engine 可以在一眨眼的时间内启动你的 Go 程序的一个实例。Python 或 Node 中的相同体验通常会导致 3-5 秒(或更长时间)的等待，因为所需的虚拟环境也会随着新实例一起启动。

# 2.Go 没有针对程序依赖性的集中托管服务

为了访问已发布的围棋程序，开发者不依赖于集中托管的服务，比如 Java 的 [Maven Central](https://search.maven.org/) 或 JavaScript 的 [NPM](https://www.npmjs.com/) registry。相反，项目通过它们的源代码库(最常见的是 Github)共享。`go install`命令行允许以这种方式下载存储库。

我为什么喜欢这个功能？我一直认为像 Maven Central、PIP 和 NPM 这样的集中托管的依赖服务是有些令人生畏的黑匣子，可能会抽象出下载和安装依赖项(以及依赖项的依赖项)的麻烦，但不可避免地会在出现依赖项错误时导致令人害怕的心脏停止跳动(我经历过太多了，无法计数)。

我常常感到沮丧，因为我从来没有完全理解他们是如何在幕后工作的。通过去掉一个中心服务，安装、版本控制和管理你的 Go 项目的依赖项的过程变得非常清晰，从而更加干净。

此外，让你的模块对其他人可用就像把它放在版本控制系统中一样简单，这是一种非常简单的分发程序的方式。

# 3.Go 是按值调用

在 Go 中，当你提供一个原语(数字、布尔或字符串)或一个结构(类对象的粗略等价物)作为函数的参数时，Go 总是会制作一个变量的值的**副本。**

在 Java、Python 和 JavaScript 等许多其他语言中，原语是通过值传递的[，但是对象(类实例)是通过**引用**传递的，这意味着接收函数实际上接收的是一个指向原始对象的**指针**，*而不是其副本*。](/the-power-of-functional-programming-in-javascript-cc9797a42b60)

这意味着在接收函数**中对对象所做的任何改变都会反映在原始对象**中。

在 Go 中，默认情况下，通过使用*星号*操作符，通过值传递结构和原语，并且可以选择传递一个[指针](https://www.ardanlabs.com/blog/2017/05/language-mechanics-on-stacks-and-pointers.html):

```
*// pass by value*
func MakeNewFoo(f **Foo**) (Foo, error) {
   f.Field1 = "New val"
   f.Field2 = f.Field2 + 1
   return f, nil
}
```

上面的函数接收 Foo 的一个副本，而**返回一个新的 Foo 对象**。

```
*// pass by reference*
func MutateFoo(f ***Foo**) error {
   f.Field1 = "New val"
   f.Field2 = 2
   return nil
}
```

上面的函数接收一个指向 Foo 的指针，然后**改变原始对象**。

这种按值调用和按引用调用的明显区别使您的意图显而易见，并减少了调用函数无意中改变传入对象的可能性(这是许多初级开发人员难以掌握的)。

正如麻省理工学院[总结](http://web.mit.edu/6.031/www/fa20/classes/08-immutability/) : *“可变性使得理解你的程序正在做什么变得更加困难，并且更难执行契约”*

此外，按值调用大大减少了垃圾收集器的工作，这意味着应用程序速度更快，内存效率更高。[这篇文章](https://www.forrestthewoods.com/blog/memory-bandwidth-napkin-math/)的结论是，指针追踪(从堆中检索指针值)比从连续堆栈中检索值要慢 10 到 20 倍。要记住的一个好的经验法则是:*从内存中读取的最快方法是顺序读取，这意味着将随机存储在 RAM 中的指针数量减少到最小*。

# 4.“延期”关键字

在 NodeJS 中，在我开始使用 [knex.js](https://knexjs.org/) 之前，我会在我的代码中手动管理数据库连接，方法是创建一个 DB 池，然后在每个函数中从池中打开一个新的连接，一旦所需的数据库 CRUD 功能完成，就在函数末尾释放连接。

这有点像维护的噩梦，因为如果我不在每个函数结束时释放连接，未释放的数据库连接的数量将会慢慢增加，直到连接池中不再有空闲连接，然后应用程序就会中断。

现实是，程序经常需要释放、清理和拆除资源、文件、连接等。所以 Go 引入了 **defer** 关键字作为管理这种情况的有效方法。

任何以 **defer** 开头的语句都会延迟其调用，直到周围的函数退出。这意味着你可以把你的清理/删除代码放在一个函数的顶部(很明显的地方)，知道一旦函数完成，它就会完成它的工作。

```
func main() {                        
    if len(os.Args) < 2 {   
        log.Fatal("no file specified")
    }  
    f, err := os.Open(os.Args[1])                        
    if err != nil {                         
        log.Fatal(err)                        
    }                        
    **defer f.Close()**                        
    data := make([]byte, 2048)                        
    for {                         
        count, err := f.Read(data)                                               
        os.Stdout.Write(data[:count])                        
        if err != nil {                          
            if err != io.EOF {                           
                log.Fatal(err)                          
            }                          
            break                         
        }                        
    }                       
}
```

在上面的例子中，文件关闭方法被延迟。我喜欢这种模式:在函数的顶部声明你的管家意图，然后忘记它，知道一旦函数退出，它就会完成它的工作。

# 5.Go 采用了函数式编程的最佳特性

函数式编程是一种高效和创造性的范式，谢天谢地，Go 采用了函数式编程的最佳特性。在 Go 中:

—函数是值，这意味着它们可以作为值添加到映射中，作为参数传递到其他函数中，设置为变量，并从函数中返回(称为“高阶函数”&经常在 Go 中使用 decorator 模式创建中间件)。

—可以创建和自动调用匿名函数。

—在其他函数内部声明的函数允许闭包(其中在函数内部声明的函数能够访问和修改在外部函数中声明的变量)。在惯用的 Go 中，闭包被广泛用于限制函数的作用域，并设置函数在逻辑中使用的状态。

```
func StartTimer (name string) func(){
    t := time.Now()
    log.Println(name, "started")
    return func() {
        d := time.Now().Sub(t)
        log.Println(name, "took", d)
    }
}func RunTimer() {
    stop := StartTimer("My timer")
    defer stop()
    time.Sleep(1 * time.Second)
}
```

以上是一个关闭的例子。“StartTimer”函数**返回一个新函数**，该函数通过**闭包**访问“t”值，该值在其出生范围内设置。然后，该函数可以将当前时间与“t”的值进行比较，从而创建一个有用的计时器。感谢 [Mat Ryer](https://twitter.com/matryer) 的这个例子。

# 6.Go 有隐式接口

任何读过关于[可靠](https://en.wikipedia.org/wiki/SOLID)编码和[设计模式](https://en.wikipedia.org/wiki/Software_design_pattern)的文献的人可能都听过这样一句口头禅*‘偏好组合胜过继承’。*简而言之，这表明你应该将你的业务逻辑分解到不同的接口中，而不是依赖于从父类继承的属性和逻辑。

另一个流行的是*“编程到一个接口，而不是实现”:*一个 API 应该只发布其预期行为(其方法签名)的契约，而不是关于该行为如何实现的细节。

这两者都指出了**接口**在现代编程中的至关重要性。

因此，毫不奇怪，Go 支持接口。事实上，接口是 Go 中唯一的抽象类型。

然而，与其他语言不同的是，Go 中的接口不是显式地实现的，而是隐式地实现的。具体类型不声明它实现接口。相反，如果该具体类型的方法集包含底层接口的所有方法集， **Go 认为该对象实现了接口**。

这种**隐式**接口实现(正式名称为结构化类型化)允许 Go 加强类型安全和解耦，保留了动态语言中展现的灵活性。

相比之下，显式接口将客户端和实现绑定在一起，使得在 Java 中替换依赖关系比在 Go 中要困难得多。

```
*// this is an interface declaration (called Logic)*
type Logic interface {
    **Process**(data string) string
} type LogicProvider struct {}*// this is a method called 'Process' on the LogicProvider struct*
func (lp LogicProvider) **Process**(data string) string {
    // business logic
}*// this is the client struct with the Logic interface as a property*
type Client struct {
    L Logic
}func(c Client) Program() {
    // get data from somewhere
    c.L.Process(data)
}func main() {
    c := Client {
        **L: LogicProvider{},** }
    c.Program()
}
```

在 *LogicProvider* 中没有声明任何东西来表明它符合*逻辑*接口。这意味着客户端在将来可以很容易地替换它的逻辑提供者，只要该逻辑提供者包含底层接口的所有方法集(*逻辑*)。

# 7.错误处理

Go 中错误的处理方式与其他语言有很大不同。简而言之，Go 通过**返回一个 error 类型的值作为函数**的最后一个返回值来处理错误。

当函数按预期执行时，为错误参数返回 **nil** ，否则返回**错误值**。然后，调用函数检查错误返回值，并处理错误，或者抛出自己的错误。

```
*// the function returns an int and an error*
func calculateRemainder(numerator int, denominator int) (**int, error**) {
   *// Error returned*
   if denominator == 0 {
      return 9, **errors.New("denominator is 0")** } *// No error returned*
   return numerator / denominator, **nil** }
```

Go 以这种方式运行是有原因的:它迫使编码人员考虑异常并正确处理它们。传统的 try-catch 异常还会在代码中添加至少一个新的代码路径，并以难以理解的方式缩进代码。Go 更喜欢将“快乐路径”视为非缩进代码，任何错误都会在“快乐路径”完成之前被识别并返回。

# 8.并发

可以说，并发性是 Go 最著名的特性，它允许在机器或服务器上的多个可用内核上并行运行处理。当独立的进程**不相互依赖**(不需要按顺序运行)并且时间性能至关重要时，并发最有意义。I/O 需求经常是这种情况，在这种情况下，对磁盘或网络的读写比除了最复杂的内存中进程之外的所有进程都要慢几个数量级。

函数调用前的' *go* '关键字将同时运行该函数。

```
func process(val int) int {
   *// do something with val*
}*// for each value in 'in', run the process function concurrently, 
// and read the result of process to 'out'*
func runConcurrently(in <-chan int, out chan<- int){
   **go** func() {
       for val := range in {
            result := process(val)
            out <- result  
       }
   }
}
```

Go 中的并发性是一个深入且相当高级的特性，但是在它有意义的地方，它提供了一个有效的方法来确保程序的最佳性能。

# 9.Go 标准库

Go 有一个“包含电池”的哲学，现代编程语言的许多要求都被放入标准库中，这使得程序员的生活简单多了。

如前所述，Go 是一种相对年轻的语言，这意味着现代应用程序的许多问题/需求都在标准库中得到了解决。

首先，Go 为网络(特别是 HTTP/2)和文件管理提供了世界级的支持。它还提供本机 JSON 编码和解码。因此，设置一个服务器来处理 HTTP 请求和返回响应(JSON 或其他)是非常简单的，这解释了 Go 在基于 REST 的 HTTP web 服务开发中的受欢迎程度。

正如 Mat Ryer 所指出的，标准库是开源的，是学习围棋最佳实践的好方法。

# 10.调试:Go 游乐场

在任何语言中进行调试都是一项关键要求。大多数语言依赖第三方在线工具或智能 ide 来提供调试工具，允许开发人员快速检查他们的代码。Go 已经提供了一个免费的在线工具—【https://play.golang.org】[Go Playground，在这里你可以尝试和分享小程序。这是一个非常有用的工具，它使调试变得非常简单。](https://play.golang.org)

# 参考

[](https://golang.org/doc/) [## 证明文件

### Go 编程语言是一个开源项目，旨在提高程序员的工作效率。围棋富有表现力，简洁…

golang.org](https://golang.org/doc/) [](https://github.com/learning-go-book) [## 学习围棋

### 学习围棋的 GitHub repo。Learning Go 有 17 个可用的存储库。在 GitHub 上关注他们的代码。

github.com](https://github.com/learning-go-book) [](https://www.oreilly.com/library/view/learning-go/9781492077206/) [## 学习围棋

### Go 正迅速成为构建 web 服务的首选语言。虽然有大量的教程可用…

www.oreilly.com](https://www.oreilly.com/library/view/learning-go/9781492077206/)