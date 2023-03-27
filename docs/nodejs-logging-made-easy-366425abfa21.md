# NodeJS 日志记录变得简单

> 原文：<https://itnext.io/nodejs-logging-made-easy-366425abfa21?source=collection_archive---------3----------------------->

![](img/a8a530c7cb18220a26c97b98cb91272c.png)

>我现在有了一个闪亮的新博客。阅读这篇文章，了解 https://blog.goncharov.page/nodejs-logging-made-easy[的最新动态](https://blog.goncharov.page/nodejs-logging-made-easy)

对于您想要记录的每一个服务方法，您写了多少次`logger.info('ServiceName.methodName.')`和`logger.info('ServiceName.methodName -> done.')`？你希望它是自动化的，并在你的整个应用程序中保持相同的签名吗？如果是这样的话，我们非常相似，我们遭受同样的痛苦太多次了，现在我们终于可以尝试解决它。一起。女士们先生们，让我来介绍...[类记录器](https://github.com/keenondrums/class-logger)！

# [类记录器](https://github.com/keenondrums/class-logger)的“为什么”

工程师往往是完美主义者。完美主义者到了极点。我们喜欢简洁的抽象。我们喜欢干净的代码。我们在别人甚至看不懂的人工语言中看到了美。我们喜欢制造小型数字世界，按照我们设定的规则生活。我们喜欢所有这些，可能是因为我们非常懒。不，我们不怕工作，但是我们讨厌做任何可以自动化的工作。

只编写了几千行日志代码，我们通常会提出某些模式，标准化我们想要记录的内容。然而，我们仍然必须手动应用这些模式。因此[类记录器](https://github.com/keenondrums/class-logger)的核心思想是提供一种声明性的、高度可配置的标准化方法来记录类方法执行前后的消息。

# 快速启动

让我们立即投入运行，看看实际的代码是什么样的。

该服务将记录三次:

*   创建时将一个参数列表传递给构造函数。
*   在`eat`被执行之前，它的一个参数列表。
*   在`eat`被执行后，它带有一个参数和结果的列表。

用代码的话来说:

[现场演示](https://stackblitz.com/edit/class-logger-demo-cats-quick-start)。

我们还能记录什么？以下是事件的完整列表:

*   班级建设前。
*   同步和异步之前的静态和非静态方法和功能属性。
*   经过同步和异步静态和非静态方法和功能属性。
*   同步和异步静态和非静态方法和函数属性的错误。

> *功能属性是分配给* `*class ServiceCats { private meow = () => null }*` *属性的箭头功能。*

# 根据我们的需要调整它

到目前为止还不错，但我们已经承诺“可定制”，对不对？那么我们该如何调整它呢？

[类记录器](https://github.com/keenondrums/class-logger)提供三层分层配置:

*   全球的
*   班级
*   方法

在每一次方法调用时，它们都被评估并从上到下合并在一起。有一些默认的全局配置，所以你可以不用任何配置就可以使用这个库。

## 全局配置

这是应用程序范围的配置。可以用`setConfig`呼叫设置。

## 类别配置

它对你的类的每个方法都有影响。它可以覆盖全局配置。

## 方法配置

它只影响方法本身。重写类配置，因此也重写全局配置。

[现场演示](https://stackblitz.com/edit/class-logger-demo-hierarchical-config)

# 配置选项

好吧，我们已经学会了如何改变默认设置，但它不会伤害到覆盖有什么配置，是吧？

> [*在这里你可以找到很多有用的配置覆盖*](https://github.com/keenondrums/class-logger#examples) *。* [*这里是 config 对象接口的链接，以防你打字稿说得比英语好:)*](https://github.com/keenondrums/class-logger#configuration-object)

配置对象具有以下属性:

## 原木

这是一个实际记录最终格式化消息的函数。它用于记录这些事件:

*   班级建设前。
*   同步和异步之前的静态和非静态方法和功能属性。
*   经过同步和异步静态和非静态方法和功能属性。

默认:`console.log`

## 对数误差

这是一个实际记录最终格式化错误消息的函数。它用来记录这唯一的事件:

*   同步和异步静态和非静态方法和函数属性的错误。

默认:`console.error`

## 格式程序

它是一个有两种方法的对象:`start`和`end`。它将日志数据格式化为最终的字符串。

`start`格式化这些事件的消息:

*   班级建设前。
*   同步和异步之前的静态和非静态方法和功能属性。

`end`格式化这些事件的消息:

*   经过同步和异步静态和非静态方法和功能属性。
*   同步和异步静态和非静态方法和函数属性的错误。

默认:`new ClassLoggerFormatterService()`

## 包括

消息中应包含的内容的配置。

## 包含. args

它可以是布尔值，也可以是对象。

如果是布尔值，它设置是否包含参数列表(还记得那个`Args: [milk]`？)分为两部分，开始(构造前和方法调用前)和结束(方法调用后，错误方法调用)，消息。

如果是一个对象，它应该有两个布尔属性:`start`和`end`。`start`包括/排除开始消息的参数列表，`end`对结束消息也是如此。

默认:`true`

## 包含.构造

设置是否记录类构造的布尔标志。

默认:`true`

## 包含.结果

另一个布尔标志设置是否包括方法调用的返回值或由它抛出的错误。还记得`Res: purr`吗？如果您将此标志设置为`false`，将不会有`Res: purr`。

默认:`true`

## include.classInstance

再次强调，要么是布尔值，要么是对象。如果您启用它，您的类实例的字符串化表示将被添加到日志中。换句话说，如果您的类实例有一些属性，它们将被转换成 JSON 字符串并添加到日志消息中。

并非所有属性都将被添加。[类记录器](https://github.com/keenondrums/class-logger)遵循以下逻辑:

*   获取实例的自身(非原型)属性。
    *为什么？当你的原型动态变化时，这种情况很少见，因此记录它几乎没有任何意义。*
*   删除任何具有`function`类型的文件。
    *为什么？大多数时候* `*function*` *属性只是不可变的箭头函数用来代替常规的类方法来保存* `*this*` *上下文。用这些函数的字符串体来膨胀日志没有多大意义。*
*   丢弃任何不是普通对象的对象。
    *什么物体是素色的？* `*ClassLoggerFormatterService*` *如果一个对象的原型严格等于* `*Object.prototype*` *，则认为该对象是普通对象。
    为什么？通常我们包含其他类的实例作为属性(注入它们作为依赖)。如果我们包含这些依赖项的字符串版本，我们的日志会变得非常庞大。*
*   剩下的纤维。

默认:`false`

# 控制格式

那么，如果你喜欢整体的想法，但希望你的信息看起来不同呢？您可以通过自己的自定义格式化程序来完全控制格式化。

您可以从头开始编写自己的格式化程序。完全同意。然而我们不打算在这里讨论这个选项(如果你真的对此感兴趣，请阅读自述文件的[“格式化”部分](https://github.com/keenondrums/class-logger#formatting))。

最快也可能是最简单的方法是创建一个内置默认格式化程序的子类— `ClassLoggerFormatterService`。

`ClassLoggerFormatterService`有这些受保护的方法，充当最终消息的构建块:

*   `base` 返回带有方法名的类名。例子:`ServiceCats.eat`。
*   `operation` 根据方法的成功执行或错误执行返回`-> done`或`-> error`。
*   `args` 返回参数的字符串列表。例:`. Args: [milk]`。它使用[快速安全纤维](https://github.com/davidmarkclements/fast-safe-stringify)用于引擎盖下的物体。
*   `classInstance` 返回一个字符串化的类实例。例:`. Class instance: {"prop1":42,"prop2":{"test":42}}`。如果你选择包含类实例，但是它不可用(静态方法和类构造就是这样)，它返回`N/A`。
*   `result` 返回执行的字符串化结果(即使它是一个错误)。使用[快速安全字符串](https://github.com/davidmarkclements/fast-safe-stringify)来序列化对象。一个字符串化的错误将由以下属性组成:
    -创建错误的类(函数)的名称(`error.constructor.name`)。
    -错误代码(`error.code`)。
    -错误信息(`error.message`)。
    ——错误名称(`error.name`)。
    -堆栈跟踪(`error.stack`)。
*   `final` 返回`.`只是`.`

`start`信息包括:

*   `base`
*   `args`
*   `classInstance`
*   `final`

`end`信息包括:

*   `base`
*   `operation`
*   `args`
*   `classInstance`
*   `result`
*   `final`

您可以覆盖这些构造块方法中的任何一个。让我们看看如何添加时间戳。我不是说我们应该。pino 、 [winston](https://github.com/winstonjs/winston) 和许多其他的记录器能够自己添加时间戳。所以这个例子纯粹是教育性的。

[现场演示](https://stackblitz.com/edit/class-logger-demo-custom-formatter-add-timestamp)

# 结论

请不要忘记遵循[安装步骤](https://github.com/keenondrums/class-logger#installation)，并在决定使用该库之前熟悉[要求](https://github.com/keenondrums/class-logger#requirements)。

希望你已经找到了对你的项目有用的东西。请随时将您的反馈传达给我！我非常感谢任何批评和问题。