# 对 SpringBoot 属性和轮廓的精细控制

> 原文：<https://itnext.io/fine-control-over-springboots-properties-and-profiles-25068bf25f1f?source=collection_archive---------1----------------------->

![](img/58220e29b8ebfa91d26c40a1dffb64c4.png)

SpringBoot 为您提供了对应用程序属性的大量控制。使用概要文件，您可以轻松地切换这些属性，而无需接触您的代码。

但是有时我们希望对每个概要文件中的活动属性有更多的控制。

想象一个奇特的代码，其中您需要一个来自(外部)配置的属性，如下所示:

为了简洁起见，我使用`@Value`。使用`[@ConfigurationProperties](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config.typesafe-configuration-properties)`是在应用程序内部使用外部属性的一种更安全的方式(感谢 [@matthenry87](https://medium.com/@matthenry87) 指出这一点)。

为了给这个`property`提供一个值，您可以在您的`application.properties`文件中指定它，如下所示:

```
# application.propertiesproperty=default
```

现在你的代码打印出来`default`。🎉

注意:有几种其他的方法来设置属性值，但是为了这篇文章，我总是使用一个. properties 文件。

您可以通过指定想要使用的配置文件来基于您的配置文件更改属性(这也可以通过几种方式来完成，但我将只在`application.properties`文件中指定它)。

让我们使用一个具有不同值的`dev`配置文件:

```
# application.propertiesproperty=default
spring.profiles.active=dev
```

让我们为`dev`概要文件定义一个. properties 文件:

```
# application-dev.propertiesproperty=dev
```

现在你的代码打印出来了`dev`。🎉🎉

您可以创建几个配置文件，并为每个配置文件创建一个. properties 文件。那对你有很大帮助。

但是想象一下，例如，您想在除一个之外的所有配置文件中设置一个属性。据我所知，没有办法只是取消一个属性。所以不能在你的默认中声明。属性，并在不需要的地方重写。

# 多文档文件

使用[多文档文件](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#features.external-config.files.multi-document)，您可以将. properties 文件的内容分成几个逻辑部分。使用[激活属性](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#features.external-config.files.activation-properties)，您可以添加逻辑来启用/禁用每个部分。

您可以定义一个。像这样的属性文件:

```
# application.propertiesproperty=default
spring.profiles.active=unique
#---
spring.config.activate.on-profile=unique
property=unique
```

注意，这是一个单独的`application.properties`文件。

而现在你的花式代码打印`unique`。🎉🎉🎉

如果你把你的`spring.profiles.active`改成`dev`，你的代码就会打印出`default`。🎉🎉🎉🎉

## 这里发生了什么事？

那个`#---`真的**重要吗**。它表示这是一部分的结束和下一部分的开始。在每个部分的开始，你可以告诉哪个概要文件使用它(用`spring.config.activate.on`属性)。

这是一个简单的例子，但是你可以想象这个特性的威力。

你可以在你的`spring.config.activate.on-profile`财产上使用更多花哨的东西。

如果您想在几个配置文件中激活它，您可以这样定义它:

```
spring.config.activate.on-profile=local | dev
```

如果您**不想在特定的个人资料中激活它，您可以这样做:**

```
spring.config.activate.on-profile=**!**unique
```

给你。现在你可以用你的属性做真正复杂的逻辑了！如果你应该做，那是另一回事。😈

有几件事需要注意:

*   对于 YAML 文件，分隔符只是`---`。这是标准的 YAML 风格。
*   您**不能**在分隔符前后使用注释。
*   始终避免部署包含不同概要文件配置的. properties 文件。您可能会意外地启用或禁用一个概要文件，而您应用程序仍然可能会在带有错误概要文件的环境中意外地运行。您不希望您的生产环境使用开发数据库。(不，这种事从未发生在我身上。😐好的。确实发生过一次。😢)
*   在代码中引入复杂和不寻常的语法之前，一定要彻底考虑你的问题。

在某些边缘情况下，这给了我很大的帮助。我希望它也能帮助你。了解你手中所有的武器总是好的。

# 链接

 [## Spring Boot 参考文献

### 导入 org . spring framework . boot . spring application；导入 org . spring framework . boot . spring boot configuration；导入…

docs.spring.io](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/) [](https://spring.io/blog/2020/08/14/config-file-processing-in-spring-boot-2-4) [## Spring Boot 2.4 中的配置文件处理

### Spring Boot 2.4.0.M2 刚刚发布，它带来了一些有趣的变化

spring.io](https://spring.io/blog/2020/08/14/config-file-processing-in-spring-boot-2-4)