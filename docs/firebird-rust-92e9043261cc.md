# 如何用 Rust 语言使用火鸟数据库

> 原文：<https://itnext.io/firebird-rust-92e9043261cc?source=collection_archive---------0----------------------->

![](img/736077a7b459e6971d90ae8afa3b709a.png)

# 介绍

首先，我想向你展示一下 Rust 编程语言。它是一种开源语言，提供了一个没有运行时或垃圾收集器的平台，速度非常快，内存管理非常棒。同样非常重要的是，它有一个[优秀的文档](https://doc.rust-lang.org/book/)。

对于数据存储，我们有[火鸟数据库](https://firebirdsql.org)。它有许多特点，是稳定和非常快。

[我们制作了](https://github.com/fernandobatels/rsfbclient/graphs/contributors)rsfbclient 机箱，用于提供访问火鸟的安全 API。

[*葡萄牙语*](https://medium.com/@luisfbatels/como-utilizar-o-banco-de-dados-firebird-com-a-linguagem-rust-812630dc0058) *。*

# 设置环境

你一定已经安装了 [rust toolkit](https://rustup.rs/) 。而且，如果你想使用原生 firebird 连接器，你还需要安装[Firebird](https://firebirdsql.org/en/firebird-3-0-7/)。

然后，你只需要使用 [cargo](https://doc.rust-lang.org/cargo/) 创建一个新的 rust 项目:

```
cargo new fbtest
```

测试项目:

```
cd fbtest
cargo run
```

# 选择客户端变体

rsfbclient 提供了三种访问数据库的实现:动态链接、动态加载和纯 rust。[功能](https://doc.rust-lang.org/cargo/reference/features.html) *联动*、*动载荷*和*纯锈*使能哪一个。

默认情况下，板条箱使用第一个选项。因此，我们只需要将其添加到 ***Cargo.toml*** 的依赖项部分即可:

```
[dependencies]
rsfbclient = “0.16.0”
```

测试项目:

```
cargo run
```

# 准备项目

在 ***main.rs*** 上，我们需要导入 crate 并修改 main 函数来返回它产生的错误:

测试项目:

```
cargo run
```

# 数据库连接

在选择了客户机变体之后，我们需要选择连接类型。板条箱有两个选项:连接生成器或字符串:

## 使用连接生成器:

## 使用字符串:

# 从数据库获取数据

现在，让我们从数据库中获取一些行。在我们的测试中，我们将使用 ***mon$attachments*** 表。rsfbclient 提供了一个 API，我们只需要在其中声明预期的类型，它将负责执行强制转换和其他操作。

随着连接的打开，我们只需要调用[查询](https://docs.rs/rsfbclient/0.16.0/rsfbclient/prelude/trait.Queryable.html#method.query)方法。此方法可用于以下格式:

## 元组:

## [行](https://docs.rs/rsfbclient/0.16.0/rsfbclient/struct.Row.html)结构:

在这种格式下，您可以动态地访问列。

# 插入数据

要在数据库上插入数据，或者在没有光标的情况下执行任何其他操作符，我们需要使用 [execute](https://docs.rs/rsfbclient/0.16.0/rsfbclient/trait.Execute.html) 方法。例如:

与 ***查询*** 方法一样，该方法也支持以下格式的参数:

## 元组:

## 结构:

所有实现 [IntoParams](https://docs.rs/rsfbclient/0.16.0/rsfbclient/trait.IntoParams.html) 特征的结构都可以用作参数。

# 关闭连接

由于 rust 的[作用域规则](https://doc.rust-lang.org/rust-by-example/scope.html)，一旦连接变量超出作用域，连接就会被关闭。

不过，您可以使用[关闭](https://docs.rs/rsfbclient/0.16.0/rsfbclient/struct.Connection.html#method.close)方法手动关闭连接。

## 考虑

本文仅仅展示了语言和数据库之间的少量交互。因此，一些特性和资源被遗漏了，但是你可以在[机箱文档](https://docs.rs/rsfbclient/0.16.0)中看到它们。此外，我们已经实现了[许多示例](https://github.com/fernandobatels/rsfbclient/tree/master/examples)。

为了跟踪 rsfbclient 的开发，您可以访问我们的资源库:[https://github.com/fernandobatels/rsfbclient](https://github.com/fernandobatels/rsfbclient)

感谢您的关注，下次再见！

[![](img/d9b1c672bb1cb70e8343c6c35d2a6084.png)](https://ko-fi.com/L3L843YUI)