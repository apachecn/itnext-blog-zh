# Rust 中的 Shell 脚本？

> 原文：<https://itnext.io/shell-scripting-in-rust-2bdb8c738c94?source=collection_archive---------0----------------------->

**TL；dr:** 是的，对于一种面向性能的系统级语言来说，Rust 中的脚本非常实用，在这种情况下与 Python 不相上下。与使用 C 或 C++这样的语言相比，Cargo build/package manager 提供了三大优势。

![](img/558f1ab53825aebd101178de406911c6.png)

对于一个花园来说，也许有些过分了…但是看起来很有趣！[奥米德·罗山](https://unsplash.com/@oommiidd?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上的照片

# **照片排序**

Gphoto-sort 最初是一个小型 Bash 脚本，当时谷歌终止了它在谷歌照片和谷歌驱动之间的链接。我使用 Google Drive sync 工具轻松备份了我们所有的照片。后来它被谷歌外卖取代，可以让我下载我所有的照片。每一个。单身。时间。在不同的目录结构中。

大多数照片在文件名中都有时间戳，所以我想如果文件不存在的话，我可以很容易地用脚本将文件从外卖存档移动到现有的树中。这样，我可以保留我的目录结构，而不必再次进行完整的云备份。

除了我真的想包括我妻子的外卖存档和重复数据删除，这意味着比较文件校验和。诸如此类……随着大多数软件项目范围的扩大，我的 Bash-foo 突然不能胜任任务了。通常我会将这样的脚本转移到 Python，但我也在学习 Rust 语言，我很好奇，这在 Rust 中有多实用，Rust 是一种像 C/C++一样的系统级语言？

我职业生涯的大部分时间都在编写 C++代码，我真的很喜欢编写面向性能、接近金属的代码。但是在 C++中，我永远不会这么做。即使我已经在家安装并构建了 Boost 和 Makefile 结构设置，我还是会使用 Python，因为有更高级别的库可以构建。

# **真的，Rust 里的脚本？**

事实证明，效果很好。非常好。我把这归因于 Rust 在这方面的三个优势:货物，货物，货物。

## **Cargo 让代码重用变得切实可行**

如果你不熟悉的话， [Cargo](https://doc.rust-lang.org/cargo/) 是 Rust 的包管理器和构建工具。它处理下载、更新和构建依赖项，然后编译和链接最终的可执行文件。如果你来自 Javascript 世界，使用 NPM，或者使用 Ruby 和 Gem，这已经过时了。与 C++相比，甚至在某种程度上与 Python 相比，这就相当于给了一台洗衣机，而不必把脏衣服拖到最近的小溪里去扔石头。

例如，我将下面一行添加到我的 Cargo.toml 中:

```
rust-crypto = "0.2"
```

突然，我有了一个 MD5 哈希库(Crate)来计算文件的校验和。就是这样。不需要弄清楚每个包的构建系统的特性和它们的依赖关系。

## **货物造就了丰富的板条箱生态系统**

即使是很小的模块也很容易重用，这意味着一个丰富的支持箱生态系统已经迅速发展起来。像 [Walkdir](https://github.com/BurntSushi/walkdir) 和 [Clap](https://github.com/clap-rs/clap) (命令行解析)这样优秀的板条箱意味着组装一个文件处理实用程序的构件只需要几分钟。同样，这对于许多高级语言来说并不新鲜，但在系统级并不常见。

## **货物鼓励小板条箱，因此更小的模块接口**

Rust Crate 生态系统中模块的精细粒度让我吃惊。比如 Walkdir 本身就不是很大，有三个依赖项。甚至还有一个(有用！)单功能机箱: [GetHostName](https://crates.io/crates/gethostname) 。为什么？

我认为，部分原因是 Rust 中的模块支持。这使得用定义良好的接口来定义甚至很小的模块变得很容易。然而，这与创建一个 C++类或者一个头文件加上内部细节头文件并没有什么不同。

不同的是，将一个 C++类拆分到一个单独的目录中，有一个单独的构建目标，这(根据我的经验)要麻烦得多:每个消费者都必须链接。o 文件、静态库或共享库。但是，每个消费者还需要知道要链接的任何依赖项，并从那时起保持这些依赖项最新。这就是 Cargo 改变游戏规则的地方，因为突然间我的构建只关注我的直接依赖项。这允许将内部模块分解成可重用的组件，这些组件可以独立于主应用程序进行修改。

# 例子

举例来说，下面是递归遍历当前目录下的目录树并打印找到的所有文件和目录所需的代码:

```
for ent in WalkDir::new(".") {
    if let Ok(ent) = ent {
        let path = ent.path();
        if path.is_dir() {
            println!("{:?} is a directory", path);
        } else {
            println!("{:?} is a file", path);
        }
    }
}
```

 [## Rust Playground —在此运行示例](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=8d69c19a9ae2ef00fecd4e27216d8afb) 

`if let Ok(ent) = ent`表达式再次将每个“Ok”(不是错误)的条目输入到变量`ent`中。然后代码检查路径是文件还是目录并打印出来。

一个类似的例子可以用 Rust 的迭代器支持写成管道，如下所示:

```
let count = WalkDir::new(".")
    .into_iter()
    .filter_map(|e| e.ok())
    .filter(|e|e.path().is_file())
    .count();
 println!("Found {} files", count);
```

这个示例使用 filter_map 来删除任何错误条目，然后过滤以只保留文件条目，最后产生这些文件的计数。

真正有趣的一点是，这个基于迭代器的版本可以很好地处理名为 [Rayon](https://crates.io/crates/rayon) 的数据并行机箱。Rayon 可以使用这样的管道，使用线程池在多个 CPU 内核之间传播处理，并几乎神奇地聚合最终结果。为了并行化我们的简单示例，我们只添加了一行代码，`.par_bridge()`:

```
let count = WalkDir::new(".")
    .into_iter()
    .filter_map(|e| e.ok())
    .**par_bridge()**
    .filter(|e|e.path().is_file())
    .count();
 println!("Found {} files", count);
```

 [## Rust Playground —在此运行示例](https://play.rust-lang.org/?version=stable&mode=release&edition=2018&gist=dac9befb1f916629ee6408be0f88e00f) 

现在，在数千个文件上调用`is_file`的“繁重处理”可以跨多个 CPU 核心共享。当然，这在这里是没有意义的，但是如果过滤步骤在计算上更加昂贵，这可能会产生很大的不同。

# **总结**

简而言之，我对 Rust 作为一种系统级语言的“动态范围”印象深刻，这也让我能够在不到 300 行的代码中编写出相当于美化了的 shell 脚本——注释、单元测试等等。这确实反映了那里板条箱的质量和货物的想法。

有必要用低级语言写这个吗？一点也不！然而，令人惊讶的是，这种文件系统级别的操作经常出现在大型应用程序中。即使在应用程序堆栈的最底层也能高效地完成这项工作，这是一个很好的生产力提升。