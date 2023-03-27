# 带避难所的弹性 JavaScript

> 原文：<https://itnext.io/resilient-javascript-with-sanctuary-858bf3ca1790?source=collection_archive---------2----------------------->

![](img/f3af2c5fe29ab3fc7f73ab899bb4d365.png)

圣米格尔·阿尔坎霍堡|法罗林·达·纳扎雷

JavaScript 真的很好，但它是一把双刃剑:一方面，它很容易开始和构建一些东西，而另一方面，它很容易中断流程。

如果你正在阅读这篇文章，你可能会明白我在说什么，但仍然不相信 JavaScript 的健壮性(或者仍然在寻找圣杯)。

在这篇文章中，我想展示一个简单的库如何帮助我们用更安全、更有弹性和更可预测的东西来增强我们的代码、逻辑和算法:**避难所**！

[](https://github.com/sanctuary-js/sanctuary) [## 避难所-js/避难所

### 躲避不安全的 JavaScript。通过在…上创建帐户，为 sanctuary-js/sanctuary 开发做出贡献

github.com](https://github.com/sanctuary-js/sanctuary) 

这个库依赖于**函数式编程、幻想世界**和 **Haskell** 概念，所以如果你不熟悉这些概念，可能会有点困难。如果是这样的话，我建议先查看一下这些资源(这样会让整个事情变得简单很多):

[](http://adit.io/posts/2013-04-17-functors,_applicatives,_and_monads_in_pictures.html) [## 图片中的函子、应用和单子

### 这里有一个简单的值:我们知道如何对这个值应用一个函数:非常简单。让我们…

adit.io](http://adit.io/posts/2013-04-17-functors,_applicatives,_and_monads_in_pictures.html) [](https://sanderv1992.github.io/fp/) [## 函数式编程(幻想世界 JavaScript 规范指南)

### 函数式编程(幻想世界 JavaScript 规范指南)

函数式编程(幻想世界 JavaScript 规范指南)sanderv1992.github.io](https://sanderv1992.github.io/fp/) 

为什么我选择了*避难所*？

因为除了公开的函数和实用程序，它还依赖于一些代数数据类型，例如，**可能是**:

![](img/c93bb85a9a968c126853291fd217de02.png)

[http://adit . io/posts/2013-04-17-functors，_applicatives，_ and _ monads _ in _ pictures . html](http://adit.io/posts/2013-04-17-functors,_applicatives,_and_monads_in_pictures.html)

基本上，如上图所示，a Maybe 可能是**只是**(我把它看作是一个“盒子”，通过例如 *map* 和 **Nothing** 可以访问它的包装值)(是的，Nothing，没有空的、未定义的或类似的奇怪的东西)。因此，在处理“烫手山芋”时，保持一定程度的安全是可能的。

与 Maybe 一起，**或者**也被支持，由**左**(通常处理错误)和**右**(通常处理成功值)组成。

现在我们已经，某种程度上，提出了基本的概念，避难所实际上给我们提供了什么？

让我们从一个简单的例子开始: **parseInt** (它接受 2 个参数——string 和 radix——并返回一个数字(如果有效)或 NaN(如果无效)——哦对了，加上 null 和 undefined:)。

Sanctuary 提供了以下解决方案:

`parseInt :: Radix -> String -> Maybe Integer`

如果不熟悉这种语法，阅读起来可能有点棘手，但它实际上为每个函数(基数和字符串)接受一个参数，并返回一个 *Maybe(整数)*，它由 *Just(整数值)*或 *Nothing* (不是 n an、null、undefined 或其他什么)组成。

```
S.parseInt *(*10*) (*'-42'*)* // Just *(*-42*)* S.parseInt *(*16*) (*'0xFF'*)* // Just *(*255*)* S.parseInt *(*16*) (*'0xGG'*)* // Nothing
```

这很好，因为我们可以在流中统一我们的数据，使之变得简单易懂。

另一个例子可能是“ *JSON.parse* ”，其中 Try/Catch 位于角落后面:

`parseJson :: (Any -> Boolean) -> String -> Maybe a`

这种情况看起来有点不同，但您可能会注意到一个额外的函数，它实际上是我们试图解析的内容的验证器，当 truthy 时，它返回包装在*中的有效对象，只是*或 *Nothing* 。

```
S.parseJson *(*S.is *(*$.Array *(*$.Integer*))) (*'['*)* // Nothing

S.parseJson *(*S.is *(*$.Array *(*$.Integer*))) (*'["1", "2", "3"]'*)* // Nothing

S.parseJson *(*S.is *(*$.Array *(*$.Integer*))) (*'[0, 1.5, 3, 4.5]'*)* // Nothing

S.parseJson *(*S.is *(*$.Array *(*$.Integer*))) (*'[1, 2, 3]'*)* // Just *([*1, 2, 3*])*S.parseJson *(_ =>  true) (*'["a", "b", "c"]'*)* // Just *(["a"*, "b", "c"*])*
```

注意:没有**try/catch**，因为该函数为我们处理错误，并且它不返回任何内容。太神奇了！

让我们转移到 JS 的另一个热门话题，*点符号*。

```
const foo = *{* bar: *{* inner: *{* thing: *{* exists: true
            *}
        }
    }
}*;
foo.bar.inner.thing.exists
// true
```

即使我们非常关注我们的数据，这也可能在任何时候中断，因为链中的一个属性可能没有被定义。为了访问深层结构，我们可以使用“ **S.gets** ”，这有助于处理棘手的情况:

`gets :: (Any -> Boolean) -> Array String -> a -> Maybe b`

```
const foo = *{* bar: *{* thing: *{* exists: true
        *}
    }
}*;

S.gets *(*S.is*(*$.Boolean*)) ([*'bar', 'thing', 'exists'*]) (*foo*)* // Just*(*true*)* S.gets *(*S.is*(*$.Number*)) ([*'bar', 'thing', 'exists'*]) (*foo*)* // Nothing

S.gets *(*S.is*(*$.Number*)) ([*'data', 'user', 'name'*]) ({})* // Nothing
```

得益于此，我们可以实现两件事:

*   以安全的方式访问嵌套属性；
*   验证遍历的值；

通常，如果路径中断或者值不符合我们的期望，*不会返回任何内容*。

另一个例子可能是 ***头*** 和 ***尾*** ，来自功能编程字段:

```
S.head *([*40, 20, 30, 50*])* // Just *(*40*)* S.head *([])* // NothingS.tail *([*40, 20, 30, 50*])* // Just *([*20, 30, 50*])* S.tail *([])* // Nothing
```

理解起来并不复杂，而且有很多实用程序可以安全地使用。

我真正一直依赖的是“ ***管道***”*它执行一系列函数*的从左到右的组合。

```
S.pipe *([* S.add *(*1*)*,
    Math.sqrt,
    S.sub *(*1*)
]) (*99*)**// 9*
```

以我的拙见，这是描述给定逻辑的一系列步骤的美丽方式，它让我想起了 Clojure 方式(使用 [thread-last 宏](https://clojuredocs.org/clojure.core/-%3E%3E))。

例如，我们可能有这样一个简单的算法:

```
S.pipe*([* S.map*(*S.pow*(*2*))*,  // Map each item to power of 2
    S.filter*(*S.even*)*, // Filter even numbers
    S.take*(*10*)*,       // Takes first 10 items  
    S.map*(*S.sum*)      // Sum all values ('map' accesses Maybe)
])
(*S.range*(*0*)(*100*)) // Creates an array with numbers from 0 to 100*// Just (1140)
```

第一:这真的很容易读懂和预测。当进行代码审查时，这可能就像读一首诗，而不是一场悲剧。

第二:是的，这是对边缘情况的反应。如果我们更改为“S.range(0)(10)”或一个空数组作为输入，它将不会返回任何内容，因为逻辑将无法从数组中获取 10 个项目(我们过滤掉偶数，从而减少了可用列表)。

更复杂的例子呢？

真实的场景是遍历和聚合嵌套数据结构的一部分。我将用 Github 提供的 Seach Commits API 展示一个例子，并只选择与给定用户相关的提交消息。一个要求:逻辑需要坚如磐石。

这是我们(假设)收到的一些数据——这只是取自 Github API 指南的一个例子，并混有模拟数据):

我们的逻辑可能如下:

*   接受集合和用户；
*   访问包含结果的“项目”(是吗？！);
*   过滤将“提交者”电子邮件与所需用户的电子邮件相匹配的结果项；
*   仅聚合提交消息并返回找到的列表；

```
const retrieveCommitMessagesByUser = *(*commits, user*)* => *{* return S.pipe*([

    ])(*commits*)*;
*}*;

getCommitMessagesByUser*(*commitsCollection, 'octocat@nowhere.com'*)*;
```

很简单，我们可以使用一个接受提交列表和期望用户的电子邮件的函数，它真的很神奇。

因此，现在我们可以尝试以一种更好的方式访问商品列表:

```
const getCommitsMessagesByUser = *(*commits, user*)* => *{* return S.pipe*([* S.get*(*S.is*(*$.Array*(*$.Object*)))(*'items'*)*,
    *])(*commits*)*;
*}*;

getCommitsMessagesByUser*(*commitsCollection, 'octocat@nowhere.com'*)*;// Output: 
// Just ([...truncated items...])
```

我们可以使用“ *S.get* ”来访问对象中的项目，并进行验证，指定它需要是一个对象数组。如果成功，它将返回*只是*，否则将返回*什么都不是*。

接下来，我们需要过滤用户的提交:

```
const committerIs = committer => *{* return _ => S.pipe*([* S.gets*(*S.is*(*$.String*))([*'commit', 'committer', 'email'*])*,
        S.equals*(*S.Just*(*committer*))
    ])(*_*)*;
*}*;

const getCommitsMessagesByUser = *(*commits, user*)* => *{* return S.pipe*([* S.get*(*S.is*(*$.Array*(*$.Object*)))(*'items'*)*,
        S.map*(*S.filter*(*committerIs*(*user*)))*,
    *])(*commits*)*;
*}*;

getCommitsMessagesByUser*(*commitsCollection, 'octocat@nowhere.com'*)*;// Output: 
// Just ([...truncated user's items...])
```

由于前面的指令返回一个包装的值，为了再次访问它，我们需要使用" *map* 和" *filter* "集合。我创建了一个函数来执行深度检查。最后，这是函数式编程，我们在代码中组合/重用函数。

返回的输出是与我们想要的用户相关的提交消息的包装值。

现在我们已经有了一组经过筛选的数据，我们可以只接收以下消息:

```
const retrieveCommitMessagesByUser = *(*commits, user*)* => *{* return S.pipe*([* S.get*(*S.is*(*$.Array*(*$.Object*)))(*'items'*)*,
        S.map*(*S.filter*(*committerIs*(*user*)))*,
        S.map*(*S.map*(*S.gets*(*S.is*(*$.String*))([*'commit', 'message'*])))
    ])(*commits*)*;
*}*;

retrieveCommitMessagesByUser*(*commitsCollection, 'octocat@nowhere.com'*)*;// Output:
// Just ([Just ("Create styles.css and updated README"), Just ("Updated eslint configuration"), Nothing])
```

使用第一个"*映射*，我们访问*可能是*，使用第二个"*映射*，我们迭代列表中的项目。" *S.gets* 执行对" *commit.message* "的访问，并检查检索到的值是否是我们所期望的:*一个字符串*。您可能会注意到“ *Nothing* ”和“Just”，因为集合中的一个项目有正确的提交者，但出于某种原因，*消息被设置为“null”*(这只是模拟数据，但从外部 API 返回的数据可能会发生这种情况，因此处理它完全有意义)。

之后，让我们仅用有效值来清理输出:

```
const retrieveCommitMessagesByUser = *(*commits, user*)* => *{* return S.pipe*([* S.get*(*S.is*(*$.Array*(*$.Object*)))(*'items'*)*,
        S.map*(*S.filter*(*committerIs*(*user*)))*,
        S.map*(*S.map*(*S.gets*(*S.is*(*$.String*))([*'commit', 'message'*])))*,
        S.map*(*S.justs*)
    ])(*commits*)*;
*}*;retrieveCommitMessagesByUser*(*commitsCollection, 'octocat@nowhere.com'*)*;// Output:
// Just (["Create styles.css and updated README", "Updated eslint configuration"])
```

酷，到目前为止一切顺利！！

最后一步可能是解开来自*或者*的值:

```
const retrieveCommitMessagesByUser = *(*commits, user*)* => *{* return S.pipe*([* S.get*(*S.is*(*$.Array*(*$.Object*)))(*'items'*)*,
        S.map*(*S.filter*(*committerIs*(*user*)))*,
        S.map*(*S.map*(*S.gets*(*S.is*(*$.String*))([*'commit', 'message'*])))*,
        S.map*(*S.justs*)*,
        S.fromMaybe*([])
    ])(*commits*)*;
*}*;

retrieveCommitMessagesByUser*(*commitsCollection, 'octocat@nowhere.com'*)*;// Output:
// [ 'Create styles.css and updated README',
     'Updated eslint configuration' ]
```

" **S.fromMaybe** 仅从*获取*的值，或者当 *Nothing* 时，它返回传入的默认值(本例中为空数组)。

就是这样！

一开始看起来可能有点奇怪，但是当熟悉这种方法和函数式编程时，这就有意义了。很少几行代码处理我们的逻辑，并对边缘情况和不愉快的路径(一些空的/未定义的，不是我们期望的类型，等等)具有弹性。

有大量的用例，结合其他技术和方法，它将整个事情带到了一个不同的层面。

我创建了一个 REPL 游乐场，你可以在那里玩耍:

更多资源:

[视频:ReactiveConf 2018 —大卫·钱伯斯:在不确定的世界中安全编程](https://www.youtube.com/watch?v=a2astdDbOjk)

[幻灯片:JS 中的 FP](https://www.slideshare.net/TychoGrouwstra/fp-in-js)

[文献:避难所](https://sanctuary.js.org/)

我希望你喜欢它！

干杯:)