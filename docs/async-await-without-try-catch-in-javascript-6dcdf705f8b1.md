# JavaScript 中不带 try/catch 的异步/等待

> 原文：<https://itnext.io/async-await-without-try-catch-in-javascript-6dcdf705f8b1?source=collection_archive---------0----------------------->

![](img/3d174c9343a9f6ddb1048c6cb7eec7c1.png)

*“如何确保正确处理异步错误”*

当 *async/await* 发布时，它成为了 JavaScript 开发的游戏改变者。它允许以同步的方式编写代码，我们不需要连锁的承诺处理程序:

如何用*异步/等待*语法重构这段代码:

现在遵循代码更容易了。但是我们仍然可以对代码做一些改进，使其更加整洁。

因为我们用 *try/catch* 包装了我们的异步函数，我们期望在下面的 *catch* 块中处理来自这个函数的所有错误。但是如果我们在这个 *try* 块中继续我们的代码呢？这意味着所有错误都将落入最近的 *catch* 块。

我们添加的代码越多， *catch* 块就变得越不清晰，越普通。我们防止这种情况的唯一方法是仔细检查代码。但是我们怎么能只对 *fetchPosts* 函数进行错误处理呢？

即使我们的代码看起来不像是在履行承诺，但别忘了我们仍然在履行承诺。每一个用 *async* 关键字声明的函数都返回一个承诺，即使我们没有明确的返回语句。因为我们有一个承诺，我们可以使用它的所有方法。我们感兴趣的是[promise . prototype . catch](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/catch)这个方法也返回一个承诺，所以我们可以将它与一个 *await* 关键字结合起来。

这允许显式地捕捉异步函数的错误，并且当 *doSomethingWithPosts* 中发生某些事情时，它不会进入*fetch post catch*块，我们可以单独处理错误。

# 结论

当使用 *async/await* 时，我们可以使用常规承诺的所有函数来避免代码中的 *try/catch* 噪声，有助于显式处理错误并保持变量为常量。