# 将冻结值与构建值进行比较

> 原文：<https://itnext.io/comparing-freezed-to-built-value-3ff978c8647?source=collection_archive---------2----------------------->

[更新，2020 年 3 月 20 日:我在 freezed 发布后不久写了这篇文章。这篇文章的许多观点不再有效]

Remi Rousselet 最近发布了 [freezed](https://pub.dev/packages/freezed) ，这是一个用于生成不可变数据类和联合类型的库。

很自然，我很好奇。我们使用 built _ value 来创建不可变的模型对象，使用 JSON 序列化和反序列化来与 API 通信，我们对此非常满意。

然而，我们对一些具有内置价值的东西感到恼火:

*   这需要大量的样板文件
*   它是为 Dart 1 编写的，这一点仍然值得注意。例如，它进行手动类型检查，而不是依赖编译器、泛型和类型推断。built _ value 引入了一些新的方法和构造函数来解决这个问题，但在某些情况下它仍然存在。
*   代码生成非常慢。对于一个大约 24k 行代码的项目(不包括生成的代码、空白和注释)，代码生成大约需要 1 分钟。也就是创建一个 build.yaml 文件，这样 built _ value generator 只在模型对象上运行。结果应该被缓存，但是根据我的经验，即使是很小的改变也会触发一次完整的重建。

所以我决定看看冷冻是否更适合我们。以下是我的发现:

## 简单的不可变模型对象

这是冻结价值和构建价值的主要来源。冰封真的在这里大放异彩。

使用冻结的简单不可变对象

内置值:

使用 built _ value 的简单不可变对象

冷冻版要短得多，没有重复。对于 built _ value，如果希望拥有一个接受对象所有变量的工厂，需要两次输入所有变量。

您还可以使用代码模板来生成新模型:

从模板生成新的数据类

Android Studio +冻结的实时模板

**JSON 序列化/反序列化**

freezed 根本不处理这个问题，但是它将所有工作委托给 json_serializable。对于简单对象，我们所要做的就是添加另一个名为 fromJson 的工厂:

使用冻结的 JSON(反)序列化

就是这样，a .托吉森()和。fromJson()方法将被创建。

对于 built _ value，需要做更多的工作:

使用 built _ value 的 JSON(反)序列化

您必须创建一个序列化程序，并且需要创建自己使用该序列化程序的 toJson()和 from()方法。此外，您必须创建一个全局 serializer 对象，其中您需要注册所有想要序列化的根类型(此处省略)

## 枚举

我不喜欢“字符串型”的应用程序或神奇的常数，我喜欢对只能包含少量可能值的字段使用枚举。

built _ value 和 freezed+json_serializable 是如何处理这些的？

冻结:

使用 freezed(带有 json_serializable)的枚举(反)序列化

内置值:

使用 built _ value 的枚举(反)序列化

freezed with json_serializable 支持 dart 枚举，这需要更少的样板文件。
然而，built _ value 提供了一个 json_serializable 没有的好特性:

您可以指定一个后备值，以防服务器添加应用程序尚不支持的新字段。在 json_serializable 中，如何在拥有枚举的对象上处理这个问题，使用类似于:

使用 freezed 进行回退的枚举反序列化

但是如果在多个对象中有相同的枚举，就需要在每个对象中指定回退。

## 联合

冻结:

Unions 是 freezed 的一流特性，使用起来很容易:

冻结的工会

就是这样！您可以在不检查具体类型的情况下访问 sharedValue，freezed 为您生成了处理所有可能类型的好方法:

built _ value 并不真正支持联合类型。
你能做的最接近的事情是:

但是您仍然必须使用 **is** 操作符来获取联合的具体类型。

## 更新

当然，即使你使用不可变的对象，我们也想更新它们。

freezed 为每个对象提供了一个简单的 copyWith 方法:

用冻结更新字段

built _ value 使用一个可变的 builder-object 来创建一个新模型:

使用 built _ value 更新字段

样板文件稍微多一点，但是如果您想要更改嵌套在几个对象中的值，built _ value 具有更好的语法:

使用 built _ value 更新嵌套对象

在 freezed 中，语法应该是:

如果更新对象，默认情况下，built _ value 会为所有字段创建可变的构建器，这使得语法更加简洁。对于大的嵌套对象来说，这会带来性能损失。不过，可以禁用这个特性(代价是更新的语法更加冗长)。

## **处理收款**

当使用不可变数据类时，您通常不希望使用标准库中的可变集合，否则，您会失去许多好处，并且不再保证对象的不变性。如果您确实使用标准库的集合，您有以下选择:

*   让它们变异，而不是依赖于不变性
*   不要改变它们，而是确保每次都复制它们，也许用不可修改的视图包装它们。但是这需要纪律，并且容易出错。

原则上冷冻并不限制你在这里的选择。它既支持来自标准库的集合(并使用 DeepCollectionEquality 对它们进行比较)，也支持像 built _ collection 或 kt.dart 这样的自定义不可变集合。

但是，json_serializable 在这里更受限制。开箱即用，json_serializable 只支持标准列表。您可以指定转换器，但是您必须为您想要序列化的每个类型编写一个新的转换器(例如`ImmutableList<MyModel>`和`ImmutableList<MyOtherModel>`)。此外，您必须用这个转换器注释每个列表字段:

在带有需要可序列化的定制集合的模型上使用 freezed 会给代码添加一大块样板文件，不幸的是，这会否定大多数(如果不是全部)从 built _ value 保存的样板文件。

通过为 json_serializable 创建一个定制的`TypeHelper`也许可以避免这种样板文件——但是，没有关于如何做的文档，并且似乎涉及到编写一个运行代码生成的定制构建器——从长远来看，这可能是值得做的，但是似乎不那么容易设置。

编辑:我为 json_serializable 编写了一个支持不可变集合的实验性构建器。[看看吧！](https://pub.dev/packages/json_serializable_immutable_collections#-readme-tab-)

## 小绳索

i̶t̶'̶s̶̶u̶s̶e̶f̶u̶l̶̶t̶o̶̶s̶e̶t̶̶d̶e̶f̶a̶u̶l̶t̶̶v̶a̶l̶u̶e̶s̶̶o̶n̶̶o̶b̶j̶e̶c̶t̶s̶.̶̶f̶r̶e̶e̶z̶e̶d̶̶d̶o̶e̶s̶̶n̶o̶t̶̶s̶u̶p̶p̶o̶r̶t̶̶t̶h̶i̶s̶̶a̶t̶̶t̶h̶e̶̶m̶o̶m̶e̶n̶t̶,̶̶b̶u̶i̶l̶t̶_̶v̶a̶l̶u̶e̶̶d̶o̶e̶s̶.̶

b̶u̶i̶l̶t̶_̶v̶a̶l̶u̶e̶̶s̶u̶p̶p̶o̶r̶t̶s̶̶c̶o̶m̶p̶u̶t̶e̶d̶̶p̶r̶o̶p̶e̶r̶t̶i̶e̶s̶̶t̶h̶a̶t̶̶c̶a̶n̶̶b̶e̶̶m̶e̶m̶o̶i̶z̶e̶d̶；̶̶f̶r̶e̶e̶z̶e̶d̶̶d̶o̶e̶s̶̶n̶o̶t̶̶s̶u̶p̶p̶o̶r̶t̶̶c̶o̶m̶p̶u̶t̶e̶d̶̶p̶r̶o̶p̶e̶r̶t̶i̶e̶s̶̶d̶i̶r̶e̶c̶t̶l̶y̶,̶̶b̶u̶t̶̶y̶o̶u̶̶c̶a̶n̶̶w̶r̶i̶t̶e̶̶a̶n̶̶e̶x̶t̶e̶n̶s̶i̶o̶n̶̶o̶n̶̶y̶o̶u̶r̶̶m̶o̶d̶e̶l̶̶o̶b̶j̶e̶c̶t̶.̶

编辑:Remi 给 freezed 添加新功能几乎比我写这篇文章还要快！freezed 现在也支持计算的、缓存的属性和默认值！

built _ value 允许您定制在生成`==`方法时应该比较哪些字段，freezed 当前总是比较所有实例字段。

## 代码生成性能

在将我们的项目转换为 freezed 后，代码生成在我的开发机器上需要大约 40 秒，这比之前的 60 秒有了很好的改进。此外，似乎缓存与 freezed 一起工作得更好，经过一些更改后，build_runner 通常只需要几秒钟。

# 结论

freezed 是一个令人惊叹的库，它使用巧妙的技巧来提供几乎没有样板文件的数据类和联合。
然而，仍然有一些功能缺失，特别是在使用 json_serializable 时对不可变集合的支持的缺失对我们来说是一个很大的缺点(虽然技术上不是 freezed 的错)。

freezed 几天前才发布，我相信它会很快赶上来。对于新项目，我肯定会选择 freezed 而不是 built _ value 在我看来，freezed 的长期前景更好。

我错过了什么重要的事情或者有什么不正确的吗？请让我知道！