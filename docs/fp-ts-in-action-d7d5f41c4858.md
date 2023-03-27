# 实践中的 fp-ts🚀

> 原文：<https://itnext.io/fp-ts-in-action-d7d5f41c4858?source=collection_archive---------0----------------------->

👊2021 年第一帖！

![](img/dbcb14274541ab5c5a47ade439431054.png)

最近我决定在一个副业项目中采用 fp-ts lib。

什么是`**F**[**p-ts**](https://github.com/gcanti/fp-ts)**?**`

它是一个库，提供了类型化函数语言的抽象，使得在 TypeScript 中开发时应用函数模式变得容易。

我采用 fp-ts 的动机是:

*   我被 Haskell 和 Rust 这样的语言迷住了，想在代码中使用一些熟悉的 [ADTs](https://dev.to/gcanti/functional-design-algebraic-data-types-36kf) (选项/也许，只是/一些，要么/结果，无)。
*   我喜欢错误处理方法。
*   我想弄清楚，在享受函数式风格的同时，它是否真的能改进我的 typescript 项目。

## 简单的启动

我已经阅读 [fp-ts 文档和帖子](https://gcanti.github.io/fp-ts/learning-resources/)有一段时间了，所以剩下的就是选择一个简单的项目代码，将其一般化一点，并开始使用 fp-ts:)

我选择了一个简单的程序来获取我在 google-sheets 表中管理的数据，如下所示:

*   从私有本地文件中读取工作表凭据
*   鉴定
*   运行并行获取所有表的 fetchTables(每个表都有自己的路由)

注意:[以下食谱](https://grossbart.github.io/fp-ts-recipes/#/basics)超级有用📓

让我们回顾一下 fetchTable 函数 sig:

```
function fetchTable<T>(sheets: Sheets, range: any, spreadsheetId: any): Promise<T> 
```

`fetchTable`接受 3 个参数并返回包装在承诺中的某个 T 类型的值。

但是有一种更好的方式来写这个函数，来表达失败的可能性。

fp-ts 引入**要么:**

`[Either<E, A>](https://dev.to/gcanti/getting-started-with-fp-ts-either-vs-validation-5eja)`[类型表示一个计算可能会因类型](https://dev.to/gcanti/getting-started-with-fp-ts-either-vs-validation-5eja) `[E](https://dev.to/gcanti/getting-started-with-fp-ts-either-vs-validation-5eja)` [的错误而失败，或因类型](https://dev.to/gcanti/getting-started-with-fp-ts-either-vs-validation-5eja) `[A](https://dev.to/gcanti/getting-started-with-fp-ts-either-vs-validation-5eja)`的值而成功

让我们重构函数来使用它:

## **任务**和**任务任一:**

**任务**:异步计算，产生给定类型的值，并且**从不失败* *例如:

```
const task1: Task<string> = () => ***Promise***.resolve('task1')
```

添加错误处理:

**任务任一**:可能失败的异步计算

**tryCatch** 从一个可能抛出。readCreds()是一个在成功时执行的函数，fail()将在失败时运行。

## 管道:

在从左到右的函数链序列中使用时，第一个参数可以是初始值，然后我们以无指针的方式传递函数。

## 提取全部作为任务序列

序列可以是一系列任务，下面的实用指南很好地解释了这一点。

让我们回顾一下代码:

在第 5 行中，我们定义了一个内部异步函数，它最终在 auth 成功的情况下处理我们的 fetch 调用(第 21 行)。

我们可以看到表 fetch 调用作为任务被映射到一个任务类型列表中(第 8 + 9 行)。

任务被传递给由 **array.sequence** 返回的函数，该函数返回一个异步函数(第 11 行)。

我们用管道将承诺列表作为初始值，并用折叠函数将结果处理为错误情况或成功情况(第 12–21 行)。

完整的代码可以在这里找到，如果你觉得有用的话，请随意使用《⭐️》!

这个帖子就说到这里，

## 结束想法:

我必须说，我在采用 fp-ts 的时候很开心，但是在写作开始流畅之前还有一个小的学习曲线要通过:)

关于我是否会在未来的项目中采用它，就个人而言，有一次可能会，在 fp-ts 提供的范畴理论的上下文中有一些非常酷的抽象和类型，在这篇文章中没有提到，我也很想利用它们。

看看这个精彩的系列:

[](https://dev.to/gcanti/getting-started-with-fp-ts-semigroup-2mf7) [## fp-ts 入门:半群

### 由于半群是函数式编程的一个基本抽象，这篇博文将会比…

开发到](https://dev.to/gcanti/getting-started-with-fp-ts-semigroup-2mf7) 

感谢阅读🙏

干杯，勒荣。