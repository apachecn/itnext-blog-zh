# 颤振最佳实践—第 3 部分

> 原文：<https://itnext.io/flutter-best-practices-part-3-747f1bfaec6b?source=collection_archive---------1----------------------->

![](img/7d76ede5bf10ec08dd5e7145dfe6c98c.png)

颤振最佳实践

# **视频教程**

你可以在下面找到这个系列的前两部分

[](/flutter-best-practices-part-1-e89467ea4823) [## 颤振—最佳实践—第 1 部分

### 这是颤振最佳实践的第一部分…

itnext.io](/flutter-best-practices-part-1-e89467ea4823) [](/flutter-best-practices-part-2-e9e5c79ccb16) [## 颤振最佳实践—第 2 部分

### 让我们开始…

itnext.io](/flutter-best-practices-part-2-e9e5c79ccb16) 

*让我们看看下面的 5 个最佳实践。*

# 1.使用原始字符串

原始字符串可以用来避免只转义反斜杠和美元。

*不要*

```
var s = ‘This is test string \\ and \$’;
```

*做*

```
var s = r’This is test string \ and $’;
```

# 2.避免打`print()`电话

您可以使用 **print()** 或 **debugPrint()** 进行记录。

**print()** 如果太多可能会截断 android 中的一些日志。

使用 **debugPrint()** 或 **debugPrintThrottled()** 来避免这种情况。

**log** 也有助于 dart 开发工具显示格式化日志。

如果您记录了太多的数据，那么

```
import "dart:developer"log('your log')
```

# 3.仅在调试模式下使用 print 语句

**print** 和 **log** 语句只能在应用程序的调试模式下使用。

使用 *kDebugMode* 检测调试或发布模式

> kReleaseMode ，在发布版本中也是如此。
> 
> [kProfileMode](https://api.flutter.dev/flutter/foundation/kProfileMode-constant.html) ，这在概要文件构建中是正确的。

```
if(kDebugMode){
    print("I am running in Debug Mode")
}
```

# 4.私有变量使用前导下划线(' _ ')

在变量前添加“ **_** ”，使其成为类的私有变量。

函数中的变量不应该使用“ **_** ”作为变量，因为它已经是该函数的私有变量。

# 5.对未修改的变量使用 final/const 关键字

" **final** "表示单赋值:一个 final 变量或字段必须有一个初始化器。一旦赋值，最终变量的值就不能更改。final 修改变量。

如果在编译时不知道值，final 应该用在 const 之上，它会在运行时被计算/抓取。

如果你有一个 const 集合，里面的所有东西都在 const 中。如果你有一个最终的收藏，里面的一切都不是最终的。

有更多的差异和更多的理解的例子。

*不要*

```
String firstName = "John"
int a = 1
```

*做*

```
final String firstName = "John"
const a = 1
```

***更来了……***

***敬请期待，成为颤振中的亲…***

***请在下面分享你的宝贵意见。***

*更多* [*编写教程*](https://coderzheaven.com)

[*视频教程*](https://www.youtube.com/user/coderzheaven)