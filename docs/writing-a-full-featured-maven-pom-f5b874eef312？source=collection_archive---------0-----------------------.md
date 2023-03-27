# 编写全功能的 Maven Pom

> 原文：<https://itnext.io/writing-a-full-featured-maven-pom-f5b874eef312?source=collection_archive---------0----------------------->

![](img/81275a49cdbc000d74f3a842f4c1c093.png)

由 CCO 旗下 [Pexels](https://www.pexels.com/photo/2-hands-holding-1-jigsaw-puzzle-piece-each-164531/) 的 [Pixabay](https://www.pexels.com/@pixabay)

Maven 是一个具有结构化构建管道的构建工具。它合金广泛的
定制为您自己的需求。你可以在 pom 中使用很少的
来构建。但是一个好的构建，遵循你可能应该
在 CI/CD 管道中做的所有事情，需要更多的时间。

我们将讨论所有不同的部分和我们推荐的配置。如果您想直接跳过完整的代码和示例，可以在我的 GitRepo:

[](https://github.com/efenglu/maven) [## 埃丰卢/马文

### 包含带有许多插件的 pom 的示例项目

github.com](https://github.com/efenglu/maven) 

## 基本信息

每个项目都应该描述自己。对于 maven 来说，这意味着至少要定义:

但是你真的应该包括:

*   **名称**:项目的人类可读名称
*   描述:更多关于你的项目的人性化的东西
*   **URL** :获取项目信息的主 URL，通常与 scm url 相同，但不总是如此
*   **问题管理**:用户有问题可以去哪里
*   **许可证**:什么许可证控制你的代码的使用
*   **SCM** :代码存放在哪里

示例:

# 编码

Maven 需要知道它应该如何处理你的文本。这是通过 enoding 项目完成的。

几乎总是这是**UTF 8**:

# 编译程序

现在我们已经有了基础知识，我们需要开始编码，我们所说的编码当然是指编译。

## Java 语言(一种计算机语言，尤用于创建网站)

默认情况下，javac 编译器基本上已经设置好了。但是，我们需要做一些家务。

*   选择我们要针对的源代码和编译类的 java 版本
*   包括调试标志
*   任何其他 javac 标志或自定义参数

对于我们的大多数项目，我们遵循一个非常简单的模式:

进一步阅读

*   [Apache Maven 编译器插件](https://maven.apache.org/plugins/maven-compiler-plugin/)

## 绝妙的

对于我们的大多数项目，我们使用的是 [Spock 测试框架。这意味着我们需要编译 groovy 代码来运行测试。](http://spockframework.org)

我们已经尝试了各种各样的 groovy 编译器。各有利弊。

目前我们选择了 GMavenPlus。

进一步阅读

*   [GMavenPlus](https://github.com/groovy/GMavenPlus)

## 龙目岛

Lombok 是降低 Java 代码复杂性的一个好方法。我们在创建 POJO 时广泛使用它

在很大程度上，简单地包括 lombok 作为一个**提供**依赖*应该*足够了。

延伸阅读:

*   [龙目岛](https://projectlombok.org)
*   [龙目岛美文](https://projectlombok.org/setup/maven)

# 代码生成

这是一个可选的部分。有许多工具*通过在构建时为你生成代码来帮助你。这方面有几个不同的例子。*

## 原蟾蜍

协议缓冲区是一种快速发展的二进制交换格式，最初由 Google 编写。协议缓冲器在*中定义。原型*文件。这些然后*编译*成各种语言。

下面是我们如何着手*编译*我们的原型文件:

**os-maven-plugin** 将检测我们的构建环境平台。这然后用于确定下载哪个平台特定的*协议*可执行文件。然后 **protobuf-maven-plugin** 使用该可执行文件编译实际的*。将原型*文件转换成 java 代码。

延伸阅读:

*   [协议缓冲区](https://developers.google.com/protocol-buffers/)
*   [Maven Protobuf 插件](https://www.xolstice.org/protobuf-maven-plugin/)

# 单元测试

好的单元测试对于任何成功项目的未来和现在都是至关重要的。

幸运的是，设置单元测试相当容易:

您可能已经注意到了 includes 元素。我们为我们的测试执行一个命名标准。对于 JUnit 测试，它们必须以“**测试**”、“**测试**”结尾，或者以“**规格**”、“**规格**”或 Spock 规格结尾。

延伸阅读:

*   [Maven Surefire 插件](https://maven.apache.org/surefire/maven-surefire-plugin/index.html)

# 代码覆盖率

确保代码得到实际测试是至关重要的。确定代码是否经过测试的一个很好的度量是代码覆盖率。这不是一个完美的衡量标准，你可能永远不会要求 100%的覆盖率。但是这是确保
至少在某种程度上符合测试的好方法。但话说回来，我们都遵循 TDD，这应该不是问题，对不对？

谈到代码覆盖率，有两个大玩家，Jacoco 和 Cobertura。我们用 Jacoco。

我们强制执行 70%的代码覆盖率。我们还从覆盖率统计中过滤所有的测试类。

延伸阅读:

*   [Jacoco Maven 插件](https://www.eclemma.org/jacoco/trunk/doc/maven.html)

# 集成测试

有些测试比其他测试耗时更长。有些测试更复杂，需要更多的设置和资源。这些是集成测试很好的例子。大多数集成测试都在构建过程的后期运行。有时候，集成测试根本不会在开发人员工作站上运行。

我们遵循将集成测试放入一个 *src/it/java* 路径结构的模式。

这将集成测试与常规测试区分开来。然而，由于 maven】并没有真正认识到集成测试的概念，因此也有必要对测试进行不同的命名，因为所有的测试
都被编译到一个测试类路径中。

对于我们的项目，所有集成测试必须在* **IT 中结束。*** 。

# 静态代码分析

静态代码分析是防止错误发生的另一种方法。您可以扫描您的代码库，查找常见的编程错误，并尽早发现它们。您还可以确保代码质量和编码标准的水平，以确保代码易于阅读。

有许多不同的 SCA 工具:

*   [检查样式](http://(https://maven.apache.org/plugins/maven-checkstyle-plugin/)
*   [PMD](https://maven.apache.org/plugins/maven-pmd-plugin/)
*   [CPD](https://maven.apache.org/plugins/maven-pmd-plugin/cpd-mojo.html)
*   [spot bugs](https://spotbugs.github.io)(Findbugs 的继任者)
*   [Codenarc](https://gleclaire.github.io/codenarc-maven-plugin/)
*   等等

## 检查样式

Checkstyle 强制代码格式化，并检查一些非常基本的不良编程实践。

## 值得注意的事情

我们在**插件管理**部分配置 checkstyle，以确保如果用户从命令行调用 checkstyle，他们将获得正确的配置。

还建议您定义自己的 checkstyle 配置。您应该将它作为自己的工件部署，并在 checkstyle 插件中使用它。您可以看到我在添加到 checkstyle 插件的依赖项中定义了自己的版本。这个 jar 可用于类路径上的 checkstyle，并且可以使用 **configLocation** 元素从该 jar 加载资源。

# Maven Enforcer

就像代码 SCA 一样，Maven Enforcer 是对 maven poms 的一种静态代码分析。

我们使用 enforcer 来确保:

*   **要求版本** : 3.5.0
*   **要求免除** : 1.8.0
*   **required No repositories**:我们的 pom 中没有其他存储库元素
*   **requireleasedeps**:我们没有在发布工件中引用快照工件
*   requireUpperBoundDeps :我们的依赖关系中没有版本冲突

进一步阅读

*   [Maven Enforcer 插件](http://(https://maven.apache.org/enforcer/maven-enforcer-plugin/)

# Maven 依赖性分析器

随着依赖项数量的增长，对我们的依赖项进行静态分析也变得很有必要。

*   我们还在使用它们吗？
*   我们的代码从可传递依赖项中导入类吗？

这通常是个坏主意。如果一个依赖库提供了一个我们过渡使用的依赖，如果这个库突然移除了这个依赖会发生什么？一般来说，如果你有一个依赖，你也应该把它叫出来。

# 包装

Maven 已经向创建的清单文件中添加了一些内容。但是我们可能应该增加一些其他的。

## 添加 Git 信息

我们喜欢将代码编译时使用的提交、标记/分支信息放入 jar 本身。为此，我们需要获取 git 信息，以便在 pom 中使用它。[**git-commit-id-plugin**](https://github.com/git-commit-id/maven-git-commit-id-plugin)将这些信息作为属性添加到 pom 中。

**最有男子气概的参赛作品**

现在我们已经获得了所需的所有信息，让我们将它作为清单条目添加到生产的 jar 中。

# 源罐

当您将代码发布到工件存储库时，同时发布一个源代码 jar 是很重要的。这将允许消费者很容易地查看您的代码。

延伸阅读:

*   [Maven 源码插件](https://maven.apache.org/plugins/maven-source-plugin/)

# 胖罐子？

有时有必要将所有的依赖项都汇总到一个 jar 文件中。这在运行脚本时很有用。胖罐子也有缺点。你可能会遇到类路径冲突、文件
重复和其他文件冲突的问题，

没有一个构建 fat jars 的工具是完美的，但是我在使用[着色器插件](https://maven.apache.org/plugins/maven-shade-plugin/)时运气最好。它做得很好的一件事是加入 spring，并提供 jars 服务。这些 jar 通常有驻留在 jar 的 MANIFEST 文件夹中的文件。但是在构建 fat jar 时，它们经常会发生冲突，导致您丢失一些必要的配置信息。着色器插件将通过尽可能地合并和组合这些文件来避免这种情况。

# 共享配置

好的，那就很多了。您的 pom 现在可能已经超过 1000 行了。不仅如此，它非常复杂，难以复制。当然，它做了我们想要的一切，但是如果我们不能轻松地在许多项目中维护配置，那有什么好处呢？

实际上有两种方法可以做到这一点。没有一个是完美的，但他们在一起可以非常接近。

## 父 POM

Maven 支持 pom 配置继承的概念。通过在 pom 中指定一个 ***父*** 元素，可以很容易地做到这一点。

子节点现在将从父节点接收所有配置。您可以覆盖孩子中的设置，并向您的 hearts 内容添加附加内容。

父母是设置 ***依赖管理*** 和 ***插件管理*** 部分的好地方。这确保了你所有的子模块将在他们的构建中使用相同版本的依赖和相同版本的插件。

一些需要注意的事情。

很难排除某些东西。也就是说，如果父母定义了一个插件，孩子就不容易关掉它。因此，在将所有配置添加到单个 monster 父节点时要小心。

这就是第二种方法有用的地方

## 瓷砖

Maven 本身并不支持配置组合的概念。但是一个名为 [Maven Tiles](https://github.com/repaint-io/maven-tiles) 的第三方工具带来了这种能力。

Maven tiles 允许您在项目配置中编写:

**注意:**您不能将父级中的图块作为所有子级获取图块的方式。事情不是这样的。瓷砖应该直接用在需要的地方。

## 瓷砖中的瓷砖

当然，您可以使用另一个图块将图块组合在一起，而不必在特定模块中列出您想要的所有图块。

## 父 Pom 和瓷砖

我建议你在你的父 POM 中使用一些常见的配置，并在大多数构建配置方面使用 tiles。

**是否包含在您的父 pom 中:**

*   **基本项目配置(scm、url、问题管理等)**
*   **插件管理**
*   **依赖性管理**
*   **项目编码**

****不要**在您的父 pom 中包含:**

*   **属国**
*   **瓷砖**

**在你所有的叶项目中使用 tiles 来以可重复的方式配置构建。**

# **完成的示例**

**现在我们有了自己的 tiles 和一些父 POM，我们有了很多精彩的特性来帮助我们成为优秀的程序员，成为未来灵活社区的积极成员。**

# **变平**

**哦，等等，还有呢！**

**现在你有了你的父母和所有的插件，你还有最后一个机会。如果有人使用你的库，他们也必须下载你所有的父 POM。最重要的是，您部署到您的工件存储库中的所有 POM 将仍然包含您的所有构建配置。**

**为了清理那些 pom 并消除下载父对象的需要，我们可以*展平*一个 POM。**

**扁平化 pom 会将所有的依赖项合并到一个 pom 中。它还在部署时从 pom 中删除了许多不必要的信息。比如，**插件**，**属性**，**报告，**等等。**

**进一步阅读**

*   **[展平 Maven 插件](https://www.mojohaus.org/flatten-maven-plugin/)**

# **结论**

**我们在这里讨论了很多东西，仅仅触及了你如何定制我们在这里提到的插件的表面。希望这为您自己的 maven 努力提供了一个构建块。**

**不要忘记在我的 github repo 查看所有提到的代码，以及完整的示例和每个插件的图片:**

**[](https://github.com/efenglu/maven) [## 埃丰卢/马文

### 包含带有许多插件的 pom 的示例项目

github.com](https://github.com/efenglu/maven)**