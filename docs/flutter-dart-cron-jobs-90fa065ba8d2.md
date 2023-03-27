# 飞镖克朗乔布斯

> 原文：<https://itnext.io/flutter-dart-cron-jobs-90fa065ba8d2?source=collection_archive---------2----------------------->

![](img/8b0bdf7df75649e17b4e47054853e694.png)

有时我们需要运行定期操作，无论是在前端还是后端。

包`[style_cron_job](https://pub.dev/packages/style_cron_job)`为您的作业提供了简单的语法和执行器。

对于`second`、`minute`、`hour`、`day`、`week`和`month`时计，您可以指定“y 时间内每个 x”或“y 时间内每个 x”等时间段。

用`only`添加自定义条件

并控制`CronJobController`中的所有作业

# 开始第一项工作

*除了 13 秒，每隔一秒说一声你好。秒*

```
every.x(3).second.only((t) => t.second != 13).listen((time) {
  print("Hello World!");
});
```

# 使用

将[包](https://pub.dev/packages/style_cron_job)添加到您的 pubspec.yaml

开始定义周期有两种不同的选项:`each`、`every`。

`each`用于各月、周、日、时、分、秒。
`every`用于每 3 天一次、每 2 个月一次等需求。

```
each.**
every.x(3).**
```

`**`必须是`month`，`week`，`day`，`hour`，`minute`，`second`两者都有，

> 每天的示例:`each.day..;`
> 
> 每 3 分钟一次:`every.x(3).minute..;`

## 可以指定钟表的子分段

例如`Each day ay 10:20:30`

```
each.day.atHour(10).atMinute(20).atSecond(30);
```

或者当时

```
each.day.fromNowOn(DateTime(1970 , 1 , 1 , 10 , 20 , 30));
```

## 添加自定义条件

每天 10:20，星期日除外

```
each.day.atHour(10).atMinute(20).only((time) {  
  return time.weekday != 7; // except sunday
});
```

## 开始倾听

倾听有四种方式

1.  创建一个流

```
var stream = each.minute.fromNowOn().asStream();
// eg starting 00:10:38
stream.listen((time){
   // Do on each minute on 38\. second
});
// Don't forget
stream.cancel();
```

2.直接听

与流相同

```
var period = each.day.onMinute(10);
period.listen((t) {  
    // do on each day 00:10
});
period.dispose();
```

3.控制器

控制器用单个流管理所有操作。所以这是最有效的使用方式。

```
var runner = CronJobController();runner.add(each.second.asRunner((time) {  
  // Do on each second
}));
runner.add(every.x(10).second.asRunner((time) {  
  // Do on every 10 seconds
}));

// Start
runner.start();
```

4.习俗

```
var period = each.second.period;
var necessary = period.isNecessary(DateTime.now());
if (necessary) {
   // Do
}
```

用法示例

*   每 3 分钟

```
every.x(3).minute
```

*   每 10 秒钟

```
every.x(10).second
```

*   每 30 分钟一次。第二

```
each.minute.atSecond(30)
```

*   每周周六 23:45(晚上 11:45)

```
each.week.onWeekDay(6).atHour(23).atMinute(45);
```

*   每周一 09:00

```
each.week.onWeekDay(1).atHour(9);
```

*   每月 15 日 09:00

```
each.month.onDay(15).atHour(9);
```

*   每 3 天一次，时间为 00:00:59

```
every.x(3).day.atSecond(59);
```