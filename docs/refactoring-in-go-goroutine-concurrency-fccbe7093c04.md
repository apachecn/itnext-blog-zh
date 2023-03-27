# Go 中的重构:goroutine 并发

> 原文：<https://itnext.io/refactoring-in-go-goroutine-concurrency-fccbe7093c04?source=collection_archive---------3----------------------->

![](img/becd03b9492e08c557a6db26ba9cd7c3.png)

约翰·卡莱尔在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

惊喜！这几天我写围棋。

最近，我在野外发现了一些代码，它们使用了简单的并发解决方案。鉴于我多次看到类似的模式，我的理论是，它可能是受基本的 goroutines 示例代码的启发。

# 场景

假设您想要运行特定数量的易于并行化的任务(没有副作用，没有外部依赖性，等等)。)，并且您希望存储它们每个的结果。在围棋中做到这一点的方法是使用 Goroutines。

我重构的真实代码通过调用`net.LookupHost` 100 次来计算我们系统中的平均 DNS 延迟。让我们看看。

# 在野外发现的实际代码

这是在野外发现的(改编并简化的)代码。看一看，下面我们来讨论一下:

```
func AverageLatency(host string) (latency int64, err error) {
    CONCURRENCY := 4
    REQUESTS_LIMIT := 100

    dnsRequests := make(chan int, REQUESTS_LIMIT)
    results := make(chan int64, REQUESTS_LIMIT)
    errorsResults := make(chan string, REQUESTS_LIMIT)

    for w := 1; w <= CONCURRENCY; w++ {
        go dnsTest(dnsRequests, results, errorsResults, host)
    }

    for j := 1; j <= REQUESTS_LIMIT; j++ {
        dnsRequests <- j
    }
    close(dnsRequests)

    requestsDone := 1
    for a := 1; a <= REQUESTS_LIMIT; a++ {
        select {
        case latencyLocal := <-results:
            latency = latency + latencyLocal
            requestsDone = requestsDone + 1
        case errorMsg := <-errorsResults:
            return 0, errors.New(errorMsg)
        case <-time.After(time.Second * DURATION_SECONDS):
            return latency / int64(requestsDone), nil
        }
    }
    return latency / int64(requestsDone), nil
}

func dnsTest(jobs <-chan int, results chan<- int64, errResults chan<- string, host string) {
    for range jobs {
        start := time.Now()
        if _, err := net.LookupHost(host); err != nil {
            errResults <- err.Error()
        }
        results <- time.Since(start).Nanoseconds() / int64(time.Millisecond)
    }
}
```

主代码按顺序执行以下步骤:

*   启动`CONCURRENCY`个 goroutines `dnsTest`
*   使用通道向 goroutines 发送`REQUESTS_LIMIT`个作业(DNS 请求)
*   关闭 dnsRequests 通道，通知 goroutines 不再有作业了(这样它们就不会一直等待)
*   它等待来自`results`的`REQUESTS_LIMIT`数量的值，我们将每个值添加到`latency`变量中。它还监听错误和超时，如果出现任何错误和超时，就退出函数。
*   在超时的情况下，它用我们到目前为止得到的值计算平均延迟，然后退出。
*   最后，它将`latency`中累积的值除以完成的请求数量，以确定平均延迟。

咻。那是一种做它的方法…但是它是惯用的吗？

# Goroutines 很便宜，让我们利用这一点

Goroutines 是轻量级线程，这意味着我们可以在不太影响程序性能的情况下启动大量线程。那么，为什么不推出和我们现有的工作一样多的 goroutines 呢？这样我们就摆脱了一个仅仅用于迭代的渠道，以及所有与之相伴的业务逻辑。我们还可以将 goroutine 代码移到循环内部，假设它足够小:

```
func AverageLatency(host string) (latency int64, err error) {
    REQUESTS_LIMIT := 100

    results := make(chan int64, REQUESTS_LIMIT)
    errorsResults := make(chan string, REQUESTS_LIMIT)

    **for w := 1; w <= REQUESTS_LIMIT; w++ {
        go func() {
            start := time.Now()
            if _, err := net.LookupHost(host); err != nil {
                errorResults <- err.Error()
                return
            }
            results <- time.Since(start).Nanoseconds() / int64(time.Millisecond)
        }
    }**

    requestsDone := 1
    for a := 1; a <= REQUESTS_LIMIT; a++ {
        select {
        case latencyLocal := <-results:
            latency = latency + latencyLocal
            requestsDone = requestsDone + 1
        case errorMsg := <-errorsResults:
            return 0, errors.New(errorMsg)
        case <-time.After(time.Second * DURATION_SECONDS):
            return latency / int64(requestsDone), nil
        }
    }
    return latency / int64(requestsDone), nil
}
```

