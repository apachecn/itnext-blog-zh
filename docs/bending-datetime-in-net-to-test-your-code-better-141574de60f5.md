# 在中弯曲日期时间。NET 来更好地测试您的代码

> 原文：<https://itnext.io/bending-datetime-in-net-to-test-your-code-better-141574de60f5?source=collection_archive---------3----------------------->

## 模拟中的日期时间。网络是痛苦的，这里有一个让它不那么痛苦的方法。

![](img/63cb880df545af78c0325e4e6fcf3d1c.png)

好吧，我会减少夸张，在我作为一名工程师的这么多年里(我感觉自己老了)，我遇到过一个问题，我的代码包含了一个`DateTime.Now`或`DateTime.UtcNow`。当我编写一个测试时，我不能验证实际的时间，因为从代码运行到我的测试开始验证已经过去了几毫秒。这不是一个很大的问题，但很烦人，因为我喜欢验证一切，以确保我不会在其他地方意外地操纵这些值。

对此有一个简单的解决方案，在我详细说明我所使用的解决方案之前，我需要为此调用[灵感](https://github.com/stphnwlsh/SimpleDateTimeProvider#inspiration)。TL；DR 就是你可以消费我的[SimpleDateTimeProvider NuGet 包](https://www.nuget.org/packages/SimpleDateTimeProvider/)帮你解决这个。这方面的代码实现如下。

# 解决方案

我创建了一个由一个接口和该接口的两个实现组成的`DateTimeProvider`。一个实现返回系统值，另一个返回用户预设的模拟值。

# 界面

```
public interface IDateTimeProvider
{
    DateTime Now { get; }
    DateTime Today { get; }
    DateTime UtcNow { get; }
}
```

# 系统实现

```
public class SystemDateTimeProvider : IDateTimeProvider
{
    public DateTime Now => DateTime.Now;
    public DateTime Today => DateTime.Today;
    public DateTime UtcNow => DateTime.UtcNow;
}
```

# 模拟实现

```
public class MockDateTimeProvider : IDateTimeProvider
{
    public DateTime Now
    {
        get => this.now.ThrowIfNotSet(DateTimeType.Now);
        set => this.now = value;
    }
    public DateTime Today
    {
        get => this.today.ThrowIfNotSet(DateTimeType.Today);
        set => this.today = value;
    }
    public DateTime UtcNow
    {
        get => this.utcNow.ThrowIfNotSet(DateTimeType.UtcNow);
        set => this.utcNow = value;
    }
}
```

# 弯曲日期和时间

这是一个简单的解决方案，使用提供者很容易开始，只需在功能代码中的`IDateTimeProvider`接口下注入系统提供者。如果你正在使用另一个库，你会知道语法，但遵循相同的公式。

```
_ = services.AddSingleton<IDateTimeProvider, SystemDateTimeProvider>();
```

下一步是创建您的类，并使用我们刚刚通过`IDateTimeProvider`接口创建的注册的`SystemDateTimeProvider`。然后使用提供程序在您的类中设置`DateTime`值。

```
public class Service
{
    private readonly IDateTimeProvider dateTimeProvider; public Service(IDateTimeProvider dateTimeProvider)
    {
        this.dateTimeProvider = dateTimeProvider;
    } public string DateTimeNow()
    {
        return $"DateTime.Now is {this.dateTimeProvider.Now}";
    }
}
```

这样做的全部目的是考虑到可测试的代码。现在你有了上面的类，你可以在它的位置注入`MockDateTimeProvider`来控制测试中的`DateTime`值。以下示例显示了如何在 [XUnit](https://xunit.net/) 中编写测试，使用 [Shouldly](https://shouldly.io/) 进行断言。

```
[Fact]
public void Today_ShouldReturn_MockedToday()
{
    // Arrange
    var provider = new MockDateTimeProvider();
    var service = new Service(provider);
    var today = DateTime.Today; provider.Today = today; // Act
    var result = service.DateTimeToday(); // Assert
    _ = result.ShouldBeOfType<string>();
    result.ShouldBe($"DateTime.Today is {today}");
}
```

# 我能在哪里找到这个？

所有这些都是开源的。你可以在[SimpleDateTimeProvider Repository](https://github.com/stphnwlsh/SimpleDateTimeProvider)找到我在 GitHub
上的工作，在[SimpleDateTimeProvider NuGet](https://www.nuget.org/packages/SimpleDateTimeProvider/)找到发布的包。

# 支持

如果你喜欢这个，在 [GitHub](https://github.com/stphnwlsh) 上看看我的其他例子，并考虑在[给我买杯咖啡](https://www.buymeacoffee.com/stphnwlsh)上支持我。

![](img/d391e374cf2d7e544d2a5e8768be87e6.png)

*原载于 2022 年 1 月 19 日*[*https://dev . to*](https://dev.to/stphnwlsh/bending-datetime-in-net-to-test-your-code-better-2lon)*。*