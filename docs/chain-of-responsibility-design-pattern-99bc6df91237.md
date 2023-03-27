# 责任链设计模式

> 原文：<https://itnext.io/chain-of-responsibility-design-pattern-99bc6df91237?source=collection_archive---------6----------------------->

## SpringBoot 应用程序中的实用概述

![](img/fe8aa5a81d083619562b53df4fdcd5aa.png)

照片由 [JJ 英](https://unsplash.com/@jjying?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)

**责任链**设计模式是[四人组](https://en.wikipedia.org/wiki/Design_Patterns) (GoF)设计模式之一。

它通常解决以下问题:

*   应该避免将请求的发送方耦合到接收方
*   一个以上的接收者可以处理一个请求应该是可能的

反正如果你想更进一步，更多信息可以在[这里](https://en.wikipedia.org/wiki/Chain-of-responsibility_pattern)找到。

# 让我们直接进入实际的…

在这个故事中，我们将看到如何从零开始创建一个 Maven 项目，将需要的依赖项导入到 *pom.xml* 中，以及最后如何开发责任链设计模式。

## Maven 原型和依赖性

首先，我们必须创建 Maven 项目。创建 maven 项目最容易和最简单的方法是从[快速入门 Maven 原型](http://maven.apache.org/archetypes/maven-archetype-quickstart/)开始:它生成一个示例 Maven 项目。

> *显然，可以随意使用*[*spring boot Initializr*](https://start.spring.io/)*来执行这个任务*。

因此，打开一个新的终端，导航到您想要的位置，并复制粘贴以下命令；然后按回车。

```
mvn archetype:generate \
-DgroupId=com.mycompany.app \
-DartifactId=my-app \
-DarchetypeArtifactId=maven-archetype-quickstart \
-DarchetypeVersion=1.4 \
-DinteractiveMode=false
```

干得好！现在您已经成功地创建了项目，让我们用您最喜欢的 IDE 打开它。

首先要做的是添加父标记，如下所示

```
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.2.2.RELEASE</version>
</parent>
```

> Maven 从您的本地存储库中读取父 *pom* ，然后它能够通过合并来自父模块和 *pom* 模块的数据来创建一个有效的 *pom* 。这样做的原因是有一个中心点来存储可以在所有模块中使用的信息。

在此之后，我们想要添加的唯一两个依赖项是*spring-boot-starter-web*和 [*Lombok*](https://projectlombok.org/) 。

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency><dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <scope>provided</scope>
</dependency>
```

真的很好！现在，我们准备开发我们的服务。

## 责任链

和许多其他人一样，我喜欢将类分为两类:**事件**和**服务**。

事件是来自外部环境的所有那些对象，例如通过 REST 调用。另一方面，服务负责处理这些传入的事件。每个服务也负责回答这个问题:“我能够处理这个事件吗？”。如果答案是 **yes** ，获取事件并处理。**否则**，让我们将事件传递给下一个服务/处理器，而不处理它。

## 事件

在本节中，我们将创建 events 类。

抽象事件类

事件类的具体事件子类

## 服务

现在，让我们转到服务。

抽象处理程序

混凝土搬运工

抽象处理程序类对于在链中自动注入所有具体的处理程序子类是至关重要的，我们将在下面看到。

## 配置

继续，创建一个能够准备所有需要的处理程序链的配置类。这里是所有处理程序被注入到链中的地方。

显然，您必须修改主类，如下所示

```
@SpringBootApplication
public class App { public static void main(String[] args) {
        SpringApplication.run(App.class, args);
    }}
```

## 休息控制器

SpringBoot 应用程序如何接收传入事件？通过休息控制器。由于有了 **@class 字段**(在抽象事件类中定义)，该控制器使用 Jackson 映射来获取传入事件的具体实例。

# 结论

太好了！我们刚刚看到了如何在 SpringBoot 应用程序中开发责任链设计模式。

> 如果你喜欢，请随意评论和鼓掌。

下一个故事再见！