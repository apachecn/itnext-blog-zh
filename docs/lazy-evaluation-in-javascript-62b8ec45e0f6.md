# JavaScript 中的惰性求值

> 原文：<https://itnext.io/lazy-evaluation-in-javascript-62b8ec45e0f6?source=collection_archive---------1----------------------->

![](img/2b7da81fcc2f1c56942f8e5ac5e8427d.png)

照片由 [Manja Vitolic](https://unsplash.com/@madhatterzone?utm_source=medium&utm_medium=referral) 在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄

在编程语言设计中，要做的许多架构决策之一涉及到语言应该如何评估它的表达式。它应该在代码中声明它们的时候对它们进行全面评估吗？还是应该将这种评估延迟到实际需要这些值的时间点(*例如*，打印到控制台或写入数据库)？

这组问题的两个主要答案是设计一种严格或非严格的语言。严格求值意味着当表达式到达时，它的值就被确定了，而不管这个值何时(甚至是否)会被使用。因此`3 + 5`总是立即减少到`8`。而非严格，则相反。只有当代码中确实需要表达式的值时，它才会减少`3 + 5`,并将加法运算延迟到那个时候。通常我们使用各自的术语“渴望”和“懒惰”来给这些评估策略取通俗的名字。

大多数编程语言(包括 JavaScript)都倾向于渴望。这意味着如果我们调用一个函数`sequence`定义为

```
const sequence = (l, u) =>
    Array(u - l + 1)
        .fill(0)
        .map((_, idx) => l + idx);
```

它在`l`和`u`之间创建一个整数数组，然后将每个数字映射到它的平方

```
const squares = sequence(1, 5).map(x => x ** 2);
```

在映射数组中的每个值之前，我们的语言将全面评估`sequence(1, 5)`。然后，我们剩下一个数组`squares`，在我们继续下一行代码之前，它已经被完全评估过了。

在 eager 语言中，当使用具有复杂操作的大型数据结构时，我们应该小心。如果要执行昂贵的计算，需要多次遍历数组，并且必须分配和释放大量内存来支持所有这些操作，那么映射数千个数组项会使程序的性能下降。如果所有的工作都完成了，而程序只需要其中的几个值(如果有的话),这是非常浪费的。

当然，有很多方法可以缓解这些问题。人们可以通过使用[换能器](https://medium.com/javascript-scene/transducers-efficient-data-processing-pipelines-in-javascript-7985330fe73d)将大型阵列中的多次迭代减少到一次。人们可以操纵控制流，以防止在程序的生命周期中过早地评估许多昂贵的计算。或者可以使用突变来就地改变数组，而不是为任何操作创建新的副本。这些都没有回避这样一个事实，即在某个时刻，程序会到达这样一个表达式，并在继续下一条指令之前对其进行充分评估。

非严格语言没有这些限制。通过推迟到必须评估一个值的最后一分钟，以及采取只评估必要的值的方法，许多有趣的特性成为语言中的一等公民。为了说明这些，我们将使用 Haskell，一种最著名的惰性语言。

在 Haskell 中，创建无限列表很简单:

```
infiniteList = [1..]
```

请注意，我们永远不能用一种热切的语言来创建它，因为无限列表在被绑定到一个变量之前必须被完全评估，这将导致环境在计算列表中的下一个数字时挂起(直到它耗尽内存并崩溃)。

但是真正强大的是，我们可以在这个无限列表上执行许多操作，而不需要立即对它们进行评估。

```
-- Filter out odds
infiniteEvens = filter even infiniteList-- Add 5 to each item
infiniteEvensPlus5 = map (+ 5) infiniteEvens
```

最终，我们可能想要获取这个列表的一部分，以便可以在控制台中显示它:

```
first5Elements = take 5 infiniteEvensPlus5
show first5Elements -- "[7,9,11,13,15]"
```

尽管我们创建了一个无限列表，指定了一个过滤器，然后在该列表上指定了一个映射操作，并对其中的五个元素进行了强制求值，但是我们的操作没有一个遍历了该列表的整个长度——它们只是尽可能地对列表中的前五个元素进行求值。然而，列表的其余部分还没有被评估，等待着需要列表的另一部分的东西出现。

# JavaScript 中的惰性求值

随着 JavaScript 成为一种渴望的语言，我们可以问这样一个问题，我们是否可以以某种方式达到我们在 Haskell 中看到的相同效果。我们希望能够创建一个(潜在的)无限的数据结构，并在其上指定除非绝对必要否则不会执行的操作。

目前 JavaScript 中没有内置的数据结构适合这个目的。数组、对象、集合、地图、*等。*，都是由严格评价的必然性所限定的。为了避开这种严格性，我们需要创建我们自己的数据结构，这样它就不仅仅是简单地*存储*值(比如上面所做的),而且还有一种方式*根据需要任意程度地生成*值。

幸运的是，JavaScript 自带了一个机制来实现这一点，这将作为我们惰性数据结构的基础:[生成器函数](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function*)。

虽然我们不会深入解释生成器函数(或它们返回的值)，但我们会注意到，这些函数与常规函数不同，常规函数可以返回任何值，而生成器函数则返回一个特殊的生成器对象。该对象允许控制程序在需要时执行生成器函数体中定义的代码，并且该代码甚至可以在执行过程中在指定的断点处暂停，它可以在将来某个不确定的时间(如果有的话)从该断点恢复执行。正是暂停能力的这一特性，我们将利用它来实现懒惰。

## 惰性序列

虽然生成器将用于数据结构本身，但它们也非常适合表示我们的惰性列表的基本构建块:一个惰性的无限数字序列。

```
const range = (l, u = Infinity) => function* () {
    while (true) {
        if (l < u) {
            yield l++;
            continue;
        } return l;
    }
};
```

这个`range`函数的构造将为我们如何设计我们的列表打下基础。首先，应该注意它返回一个生成器函数。这将是贯穿始终的一个共同主题——包装懒惰总是涉及到创建生成器函数，懒惰的消费者可以在下游利用这些函数。第二，惰性序列不必是无限的，所以我们需要列出当我们的惰性列表执行它的一些操作时，由生成器对象产生的值是已经被`return`处理还是仅仅被`yield`处理

## 懒惰列表

使用我们上面的 Haskell 代码作为灵感，我们希望能够有一个列表，我们可以映射，过滤，切片，*等。*，但是服从某一组约束。

首先，除非绝对必要，否则我们不想执行任何操作。因此，尽管`Array.prototype.map`会立即用新值创建一个新数组，但我们只希望我们的列表的 map 操作只在我们需要从列表中获取值时运行。第二，我们只想尽可能多地评估列表以获得我们的价值，仅此而已；如果我们想得到列表的第三个值，我们必须创建第一个和第二个值，但绝对不能创建第四个值。第三，我们不希望在同一个列表上多次执行相同的操作，所以我们应该缓存构建的任何值。

在构建动作方面，我们将采用基于类的方法，这样我们就可以将 Haskell 操作写成

```
lazyList
    .filter(n => (n % 2) === 0)
    .map(n => n + 5)
    .take(5);
```

由此，我们可以通过从列表中获取，来强制评估这五个值中的任何一个。

我们类的框架将会是这样的:

```
class LazyList {
    #generator;
    #generatedValues = [];
    #done = false; constructor(generatorFn) {
        this.#generator = generatorFn();
    } get(n) {} take(n) {} map(fn) {} filter(fn) {}
}
```

它将有三个私有属性:

*   `#generator`保存表示要创建的序列的生成器对象，该对象最终将从我们上面创建的惰性`range`序列中派生出来
*   `#generatedValues`是一个已经计算过的值的缓存，所以如果我们从一个列表中得到第十个值，我们将把它存储在这个数组中，这样我们就可以在以后的任意次数中获取它，而无需重新计算
*   `#done`将跟踪我们的生成器对象是否已经完成；稍后我们将看到为什么这是一个必要的属性

我们的`get`方法将是列表中唯一强制评估值的方法——否则我们无法获得第一个、第二个或第`n`个值。它将从`#generatedValues`数组中获取数据，因此它的部分操作是将数组填充到传递给它的任何参数`n`中。

让我们创造`get`:

```
class LazyList {
    #generator;
    #generatedValues = [];
    #done = false; constructor(generatorFn) {
        this.#generator = generatorFn();
    } #next() {
        const { value, done } = this.#generator.next(); if (done) {
            this.#done = true;
        } if (value !== $SKIP) {
            this.#generatedValues.push(value);
        }
    } get(n) {
        while (this.#generatedValues.length <= n && !this.#done) {
            this.#next();
        } return this.#generatedValues[n];
    }
}
```

注意，上面的`get`检查已经生成了多少个值，并生成任何缺失的值，直到第`n`个位置。如果底层生成器在到达该位置之前已经完成，它将提前中止，在这种情况下，它将返回`undefined`(因为我们要求的值超出了`#generatedValues`数组的界限)。

在`get`使用的`#next`方法中，我们确保不将任何标记为`$SKIP`的值视为有效。很少有情况会产生这种情况，所以这是一种边缘情况。但是我们将在下面看到，如果没有它，我们可能会有意想不到的行为。

对于我们的每一个懒惰方法`take`、`map`和`filter`，我们有两个逻辑要实现:获取/映射/过滤的实际操作，以及每个操作应该如何与`LazyList`类交互。

在操作方面，为了保持惰性，我们希望创建接受生成器并返回执行所需操作的生成器的函数。这些人都不需要知道*他们正在接收什么*生成器，尽管他们确实需要尊重那个生成器指定的`done`属性。由于这些函数的操作没有外部环境，所以我们的三种方法的所有逻辑都可以提取为效用函数。

```
const take = (n) => (generator) => function* () {
    while (n-- > 0) {
        const { value, done } = generator.next(); if (done) {
            return value;
        } yield value;
    }
};
```

简而言之，我们对`generator`参数的值循环`n`次，然后将它们丢弃。我们必须注意它是否已经完成(因为我们不知道它是否创建了一个有限或无限的序列),并且同样通过选择`return`或`yield`来传递信息。

映射和过滤是相似的。

```
const map = (fn) => (generator) => function* () {
    while (true) {
        const { value, done } = generator.next(); if (done) {
            return fn(value);
        } yield fn(value);
    }
};const $SKIP = Symbol('skip');const filter = (fn) => (generator) => function* () {
    while (true) {
        const { value, done } = generator.next(); if (done && fn(value)) {
            return value;
        } if (done && !fn(value)) {
            return $SKIP;
        } if (!fn(value)) {
            continue;
        } yield value;
    }
};
```

我们可以看到我们的过滤函数利用了前面介绍的`$SKIP`值。原因如下。通过过滤，我们的谓词函数(`fn`)决定了`generator`给出的值应该被产生还是被丢弃。如果`fn`返回`false`，我们希望继续遍历生成器的值，直到它返回`true`为止。如果由于某种原因，我们的生成器完成了(因为它是有限的)，我们仍然需要检查那个最终值，看看它是否应该被传递。最好的情况是它应该是，我们可以简单地`return`这个值。然而，有时它应该*而不是*被传递(例如过滤掉范围`[1, 3]`中的奇数值，其中`3`是最终值并且也是奇数值，不应该被返回)。在这种情况下，我们仍然需要给*一些东西*，所以我们返回一个特殊的符号`$SKIP`，下游处理会考虑这个符号。

完成这三个操作的纯实现后，他们现在需要与`LazyList`类交互。概括来说，我们将执行以下步骤:

*   将类打包成一个生成器函数
*   将该生成器函数提供给适当的实用函数
*   将实用程序返回的生成器函数输入到新的`LazyList`实例中。

当将类实例转换为生成器时，请记住，我们需要考虑已经生成的值，并在可用时使用这些值，并且只生成我们必须生成的新值。

我们将在我们的类中添加一个新的实用方法，将其重新打包为一个生成器函数。

```
class LazyList {
    #generator;
    #generatedValues = [];
    #done = false; constructor(generatorFn) {
        this.#generator = generatorFn();
    } #next() {
        const { value, done } = this.#generator.next(); if (done) {
            this.#done = true;
        } if (value !== $SKIP) {
            this.#generatedValues.push(value);
        }
    } #asGenerator() {
        function* toGenerator() {
            let cursor = 0; while (true) {
                if (!Reflect.has(this.#generatedValues, cursor)) {
                    this.#next();
                } if (this.#done) {
                    return this.#generatedValues[cursor++];
                }

                yield this.#generatedValues[cursor++];
            }
        } return toGenerator.apply(this);
    }
}
```

我们创建一个名为`toGenerator`的新生成器函数，它将`yield`当前惰性列表类的每个值(在需要时通过调用`#next`来生成它)，直到底层生成器用完为止。当然，如果我们正在生成一个像 fibonacci 数一样的无限序列，我们将永远不会耗尽生成器，所以这将是我们的懒惰实现是否工作的真正测试:如果它挂起，我们将尝试评估整个底层列表，而不仅仅是我们下一个操作的本质。

我们对类实例本身执行`apply`这个生成器函数，以便它在访问它的一些私有属性时有正确的`this`上下文。

结果方法本身是简单的单行实现。整个班级都会

```
class LazyList {
    #generator;
    #generatedValues = [];
    #done = false; constructor(generatorFn) {
        this.#generator = generatorFn();
    } #next() {
        const { value, done } = this.#generator.next(); if (done) {
            this.#done = true;
        } if (value !== $SKIP) {
            this.#generatedValues.push(value);
        }
    } #asGenerator() {
        function* toGenerator() {
            let cursor = 0; while (true) {
                if (!Reflect.has(this.#generatedValues, cursor)) {
                    this.#next();
                } if (this.#done) {
                    return this.#generatedValues[cursor++];
                }

                yield this.#generatedValues[cursor++];
            }
        } return toGenerator.apply(this);
    } get(n) {
        while (this.#generatedValues.length <= n && !this.#done) {
            this.#next();
        } return this.#generatedValues[n];
    } take(n) {
        const generator = take(n)(this.#asGenerator());
        return new LazyList(generator);
    } map(fn) {
        const generator = map(fn)(this.#asGenerator());
        return new LazyList(generator);
    } filter(fn) {
        const generator = filter(fn)(this.#asGenerator());
        return new LazyList(generator);
    }
}
```

## 测试懒惰列表

现在我们有了懒惰生成的列表，我们应该测试我们的主要目标是否已经达到。记住我们的三个目标是:

*   除非绝对必要，否则不做评估
*   没有多余的评估
*   没有值会被评估一次以上

为了测试这一点，我们将在相关的函数和方法中添加控制台日志，并记录这些控制台日志的执行时间。

```
const plus1 = (n) => {
    console.log('Adding one to', n);
    return n + 1;
};const oneToFive = new LazyList(range(1, 5));
const oneToFivePlus1 = oneToFive.map(plus1);// No console logging yet.oneToFivePlus1.get(0);// log: 'Adding one to 1'oneToFivePlus1.get(0);// No console logoneToFivePlus1.get(2);// log: 'Adding one to 2'
// log: 'Adding one to 3'
```

从这段摘录中我们可以看到，我们的三个条件都得到了满足。在我们的素数序列上的映射不执行任何直接的计算。获得序列中的第`0`项仅触发了一次评估。并且获取第`0`和第`2`项时，除了各自的位置之外，不会评估任何东西。

我们也可以建立更复杂的操作。

```
const notDivisibleBy3 = new LazyList(range(1))
    .map(n => {
        console.log('Adding one to', n);
        return n + 1;
    })
    .filter(n => {
        console.log('Evaluating', n);
        return (n % 3) !== 0;
    })
    .take(5);notDivisibleBy3.get(0); // 2// log: 'Adding one to 1'
// log: 'Evaluating 2'notDivisibleBy3.get(0); // 2// No console lognotDivisibleBy3.get(2); // 5// log: 'Adding one to 2'
// log: 'Evaluating 3'
// log: 'Adding one to 3'
// log: 'Evaluating 4'
// log: 'Adding one to 4'
// log: 'Evaluating 5'notDivisibleBy3.get(5); // undefined// Similar console logs as above but for numbers 5 - 8
```

注意，当我们得到不能被 3 整除的序列的第`2`个元素时，我们做了三次评估。其中一个数字*可被 3 整除，因此我们将其作为无效序列丢弃，并生成两个适当的值(4 和 5)。*

## 关于懒惰名单的思考

从我们的惰性列表数据结构可以看出，我们可以定义相当有效的管道，允许我们在无休止的输入流上创建复杂的操作系统。通过缓存评估值，并且每次只处理我们需要的量，我们可以在相当昂贵的操作上获得相当好的性能。

尽管我们已经实现了所有这些好处，但值得注意的是，最终我们并没有真正的懒惰评估(撇开命名不谈)。我们可以称我们的列表为“懒惰的”,但是在每个操作的每一步，运行时已经完全评估了每个表达式。只是那个运行时也有一些特定的构造，这些构造模拟了延迟求值的编程语言的某些特性。不幸的是，因为评估策略是语言的基本方面，懒惰不是 JavaScript 的首要元素，所以模拟懒惰与懒惰不是一回事。我们仍然必须编写在适当的时候调用评估的代码，因为运行时本身没有执行评估的内置概念。

这意味着 Haskell 可以做的非常符合人体工程学的事情，我们只能通过变通办法和额外的实现代码以及增加的复杂性来做(当同时执行多个操作时，我们的懒惰列表必须使用的生成器函数层会变得非常复杂)。

虽然我们只能粗略地估计 Haskell 能做的事情，但我们在 JavaScript 中拥有这样的特性仍然是一件好事。能够定义一组只对少量值调用(或者根本不调用)的昂贵的操作，仍然为运行时引擎执行其他重要任务节省了宝贵的时间。

列表也只是我们可以惰性生成的许多东西中的一种。我们可以扩展我们的代码来生成无限的树或分形，甚至是近似的可计算的实数，只要我们可以为我们试图生成的任何东西编码“下一次迭代”的概念。

这里是我们为惰性列表生成的代码的[要点](https://gist.github.com/ryandabler/f4f19256183bec068bcb4e2094678f5f)，以及一些在 Haskell 等语言中常见的列表操作方法，我们没有时间在本文中开发。