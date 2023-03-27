# 底部基准测试——迭代列表。网

> 原文：<https://itnext.io/benchmarking-your-way-to-the-bottom-iterating-lists-7ab4e6d2dbf6?source=collection_archive---------3----------------------->

## 中访问列表值时找到迭代列表的最快方法。NET 使用[BenchmarkDotNet](https://benchmarkdotnet.org/)验证大规模性能。

![](img/61fad17c4f925a99c0c40e8a7d584b98.png)

BenchmarkDotNet —一个强大的。用于基准测试的. NET 库

我最近花了一些时间查看一个旧系统，想知道如何让它性能更好。有许多方法可以使这样的系统更快、性能更好，但是我想避免大规模的重构，以将风险降到最低。具体来说，我关注的是那些可以累积起来产生更大影响的小项目。最明显的是整个系统的升级，但当这不是一个选项时，这些小变化可能是一个工程师必须做出的改变。

做这项工作已经激发了我的兴趣，首先，我想知道哪种方式是做这些日常小事的最佳方式。我正在慢慢构建一个小的任务系列。它可能会消耗不同的库来完成一个任务，或者直接比较每天的任务。无论哪种方式，都可以学到教训，获得乐趣。

# 迭代和访问列表

今天我决定看看你可以迭代一个[列表](https://docs.microsoft.com/en-us/dotnet/api/system.collections.generic.list-1)的许多不同方法。我一直在做的应用程序有许多长列表的实例，这些长列表由于各种原因而被迭代。这让我想知道如何最快地完成[列表](https://docs.microsoft.com/en-us/dotnet/api/system.collections.generic.list-1)中的所有项目。

在我第一次运行时，我忽略了将被迭代的项赋给一个变量。单纯地迭代[列表](https://docs.microsoft.com/en-us/dotnet/api/system.collections.generic.list-1)而不访问条目会导致错误的结果。**两组结果**都将在结果中列出。

以下是通过 [BenchmarkDotNet](https://benchmarkdotnet.org/) 使用和测试的方法。

*   [为](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/statements/iteration-statements#the-for-statement)
*   [ForEach](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/statements/iteration-statements#the-foreach-statement)
*   [LINQ ForEach](https://docs.microsoft.com/en-us/dotnet/api/system.collections.generic.list-1.foreach)
*   [获取枚举器](https://docs.microsoft.com/en-us/dotnet/api/system.collections.generic.ienumerable-1.getenumerator)
*   [跨度为](https://docs.microsoft.com/en-us/dotnet/api/system.span-1)
*   [Span ForEach](https://docs.microsoft.com/en-us/dotnet/api/system.span-1)

# 理论

我最近读到过`for`循环实际上比`foreach`循环要快。我预计我会发现在`List`上表现会稍微好一点。为了涵盖遍历列表的多种不同方式，我已经将 LINQ 和`for`作为其他选项。我预计 LINQ 会稍微慢一点，我以前从未用过`Span`，所以我不知道会发生什么。

# 设置

使用 [BenchmarkDotNet](https://benchmarkdotnet.org/) 创建一个控制台应用程序使得运行这个基准测试变得非常简单。为了确保涵盖旧系统，这些测试将在两个**中运行。NET Framework 4.8** 和**。NET 6.0** 因为这是当前对. NET 的长期支持版本。一个用于旧世界，一个用于新世界。

前提很简单。创建一个 100 和 10，000 深的字符串`List`。然后运行测试来迭代每一项，并重复这样做，直到 [BenchmarkDotNet](https://benchmarkdotnet.org/) 乐意给我们一个结果。这应该给我们一个合理的假设，最快的方式去做这件事。

迭代列表的设置方法

# 方法

接下来是为我想要测试的每种类型的`list`迭代创建一个方法。每种方法都做一件事来使它成为一个均匀的基准。每个方法将遍历列表中的每个项目，访问列表中的项目并将其赋给一个变量。一旦所有的项目都被处理，它将完成并记录完成任务所用的时间。

迭代列表的测试方法

# 结果

对我们来说幸运的是， [BenchmarkDotNet](https://benchmarkdotnet.org/) 包括一个排名栏，可以清楚地显示哪一轮测试是赢家。当您阅读这些结果时，您可以使用 Mean 列来比较每个测试之间的运行时间差异。其中一些有明显的赢家，而其他的差别可以忽略不计。

## 。NET Framework 4.8 —迭代和访问

```
| Rank | Method        | N     |        Mean |
| ---: | ------------- | ----- | ----------: |
|    1 | For           | 100   |    344.8 ns |
|    2 | ForEachLinq   | 100   |    559.9 ns |
|    3 | ForEach       | 100   |    659.4 ns |
|    3 | GetEnumerator | 100   |    659.5 ns |
|    4 | For           | 10000 | 33,562.8 ns |
|    5 | ForEachLinq   | 10000 | 53,895.5 ns |
|    6 | GetEnumerator | 10000 | 63,317.0 ns |
|    6 | ForEach       | 10000 | 63,356.6 ns |
```

## 。NET 6.0 —迭代和访问

```
| Rank | Method        | N     |        Mean |
| ---: | ------------- | ----- | ----------: |
|    1 | SpanForEach   | 100   |    144.3 ns |
|    2 | SpanFor       | 100   |    176.3 ns |
|    3 | ForEach       | 100   |    251.0 ns |
|    3 | GetEnumerator | 100   |    251.0 ns |
|    3 | For           | 100   |    251.4 ns |
|    4 | ForEachLinq   | 100   |    352.1 ns |
|    5 | SpanForEach   | 10000 | 12,260.4 ns |
|    6 | SpanFor       | 10000 | 16,088.1 ns |
|    7 | ForEach       | 10000 | 24,109.4 ns |
|    7 | GetEnumerator | 10000 | 24,111.0 ns |
|    7 | For           | 10000 | 24,113.8 ns |
|    8 | ForEachLinq   | 10000 | 34,713.9 ns |
```

结果被格式化为清晰明了的显示，完整的输出你可以在 GitHub 上查看[动作的输出。](https://github.com/stphnwlsh/Benchmarking/actions)

# 思想

## 。NET 框架 4.8

在运行基准测试之后，让我吃惊的是`foreach`的表现有多差。它与最差的`GetEnumerator`不相上下，被 LINQ `foreach`超越。我肯定不会在任何旧系统中使用它。如果能有一些`Span`结果进行比较就更好了，但是这个特性不可用。NET 框架。这里明确的选择是将所有的`List`迭代重构为`for`循环，过上幸福的生活。

## 。NET 6.0

的。净结果令我相当惊讶。我开始运行这个并不太了解`Span`，现在我准备做一些研究来了解更多。当访问该项目时，`Span`似乎比`for`和`foreach`更有优势，为了性能提升，似乎值得重构整个代码库。这里的关键结果是，`List`的任何迭代都应该将列表转换为`Span`。除此之外，当迭代一个`Span`时，在`for`和`foreach`之间有一些不同，但是我不确定是否值得改变所有这些。

## 停止使用 LINQ FOREACH 语句

在这两部影片中，他们都是性能杀手。NET 6.0 和**。NET Framework 4.8** 比其他版本慢得多。立即重构它们。对于被很多人使用的东西来说，这实在是太可怜了。

# 密码

所有这些代码都是开源的。你可以在 [GitHub](https://github.com/stphnwlsh/Benchmarking) 上找到我的基准测试工作。结果是运行基准测试的 GitHub 动作的输出，您应该能够在该存储库中找到详细的输出。

```
**Connect or Support?**If you like this, or want to checkout my other work, please connect with me on [LinkedIn](https://www.linkedin.com/in/stphnwlsh), [Twitter](https://twitter.com/stphnwlsh) or [GitHub](https://github.com/stphnwlsh), and consider supporting me at [Buy Me a Coffee](https://www.buymeacoffee.com/stphnwlsh).
```