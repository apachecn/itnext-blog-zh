# 使用 TaskCompletionSource 组装您自己的任务

> 原文：<https://itnext.io/assembling-your-own-tasks-with-taskcompletionsource-82bfdab5b995?source=collection_archive---------3----------------------->

![](img/e986eaa71a3c6d9e67f74cb0b961990e.png)

在我们的日常工作中，我们几乎只处理具有预定义和可预测集合的任务。例如，您的代码中可能有一个如下所示的方法:

```
private async Task<string> GetInfo()
{
    var response = await _httpClient.GetAsync("[https://medium.com](https://medium.com)");
    return await response.Content.ReadAsStringAsync();
}
```

当然，这种方法可能仍然需要很长的时间来完成，因为许多变量会影响连接等。—然而，方法本身仍然是非常可预测的。我们有一组必须完成的指令，然后任务就完成了。

然而，如果我们想让一个任务代表一个预先不可预测的操作，我们该怎么做呢？基于事件的 API 就是一个例子。在某些情况下，您可能不得不放弃对任务的控制，并在特定情况发生时做出反应:

```
public class EventTracker
{
    private void OnEvent(EventData eventData)
    {
        // Raised when event is received
    } public async Task WaitForNextEvent()
    {
        await ...?
    }
}
```

我们该怎么做？

# 任务完成资源拯救世界

对于这种情况。Net 提供了*TaskCompletionSource<T>*类。这个类提供了一个底层任务来分发给消费者，它可以像任何其他任务一样使用。然而，另一方面，生产者可以明确地控制这个任务的生命周期。

为了做到这一点，*TaskCompletionSource<T>公开了一组方法:*

*   *SetResult(T result)* —设置底层任务的特定结果，并将其移入 *RanToCompletion* 状态。
*   *SetCanceled()* —将任务移动到*已取消*状态。
*   *set Exception(Exception exc)/set Exception(IEnumerable<Exception>exceptions)—*将任务移入 *Faulted* 状态，并向其添加相应的异常。

注意，所有这些方法都有一个 *Try…* 版本来防止已经发生的任务消耗。

听起来不错，所以让我们看看如何在前面的例子中使用它。

```
public record EventData();public class EventTracker
{
    private TaskCompletionSource<object> _tcs; public void OnEvent(EventData eventData)
    {
        _tcs.SetResult(null);
    } public async Task WaitForNextEvent()
    {
        _tcs = new TaskCompletionSource<object>();
        await _tcs.Task;
    }
}public static async void Main(string[] args) 
{
    var eventTracker = new EventTracker();
    _ = Task.Run(async () => {
        await Task.Delay(2000);
        Console.WriteLine("Triggering event");
        eventTracker.OnEvent(new EventData());
    });

    Console.WriteLine("Awaiting event...");
    await eventTracker.WaitForNextEvent();
    Console.WriteLine("Event awaited!");
}// Output:
Awaiting event...
Triggering event
Event awaited!
```

完美！那么，到底发生了什么？

1.  我们创建一个新的 EventTracker 实例。
2.  出于嘲弄的目的，我们在这里开始一个小任务，它将在 2 秒钟后“引发”一个事件。
3.  现在，我们进入 EventTracker 实例的 *WaitForNextEvent()* 方法。在这里，我们创建了我们的 *TaskCompletionSource <对象>* 的一个新实例，并获取底层任务，我们将立即等待它。注意，不幸的是没有非通用版本的 *TaskCompletionSource* ，所以对于一个“中性”版本，我们需要使用 *TaskCompletionSource <对象>。*
4.  2 秒钟后，我们的后台任务触发*OnEvent(event data event data)*方法，并在 *TaskCompletionSource* 上设置结果。此时，任务被标记为已完成，我们的等待方法可以返回。

# 逮到你了

我想提两个问题。

第一个是指默认情况下， *TaskCompletionSource* 同步运行所有延续的特殊行为。为了安全起见，您应该始终像这样构造 TaskCompletionSource:

```
new TaskCompletionSource<object>(TaskCreationOptions.RunContinuationsAsynchronously);
```

如果你想了解更多这方面的内容，我绝对可以推荐[这篇](https://devblogs.microsoft.com/premier-developer/the-danger-of-taskcompletionsourcet-class/)文章——在这里谈论可能会有点太长。

除了这个具体的技术问题之外，我还想指出这样一个事实，即虽然使用 *TaskCompletionSource* 作为一种基于事件的 API 创建任务的方法很有吸引力，但是您也应该记住，很多事情都可能出错，特别是当事件通过网络传入时，这可能会导致任务被无限期地等待。

一个可能减轻这种情况的例子是创建一个定制的 *TaskCompletionSource* ，或者创建一个提供 CancellationToken 的工厂，这可以确保任务不会永远挂起:

```
public class TimeoutTaskCompletionSource<T> : TaskCompletionSource<T>
{
    public TimeoutTaskCompletionSource(TaskCreationOptions        creationOptions)
         :base(creationOptions)
        {
            var cts = new CancellationTokenSource(1000);
            cts.Token.Register(SetCanceled);
        }
}
```

例如，这个实现将在一秒钟后自动取消任务。不过，我想你明白我要去哪里。当你控制了任务的生命周期，你也控制了悲观的路径，而不仅仅是乐观的路径。

因此，总结一下:如果您需要对任务对象进行显式控制，例如在基于事件的架构中，那么 *TaskCompletionSource* 就是您的完美 API！