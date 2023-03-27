# 颤振中的 TDD 第二部分:如何编写单元测试？

> 原文：<https://itnext.io/tdd-in-flutter-part-2-how-to-write-a-unit-test-8493044792f?source=collection_archive---------0----------------------->

本文是《Flutter 第 1 部分:测试驱动开发简介 中第一部分 [***TDD 的延续。***](/tdd-in-flutter-part-1-introduction-to-test-driven-development-c130b9e82f36)

![](img/8d86239268090316ddb2997da442d57f.png)

# 编写您的第一个单元测试

让我们假设您已经创建了一个从 API 获取数据应用程序。如何测试您将正确地获取和解析您的数据到一个模型类中？

在这个例子中，我将使用 [JSON 占位符 API](https://jsonplaceholder.typicode.com/) 。你应该做的第一件事是获取一些你可以在测试中使用的响应，这是我的:

随机 _ 用户. json

下一步是创建一些`UserModel`类，但是请记住，我们正在尝试应用 TDD，所以您应该做的第一件事是在`test/`文件夹中创建一个测试文件，这将是定义我们的类的基础:

测试/用户测试. dart

注意，我把这个文件叫做`user_test.dart`，文件名的“_test”部分很重要。这是必要的，这样您的文件将被识别为测试文件。

我们在文件中所做的只是定义我们需要一个`UserModel`类，它将接受 JSON 中定义的所有四个参数，并将存储它们的值，以便我们可以访问它们。既然我们有一个失败的测试，我们可以用代码中所有需要的属性创建我们的`UserClass`。

用户模型. dart

我们现在可以尝试使用命令`flutter test`运行我们的测试，您应该会得到类似如下的输出:

```
00:00 +0: test 1
00:00 +1: All tests passed!
```

因为我们有一个通过测试，我们需要写另一个或改变我们目前的编码前再次。既然我们已经定义了模型，下一步就是解析一个样本 JSON 文件，这样我们就可以确认我们的`UserModel`可以用来解析 API 数据。要在您的测试中使用一个文件，您可以简单地在`test/`中创建一个新的文件夹，并在其中添加您所有的测试资源。

```
- test/
  - test_resources/
    - random_user.json
  - user_test.dart
```

你可以像这样从你的代码中获取它:

现在，我们已经具备了添加新测试所需的所有条件:

如你所见，我们已经定义了一个新的构造函数`UserModel.fromJson`，我们需要像这样编码:

你完了！您的类已经过全面测试，最后一步是模拟一个 HTTP 客户端。

# 模仿 HTTP 客户端

在这篇文章中，我将使用包 [http](https://pub.dev/packages/http) ，因为它最容易模拟和测试。您应该做的第一件事是定义将用于 API 调用的基类和方法。

我们想要做的是简单地使用包提供的`MockClient`来模拟假的 API 调用，而不是从 API 接收响应，我们将返回我们的样本文件`random_user.json`。

现在我们需要编写我们的`ApiProvider`类及其方法`getUser()`:

正如你所看到的，我们的`ApiProvider`将一个 HTTP 客户端作为参数，通过这样做，你可以传递一个`MockClient`或者一个真正的`Client`对象来执行你的请求。现在您应该能够测试您的 API 调用了。

# 结论

恭喜，这是第二部分的结束。一如既往，我希望我的解释是清楚的，如果你喜欢这篇文章并想支持这个系列，你可以鼓掌并在评论区留下你的印象。

我要感谢社区对第一篇文章的支持，这真的很激励人。第三部分将致力于使用`testWidgets`的小部件测试和使用`golden_toolkit`的快照比较。