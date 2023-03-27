# 生成器和协程的流类型

> 原文：<https://itnext.io/flow-types-for-generators-and-coroutines-ddee13c30ba1?source=collection_archive---------8----------------------->

自从 ECMAScript 6 引入了`yield`关键字，协程变得更加常见。最著名的例子可能是用于并发的`async` / `await`框架，但是协程也构成了 [redux-saga](https://redux-saga.js.org/) 的主干，并且已经进入了 [bluebird](http://bluebirdjs.com/docs/getting-started.html) 。

似乎很少有关于如何将[流](https://flow.org/en/docs/)类型添加到生成器或协程的文档。这篇文章通过给出许多不同类型生成器的例子来缓解这个问题。代码测试在 [Github](https://github.com/pbugnion/flow-types-for-coroutines) 上。

# 生成器与协程

协程和生成器是非常不同的概念。*生成器*让你创建消费者看来像迭代器的函数。

协程是传统函数概念的扩展。一个函数将通过`return`语句把控制权交还给它的调用者一次。协程可以通过调用`yield`将控制权交还任意次。当协程让步时，它的内部状态被保留。

生成器的消费者将根据需要从生成器中获取数据。协程的调用者将*将*数据推入协程。

生成器和协程经常被混为一谈，因为它们使用相同的底层机制。例如，在 Python 和 JavaScript 中，都使用`yield`关键字来移交控制权。生成器和协程都可以暂停，稍后再继续。

尽管有这些相似之处，但它们在不同的上下文中使用:生成器用于轻松创建迭代器。协程用于引入并发或复杂的控制流。

如果你不熟悉协程和生成器，我发现*探索 JavaScript* 的[这一章](http://exploringjs.com/es6/ch_generators.html#sec_overview-generators)非常有用。

# 一种类型来统治他们

因为协程和生成器使用相同的底层机制，所以在 Flow 中只有一个通用类型:

```
Generator<Yield, Return, Next>
```

这里:

*   `Yield`是生成器生成的数据类型。如果您的生成器/协程中有类似于`yield "my-string"`的语句，`Yield`将会是`string`。
*   `Return`是生成器返回语句的类型。如果你有一个类似于`return "my-string"`的语句，`Return`的类型就会是`string`。
*   `Next`是由`yield`注入函数的值的类型。如果你有一个类似于`const nextItem: string = yield`的语句，`Next`的类型就会是`Yield`。

```
**function*** example: Generator<string, boolean, number> {
  **const** toYield = 'this is a string'
  **const** received: number = **yield** toYield
  **return** false
}
```

![](img/ec0314f338ce88516428f917f2bff499.png)

# 打字生成器

生成器发布数据。从简单(也很无聊)开始，让我们创建一个发布偶数的生成器:

```
**function*** evens(): Generator<number, **void**, **void**> {
  **let** current = 0;
  **while** (**true**) {
    **yield** current;
    current += 2;
  }
}
```

在本例中，我们:

*   产量数字
*   不要退回任何东西
*   当我们让步时，不要期望注入任何东西(没有变量绑定到`yield`的左边)。

因此，我们的发电机类型是`Generator<number, void, void>`。对于生成器，`Next`类型参数通常是`void`:很少将值注入到生成器中(尽管我们确实看到许多人为的例子，人们试图重新开始一个斐波那契数列)。

让我们试试稍微复杂一点的。我们可以创建一个递归遍历文件树的生成器:

```
**import** fs from 'fs'
**import** path from 'path'

**function*** walkDirectories(root: string): Generator<string, **void**, **void**> {
  **for** (**const** name **of** fs.readdirSync(root)) {
    **const** filePath = path.join(root, name)
    **const** stat = fs.lstatSync(filePath)
    **if** (stat.isFile()) {
      **yield** filePath
    } **else** **if** (stat.isDirectory()) {
      **yield*** walkDirectories(filePath)
    }
  }
}
```

我们读取根目录的内容，对于每一项，如果它是一个文件，就发布它；如果它是一个目录，就递归地发布它的内容。我们仍然不返回任何东西，或者绑定到`yield`语句，所以我们的生成器仍然是`Generator<?, void, void>`。我们要么直接产生字符串，要么子目录中的任何内容也是字符串。因此`Yield`的类型参数为`string`，构成整体类型`Generator<string, void, void>`。

# 输入消耗生成器的函数

我们现在有了函数`walkDirectories`的类型。我们发电机的消费者呢？让我们编写一个函数，按文件扩展名对文件进行分组，并计算每个扩展名的文件数:

```
**function** countFilesByExtension(files): {[string]: number} {
  **const** total = {};
  **for** (**const** f **of** files) {
    **const** extension = path.extname(f)
    **const** currentCount = total[extension] || 0
    total[extension] = currentCount + 1
  }
  **return** total
}
```

在这里，参数`files`是我们的生成器。`files`是什么类型的？当然，我们可以将其输入为:

```
// overly specificfunction countFilesByExtension(
  files: Generator<string, void, void>
): {[string]: number} { // ...
```

或者，更好的说法是:

```
// still overly specific function countFilesByExtension(
  files: Generator<string, any, any>
): {[string]: number} { // ...
```

但实际上，我们只关心`files`是否可以被迭代。例如，传入一个数组是合法的。因此，推荐使用的类型是`Iterator`:

```
function countFilesByExtension(
  files: Iterator<string>
): {[string]: number} { // ...
```

这将与我们的生成器一起工作，因为`Generator<T, any, any>`实现了`Iterator<T>`接口。

我们可以如下使用整个管道:

```
**import** os from 'os'

**const** rootDirectory = os.homedir()

console.log(
  countFilesByExtension(
    walkDirectories(rootDirectory)
  )
)
```

![](img/7cf88693a9de97e23eb9640c9579f330.png)

# 键入协程

在围绕生成器构建的程序中，信息是由消费者拉取的。在围绕协程构建的程序中，信息被推送给消费者。让我们创建一个管道，打印出给定目录中的文件。我们将在节点的`[fs](https://nodejs.org/api/fs.html)`模块中包装异步的、基于回调的函数。本节中的示例大致基于*探索 JavaScript* 的[生成器部分](http://exploringjs.com/es6/ch_generators.html#sec_overview-generators)。

通常，协程管道中的阶段采用一个`target`参数来标识管道中的下一个阶段。因此，如果我们有一个函数将类型为`T`的数据推入管道，那么该函数的类型签名将是:

```
function (target: Generator<any, any, T>): void { /* ... */ }
```

让我们从编写源函数开始。源本身通常不是一个协程，但是它将一个协程作为目标。

我们将使用 Node 的`fs.readdir`将一个给定目录中所有条目的列表推送到目标:

```
**const** pushFiles = **function**(
  directory: string,
  target: Generator<any, any, string>
): **void** {
  fs.readdir(
    directory,
    { encoding: 'utf-8' },
    (error, fileNames) => {
      **if** (error) {
        **throw** error
      } **else** {
        **for** (**const** fileName **of** fileNames) {
          **const** filePath = path.join(directory, fileName)
          target.next(filePath)  *// push file paths to a coroutine*
        }
      }
    }
  )
}
```

这里，目标必须是一个`Generator<any, any, string>`:它必须接受 yield 语句左侧的字符串。目标将包括如下陈述:

```
const file: string = yield
```

接下来，让我们创建一个协程，它只记录传递给它的所有内容:

```
**const** log = **function*** (): Generator<**void**, **void**, any> {
  **while** (**true**) {
    **const** item: any = **yield**
    console.log(item)
  }
})
```

我们的协程类型是`Generator<void, void, any>`:

*   `Yield`是无效的，因为它不产生任何东西
*   `Return`无效，因为函数不返回
*   `Next`是`any`，因为它接受来自上游的任何类型。

有点烦人的是，我们不能不初始化就直接使用我们的`log`协程，因为协程需要前进到第一个产出。我们需要写:

```
**const** logCoroutine = log()
logCoroutine.next() *// initialize coroutine*

*// Read files in home directory and push them to logCoroutine*
pushFiles(os.homedir(), logCoroutine)
```

为了减少这种样板文件，通常编写一个助手函数来创建生成器，通过调用它的`next`方法来初始化它并返回它:

```
**function** coroutine(generatorFunction) {
  **return** **function**(...args) {
    **const** generator = generatorFunction(...args)
    generator.next()
    **return** generator
  }
}
```

键入这个 helper 方法有点棘手。我们真正要告诉 Flow 的是，我们返回一个与`generatorFunction`同类型的函数。我们还需要指定`generatorFunction`是一个接受变量类型并返回生成器的函数。例如，我们可以写:

```
**function** coroutine<G: Generator<any, any, any>>(
  generatorFunction: ((...args: Array<any>) => G)
) {
  **return** **function**(...args: Array<any>): G {
    **const** generator = generatorFunction(...args)
    generator.next()
    **return** generator
  }
}
```

这里，我们指定`generatorFunction`必须是返回任何生成器的函数，并且我们的协程函数将自己返回返回该生成器的函数。不幸的是，在 Flow 中没有好的方法来对多态变量函数进行类型化(见[本期](https://github.com/facebook/flow/issues/1251))，所以我们在`generatorFunction`的参数中失去了类型安全性。我们可以通过显式地键入协程来避免这种泄漏到程序的其他部分。我们的`log`功能现在变成了:

```
**const** log: () => Generator<**void**, **void**, any> = coroutine(**function*** () {
  **while** (**true**) {
    **const** item: any = **yield**
    console.log(item)
  }
})
```

因此，通过将类型移至`log`，而不是直接基于`coroutine`的参数，我们可以在程序的剩余部分以类型安全的方式使用`log`。

我们现在不再需要初始化我们的协程来使用它了:

```
pushFiles(os.homedir(), log())
```

最后，让我们在管道中创建一个中间步骤，过滤掉不是简单文件的任何内容:

```
**const** isFile: (Generator<any, any, string> => Generator<**void**, **void**, string>) =
  coroutine(**function*** (target) {
    **while** (**true**) {
      **const** fileMaybe: string = **yield**
      fs.lstat(fileMaybe, (error, stat) => {
        **if** (error) {
          **throw** error
        } **else** **if** (stat.isFile()) {
          target.next(fileMaybe)
        }
      })
    }
  })
```

我们的函数接受任何接受字符串的协程作为其目标(因为这是它推出的内容)。其返回类型为`Generator<void, void, string>`:

*   `Yield`是空的，因为它不产生任何东西
*   `Return`无效，因为函数不返回
*   `Next`是`string`，因为它期望从上游获得字符串。

`Generator<void, void, string>`将满足我们在`pushFiles`中的`target`参数上指定的约束`Generator<any, any, string>`。

我们现在可以构建我们的协程管道:

```
pushFiles(os.homedir(), isFile(log()))
```

# 结论

流量很大。它捕捉了大量单元测试或手工 QA 会遗漏的错误。这也使得新用户更容易阅读代码。

不幸的是，对于更复杂的类型，很少有好的例子。如果您很难将流类型添加到一个构造中，请写下来，以便社区可以从中受益！

# 更多！

有关更多信息:

关于发电机的[流程文件](https://flow.org/blog/2015/11/09/Generators/)是有用的进一步阅读材料。

我已经提到过[探索 JavaScript](http://exploringjs.com/es6/ch_generators.html#sec_overview-generators) 。代码样本可以在 [GitHub](https://github.com/rauschma/generator-examples/tree/gh-pages/node) 上获得。

最后，为了更深入地理解协程，David Beazley 有一些很棒的幻灯片。这些是针对 Python 的，但是概念是非常可移植的。

*最初发表于*[T5【pascalbugnion.net】](https://pascalbugnion.net/blog/flow-types-for-generators-and-coroutines.html)*。*