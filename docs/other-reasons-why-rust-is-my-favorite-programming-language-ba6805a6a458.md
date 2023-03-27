# Rust 是我最喜欢的编程语言的其他原因

> 原文：<https://itnext.io/other-reasons-why-rust-is-my-favorite-programming-language-ba6805a6a458?source=collection_archive---------0----------------------->

![](img/ac8bb8460536744cf31956ae41466c6c.png)

现在围绕 Rust 有很多炒作。一些特征占据了头条:

1.  Rust 和 C 或 C++一样快。
2.  Rust 是内存安全的。

Rust 程序员也喜欢优秀的文档和优秀的包管理器，这无疑是 Rust 越来越受欢迎的原因。

Rust 语言还有一些我非常喜欢的特性，但是我还没有看到太多的讨论。我用 Rust 编写代码已经有一年了，在这段时间里，它已经成为我所知道的几十种语言中我最喜欢的一种。

下面是三个鲜为人知的语言特性，它们让 Rust 编程变得非常有趣:

1.  没有空值。
2.  强大的宏。
3.  对可变性的精确控制。

还有一个额外的特性我留到了最后，只有在解释了其他特性之后才有意义。

Rust [备忘单](https://cheats.rs/)在接下来的代码样本中快速解释了 Rust 的一些独特语法。

让我们更深入地了解每个特性。

# 没有空值

Null 可能是编程语言设计中最大的错误。[东尼·霍尔称之为他的“十亿美元的错误”](https://en.wikipedia.org/wiki/Tony_Hoare)

你有没有做过一个软件项目从来没有抛出过一个 NullPointerException 或者解引用 null 而崩溃？如果你的项目是用 Rust、Swift、Elm 或 Haskell 编写的，那么也许你有。否则，这种可能性微乎其微。

取消对 null 的引用将使程序嘎然而止。为了应对不断的危险，避免软件崩溃，程序员们编写类似[这样的代码:](https://github.com/dotnet/aspnetcore/blob/52eff90fbcfca39b7eb58baad597df6a99a542b0/src/Components/Analyzers/src/ComponentFacts.cs#L15)

这段代码被空检查所支配，以至于很难理解这个函数实际上做了什么。可怜的程序员编写了更多的代码来应对参数可能为空的情况，而不是解决他们想要解决的问题。更重要的是，程序员已经多次阅读这种空检查代码，以至于他们已经训练自己忽略它。你看到我在上面的代码中引入的 bug 了吗？

因为 Rust 没有 null，上面的示例代码可以用 Rust 写成这样:

Rust 代码不检查空值，因为两个参数都不能为空。任何传递 null 的尝试都会被编译器捕获，并报告为类型不匹配。现在我知道了 Rust，当我看到其他语言中的空检查时，我会退缩。

但是当然，null 可能非常有用。有时，类或结构可以选择保存值，函数可以选择返回值，这很方便。为此，Rust 提供了[选项<T>类型。Option < T >的一个实例要么不包含，要么包含一些(T ),但决不包含两者。](https://doc.rust-lang.org/std/option/)

让我们考虑一个表示类似“[http://www.google.com](https://www.google.com:80/)/”的 url 的结构。URL 可选地包含一个端口号，如下所示:“[http://www.google.com:80/](http://www.google.com:80/)”。在 Rust 中，我们可以用以下结构来表示解析后的 Url:

假设我们想要编写一个名为 incr_port()的方法，它增加 url 中的端口号；我们将像这样调用方法`url.incr_port();`。

天真地说，我们可能会这样写这个方法:

Rust 编译器拒绝上面的 incr_port()，因为端口不是 u16 这是一个选项<u16>。编译器防止了在其他语言中潜在的空指针异常。</u16>

```
14 |      self.port += 1;
   |      ---------^^^^^
   |      |
   |      cannot use `+=` on type `std::option::Option<u16>`
```

要增加端口，代码必须首先确认端口有一些值。

通过选项<t>，Rust 在我们需要的时候提供可选值，而不会导致程序崩溃。更重要的是，Rust 将程序员从对每个参数和对象进行空检查的苦差事中解放出来。</t>

# 强大的宏

让我们继续上面的 Url 结构。我将在 Url 结构之前添加一行代码，然后讨论这一行代码的所有好处:

通过添加这一行代码，

`#[derive(Eq, PartialEq, Ord, PartialOrd, Clone, Hash, Default, Debug)]`

我现在可以像这样比较两个 URL 是否相等:

`url == url2`

我也可以比较它们来点菜。帕蒂亚洛德和 Ord 宏按照成员在结构中出现的顺序比较结构的成员。

`url > url2`

这意味着我可以将 URL 放入已排序的容器中:

```
let mut sorted_urls = BTreeSet::new();
sorted_urls.insert(url);
```

我可以克隆一个 Url 来得到一个精确的副本:

`let url2 = url.clone();`

我可以将 URL 插入哈希容器:

```
let mut urls = HashSet::new();
urls.insert(url4);
```

我可以创建一个默认的 Url，有点像在其他语言中调用无参数的构造函数:

`let url = Url::default();`

而且，我可以为 Url 打印一个漂亮的调试字符串:

`println!(“{:?}”, url);`

输出:`Url { protocol: “http”, host_name: “www.google.com", port: Some(80), path: “” }`

一行代码中包含了大量的功能！在其他语言中，有实现相同行为的方法:代码生成器、后处理器、ide 等。有些语言使用内省。然而，我最喜欢 Rust 的解决方案，因为它不需要额外的工具，也没有额外的代码需要我或我的同事来检查和维护，而且类型错误会在编译时被捕获。

这些宏的行为都与您预期的一样。例如，Ord(ordering 或 ordered 的缩写)按顺序比较结构的每个成员。这可能会使人们在重构代码时出错，因为改变结构成员的顺序会改变排序顺序。为了避免这种情况，我可以实现 [std::cmp::PartialOrd](https://doc.rust-lang.org/std/cmp/trait.PartialOrd.html) 并根据我的需要对其进行定制。

第三方库也可以利用`#[derive()]`宏。serde 库是 Rust 事实上的标准，用于将结构序列化/反序列化为十几种格式。让我们添加一些代码来允许 Url 被序列化到 json 和从 JSON 反序列化。导入 serde 库之后，我向`#[derive()]`宏添加了两个属性:

`#[derive(…, Serialize, Deserialize)]`

现在我可以从 json 序列化和反序列化 URL。

```
let text = serde_json::to_string(&url)?;
let url2: Url = serde_json::from_str(&text)?;
```

解析错误立即通过`[?](https://doc.rust-lang.org/edition-guide/rust-2018/error-handling-and-panics/the-question-mark-operator-for-easier-error-handling.html)`[操作符](https://doc.rust-lang.org/edition-guide/rust-2018/error-handling-and-panics/the-question-mark-operator-for-easier-error-handling.html)返回给调用函数。

`#[derive()]`宏通过只在我的代码中添加两个符号，为我的 Url 结构添加了大量的功能！如果 Url 包含无法序列化为 json 的类型，编译器就会报告错误。

[权力越大，责任越大。如果你在 20 世纪 90 年代维护一个 C 或 C++代码库，那么你可能会看到人们用宏做可怕的事情。C/C++预处理器的宏可以用任何文本*替换任何符号。人们利用这种能力来创建看起来一点也不像 C 的代码，并且很难理解和维护。这里有一个那个时代的真实例子:*](https://en.wikipedia.org/wiki/With_great_power_comes_great_responsibility)

这段代码实际上定义了一个方法，并在主体中构建了一个 if-else 阶梯，但是除非你剖析了宏，否则你永远不会知道。在调试器中检查这段代码也令人困惑。BEGIN_MESSAGE_MAP()是这样的:

不出所料，有人强烈反对宏造成这种混乱。从这一经历中诞生的语言有意地缺少宏:即 Java 和 Go。

但是我认为那些语言[把婴儿和洗澡水一起倒掉了](https://en.wikipedia.org/wiki/Don%27t_throw_the_baby_out_with_the_bathwater)。这个问题不是一般的宏；问题是 C/C++预处理器。在 Rust 中使用了几十个库之后，我还没有见过像 BEGIN_MESSAGE_MAP()这样神秘的东西。事实上，我只用过两个库中的宏: [serde](https://crates.io/crates/serde) 和[总之](https://docs.rs/anyhow/1.0.41/anyhow/index.html)。

我看到了一些程序员在 Rust 中使用宏比在其他语言中更有纪律的原因:

1.  Rust 提供了[常量函数](https://doc.rust-lang.org/reference/items/functions.html#const-functions)，在很多情况下可以替代宏。例如，您可以编写一个`const sqrt()`函数，在编译时计算一个常数的平方根。
2.  使用 Rust 宏，您不能用任何想要的文本超级全局地替换一个符号。比如这是不可能的:`#define PLUS +`
3.  在 Rust 中，[宏必须扩展以完成表达式、语句、项目、类型或模式](https://docs.rust-embedded.org/book/c-tips/index.html#macros)。他们不能扩展到半个功能。它们不能有不平衡的圆括号、中括号或大括号。上面的 BEGIN_MESSAGE_MAP()宏在 Rust 中是不可能的，因为它有不平衡的大括号。
4.  在 Rust 中定义宏比用 C/C++预处理器要复杂得多。学习如何在 Rust 中编写宏就像学习一门全新的编程语言。不能只`#define max(a, b) ((a) < (b) ? (b) : (a))`。如果一个东西可以用除了宏之外的任何方式来表达，Rust 程序员喜欢选择另一种方式。

# 对可变性的精确控制

Rust 给了程序员对可变性的精确控制。可变数据可以被修改。不可变数据不能被修改。

不可变数据对于每个必须阅读代码的人来说都很棒，不管他们是人还是机器。推理不可变数据要容易得多，编译器更容易优化使用不可变数据的代码，并且不可变数据总是线程安全的。

考虑这个示例代码，它使用了上面的 Url 示例，并演示了为什么不可变数据使代码更容易推理。

因为 url 是通过不可变的引用传递给 fetch()的，所以 fetch()不可能修改 url 的内容。因此，很容易通过循环的每次迭代来预测 url 的值。我不必检查 fetch()的代码，甚至不必查看它的参数类型来确认它没有修改 url，因为我可以看到参数是作为*不可变*引用:`&url`传递的。要让 fetch()修改 url，我必须修改调用代码，将 url 作为可变引用传递，就像这样:`fetch(&mut url)`。

同时，我不必在整个循环的上下文中使 url 不可变，以使它在调用 fetch()期间不可变。我仍然可以通过在循环中调用`url.incr_port()`来修改 url。

将 url 作为不可变的引用传递也为 fetch()提供了保证。fetch()不仅不可能修改 url，而且当 fetch()对 url 有一个不可变的引用时，其他任何东西也不可能修改 url。当 fetch()正在执行时，任何其他线程都不能修改 url。这使得编写 fetch()更容易。

我说 Rust 给了程序员对可变性的精确控制，因为 Rust 不会将可变性与其他概念纠缠在一起。程序员可以选择数据是可变的还是不可变的，不管数据是分配在堆栈上还是堆上，不管它是通过引用还是通过值传递，也不管它是结构还是像整数这样的原始类型。

没有像 Rust 那样对可变性的精确控制，其他语言引入了各种模式和语言特性来提供某种形式的不变性。一些例子是[构建器模式](https://howtodoinjava.com/design-patterns/creational/builder-pattern-in-java/)，只读接口，以及多种聚合类型，每种类型都有自己的可变性规则，比如类对结构对记录。使用这些其他解决方案需要进行权衡，或者编写比 Rust 更多的代码。

# 额外特点:铁锈富有表现力

Rust 被描述为一种富于表现力的语言；表达是一个非常抽象的术语。什么是表现力，为什么有用？

在表达性语言中，不改变语言的语法，不使用[样板代码](https://en.wikipedia.org/wiki/Boilerplate_code)，就有可能引入语言设计者从未想过的新思想。事实上，许多概念纯粹是通过一个库而不是语法来合并的，这证明 Rust 是有表现力的。

例如，Option <t>类型没有内置到语言中；它只是一个标准的库。其他语言需要专用的语法来编码可选值，但是使用 Rust，现有的语言特性允许程序员使用 Option <t>有效地表达可选值；不需要对语言进行修改。</t></t>

Rust 用标准库而不是语法实现概念的另一个例子是 Rust 的[From<T>and Into<T>traits，它允许程序员编写从一种类型到另一种类型的自定义转换。其他语言增加了运算符语法，使程序员能够定义自定义类型转换。有了 Rust，现有的语言特性就足够了。从< T >到< T >都是普通性状。](https://doc.rust-lang.org/std/convert/trait.From.html)

对于语言设计师来说，这听起来很棒，但是对于我们其他人来说，这怎么可能是实际的呢？

考虑这个函数:

io::Result 返回类型可以恰好包含以下值之一:错误值或表示成功的空值()。

想象一下实现这个函数。为了确保我们总是将文件指针重置到其原始位置，在其他编程语言中找到类似于`try {} finally {}`的东西肯定会很方便。然后，我们可以简单地在 finally 块中调用`f.seek(original_offset)`。但是 Rust 没有例外，也没有`try {} finally {}`。

为了应对没有`try {} finally {}`的情况，我写了一个行为相同的宏。首先，一些免责声明:

1.  这个是我写的第一个宏，所以肯定可以改进。
2.  这是我写的第一个宏，因为我没有写宏的需要。
3.  该宏仅用于演示目的。当我几个月前在产品代码中有同样的需求时，我写的是普通的 Rust 代码。
4.  因为 Rust 有一个内置的宏叫 try！()，我必须给我的宏取个不同的名字，所以我把它命名为 tryf！().

免责声明够多了。下面是我用 tryf 实现的函数的样子！()宏:

语法有点不同，但是如果你熟悉有异常的语言，我希望这段代码的作用很明显。这就是表达性语言的美妙之处。我能接受与语言本身无关的想法，并用 Rust 有效地表达出来。

因为 Rust 富有表现力，我相信我能够用代码高效地表达我自己的各种想法，包括编译时发现的错误，以及没有[样板代码](https://en.wikipedia.org/wiki/Boilerplate_code)。

# 结论

当然，我喜欢 Rust 的原因和许多其他人一样:与 C/C++相似的速度，内存安全，优秀的文档，优秀的包管理器，通过 [WebAssembly](https://webassembly.org/) 在浏览器中运行，等等。但我觉得 Rust 也有一些隐藏的宝石，所以我写了这个帖子。

如果你正在考虑将 Rust 用于工作或爱好，我希望这篇文章能给你一些有用的见解，帮助你做出更明智的决定。要开始学习 Rust，我推荐从《Rust by Example 这本书开始。

# 承认

感谢下面的人，他们无疑对自己喜欢的语言有不同的看法，但却慷慨地审阅了这篇文章的草稿。

*   [铁锈论坛](https://users.rust-lang.org/t/seeking-comments-on-my-draft-blog-post-about-my-favorite-features-of-rust/59230)
*   乔恩·斯基特
*   丹尼尔·班克海德
*   本杰明·科

脚注

[1]有时，你可能需要给一个不可变的结构增加一个引用计数。对于这样的情况，Rust 提供了[单元](https://doc.rust-lang.org/std/cell/)、 [Rc](https://doc.rust-lang.org/std/rc/index.html) 等等。