# 用 JavaScript 映射“未来”值，这是一种更实用的方法。

> 原文：<https://itnext.io/mapping-future-values-in-javascript-to-avoid-promise-nesting-embrace-functional-techniques-9e44bc829403?source=collection_archive---------1----------------------->

![](img/274f0bf07f8b7bd2c16ca8459f313976.png)

许多 JavaScript 开发人员不再直接处理*承诺*了，像 RxJS 和/或 async/await 语言支持已经把他们从这种情况中解救了出来。但是世界上许多开发者还没有那么幸运，出于不同的原因，他们仍然直接使用*承诺*。不管这些原因，如果你属于这一类，我今天想帮助你把代码写得更好一点。我们将使用函数式编程技术，但是为了不造成挫折和混乱，或者过于理论化，我将远离像*单子*、*函子*、*态射*等术语。

在这篇文章中，我将使用函数式编程技术，它可以引导您获得更健壮、更易于维护的代码库。我不会像在这里一样谈论迭代器/生成器([https://it next . io/iterators-and-Generators-do-have-a-place-in-modern-JavaScript-d4cb 589 b 491](/iterators-and-generators-do-have-a-place-in-modern-javascript-d4cb589b491))。我将使用直截了当的承诺，以及一些功能性的“魔法”。

有一点我想从一开始就诚实地说，承诺在本质上是不纯的，所以我不会使用像纯函数式编程这样的术语，或者暗示这里显示的是函数式编程，我会将其视为“函数式技术”。让我们开始吧。

为了简单起见，让我们使用下面的内存数据:

```
const people = [{
    id: 1,
    firstName: 'John',
    age: 34
  }, {
    id: 2,
    firstName: 'Rose',
    age: 25
}];const cars = [{
    ownerId: 1,
    make: 'Ford',
    year: 1988
  }, {
    ownerId: 2,
    make: 'Audi',
    year: 2001
}];
```

我们将尝试解决的问题是:给定一个人的 id，需要在*控制台*中打印其唯一汽车的品牌。

**传统的许诺方式:**

让我们声明两个函数，它们将使用上述变量，但也可以从数据库或网络调用中检索信息。

```
const getPersonById = *id* => *Promise*.resolve(people.filter(*p* => p.id === id)[0]);const getCarByOwnerId = *id* => *Promise*.resolve(cars.filter(*c* => c.ownerId === id)[0]);
```

这两个方法都返回一个*承诺，*因为这是 **fetch** 函数([https://developer . Mozilla . org/en-US/docs/Web/API/Fetch _ API/Using _ Fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch))或者 jQuery.ajax()或者 Angular 1.x 的 http.get()等将返回的内容。

传统的代码应该是这样的:

```
getPersonById(personId)
  .then(*person* => getCarByOwnerId(person.id))
  .then(*car* => car.make)
  .then(*console*.log)
  .catch(*e* => *console*.log(`The following error occurred: ${e}`));
```

**更实用的方式:**

我们将为 **Promise.prototype** 添加一个 *Map* 函数，这将允许我们甚至在解决 *Promise* 之前对其进行转换，类似于“以这种方式转换未来值……”。

```
*Promise*.*prototype*.map = function (*fn*) {
  return **new** *Promise*((*resolve*, *reject*) => *this*.then(*x* =>                
  resolve(fn(x))).catch(*e* => reject(e)));
};
```

如您所见，该函数将采用一个“映射”函数，并将返回一个新的*承诺*，该承诺将解析通过对原始*承诺*将要解析的值应用转换函数而得到的值。它还确保在解析该原始*承诺*时出错的情况下拒绝*承诺*。我们如何使用它？

```
*Promise*.resolve(3).map(*x* => x + 2).then(*console*.log);
```

我们创建了一个解析为值 3 的*承诺*，然后指定我们希望在它解析时在未来值(3)上加 2。结果，一个新的*承诺*在被解决后，将给出值 5 (2 + 3)。在这一点上，我们只是改变了*然后*为*地图，在这篇文章的后面*你会找到我们这样做的原因。

让我们开始采取一些功能方法。我本来打算添加一个对*ramda js*([https://ramdajs.com/](https://ramdajs.com/))的引用，但是后来我意识到很多团队(或者团队经理)都怀疑(出于某种原因)将新的库引入到现有的项目中。所以我要用的来自 *ramdajs* 的函数，我们将自己实现。只要意识到它们不会像 ramdajs 的那么健壮，但是它也能帮助我们不留下任何想象的空间。

```
const prop = *propName* => *obj* => obj[propName];
```

上述名为 **prop** 的函数将获取一个属性名称，并将返回一个新函数，当通过传递一个*对象*调用该函数时，将返回给定*对象*的属性值。这怎么可能呢？让我们看看:

```
const makeOf = prop("make");
const make = makeOf(cars[0]);*console*.log(make);// 'Ford'
```

让我们用 *map* 和 *prop* 函数来一点点重构代码。

```
getPersonById(personId)
  .map(*person* => getCarByOwnerId(person.id))
  .map(prop('make'))
  .then(*console*.log)
  .catch(*e* => *console*.log(`The following error occurred: ${e}`));// 'Ford'
```

这里需要注意的一点是，在 *getPersonById* 和 *getCarByOwnerId* 中，我们仍然分别引用了变量 *people* 和*cars**、*，这超出了它们的定义，这在函数式编程中是被禁止的(被称为一种副作用)。我们稍后会解决这个问题。

有些库，比如 folk tale([https://folktale.origamitower.com/](https://folktale.origamitower.com/))，已经给了你处理“未来”值的类型，让你像我们这里一样映射它们；在民间故事的情况下叫做*任务*。顺便说一下，这种方式更有力量，让你做的不仅仅是一个承诺。无论如何，让我们更进一步。

在函数式编程中，大量使用的一种技术是函数组合。它由一系列函数组成，这些函数给定一个初始值，一个接一个地执行，每个函数都接收前面执行的函数的输出作为输入。一个简单的 *compose* 函数可能如下所示:

```
const compose = (...*fn*) => *n* => fn.reduceRight((*acc*, *fn*) => fn(acc), n);
```

同样，我们可以从 *ramdajs* 或任何其他功能性的 JavaScript 库中获取它，但是我们今天把它放在了内部！我们如何使用它？

```
const toUpper = *s* => s.toUpperCase();
const trim = *s* => s.trim();const toUpperAndTrimmed = compose(trim, toUpper);*console*.log(toUpperAndTrimmed(' hello world  '));// 'HELLO WORLD'
```

在构造函数时，我们经常希望预先将某些参数应用到函数中，因为我们事先知道它们，并且只在运行时提供最后一个参数。有一个函数可以让我们把任何一个有多个参数的函数转换成一个可以部分应用的函数。这个函数通常被称为*curry*(currying 和 partial application [有区别 https://stack overflow . com/questions/218025/what-is-the-difference-that-currying-and-partial-application](https://stackoverflow.com/questions/218025/what-is-the-difference-between-currying-and-partial-application))。虽然我将提供这个 *curry* 函数的一个简单实现，但是不要太关心它的细节，知道它做什么就足够了。来自上面的链接:“Currying 是将一个有 *n* 个参数的函数转换成每个有一个参数的 *n* 个函数”。这基本上意味着您将在传递了原始函数定义的所有参数之后获得最终结果。

```
const curry = (*fn*) => {
  const arity = fn.length; return function $curry(...*args*) {
    if (args.length < arity) {
      return $curry.bind(null, ...args);
    }
    return fn.call(null, ...args);
  };
};
```

同样为了有利于构图，让我们创建一个名为 *map* 的方法。

```
const map = curry((*fn*, *mappable*) => mappable.map(fn));
```

这个 *map* function 取一个函数(转换)和一个已知有 *map* 方法的对象(这样信任一个对象会有某个方法通常被称为 duck typing)。它在那个*可映射的*对象中调用*映射*方法，从而应用给定的转换。如你所见，我们已经使用了 *curry，*如果我们没有使用，那么我们将不得不像这样编写 *map* 函数:

```
const map = *fn* => *mappable* => mappable.map(fn);
```

*更新:*

Ralf Kistner 在评论部分提出了一个很好的观点。我们并不真的需要 **Promise.prototype.map** ，我们可以这样写上面的 *map* 函数:

```
const map = curry((*fn*, *mappable*) => mappable.then(fn));
```

虽然对于本文来说这是可行的，但在真正的代码库中这并不是一个好主意。原因是这个版本的*映射*方法只对*承诺*有用(或者对象先用*然后用*方法)。 **Promise.prototype.map** 之所以重要的真正原因是，我们可以以一种可以与任何*仿函数(*[https://github.com/fantasyland/fantasy-land#functor](https://github.com/fantasyland/fantasy-land#functor))一起使用的方式来编写 *map* ，并且 *Promises* 可以被视为另一个*仿函数*。

接下来让我们定义一个函数，给定一个数组，返回它的第一个元素:

```
const head = *a* => a[0];
```

现在我们将添加一个给定谓词和数组的函数，它过滤并获取数组中符合谓词标准的所有元素:

```
const filter = curry((*predicate*, *a*) => a.filter(predicate));
```

现在让我们定义一个函数，它将返回匹配谓词的第一个元素。

```
const first = *predicate* => compose(head, filter(predicate));
```

我们将重新定义函数 *getPersonById* 和 *getCarByOwnerId* 如下:

```
const getPersonById = *id* => first(*p* => p.id === id);const getCarByOwnerId = *id* => first(*p* => p.ownerId === id);
```

我们将定义两个新函数，给定一个源数组和第二条信息，可以分别通过 id 获取一个人，通过所有者 id 获取一辆车。

```
const getPersonByIdFrom = curry((*source*, *id*) =>   
               *Promise*.resolve(getPersonById(id)(source)));const getCarByOwnerIdFrom = curry((*source*, *person*) =>  
               *Promise*.resolve(getCarByOwnerId(person.id)(source)));
```

花一点时间来理解我们做了什么。还注意到我们使用 *Promise.resolve* 来返回一个*承诺*，就像我们最初做的那样。函数式编程很重要的一点是创建许多小函数(专注于做一件事并且非常擅长)，然后组合它们(这是一种很棒的重用技术)。这导致您以声明的方式(而不是传统的命令式方式)创建软件，这样您的最终产品肯定更加健壮和灵活。问题是，我们几乎所有人都被教导了命令式思维，并且偏向于命令式思维，这种方法挑战我们，有时会让我们沮丧。你准备好迎接挑战了吗？

接下来，我们组合我们目前所拥有的来创建负责解决我们问题的函数。

```
const getMakeForOwnerId =      
         compose(map(prop('make')), map(getCarByOwnerIdFrom(cars)),              
                 getPersonByIdFrom(people));
```

在 *compose* 函数中从右向左阅读，以跟踪流程和正在发生的事情。现在 *makeForOwnerId* 只是在等待一个人 Id 来执行所有的组合行为。最后，我们释放“魔法”并将结果记录到*控制台*。

```
getMakeForOwnerId(personId)
  .then(*console*.log);
  .catch(*e* => *console*.log(`The following error occurred: ${e}`));// Ford
```

我知道这种编程风格和思维方式并不适合每个人。我也理解复杂性(和对已知的偏见)使得许多开发人员回避它。但是如果你至少对它感到好奇，那就试一试，至少，它会成为你工具箱中的另一个工具。

编码快乐！