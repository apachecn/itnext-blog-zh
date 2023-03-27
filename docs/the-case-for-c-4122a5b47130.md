# C++的案例

> 原文：<https://itnext.io/the-case-for-c-4122a5b47130?source=collection_archive---------1----------------------->

## 为什么你应该现在就开始学习

![](img/d7652401adbaf9d7c7103dbcbfd6bf2b.png)

(来源: [pixabay](https://pixabay.com/photos/architecture-skyscraper-urban-city-768432/) )

C++是你在 2019 年应该认真考虑学习的一门语言(或者当你碰巧看到这篇文章的时候)。快速的语言现代化、更好的工具、不断增长的包容性社区和繁荣的就业市场只是 C++应该成为你下一门学习语言的部分原因。

*哇，这家伙喝了太多的™️️苦艾酒，我出局了。*

是的，我明白了。至少在我多年来参与的社区中，C++的名声一直不好。

> 代码是可怕的。
> 
> 一堆难以维护的范例。
> 
> 非托管代码？不，谢谢，我喜欢我的脚——没有洞的。

**但是请听我说完**——如果我们谈论的是 C++11 之前的版本，那么这些评论有一定的道理。**现代 C++(版本≥ 11)** 是完全不同的野兽，应该单独考虑。

```
auto genYNames = people | filter(younger_than(39))
                        | filter(older_than(19))
                        | transform(names)
                        | sort()
                        | unique();for_each(genYNames, [](auto name) {
    cout << name << "\n";
});
```

如果你不认为这段代码是现代的 C++，那么我鼓励你继续读下去，并且我会列出一些你应该重新考虑你在 C++上的位置并且现在就开始学习的理由。

# TL；DR；

*——对于那些需要对不读书感觉良好的人来说。*

C++在语言、社区和工具方面正在重塑自己。创新和语言进化的速度令人震惊。现在是进入高性能和系统级编程并利用所有最新进展的大好时机。学习 C++，做一些很酷的事情，甚至可能因此找到一份新工作。

[现在](http://shop.oreilly.com/product/0636920033707.do)，[走](https://www.manning.com/books/functional-programming-in-c-plus-plus?query=functional%20c++)，[学](https://www.udemy.com/course/advanced-c-programming/) [东西](https://www.udemy.com/course/free-learn-c-tutorial-beginners/)。

# 快速的语言现代化

大家又爱又恨的 C++很可能是 C++11 之前的。随着版本 11 的出现，出现了许多迟到的特性，这些特性可能解决了人们指出的关于该语言的许多“问题”。11 中登陆的内容包括:

*   **智能指针**——当你用完它时，自动清理你的内存。内存泄漏的可能性更小。增加了三个变体:`unique_ptr`、`shared_ptr`和`weak_ptr`。最近，我们看到 Rust 等语言试图将智能指针嵌入到语言中。
*   `**nullptr**`**——比`NULL`更安全的替代方案，它消除了隐式转换**
*   **`**auto**` —自动推导变量类型**
*   ****基于范围的** `**for**` **循环**——基于集合的迭代的现代方法(想想`for x in xs: do ...`风格)**
*   **`**override**` **和** `**final**` **关键词** —填补那些缺失的 OOP 空白**
*   ****强类型** `**enums**` —远离 C 风格的全局枚举(从`LIB_OPTIONS_A`到`LibOptions::A`)**
*   **这对于读者来说是不言自明的，但也是 C++现代化的另一个例子**
*   **`**static_assert**` —允许编译时检查，作为增加程序安全性的一种方式**
*   ****移动语义** —允许在 C++中表示适当的“所有权模型”。我们再次看到 Rust 等语言试图将这一点融入语言本身。**
*   **`**noexcept**` —指示函数不会抛出异常的说明符(与 Java 的`throws`说明符相反)**
*   **`**std::thread**` — C++是一种面向许多不同平台的低级语言，因此为线程提供*标准*支持不仅是一项重大成就，也是语言现代化和减少新来者障碍的一大进步。**
*   **`**constexpr**` —允许编译器在编译时完全评估一个函数/表达式。还有一个很好的副作用，就是不允许未定义的行为。**

**当你把这些特性加起来，你就可以开始理解为什么 C++11 应该被当作一种独立的语言。它代表了语言、标准库和[首选的代码编写风格](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md)的一次重大飞跃。**

**好吧，我同意这是很大的进步。但是 C++已经有很长时间了，我们现在不应该有更大的进步吗？**

**这很公平，但是，我应该提到 C++11 是在 2011 年问世的。从那时起，C++委员会就设定了目标，每 3 年发布一个新的主要语言版本。对于一种成熟、稳定、拥有国际标准机构的语言来说，这是一件大事。如果你在 2019 年看到这篇文章，那意味着从那以后又有 2 个版本发布了——分别是 14 个和 17 个。该语言最值得注意的新增内容是:**

*   ****通用 Lambdas** —允许使用`auto`作为 Lambdas 参数，以便更灵活地(重新)使用。**
*   **`**[[deprecated]]**` **属性** —供 API 作者将函数标记为已弃用，这将导致编译器警告(可选地带有用户定义的帮助消息)**
*   ****标准用户定义的文字**——类似于 Scala 中的`s`字符串插值:`"hello"s`是一个`std::string`，而`60s`是一个`chrono::seconds`(许多其他文字可用)**
*   ****Constexpr 扩展** —使用 use `if constexpr (...)`对`if`块进行编译时评估的能力。这个特性极大地简化和改进了 C++中的元编程([阅读更多](https://hackernoon.com/a-tour-of-c-17-if-constexpr-3ea62f62ff65))。**
*   **结构化绑定声明(Structured Binding Declarations)——在其他语言中通常被称为析构赋值或模式匹配赋值，C++现在支持像`auto [a, b] = getTwoReturnValues();`这样的语句**
*   ****类模板参数演绎** —现在可以使用模板(*泛型，* sort-of)并允许编译器确定类型，比如`std::pair p(32.5, false)`。**
*   ****文件系统库**——和 C++11 中的`std::thread`一样，FS lib 代表了语言现代化和减少新手障碍的一个重大进步。**
*   ****标准库优点** —标准库经历了许多改进，包括增加了`std::optional`、`std::string_view`、`std::any`、并行容器算法、`std::variant`、`std::byte`等。**

**嗯，你的确指出了过去几年发生了很多变化。但是我听说如果我今天想写 C++，我真的应该去学 Rust。它更现代，而且有 Mozilla 的支持。**

**诚然，Rust 是一种值得关注的伟大语言。但是它还很年轻，收养还不是我个人会考虑投入时间(除了我的爱好/娱乐时间之外)的问题😃).然而，Rust 确实给了 C++社区一个很好的推动力来改进语言并采用越来越新的特性。LLVM 开发者甚至已经开始开发一个[实验性借用检查器](https://www.youtube.com/watch?v=VynWyOIb6Bk)(生命周期分析器)，这是 Rust 相对于 C++的主要卖点之一。随着 C++20 的到来，我相信我们会看到 Rust 和 C++之间更多的特性差距正在缩小。**

**还要记住，C++是由许多大公司开发的，包括 4/5 FAANG 公司、微软和许多其他公司。也就是说，只要企业有强烈的兴趣，它的发展就不太可能放缓。**

# **社区**

**如果您在过去的 20 年里在 C 或 C++社区呆过，您可能会对我提出这个问题感到惊讶。近年来这里有了一些惊人的改进。特别是会议大大改善了他们的社会行为准则，增加了演讲者和与会者的多样性。**

**![](img/f1671e769923751b2d14a78168a6b47e.png)**

**这种变化很大程度上是由 [#include < C++ >](https://www.includecpp.org/) 小组推动的，该小组还管理着一个欢迎和邀请 [Discord](https://discord.gg/ZPErMGW) (你应该完全加入)。随着 SG-20 的引入，我们甚至在委员会层面也见证了文化的改变，SG-20 是一个专注于 C++的“可学性”或“可教性”的研究小组。这个小组的目标是向初学者推荐教授 C++的最佳实践，并为可能增加语言复杂性的语言特性提供输入。**

**C++社区也是一个[惊人的每周播客](https://cppcast.com/)和[一个](https://www.embo.io/) [可笑的](https://meetingcpp.com/) [数字](https://cppcon.org/) [的](http://cppnow.org/) [年度](https://conference.accu.org/) [会议](https://cpponsea.uk/)的所在地，其中大多数使[他们的](https://www.youtube.com/channel/UCAczr0j6ZuiVaiGFZ4qxApw) [演讲](https://www.youtube.com/user/MeetingCPP) [自由](https://www.youtube.com/user/CppCon) [在](https://www.youtube.com/user/BoostCon) [YouTube](https://www.youtube.com/channel/UCg2JbpJ-PGdFUEZEiNr0GWg) 上可用。**

# **工具作业**

## **依赖性管理**

**一个主要的，也是合理的抱怨是，在 C++应用程序中管理你的依赖关系是一件非常令人头疼的事情。令人欣慰的是，近年来两个 C++包管理器越来越受欢迎；来自微软的 [vcpkg](https://github.com/microsoft/vcpkg) (最初只针对 Windows，现在运行在 Mac + Linux 上)和来自 JFrog 的[柯南](https://conan.io/)(arti factory 和 BinTray 的制造商)。这两个工具都是通过挂钩到 CMake 来工作的，CMake 是构建 C++应用程序几乎无处不在的选择。**

**就我个人而言，我是柯南的粉丝，每个新项目都是从把它插入我的 CMake 文件开始的。它使得管理我的依赖关系像任何其他流行的依赖关系管理系统(NPM、Bundler、Cargo 等)一样简单。**

## **代码分析器**

**对该语言的一个常见批评是，代码安全和资源管理可能是一个挑战，但重要的是要记住，空指针异常和内存泄漏(仅举几个例子)在“安全”和垃圾收集语言中仍然是可能的。真正的问题是，存在什么类型的工具来帮助您跟踪那些难以诊断的错误？**

**[Clang](https://clang.llvm.org/docs/)(LLVM 项目的一部分)提供了许多不同的被称为“杀毒程序”的工具，这些工具可以追踪各种各样的[内存错误](https://clang.llvm.org/docs/AddressSanitizer.html) / [泄漏](https://clang.llvm.org/docs/LeakSanitizer.html)、[检测数据竞争](https://clang.llvm.org/docs/ThreadSanitizer.html)、发现[未定义行为](https://clang.llvm.org/docs/UndefinedBehaviorSanitizer.html)、以及[许多其他种类的错误](https://clang.llvm.org/docs/analyzer/checkers.html)。一个类似的项目， [Valgrind](http://valgrind.org/) ，提供了类似的工具，但是使用了一种不同的方法，这种方法对于捕捉不同类别的错误可能是有用的。这些工具非常非常强大，是我在专业工作过的相当多的语言中使用过的最好的工具。**

**虽然编写 C++可能是一种“脱离训练”的体验，但有许多世界级的工具可以确保您不会崩溃。😉**

## **在线工具**

**我不知道还能把这些叫做什么，但是有两个工具是不可或缺的。首先是[编译器资源管理器](https://godbolt.org/)——一个查看编译器输出的在线工具。如果您想了解编译器在为您做什么，或者想对似乎没有按照您的预期运行的代码进行故障排除，这是非常有用的。**

**第二个是[快速工作台](http://quick-bench.com/)——一个在线基准测试工具。写下你的 C++并按下*运行*，看看它的表现如何。该工具还将向您显示程序集级别的热点。与编译器资源管理器一起，您可以努力找出每一点的性能。**

# **需要更多吗？**

**如果我到目前为止所说的还不足以说服你，让我快速提醒你，FAANG 和其他主要的技术公司大量使用 C++，更不用说几乎整个游戏社区了。C++已经在编程世界中根深蒂固，创新和社区的重生只会巩固这一地位。**

**随着 C++20 的出现，现在是深入了解和学习更多知识的好时机。驾驭创新的浪潮；扩展你的技能；并加入一个由热情的开发人员组成的不断发展、友好、安全的社区。**

# **约翰·默里的其他职位**

**[](https://medium.com/@john.m.murray786/your-softwares-carbon-footprint-98d6dc2ff6d6) [## 你的软件的碳足迹

### 你的技术对环境负责吗？

medium.com](https://medium.com/@john.m.murray786/your-softwares-carbon-footprint-98d6dc2ff6d6) [](https://medium.com/@john.m.murray786/systems-programming-d5917e41353f) [## 系统编程

### 用无术语的探索揭开神秘面纱。

medium.com](https://medium.com/@john.m.murray786/systems-programming-d5917e41353f)**