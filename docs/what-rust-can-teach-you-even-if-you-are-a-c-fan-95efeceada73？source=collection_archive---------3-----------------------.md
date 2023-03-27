# Rust 有什么可以教你的，即使你是 C++迷？

> 原文：<https://itnext.io/what-rust-can-teach-you-even-if-you-are-a-c-fan-95efeceada73?source=collection_archive---------3----------------------->

![](img/e25c2f211ecb0a6c9d299e9f480601ed.png)

我开始使用 Rust 作为一个实验，去了解*他们是如何在不损失执行性能的情况下开发内存安全语言的？*

作为一名没有任何特定语言偏好的软件开发人员，我接受了一个想法`If Rust compiler compiles code without any issue, it is unlikely that it will give Segmentation fault or there will memory leak`。那很酷，不是吗？

> 没有 segfaults，没有未初始化的内存，没有强制错误，没有数据竞争，没有空指针，没有头文件，没有 makefiles，没有 autoconf，没有 cmake，没有 gdb。如果 c/c++的所有问题都被魔杖一挥就解决了，那会怎么样？未来就在这里，https://news.ycombinator.com/item?id=10886253 人
> T5

我不打算制作 holly wars，相反，在这篇文章中，我将尝试解释我在 Rust 中最喜欢的东西，以及我在 Rust 中编码的前两周所学到的东西，顺便说一下，这提高了我在所有其他编程语言中的编码技能！

所以让我们开始吧！🎉

# 1.避免可变的静态声明

Rust 编译器不允许可变的静态/全局变量声明，事实上，它只允许声明非动态的标准类型(int、const string、float 等)，这显然是因为已知的大小。有时候，我们真的需要保留一些全局变量来检查应用程序的每个部分，但是 Rust compiler 说“好吧！想怎么匹配就怎么检查，但是不要改静态变量，不安全！”

静态声明通常不是线程安全的，而且因为 Rust 语言建立在拥有`move semantics`的思想之上，如果你拥有非标准类型的静态变量，你就不能进行匹配。我不确定发生这种情况是因为语言编译器是如何构建的，还是最初的设计，但起初我很惊讶我不能将我的自定义 struct mutable 对象声明为静态/全局变量，然后当我了解 Rust 的内存管理概念时，这开始对我有意义。

```
// WRONG!!
static mut SOME_VALUE: SomeType = SomeType{abc: 10};// Correct!
static SOME_VALUE: SomeType = SomeType{abc: 10};
```

**采取一种方式:**不要声明静态/全局变量，以便稍后在其他语言中更改它们，这可能会导致问题(我知道，我知道你可以处理，跳过这一步😏).

# 2.用树形原则设计你的代码

从 Rust 的内存管理来看，它抛出了很多非连接的结构，导致“为什么我不能在这里声明我的结构对象？”。这种情况发生在几百行代码之后，但是当我面对这个问题时，我开始重新思考应用程序代码分配应该如何正确地设计代码，并遵循 Rust 的编译器规则。

*假设您有一个应用程序，它将解析配置文件，并使用该信息启动 HTTP 服务器。*

例如在 Golang 中，我会做一些名为`config`的包，用一个全局变量来表示应用程序配置。然后我将简单地导入这个包并使用这个全局变量。但是在以 Rust 的方式实现它之后，我真的很喜欢没有全局变量的方法，相反，我需要做这样的东西:

```
struct Config {
    ...
}struct MainApp {
    config: Config,
    ...
}impl MainApp {
    fn new(config_file: &str) -> MainApp {
        // Parsing config file and allocating Config object
    }
}
```

哪个代码看起来更好，你有一个单点主结构，它将分配、创建和声明所有其他操作你的应用程序所需的东西。事实上，这更有可能保持您的代码的可伸缩性，因为您知道哪个结构去哪里！(但这只是我的看法)

# 3.使用你申报的所有东西

默认情况下，Rust 的编译器会给出关于未使用变量的警告，所以如果你想通过代码编译而没有任何“非绿色”输出，你必须删除未使用的变量/结构/特征，除非你真的需要它们，只是为了扩展代码库并在以后使用它们，有一个宏告诉 Rust 编译器“我真的需要这个，不要给出警告。”。我知道 Golang 和 C++也存在这种情况(通过添加一些编译器标志)，但一般来说，内置这种特性有助于保持代码整洁。我在许多编程语言中都缺少这种类型的特性。

在经历了 Rust 中的“未使用…”警告后，我开始配置 JavaScript 的 ESLinter，为未使用的变量给出一个错误，Python linter 等也是如此。我知道你们中的一些人可能在 Rust 之前就这样做了，但对我个人来说，这是来自 Rust 的情况。

# 4.让人们能够读懂错误信息

当我第一次开始运行 Rust 编译器时，我最喜欢的就是错误信息！它们非常有帮助，非常有用，尤其是如果你来自 C++，有时 GCC 编译器会给出无用的错误消息，使你的调试更加混乱。例如，此错误消息通过标记问题发生的确切位置，给出了问题的解决方案

```
fn main() {
    let s = String::from("hello");change(&s);
}fn change(some_string: &String) {
    some_string.push_str(", world");
}# cargo build
error[E0596]: cannot borrow immutable borrowed content `*some_string` as mutable
 --> error.rs:8:5
  |
7 | fn change(some_string: &String) {
  |                        ------- use `&mut String` here to make mutable
8 |     some_string.push_str(", world");
  |     ^^^^^^^^^^^ cannot borrow as mutable
```

通过使用 rust compiler，我开始在应用程序中犯这种错误，这改变了我们开始处理团队内部问题的方式。

# 5.尽可能编写更高层次的抽象

在许多语言中，更高层次的抽象意味着更低的性能，但 Rust 并非如此。事实上，你会访问他们的主页，他们首先谨慎的是“抽象的零成本”。他们为此做了大量的编译器优化，并使用非面向对象的原则来帮助设计零成本的抽象(以及宏声明)。

但当我在 Rust 之后用其他语言写代码时，默认情况下我会尝试更具表达力的抽象，因为这是拥有可伸缩和更可读代码库的关键。即使对于基本的数组操作，Rust 也给出了`map, filter, zip, etc...`抽象来防止多行循环和迭代器声明。当然，这是在抽象内部完成的，但是它们是以更优化的方式完成的，这将在代码中实现。

```
// Calculate the intermediate sum of this segment:
let result = data_segment
            // iterate over the characters of our segment..
            .chars()
            // .. convert text-characters to their number value..
            .map(|c| c.to_digit(10).expect("should be a digit"))
            // .. and sum the resulting iterator of numbers
            .sum();
```

在我们的 Python、Go 或 JS 代码中进行这种抽象，提供了更好的应用程序设计方式，新员工开始更快地学习代码，因为他们大多只是重用我们的抽象。

这 5 点都是基于我写了 6 个月的 Rust 应用程序的个人经验。

Rust 对我和我们的团队来说并不是主要的编程语言，但我们从它身上学到了很多，尤其是如何用其他编程语言编写更高效、更可扩展的代码库。

所以，如果你是一个 C++爱好者，认为花时间在 Rust 上是一件很糟糕的事情，我肯定你是对的，但是为什么不试一试，然后给出一个理由。我很想听听你的意见。

感谢阅读！💥，给一个👏并分享这篇文章或让我知道你对这个话题的看法。