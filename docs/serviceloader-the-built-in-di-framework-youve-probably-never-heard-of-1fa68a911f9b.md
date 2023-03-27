# ServiceLoader:您可能从未听说过的内置 DI 框架

> 原文：<https://itnext.io/serviceloader-the-built-in-di-framework-youve-probably-never-heard-of-1fa68a911f9b?source=collection_archive---------0----------------------->

![](img/fc8c0f89303c8c9114d425164587d3c2.png)

照片由[负空间](https://www.pexels.com/@negativespace?utm_content=attributionCopyText&utm_medium=referral&utm_source=pexels)发自[像素](https://www.pexels.com/photo/grayscale-photo-of-computer-laptop-near-white-notebook-and-ceramic-mug-on-table-169573/?utm_content=attributionCopyText&utm_medium=referral&utm_source=pexels)

大多数 java 开发人员已经使用 [Spring](https://spring.io) 进行依赖注入(DI)。有些人甚至使用了谷歌 Guice 或者 OSGi 服务。但是很多人不知道 Java 其实有一个内置的 DI 系统。这是 Java 11 或 12 的新特性吗？不，它从 Java 6 开始就存在了。

[ServiceLoader](https://docs.oracle.com/javase/7/docs/api/java/util/ServiceLoader.html) 提供了查找和实例化接口或抽象类的注册实例的能力。对于那些熟悉 Spring 的人来说，它非常类似于 [Bean](https://www.baeldung.com/spring-bean-annotations) 和[自动连线](https://www.baeldung.com/spring-autowire)注释。让我们看一个使用 Spring 的例子和一个使用 ServiceLoader 的例子。我们将讨论相同点和不同点。

![](img/c2a938e137c8c3f4cd4bbf9089a8f700.png)

[spring.io](https://spring.io)

# 弹簧依赖注入

首先，让我们回顾一下用 Spring 做简单 DI 的方法。

我们将创建一个简单的界面:

```
**public interface** SimpleService {

    String echo(String value);

}
```

然后是接口的实现:

```
**import** org.springframework.stereotype.Component;

@Component
**public class** SimpleServiceImpl **implements** SimpleService {
    **public** String echo(**final** String value) {
        **return** value;
    }
}
```

注意**@组件**。这将把类作为 bean 添加到应用程序上下文中。

最后，我们的主类:

```
@SpringBootApplication
**public class** SpringExample **implements** CommandLineRunner {

    @Autowired
    List<SimpleService> **simpleServices**;

    **public static void** main(String[] args) {
        SpringApplication.*run*(SpringExample.**class**, args);

    }

    **public void** run(**final** String... strings) **throws** Exception {
        **for** (SimpleService simpleService : **simpleServices**) {
            ***log***.info(**"Echo: "** + simpleService.echo(strings[0]));
        }
    }
}
```

注意@ **自动连线的**简单服务列表。@**spring boot 应用程序**将自动扫描组件包。然后在运行时，这些被连接到 *SpringExample* 类中。

# ServiceLoader 依赖注入

我们将使用与 Spring 示例相同的接口，因此这里不再重复。让我们转而关注服务的实现:

```
**import** com.google.auto.service.AutoService;

@AutoService(SimpleService.**class**)
**public class** SimpleServiceImpl **implements** SimpleService {
    **public** String echo(**final** String value) {
        **return** value;
    }
}
```

在实现中，我们用注释@ **AutoService** 来“注册”服务的一个实例。这只在编译时需要，因为 javac 注释处理器使用注释来自动生成服务注册文件:

> META-INF/services/io . github . efenglu . service loader . example . simple service

该文件包含实现该服务的类的列表:

```
io.github.efenglu.serviceLoader.example.SimpleServiceImpl
```

文件名必须与服务的完全限定名匹配。内容可以有任意数量的实现，每个实现在单独的一行上。

实现类**必须**有一个无参数的构造函数。

您可以手动生成该文件，但是使用该注释要容易得多。

和主类:

```
**public class** ServiceLoaderExample {

    **public static void** main(String [] args) {
        **final** ServiceLoader<SimpleService> services = ServiceLoader.*load*(SimpleService.**class**);
        **for** (SimpleService service : services) {
            System.***out***.println(**"Echo: "** + service.echo(args[0]));
        }
    }
}
```

**ServiceLoader.load** 被调用，加载一个 **ServiceLoader** ，可以用来获取所有服务的实例。ServiceLoader 实例是服务类型的 iterable，因此服务变量上的每个循环都有*。*

![](img/47952f2e3db0b692d6420fbf2587ff3e.png)

照片由[像素](https://www.pexels.com/photo/view-ape-thinking-primate-33535/?utm_content=attributionCopyText&utm_medium=referral&utm_source=pexels)的 [Pixabay](https://www.pexels.com/@pixabay?utm_content=attributionCopyText&utm_medium=referral&utm_source=pexels) 拍摄

# 那又怎样？

这两种实现都相对较小。两者都可以主要基于注释，因此使用起来相当简单。那么，你为什么要使用 ServiceLoader 而不是 Spring 呢？

## 属国

让我们看一下非常简单的 Spring 服务加载器的依赖树:

```
[INFO] -----------< io.github.efenglu.serviceLoader:spring-example >-----------
[INFO] Building spring-example 1.0.X-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- maven-dependency-plugin:3.1.1:tree (default-cli) @ spring-example ---
[INFO] io.github.efenglu.serviceLoader:spring-example:jar:1.0.X-SNAPSHOT
[INFO] +- org.slf4j:slf4j-api:jar:1.7.25:compile
[INFO] +- org.springframework:spring-context:jar:4.3.22.RELEASE:compile
[INFO] |  +- org.springframework:spring-aop:jar:4.3.22.RELEASE:compile
[INFO] |  +- org.springframework:spring-core:jar:4.3.22.RELEASE:compile
[INFO] |  |  \- commons-logging:commons-logging:jar:1.2:compile
[INFO] |  \- org.springframework:spring-expression:jar:4.3.22.RELEASE:compile
[INFO] +- org.springframework.boot:spring-boot-autoconfigure:jar:1.5.19.RELEASE:compile
[INFO] +- org.springframework.boot:spring-boot:jar:1.5.19.RELEASE:compile
[INFO] \- org.springframework:spring-beans:jar:4.3.22.RELEASE:compile
```

与我们的服务加载器相比:

```
[INFO] io.github.efenglu.serviceLoader:serviceLoader-example:jar:1.0.X-SNAPSHOT## Only provided dependencies for the auto service annotation
[INFO] \- com.google.auto.service:auto-service:jar:1.0-rc4:provided
[INFO]    +- com.google.auto:auto-common:jar:0.8:provided
[INFO]    \- com.google.guava:guava:jar:23.5-jre:provided
[INFO]       +- com.google.code.findbugs:jsr305:jar:1.3.9:provided
[INFO]       +- org.checkerframework:checker-qual:jar:2.0.0:provided
[INFO]       +- com.google.errorprone:error_prone_annotations:jar:2.0.18:provided
[INFO]       +- com.google.j2objc:j2objc-annotations:jar:1.1:provided
[INFO]       \- org.codehaus.mojo:animal-sniffer-annotations:jar:1.14:provided
```

如果我们忽略所提供的依赖项，则 ServiceLoader 示例没有依赖项。没错，它只需要 Java。

现在，如果你是一个 Spring 商店，这并不是什么大不了的事情，但是如果你正在写一些可以在多种不同框架中使用的东西，或者想要一个小的命令行应用程序，这将会有很大的不同。

## 速度

说到命令行应用，ServiceLoader 的启动时间比 Spring Boot 应用要快得多。这要归功于加载的代码更少，没有扫描，没有反射，没有大框架。

## 记忆

春天并不因为记忆力好而出名。如果内存对您的应用程序至关重要，并且您需要某种 DI 功能，那么您应该考虑使用 ServiceLoader。

![](img/6988aaef84c909f8c00d5a94c0223fcf.png)

照片由 [Pixabay](https://www.pexels.com/@pixabay?utm_content=attributionCopyText&utm_medium=referral&utm_source=pexels) 从[像素](https://www.pexels.com/photo/blue-white-orange-and-brown-container-van-163726/?utm_content=attributionCopyText&utm_medium=referral&utm_source=pexels)拍摄

# Java 模块

java 模块的一个关键方面是将类与模块外的代码完全隔离的能力。ServiceLoader 是允许外部代码“访问”内部实现的机制。Java 模块允许您为内部实现注册服务，同时仍然维护防火墙。

事实上，这是唯一官方认可的支持 Java 模块依赖注入的机制。Spring 和大多数现有的 DI 框架依靠反射来发现和连接它们的组件。但是这在 Java 模块中是行不通的。甚至反射也不能看到模块内部(除非你允许，但是你为什么要这么做)。

# 结论

春天很棒。它比 ServiceLoader 做得更多。但是有些时候 ServiceLoader 是正确的选择。它简单、小巧、快速且随时可用。

要查看示例的完整源代码，请查看我的 Git Repo:

[](https://github.com/efenglu/serviceLoaderExample) [## efenglu/serviceLoaderExample

### ServiceLoader 使用指南。在…上创建帐户，为 efenglu/serviceLoaderExample 开发做出贡献

github.com](https://github.com/efenglu/serviceLoaderExample)