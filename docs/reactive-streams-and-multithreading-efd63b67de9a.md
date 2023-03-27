# 反应流和多线程

> 原文：<https://itnext.io/reactive-streams-and-multithreading-efd63b67de9a?source=collection_archive---------1----------------------->

![](img/b6a4278f5a1dbf81a3033c76019aa1ee.png)

反应流是非阻塞的、异步的、多线程的、快速的、煮咖啡的……好吧，他们不煮咖啡。然而，其他属性通常与反应流相关联。然而，当开发人员第一次尝试流时，他们经常发现它们是阻塞的、同步的和单线程的。那么这一切只是炒作吗？让我们找出答案。

# 阻塞的溪流

第一个简单的例子(使用 Spring Reactor 和 Java)通常是这样的:

```
source = Flux.*just*(1,2);source
   .map(i -> i + ". element: " + i)
   .subscribe(System.*out*::println);

System.*out*.println("Before or after?");// Output:
// 1\. element: 1
// 2\. element: 2
// Before or after?
```

所以一切似乎都是同步的。要理解为什么我们需要理解流是如何工作的。第一行创建了源流，在本例中，它基于一个数组。第二行添加了一个映射操作，从而创建了一个映射流，一个映射值流。你可能知道，在我们订阅之前什么都不会发生。当我们订阅时，会发生以下情况:

*   subscribe 方法告诉映射的流它准备好了，并期待值。
*   映射的流有一个父流，并将订阅请求转发给它。
*   源流没有父流，它立即开始发出值，并一直持续到完成。

所有的事情都发生在同一个线程上。这就是为什么最后一条语句只在流完成后执行。

# 让我们异步进行

现在让我们用另一个流来代替源程序:一个定时器。

```
source = Flux.*interval*(Duration.*ofSeconds*(1))
   .take(2);//...// Output:
// Before or after?
// 0\. element: 0
// 1\. element: 1
```

我们可以看到，流已经变成了异步的。为什么？因为源流不会立即发出值。所以很明显，源决定了我们的流是同步的还是异步的。

但是我们也可以直接影响我们流的线程行为。为了更好地说明，让我们再次改变源流:

```
source = Flux.<Integer>*create*(emitter -> {
    System.*out*.println(Thread.*currentThread*().getName());
    emitter.next(1);
    emitter.complete();
});// ...// Output:
// main
// 1\. element: 1
// Before or after?
```

我们现在可以看到源(流)在主线程上运行。现在让我们在源代码后添加 publishOn 并执行代码:

```
source.publishOn(Schedulers.*single*())
// ...//Output:
// main
// Before or after?
// 1\. element: 1
```

起初，一切都和以前一样，但是 publishOn 确保每个流操作都在另一个线程上运行。这就是为什么最后一个语句打印在值之前。

让我们尝试一些不同的方法，在订阅之前插入 subscribeOn:

```
source.map(i -> i + ". element: " + i)
.subscribeOn(Schedulers.*single*())
.subscribe(System.*out*::println);// Output:
// Before or after?
// single-1
// 1\. element: 1
```

正如我们从输出中看到的，现在甚至订阅进程也在另一个线程上运行。我们最终得到了一个真正异步的、非阻塞的流。

# 结论

反应流并不总是异步的。线程行为主要取决于源代码，并且会受到运算符的影响。不过，好的一面是代码保持不变。我们可以对同步和异步流使用相同的编程模型和相同的管道。