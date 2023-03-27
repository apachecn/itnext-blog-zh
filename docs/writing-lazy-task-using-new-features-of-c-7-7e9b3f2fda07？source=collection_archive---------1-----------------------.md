# 使用 C# 7 的新特性编写“懒惰任务”

> 原文：<https://itnext.io/writing-lazy-task-using-new-features-of-c-7-7e9b3f2fda07?source=collection_archive---------1----------------------->

![](img/764a8582c90637b9d98a75396e50c5d1.png)

无论您是否“等待”它们，您在 C#代码中处理的几乎 100%的异步任务正在运行或已经完成。下面的例子可以说明这一点:

```
**static async** Task Main()
{
    **var** task = GetValueAsync();
    **await** Task.Delay(500);
    Console.WriteLine("*Awating...*"); Console.WriteLine("*Result is:* " + **await** task);
}**static async** Task<**int**> GetValueAsync()
{
    Console.WriteLine("*Async Start*");
    **await** Task.Delay(100);
    Console.WriteLine("*Async End*");
    **return** 42;
}
```

输出:

```
Async Start
Async End
Awating...
Result is: 42
```

如您所见，**等待**之前 **GetValueAsync** 已完成，但是如果行为不是您所期望的，并且您不希望任务在至少有一个“等待者”之前开始执行，该怎么办呢？当然，您可以使用**惰性**——来自**系统**名称空间的通用类:

```
**static async** Task Main()
{
    **var** task = **new** Lazy<Task<int>>(GetValueAsync);
    **await** Task.Delay(500);
    Console.WriteLine("*Awating...*");
    Console.WriteLine("*Result is:* " + **await** task.Value);
}**static async** Task<**int**> GetValueAsync()
{
    Console.WriteLine("*Async Start*");
    **await** Task.Delay(100);
    Console.WriteLine("*Async End*");
    **return** 42;
}
```

输出:

```
Awating...
Async Start
Async End
Result is: 42
```

然而，还有另一个解决方案会稍微影响您的客户端代码——它是用 **LazyTask** 替换由您的异步函数返回的**Task**——一个您可以使用新的 C#7 特性创建的类— **通用异步返回类型**

因此，您的代码将如下所示:

```
**static async** Task Main()
{
    **var** task = GetValueAsync();
    **await** Task.Delay(500);
    Console.WriteLine("*Awating...*");
    Console.WriteLine("*Result is:* " + await task);
}**static async** LazyTask<**int**> GetValueAsync()
{
    Console.WriteLine("*Async Start*");
    **await** Task.Delay(100);
    Console.WriteLine("*Async End*");
    **return** 42;
}
```

输出:

```
Awating...
Async Start
Async End
Result is: 42
```

众所周知，C#编译器将标记为 **async** 的方法转换为状态机，其中每个状态代表一个标记为 **await** 的异步操作的完成。**通用异步返回类型**允许你访问状态机执行(我在[我以前的一篇文章](https://habr.com/en/post/458692/)中详细研究了这个机制)。)在我们的例子中，我们只需要推迟第一步，直到我们至少有了一个“奖励者”。这可以通过调整**启动**的方法来实现:

> [lazytaskmethodbuilder . cs](https://github.com/0x1000000/LazyTask/blob/master/LazyTask/LazyTaskMethodBuilder.cs)

```
**public class** LazyTaskMethodBuilder<T>
{
    **public** LazyTaskMethodBuilder() => this.Task = **new** LazyTask<T>(); **public void** Start<TStateMachine>(
        **ref** TStateMachine stateMachine)..
    {
        *//instead of stateMachine.MoveNext();*
        **this**.Task.SetStateMachine(stateMachine);
    }
    ...
}
```

**Start** 方法并没有开始第一步，而是在 **LazyTask** 对象中存储了一个到状态机的链接。

只有当我们有了第一个“奖赏者”时，第一步才能被称为:

> [LazyTask.cs](https://github.com/0x1000000/LazyTask/blob/master/LazyTask/LazyTask.cs)

```
**public void** OnCompleted(Action continuation)                               {
    ... **this**._asyncStateMachine.MoveNext();
    ...}
```

*注意:有趣的是，这个技巧甚至可以在没有任何异步调用的方法中使用(没有“await”关键字)。*

仅此而已！

总之，我想说的是**通用异步返回类型**是一个强大的机制，而 **LazyTask** 只是如何利用它的一个例子。

## 链接

1.  [git nub 上的源代码；](https://github.com/0x1000000/LazyTask)

[2。“也许”monad 通过异步/等待在 C#中(没有任务！)](https://habr.com/en/post/458692/) —一篇详细分析了**广义异步返回类型**的文章。