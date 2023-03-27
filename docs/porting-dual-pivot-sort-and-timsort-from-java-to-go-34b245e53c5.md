# 将双支点排序和 Timsort 从 Java 移植到 Go

> 原文：<https://itnext.io/porting-dual-pivot-sort-and-timsort-from-java-to-go-34b245e53c5?source=collection_archive---------4----------------------->

> [hubo.dev](https://hubo.dev/2020-11-15-porting-dual-pivot-sort-and-timsort-from-java-to-go/) 的镜像

![](img/b9d33ef735a21041ff297f329e3662de.png)

Erik Lucatero 在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

大多数编程语言在其内置库中提供了排序 API。Java 和 Go 使用不同的排序算法，作为练习，我用不同的方法在 Go 中实现了 Java 的算法。用两种语言理解一些设计选择是一种有趣的方式。

# TLDR

*   Go 使用混合算法，对于不稳定排序，使用**快速排序**、**堆排序、**和**外壳排序**。
*   Go 使用**插入排序**和**稳定最小存储合并**进行稳定排序。
*   Java 对原语使用**双支点排序**，这是一种不稳定的排序。
*   Java 使用 **timsort** ，一种稳定的对象排序算法。
*   我通过从 Java 移植算法创建了一个 Go 包`[**jsort**](https://github.com/openaphid/jsort)`。可以从[github.com/openaphid/jsort](https://github.com/openaphid/jsort)进口。
*   用于排序切片的公共 API 兼容 Go 内置的' sort '包，如`jsort.Sort`、`jsort.Stable`、`jsort.Slice`、`jsort.SliceStable`等。它们都是使用 timsort 的稳定排序，在额外空间的成本上，timsort 可能比`sort.Stable`快 2-5 倍。
*   使用双支点排序提供了图元切片的专用函数，如`jsort.Ints`、`jsort.Int64s`等。它们可能比`sort.Sort`快 3 倍，而且不占用额外空间。

# 围棋的排序算法

排序 API 需要支持任意切片类型，其元素可以进行比较。如果一种语言支持泛型，通常很容易实现。Go 目前还没有泛型，预计到 2021 年底会包含在[v 1.18 beta 版](https://blog.golang.org/11years)中。Go 中的泛型有[种替代方式](https://appliedgo.net/generics/):代码生成、类型断言、反射和接口抽象。

Go 的内置`[sort](https://golang.org/src/sort/)`包使用接口抽象方法结合其他技术。排序算法依赖于接口`[sort.Interface](https://golang.org/src/sort/sort.go)`进行三种基本操作:Len、Less 和 Swap。

`sort.Slice`如果我们有时不想实现接口，函数就存在。它利用反射来派生 Len 和 Swap 操作。

`quickSort_func`函数可以在 [zfuncversion.go](https://golang.org/src/sort/zfuncversion.go) 中找到，它是由 [genzfunc.go](https://golang.org/src/sort/genzfunc.go) 从`quicksort`函数的`Interface`版本生成的。

“sort.go”实现了不稳定排序和稳定排序的两种混合算法。`[sort.Sort](https://golang.org/pkg/sort/#Sort)`中的不稳定算法有以下特点:

*   这是一个具有对数额外堆栈空间的递归就地算法，没有对临时空间的显式`make`调用。
*   该实现松散地遵循 glibc 中的`qsort`。
*   递归深度由`2*ceil(lg(n+1))`限定。一旦达到极限，它就切换到堆排序。
*   如果子问题的元素少于 12 个，它将运行一次间距为 6 的 shell 排序，然后应用插入排序。

`[sort.Stable](https://golang.org/pkg/sort/#Stable)`中的稳定算法使用了另一种方法:

*   这也是一种具有对数附加堆栈空间的递归就地算法。
*   它将输入数据分成大小为 20 的块，然后对所有块运行插入排序。
*   排序后的块通过`symMerge`函数合并，该函数实现了[稳定最小存储合并](http://itbe.hanyang.ac.kr/ak/papers/esa2004.pdf)算法。

# JDK 的排序算法

在旧的 JDK 版本中，Java 对原始数组使用了一个[调优的快速排序](http://hg.openjdk.java.net/jdk6/jdk6/jdk/file/8deef18bb749/src/share/classes/java/util/Arrays.java#l112)。排序`Object[]`使用一个[修改的合并排序](http://hg.openjdk.java.net/jdk6/jdk6/jdk/file/8deef18bb749/src/share/classes/java/util/Arrays.java#l1090)。从 JDK 1.7 开始，快速排序升级为[双枢轴排序](http://hg.openjdk.java.net/jdk7/jdk7/jdk/file/9b8c96f96a0f/src/share/classes/java/util/Arrays.java#l99)，合并排序被 [timsort](http://hg.openjdk.java.net/jdk7/jdk7/jdk/file/9b8c96f96a0f/src/share/classes/java/util/Arrays.java#l468) 取代。

2009 年，Yaroslavskiy-Bloch-Bentley 首次推出双支点排序，实际上比单支点排序更快。它不会减少比较或交换操作的数量。速度更快是因为[更少的 CPU 缓存未命中](https://algs4.cs.princeton.edu/lectures/keynote/23Quicksort.pdf)。

Timsort 是一种自适应的稳定合并排序，它利用了输入中的现有顺序。它是时间彼得斯在 2002 年为 Python 发明的。除了 Java，还被 [V8](https://v8.dev/blog/array-sort) 、 [Swift](https://github.com/apple/swift/pull/19717) 、 [Rust](https://github.com/rust-lang/rust/pull/38192) 采用。实践证明，它的速度更快，需要 O(n/2)的额外空间。这也是一个复杂的排序算法，几个实现被发现[破坏](http://www.envisage-project.eu/proving-android-java-and-python-sorting-algorithm-is-broken-and-how-to-fix-it/)。

`jsort`基于 OpenJDK 11 中[DualPivotQuickSort.java](https://github.com/AdoptOpenJDK/openjdk-jdk11u/blob/master/src/java.base/share/classes/java/util/DualPivotQuicksort.java)和[TimSort.java](https://github.com/AdoptOpenJDK/openjdk-jdk11u/blob/master/src/java.base/share/classes/java/util/TimSort.java)的代码构建。

# 从 Java 移植到 Go

除了以 Go 惯用的方式移植算法之外，我还尝试了一些仅用于评估的实验性解决方案，比如使用类型断言和 Go2 泛型。

## 原始类型

[DualPivotQuicksort.java](https://github.com/AdoptOpenJDK/openjdk-jdk11u/blob/master/src/java.base/share/classes/java/util/DualPivotQuicksort.java)为每个原语类型复制算法代码，因为 [Java 不支持专门化](https://cr.openjdk.java.net/~briangoetz/valhalla/specialization.html)。在 Go 中使用`go generate`更容易完成。模板文件 [sort_primitive.go](https://github.com/openaphid/jsort/blob/main/sort_primitive.go) 通过利用一些方便的 go 特性来实现该算法:

`// +build ignore`标志将模板排除在普通`go build`之外，别名`type primitive`创建了一个很容易被 [sort_primitive_gen.go](https://github.com/openaphid/jsort/blob/main/sort_primitive_gen.go) 替换的占位符。生成器创建一个内部包，其中包含每个原语类型的专用代码和测试，如 [internal/sort_int](https://github.com/openaphid/jsort/tree/main/internal/sort_int) 、 [internal/sort_int64](https://github.com/openaphid/jsort/tree/main/internal/sort_int64) 等。然后 [sort_export.go](https://github.com/openaphid/jsort/blob/main/sort_export.go) 导出内部符号，比如`jsort.Ints`和`jsort.Int64s`函数。`go generate`的一个漂亮的特性是，生成的代码可以在写入文件之前进行格式化。

## 通用切片类型

**#1 尝试:类型断言**

为了支持排序通用切片类型，我首先在包 [sort_slice_dps_ts](https://github.com/openaphid/jsort/tree/main/internal/sort_slice_dps_ts) 和 [sort_slice_tim_ts](https://github.com/openaphid/jsort/tree/main/internal/sort_slice_tim_ts) 中试验了类型断言。排序函数期望数据参数是一个`[]interface{}`和一个比较函数。看起来就像在 Java 中使用`Object[]`和`Comparator<Object>`。

在 Java 中，`Object[]`和`SomeClass[]`可以直接相互强制转换，没有开销成本。围棋不是这样，因为`[]interface{}`和`[]SomeStruct`是两种不同的类型。我们需要一个临时空间来帮助通过反射进行转换:

除了`[]interface{}`的额外空间之外，它还需要在比较函数中从`interface{}`到具体类型的额外强制转换操作，这是有代价的。这两个包没有被导出，因为使用类型断言在速度和内存上都不理想。

**#2 方法:接口抽象**

我首先想到的是，timsort 不能建立在内置的`sort.Interface`之上，因为算法依赖于更多的操作，比如索引[读](https://github.com/AdoptOpenJDK/openjdk-jdk11u/blob/master/src/java.base/share/classes/java/util/TimSort.java#L283)和[写](https://github.com/AdoptOpenJDK/openjdk-jdk11u/blob/master/src/java.base/share/classes/java/util/TimSort.java#L318)、[类型化数组创建](https://github.com/AdoptOpenJDK/openjdk-jdk11u/blob/master/src/java.base/share/classes/java/util/TimSort.java#L933)等。

在更彻底地阅读了 Go 的`sort`包的源代码后，我意识到接口抽象的关键是不要将数据类型泄露到排序算法中。我在包[sort _ slice _ Tim _ interface](https://github.com/openaphid/jsort/tree/main/internal/sort_slice_tim_interface)里想到了一个更好的主意，让 timsort 使用`sort.Interface`:

现在`jsort.Sort`的函数签名和 Go 的`sort.Sort`是一样的。我们可以用一个助手结构`[lessSwap](https://github.com/openaphid/jsort/blob/main/internal/sort_slice_tim_interface/sort_slice_tim_interface.go#L67)`来扩展它以支持`sort.Slice`:

接口抽象通常是解决 Go 中需要泛型的任务的惯用技术。我很高兴`jsort`可以使用与 Go 的`sort`包相同的 API 导出 timsort。需要注意的一点是`jsort.Sort`和`jsort.Stable`是 timsort 的相同函数，而`sort.Sort`和`sort.Stable`是不同的，这个规则也适用于`Slice`和`SliceStable`的函数。

**#3 实验:Go2 仿制药**

Go 团队在 2020 年 6 月发布了一款[实验工具](https://blog.golang.org/generics-next-step)**go2go**。我通过检查 Go 的一个 dev 分支在本地构建了这个工具，并用泛型语法编写了两个算法。令人印象深刻的是，我可以在不到 5 分钟的时间内构建一个完整的语言二进制文件。

go2go 工具将“ **.go2** ”代码文件翻译成“中的类型专用版本”。去吧”文件。这是一种异构的翻译，不同的专门化之间没有类型关系。翻译后的代码可以由 Go 1.x 构建和运行。

例如，我们可以用泛型为双支点排序和 timsort 声明排序函数:

正如我通过用`[]int`或`[]Person`调用函数来编写测试一样，go2go 生成了专门的函数:

奥里亚文数字零 [୦ (U+0B66)](https://www.fileformat.info/info/unicode/char/0b66/index.htm) 和奥里亚文数字八 [୮ (U+0B6E)](https://www.fileformat.info/info/unicode/char/0b6e/index.htm) 被 go2go 用作分隔符。这两个字符在 Go 的标识符中使用是有效的，go2go 假设没有人在自己的代码中使用它们。

您可以找到“. go2”代码和翻译的“.go”包中的代码 [sort_slice_go2](https://github.com/openaphid/jsort/tree/main/internal/sort_slice_go2) 。对于这个特定的任务，用 Go 的新泛型语法编写感觉就像使用 Java 的泛型一样。

由于 go2go 关闭了`GO111MODULE`，让 go2 文件与其他包共享代码并不容易。我只能玩包里面的代码。

# 错误

移植的大部分都很简单，我确实犯了一些值得注意的错误:

*   `**++**`**`**--**`**运算符**。Java 代码在很多地方使用了`a++`和`++a`。由于递增和递减运算符不是 Go 中的表达式，所以在 Go 中需要用两个语句来表示。例如，下面的 Java 代码在 if 语句中嵌入了一个增量运算符:**

**因为语义是增加`runHi`并在 if 条件中使用旧值，所以等价的 Go 代码是:**

*   ****循环**。Go 只有一个循环关键字，而 Java 有几个可选的。当它带有增量和减量操作符时，就变得更棘手了。以下代码是 Java 双枢轴排序中的插入排序路径:**

**事后的部分有一个增量操作和对两个变量的赋值，它被转换成如下:**

*   ****开关**。Switch 语句在 Java 中有隐式的失败行为，Go 要求显式声明。下面这段 timsort 是在 n 较小时对数组复制的优化:**

**我被它忽悠了，在 Go 里写错了代码。正确的版本是:**

*   **`**>>>**` **操作员**。对于无符号右移位操作，在 Go 中没有等效项。在 Java 代码中有两种情况。一种是得到中间点不发生整数溢出，这是历史上存在多年的[著名 bug](https://ai.googleblog.com/2006/06/extra-extra-read-all-about-it-nearly.html) :**

**在 Go 中，应该先转换成无符号 int，然后再转换回来:**

**Java 中的另一个地方是计算 2 的下一个最小幂:**

**我不知道如何在一行中完成，使用来自[bit twidling hacks](https://graphics.stanford.edu/~seander/bithacks.html#RoundUpPowerOf2)的想法:**

# **基准**

**我必须说，在 Go 中编写基准更加自然，因为它内置于 Go 的工具链中，而 Java 需要手动设置 [jmh](https://openjdk.java.net/projects/code-tools/jmh/) 。**

**[sort_benchmark_test.go](https://github.com/openaphid/jsort/blob/main/sort_benchmark_test.go) 设置 int 切片和 struct 切片排序的基准。在每个基准测试场景中使用了两种类型的数据集。一个是随机数据，这意味着元素大多是未排序的，只有很少的重复数据。另一种使用 XOR 运算从索引值计算数据，这使得数据部分排序。数据集的大小范围从 256 到 65536。**

## **#1 基准:比较次数**

**让我们首先通过 [sort_op_test.go](https://github.com/openaphid/jsort/blob/main/sort_op_test.go) 检查每个排序算法的比较次数。可以自己跑`make operation_stats`看看数字。我们可以发现:**

*   **Timsort 总是比其他人使用更少的比较。在 xor 数据集基准测试中，它的优势更加显著。**
*   **双支点排序比其他排序需要更多的比较。**
*   **`sort.Sort`不适应数据模式。它在随机数据集和异或数据集上使用几乎相同数量的比较。**
*   **`sort.Stable`在异或数据集上比`sort.Sort`使用更少的比较，在随机数据集上使用更多的比较。**

**从理论上讲，timsort 似乎比其他人更快。但是现实世界中的性能受到许多其他指标的影响。让我们运行性能基准来研究它。**

## **#2 基准:排序整数**

**[第一组基准](https://github.com/openaphid/jsort/blob/main/sort_benchmark_test.go#L74)用于 int 切片。我试图测试所有适用的 API:**

*   ****Dps-SpecializedInts** 使用的`jsort.Ints`是 int 的专用双枢轴排序。**
*   ****BuiltinSort-specialized ints**使用的是 Go 内置的不稳定排序算法的专业版，我在[sort _ builtin _ specialized _ test . Go](https://github.com/openaphid/jsort/blob/main/sort_builtin_specialized_test.go)中手工创建的。**
*   ****内置切片**使用内置`sort.Slice`功能。**
*   ****内置排序功能**使用`sort.Sort`功能。**
*   ****TimSort-接口**调用`jsort.Sort`即 Tim sort。**
*   ****TimSort-Slice** 调用`jsort.Slice`函数。**
*   ****内置切片表**使用`sort.SliceStable`功能。它仅用于评估，因为对基元使用稳定排序没有意义。**
*   ****内置排序稳定**使用`sort.Stable`功能。**
*   ****Dps-TypeAssertion** 和 **TimSort-TypeAssertion** 使用双枢纽排序和 TimSort 的类型断言变体。预计它们的效率较低。**

**“Dps-SpecializedInts”和“BuiltinSort-Sort”是实践中对 int slice 进行排序的推荐 API。我在图表中突出显示了它们。**

**排序随机整数的数字可以给我们一些发现:**

*   **`jsort.Ints`可以比围棋的`sort.Ints`功能快 3 倍。两者都不占用额外的空间，因为它们都是就地排序。它很好地解释了为什么 Java 采用双支点排序。**
*   **接口抽象对性能的影响不容忽视。“BuiltinSort-SpecializedInts”通过减少开销来加快速度，尽管它比`jsort.Ints`慢 10%~20%。**
*   **当比较成本较低时，Timsort 在随机数据上的速度并不快。**
*   **使用类型断言是最慢的，而且由于类型转换，它也消耗了最多的额外空间。**
*   **在 int slice 上`sort.Slice`比`sort.Sort`快，而在`jsort.Slice`和`jsort.Sort`上结果相反。我还不知道原因。**
*   **`sort.Stable`最慢。**

**对 xor 数据的测试告诉我们更多的故事:**

*   **`jsort.Ints`可以比`sort.Ints`快 5 倍。**
*   **双支点排序和 timsort 的性能都要好得多。Timsort 可以比内置的排序算法更快。**

## **#3 基准:排序结构**

**[另一组基准测试](https://github.com/openaphid/jsort/blob/main/sort_benchmark_test.go#L319)是对一个结构的片进行排序，这个结构有一个 int 字段和一个 string 字段:**

**基准测试测试了所有可以通过`Name`字段对`[]Person`进行排序的 API，它的值是一个来自 random int 或 XOR computed int 的字符串。**

*   ****Unstable-builtin Sort-Sort**使用 Go 的`sort.Sort`，这是一种不稳定排序。**
*   ****Stable-Tim sort-Interface**使用的是`jsort.Sort`，是稳定排序。**
*   ****不稳定内置排序片**使用`sort.Slice`，也是不稳定排序。**
*   ****稳定时间排序片**调用`jsort.Slice`。**
*   ****稳定内置排序稳定**调用`sort.Stable`。**
*   ****Unstable-Dps-type assertion**和**Stable-TimSort-type assertion**测试两种算法的类型断言版本。**
*   ****Stable-built insert-slice Stable**调用`sort.SliceStable`。**

**实际上，稳定排序是结构的首选。它们在图表中突出显示。**

**对于随机名称数据集，我们可以收集一些见解:**

*   **`jsort.Sort`是所有稳定排序 API 中最快的。它比不稳定的`sort.Sort`慢 15%，比`sort.Stable`快 50%~100%。**
*   **Timsort 需要额外的空间。**
*   **`sort.SliceStable`可以比`sort.Stable`慢 100%，而`jsort.Slice`和`jsort.Sort`可以慢 20%。**

**在 xor names 数据集上，`jsort.Sort`在速度上明显胜出。甚至比`sort.Sort`还快。**

## **#4 基准:Go2 泛型**

**实验性的 Go2 泛型在相同的任务上表现如何？由于`GO111MODULE`被 go2go 关闭，我无法在 go2 文件中导入`jsort`包。基准测试只将 go2 版本的 dual-pivot sort 和 timsort 与内置的排序函数进行了比较。**

*   **“Dps-Go2”的翻译代码和“Dps-SpecializedInts”基本相同，对于 int slice 来说最快也就不足为奇了。**
*   **“Timsort-Go2”的速度与“Stable-TimSort-Interface”不相上下，但它使用了更多的额外空间，因为它在内部分配了`[]Person`而不是`[]int`。如果结构`Person`的大小增加，将需要更多的空间。**

# **结论**

**Go 的排序函数可以通过使用其他算法来提高性能。如果分拣速度对您的服务至关重要，请尝试[ [jsort](https://github.com/openaphid/jsort) ] package。**