代码现在更清晰了。我们循环`REQUESTS_LIMIT`次，每次都创建一个 goroutine 来检查查找主机的延迟。然后我们等待通道`results`、`errorResults`和`time.After(..)`产生值`REQUESTS_LIMIT`次。

# 利用等待组和清理

我对最后一个循环不太满意，它看起来有点脆弱，Go 有更优雅的工具来帮助从生成的 gouroutines 中收集结果。其中之一就是`WaitGroup`。一个`Waitgroup`是一个等待一组例行程序完成的构造，而不需要我们做太多。我们只需使用`WaitGroup.Add`增加我们运行的每个 goroutine 的`WaitGroup`计数器，当一个 goroutine 结束时，我们调用`WaitGroup.Done`。

```
func AverageLatency(host string) (latency int64, err error) {
    REQUESTS_LIMIT := 100
    results := make(chan int64, REQUESTS_LIMIT)
    errorsResults := make(chan string, REQUESTS_LIMIT)

    **var wg sync.WaitGroup
    wg.Add(REQUESTS_LIMIT)**

    for j := 0; j < REQUESTS_LIMIT; j++ {
        go func() {
            **defer wg.Done()**
            start := time.Now()
            if _, err := net.LookupHost(host); err != nil {
                errorResults <- err.Error()
                return
            }
            results <- time.Since(start).Nanoseconds() / int64(time.Millisecond)
        }
    }

    **wg.Wait()**

    ...
}
```

此时，我们不需要通道来收集错误和结果，我们可以使用更传统的数据类型。

我还想收集错误的数量，以便进行监控，而不仅仅是在出现一个错误时返回。如果我们想查看实际的错误消息，可以手动检查日志:

```
**type Metrics struct {
    AverageLatency float64
    RequestCount   int64
    ErrorCount     int64
}**

func AverageLatency(host string) Metrics {
    REQUESTS_LIMIT := 100
    var errors int64
    results := make([]int64, 0, DEFAULT_REQUESTS_LIMIT)

    **var wg sync.WaitGroup
    wg.Add(REQUESTS_LIMIT)**

    for j := 0; j < REQUESTS_LIMIT; j++ {
        go func() {
            **defer wg.Done()**
            start := time.Now()
            if _, err := net.LookupHost(host); err != nil {
                fmt.Printf("%s", err.Error()) **atomic.AddInt64(&errors, 1)**
                return
            }
            **append(results, time.Since(start).Nanoseconds() / int64(time.Millisecond))**
        }
    }

    **wg.Wait()**

    return CalculateStats(&results, &errors)
}
```

我们创建了一个新的类型`Metrics`来对我们想要监控的值进行分组。我们现在将`wg`计数器设置为我们计划执行的请求数，并且当每个 goroutine 完成时，我们调用`wg.Done`,不管它是否成功。然后我们等待所有的 goroutines 完成。我们还将所有的统计数据计算提取到一个外部的`CalculateStats`函数中，这样`AverageLatency`就可以专注于收集原始值。

为了完整起见，`CalculateStats`可能是这样的:

```
// Takes amount of requests and errors and returns some stats on a 
// `Metrics` struct
func CalculateStats(results *[]int64, errors *int64) Metrics {
    successfulRequests := len(*results)
    errorCount := atomic.LoadInt64(errors)

    // Sum up all the latencies
    var totalLatency int64 = 0
    for _, value := range *results {
        totalLatency += value
    }

    avgLatency := float64(-1)

    if successfulRequests > 0 {
        avgLatency = float64(totalLatency) / float64(successfulRequests)
    }

    return Metrics{
        avgLatency,
        int64(successfulRequests),
        errorCount
    }
}
```

# 等等，暂停在哪里？！

哦，对了。在上次重构中，我们似乎忘记了恢复超时，不是吗？

使用`WaitGroup`时设置超时有点麻烦，但是使用通道变得容易了。我们不再直接调用`wg.Wait`，而是将该调用封装在一个名为`waitWithTimeout`的新函数中:

```
func waitWithTimeout(wg *sync.WaitGroup, timeout time.Duration) bool {
    c := make(chan struct{})
    go func() {
        defer close(c)
        wg.Wait()
    }()

    select {
    case <-c:
        return false
    case <-time.After(timeout):
        return true
    }
}
```

