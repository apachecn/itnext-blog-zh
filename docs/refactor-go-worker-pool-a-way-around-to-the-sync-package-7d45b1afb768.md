# [Refactor]Go Worker Pool——一种绕过同步包的方法

> 原文：<https://itnext.io/refactor-go-worker-pool-a-way-around-to-the-sync-package-7d45b1afb768?source=collection_archive---------0----------------------->

## 通过使用信号量模式而不是 sync.WaitGroup 来同步工作线程执行。

![](img/a3cef63a988ce7916768959a141dc73a.png)

> **在您继续之前**，我想让您知道，本文不会告诉您信号量比使用 WaitGroups 更好，反之亦然，但是您可以通过纯粹使用通道为我们的 WorkerPool 实现选择一种不同的方法。

# 介绍

> **【TL:DR】***你可以跳过这个* [*直接跳到实现*](#8501) *。*

上周我想出了 [***“向我解释 Go Worker Pool 模式就像我五岁一样”***](/explain-to-me-go-concurrency-worker-pool-pattern-like-im-five-e5f1be71e2b0) 的帖子，在那里我们经历了这个模式，它的组件，以及它们是如何一起工作的。在我的 LinkedIn 账户上发布了这篇文章后，我的一位前同事向我指出，我们仍然可以让`WorkerPool`照原样工作，而不需要使用 sync 包中的`WaitGroup`作为`Worker`的同步机制。

[](/explain-to-me-go-concurrency-worker-pool-pattern-like-im-five-e5f1be71e2b0) [## 向我解释 Go 并发工作池模式，就像我五岁时一样

### 简单解释一下这个模式的所有组件如何协同工作来并发处理一批作业。

itnext.io](/explain-to-me-go-concurrency-worker-pool-pattern-like-im-five-e5f1be71e2b0) 

记住上一篇文章，`WaitGroup`的目标是阻塞运行`WorkerPool`的 goroutine，直到每个`Worker`完成`Job`的处理。因此，如果我们不得不移除`WaitGroup`，我们将不得不以某种方式模拟它的功能。我们如何实现这样的行为实现？

# 关键是利用渠道！

从某种意义上说，要从组合中去掉`WaitGroup`，我们必须:

1.  创建一个`execution` `slot`计数器。每个被允许加工`Job`的`Worker`必须`**acquire**`一个`execution`一个`slot`。该操作*将* `slots`可用性减少 1。
2.  将`slot`标记为`**release**` d，这个*在`Worker`完成后增加* `slots`可用性 1。
3.  阻止`WorkerPool`返回，所以我们`**wait**`直到`Worker` s 完成他们的处决。

## 计数槽的通道

用工作计数容量配置的`execution` `slots`缓冲通道将代表`slot`总计数。

```
*type* slot *struct*{}
*type* slots *chan* slot

*type* execution *struct* {
   slots slots
}

*func* newExecutionSlots(capacity int) execution {
   slots := make(*chan* slot, capacity) // capacity = worker count
   *return* execution{slots: slots}
}*func* (e execution) **acquire**() { // **1
**   e.slots <- slot{}
}

*func* (e execution) **release**() { // **2**
   <-e.slots
}

*func* (e execution) **wait**() { // **3**
   *for* i := 0; i < cap(e.slots); i++ {
      e.slots <- slot{} // empty struct type
   }
}// close the slots channel when done with it
*func* (e execution) close() {
   close(e.slots)
}
```

1.  对于`**acquire**` 其中一个，我们必须将一个空结构推到`execution`的`slots`通道上。此*会降低* `slot`的可用性。
2.  为了`**release**`，需要从`execution`的`slots`通道中读出一个`slot`读数。这个*增加了* `slot`的可用性。
3.  到`WorkerPool`上的`**wait**`为了让`Workers`完成，我们试图在工人计数到`slots`通道时推送许多空结构(目的是将它们全部取出)。但是这是不允许的，因为这个通道被`Workers`完全`**acquire**` d(没有更多的空闲槽可用)，这会阻塞当前的 goroutine。这种情况会一直发生，直到`Worker`结束，而`**release**`成为`slot`为止。因此，通过这种方式，我们阻止了`WorkerPool`从执行中返回。

正如我们在这里看到的，`Worker`同步将由这个在`Worker`的 goroutines 间共享的*伪*计数器来管理。增加/减少`slot`可用性将由我们通过`slots`渠道执行的操作来表示。

*   `**slots**` **计数** ( `Worker`计数)= `*slot*` *通道容量*
*   **`**slots**`**计数** = `*slot*` *通道长度***
*   ****空闲** `**slots**` **计数** = ( *容量—长度*)**

**我们在这里所做的只是描述一个`**semaphore**`实现。在计算机科学中，这是一种允许进程安全并发地访问共享资源的机制。**

```
*type* **semaphore** *interface* {
   acquire()
   release()
   wait()
   close()
}
```

**为了寻找一个易于理解的概念描述，我无意中发现了维基百科的一个章节，它很好地描述了这个概念:**

> **将信号量视为在现实系统中使用的一种有用的方式是记录**有多少单位的特定资源可用**，以及在单位被**获取**或**变得自由(释放)**时安全地调整该记录的操作(即，避免竞争条件)，如果必要的话，**等待**直到一个单位的资源变得可用。**

**因此，我们可以将`execution` `slots`通道视为`slots`可用性的计数器(槽是我们的“*共享资源*”)。每个`Worker`都要声明一个`slot`才能运行。当所有的`slots`都是`**acquired**`时，`WorkerPool`将`**wait**`为它们全部完成以获得`**release**` d `slot`。**

**最后，`WorkerPool`代码将如下所示:**

```
*func* (wp WorkerPool) Run(ctx context.Context) {
   eSlots := **newExecutionSlots**(wp.workersCount) **// create new slots**
   *defer* eSlots.close() *// closing slots channel before WP returns*

   *for* i := 0; i < wp.workersCount; i++ {
      **//** ***for each worker acquire an execution slot***
      eSlots.**acquire**() *// share the slot channel to workers to let them
     // mark them as released as soon they finish      
     go* worker(ctx, wp.jobs, wp.results, **eSlots**)
   } **// wait for workers to finish its execution**   eSlots.**wait**()
   close(wp.results)
}
```

## **摘要**

**这种重构使您的代码只依赖于通道，并让您有机会完全摆脱对`sync`包的依赖，因为您只需使用通道就可以模拟`WaitGroup`行为。**

**我把 **PR** 留给你，你可以在这里检查我对`WorkerPool`的代码所做的更改以及`WaitGroup`是如何被替换的。**

**[](https://github.com/godoylucase/workers-pool/pull/3) [## 用信号量替换 WaitGroup，而不是用 godoylucase Pull 请求#3 …

### 将此建议添加到可以作为单次提交应用的批处理中。此建议无效，因为没有更改…

github.com](https://github.com/godoylucase/workers-pool/pull/3) 

欢迎任何意见或建议，感谢阅读！**