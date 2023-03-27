# Flutter: Dart 不可变对象 JSON 序列化和反序列化

> 原文：<https://itnext.io/flutter-dart-immutable-objects-json-serialization-and-deserialization-dff917b1af2?source=collection_archive---------4----------------------->

![](img/b8eab3e217c92faa0ff894df3afe90ea.png)

在[之前的一篇文章](https://medium.com/@muccy/flutter-dart-immutable-objects-and-values-5e321c4c654e)中，我讨论了关于`freezed`和`kt_dart`的内容，这两个包为 Dart 和 Flutter 带来了不可变的值类型。我忽略了`freezed`揭露的一个免费功能:与`[json_serialializable](https://github.com/dart-lang/json_serializable)`的集成。

## 装置

首先，将依赖项添加到`pubspec.yaml`文件中:

然后，像这样编辑您的不可变对象声明:

*   您需要声明另一个带有`.g.dart`后缀的`part`文件。
*   您需要声明一个名为`fromJson`的新工厂构造函数，并将其绑定到由`freezed`生成器提供的合成反序列化器。

就是这样！就是管用！

如果您提供了错误的类型或者遗漏了一些必填字段，那么``fromJson``将会抛出一个异常。

您还可以获得一个返回`Map<String, dynamic>`的免费`toJson`方法。

您可以通过使用`json_serializable` [注释](https://github.com/dart-lang/json_serializable/tree/master/json_serializable#annotation-values)来影响 JSON 模型。比如说。您可以更改字段名称:

## 不可变集合

虽然`json_serializable`能够处理普通的`List`或`Dictionary`对象，但是我们希望支持`kt_dart`包提供的不可变类型。您有两种方法来扩展此功能:前者是编写一组自定义转换器:

我很确定你已经发现了问题:因为你必须调用`cast()`你必须为每个封闭类型写一个转换的。目前还不可能写出一个通用版本。这不是一个大问题，因为它们是一行程序，但肯定有点乏味。

后一种方式是使用名为`[json_serializable_immutable_collections](https://github.com/knaeckeKami/json_serializable_immutable_collections)`的`json_serializable`扩展。你把它安装在`pubspec.yaml`文件的`dev_dependencies`里面:

然后创建一个`build.yaml`文件来指示源代码生成器使用这个扩展:

然后您可以重新启动`build_runner`和`KtList`，`KtSet`和`KtMap`将在没有特殊转换器的情况下得到支持。

## VS 代码集成

您可以通过*浏览器*选项卡中的`json_serializable`隐藏生成的文件。为此，打开*设置*并搜索*文件:排除*首选项。然后，你可以将这个模式添加到列表中:`**/*.g.dart`。