这里我们创建了一个一旦`wg.Wait()`运行就会关闭的通道，这意味着所有的 goroutines 都已完成。然后我们使用一个`select`语句来返回所有的 goroutines 完成的时间，或者已经过了`timeout`的时间。在这种情况下，我们返回`true`来通知超时已经发生。

我们的最终`AverageLatency`将最终看起来像这样:

```
func AverageLatency(host string) Metrics {
    REQUESTS_LIMIT := 100
    var errors int64
    results := make([]int64, 0, REQUESTS_LIMIT)

    var wg sync.WaitGroup
    wg.Add(REQUESTS_LIMIT)

    for j := 0; j < REQUESTS_LIMIT; j++ {
        go func() {
            defer wg.Done()
            start := time.Now()
            if _, err := net.LookupHost(host); err != nil {
                fmt.Printf("%s", err.Error())
                atomic.AddInt64(&errors, 1)
                return
            }
            append(results, time.Since(start).Nanoseconds() / int64(time.Millisecond))
        }
    }

    **if waitWithTimeout(&wg, time.Duration(time.Second*DURATION_SECONDS)) {
        fmt.Println("There was a timeout waiting for DNS requests to finish")
    }**
    return CalculateStats(&results, &errors)
}
```

在超时的情况下，我们只显示一条消息，因为我们仍然想知道发出了多少个请求和错误，而不管是否超时。

# 奖金:比赛条件！！

哈，我敢肯定你认为就是这样，世界上的一切都是好的。但是没有！我在代码中加入了一个竞争条件。你们中的一些人可能已经发现了。对于没有看过的人，我会让你看一下上面的代码。我会等的。

你找到了吗？就算你找到了也没关系，我不会对你评头论足，我自己可能也找不到。不管怎样，谢谢你迁就我！

好吧，让我们来看看这整个比赛条件是怎么回事。

## 在 Go 中，事情很少是线程安全的

每当我们用 Goroutines 操作时，我们必须小心修改外部变量，因为它们通常不是线程安全的。在我们的主 goroutine 中，我们通过使用`atomic.AddInt64`以线程安全的方式存储错误，但是我们将延迟存储在`results`片上，这不是线程安全的。绝对不能保证`results`中的条目数量与我们尝试的请求数量相匹配。哎呀。

在围棋中，有几种方法可以解决这个问题，例如:

*   返回使用通道而不是切片来收集延迟
*   使用互斥体来控制对`results`的访问
*   使用缓冲容量为 1 的通道作为队列，并在那里发送延迟

鉴于我们已经深陷 goroutines，我选择使用第三种方法。这也是我发现的最惯用的方法:

```
func AverageLatency(host string) Metrics {
    REQUESTS_LIMIT := 100
    var errors int64
    results := make([]int64, 0, REQUESTS_LIMIT) **successfulRequestsQueue := make(chan int64, 1)**

    var wg sync.WaitGroup
    wg.Add(DEFAULT_REQUESTS_LIMIT)

    for j := 0; j < REQUESTS_LIMIT; j++ {
        go func() {
            start := time.Now()

            if _, err := net.LookupHost(host); err != nil {
                atomic.AddInt64(&errors, 1)
                wg.Done()
                return
            }

            **successfulRequestsQueue <- time.Since(start).Nanoseconds() / 1e6**
        }()
    }

    **go func() {
        for t := range successfulRequestsQueue {
            results = append(results, t)
            wg.Done()
        }
    }()**

    if waitTimeout(&wg, time.Duration(time.Second*DURATION_SECONDS)) {
        fmt.Println("There was a timeout waiting for DNS requests to finish")
    }
    return CalculateDNSReport(&results, &errors)
}
```

我们正在创建一个只能缓冲一个值的通道`successfulRequestsQueue`，有效地创建了一个同步队列。我们现在可以将延迟结果发送到那里，而不是直接将其附加到`results`。然后，我们在`successfulRequestsQueue`中循环所有输入延迟，并将其附加到该 goroutine 中的`results`。我们还将`wg.Done()`调用移到了`append`之后。通过这种方式，我们可以确保每个结果都会得到处理，并且不会出现竞争情况。

# 结论

我们可以进一步重构这段代码，但是出于本文的目的，我认为我们现在可以停止了。万一你想继续下去，我提出一些改进:

*   让计算函数(现在是`net.LookupHost`)通用化，这样我们就可以在测试中使用不同的函数。
*   如果没有 goroutines，您将如何编写这段代码？
*   使代码完全独立于上下文，这样我们就可以在任何时候使用它来存储许多操作的结果。