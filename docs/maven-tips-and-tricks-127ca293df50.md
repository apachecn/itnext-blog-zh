# Maven 提示和技巧

> 原文：<https://itnext.io/maven-tips-and-tricks-127ca293df50?source=collection_archive---------3----------------------->

![](img/d498c619e87ebbebbd6cab4185044062.png)

图片由[皮克斯拜](https://pixabay.com/?utm_source=link-attribution&utm_medium=referral&utm_campaign=image&utm_content=998958)的 Gerd Altmann 提供

我写 Maven pom 已经超过 10 年了。在那段时间里，我学会了许多技巧，让 maven 按照我想要的方式工作，并为我的开发人员提供出色的体验。

# Maven 包装

Maven 包装器很像 Gradle 包装器。包装器提供了一个可执行的 shell 脚本，如果需要的话，它会下载 Maven，然后使用它来完成构建。

这允许您在代码中配置 maven 版本。这在大型团队中是非常宝贵的，因为不是每个人的系统上都安装了相同版本的 Maven。

要将 maven 包装器添加到项目中，请运行

```
mvn -N io.takari:maven:0.7.6:wrapper
```

## 进一步阅读

*   [Maven 包装](https://github.com/takari/maven-wrapper)

# 执行单元的配置

当你添加一个插件到你的构建中时，你有时可以依赖它的默认配置，但是更多的时候你会想要为你的特定项目需求配置它。这意味着提供一个配置块和可能的执行块。

执行块是将配置隔离到特定环境的好方法。这样想吧。

根配置块用于全局配置。

内部执行配置块用于本地范围的配置。

例如:

```
<**plugin**>
    <**groupId**>org.example</**groupId**>
    <**artifactId**>example-plugin</**artifactId**>
    <**version**>1.0.0</**version**>
    <**configuration**>
        <**globalValue**>someValue<**/globalValue**>
    <**/configuration**>
    <**executions**>
        <**execution**>
            <**id**>doSomething</**id**>
            <**phase**>generate-sources</**phase**>
            <**goals**>
                <**goal**>runIt</**goal**>
            </**goals**>
            <**configuration>**
                <**localValue**>otherValue</**localValue**>
                <**globalValue**>override</**globalValue**>
            </**configuration**>
        </**execution**>
        <**execution**>
            <**id**>doSomethingElse</**id**>
            <**phase**>generate-sources</**phase**>
            <**goals**>
                <**goal**>runIt</**goal**>
            </**goals**>
            <**configuration>**
                <**localValue**>otherValue2</**localValue**>
            </**configuration**>
        </**execution**>    
     </**executions**>
</**plugin**>
```

这里我们有同一个插件的两个执行。属性 **globalValue** 的全局值被指定为 **someValue** 。

第一次执行 *doSomething* 将 **localValue** 属性分配给 **otherValue** ，并用 **override 的值覆盖 **globalValue** 。**

第二次执行 *doSomethingElse* 也给 **otherValue2** 分配了一个 **localValue** 属性，然后继承了 **globalValue。**

# 执行执行块

使用执行块，您还可以执行单个步骤。例如，如果我只想运行上例中的第二个执行块，我会运行:

```
mvn org.example:example-plugin@doSomethingElse
```

现在，对于开发人员来说，查找您选择的执行 id 可能有些繁琐。尤其是当这个执行存在于其他父 pom 中时。谢天谢地，有一个解决办法。

如果您有一个希望容易地向开发人员公开的执行，那么将执行 id 设置为 *default-cli。*

这是一个特殊的 ID，当有人执行一个没有执行 ID 的插件时，maven 会寻找这个 ID。

例如:

```
mvn org.example:example-plugin
```

这里隐含了*默认 cli* 的 ID。

现在，在一般执行中，不需要 ID。但是它们非常有用，我建议你总是提供一个。

# 插件管理

你也可以使用插件管理部分来配置一个插件，而不用执行它。现在你也可以用概要文件来做这件事，但是使用*插件管理*部分来确保全局共享配置。

例如，我可以将之前的插件配置放入*插件管理*部分，如下所示:

```
<**pluginManagement**>
  <**plugins>** <**plugin**>
      <**groupId**>org.example</**groupId**>
      <**artifactId**>example-plugin</**artifactId**>
      <**version**>1.0.0</**version**>
      <**executions**>
        <**execution**>
            <**id**>doSomething</**id**>
            <**phase**>generate-sources</**phase**>
            <**goals**>
                <**goal**>runIt</**goal**>
            </**goals**>
            <**configuration>**
                <**localValue**>otherValue</**localValue**>
                <**globalValue**>override</**globalValue**>
            </**configuration**>
        </**execution**>
        ...   
     </**executions**>
    </**plugin**>
  </**plugins**>
</**pluginManagement**>
```

现在，即使我们插件的阶段被设置为生成源代码，如果我运行:

```
mvn generate-sources
```

我们的插件不会运行。这是因为通过将我们的插件配置放在 *pluginManagement* 部分，我们给了 maven 一个执行的模板，但没有告诉它执行它。

你为什么想这么做？

这是一个跨所有执行类型共享配置的好方法。我发现这对于配置静态代码分析工具特别有用，比如 check-style，我希望在所有执行、命令行和部分构建中共享配置。

# 无阶段

假设你有一个插件，它有一个默认的执行阶段，但是你不想让它运行。您可以用自己的阶段覆盖默认阶段，但它仍然会运行。

如果你根本不想让它运行呢？

然后将相位设置为**无。**

**无**相，没听说过。猜猜看，美文也没有！所以它不会执行你的插件。这不是错误，maven 只是不会执行，因为没有 *none* 阶段。

这是一个禁用插件执行的好技巧。例如，假设之前的配置在父 pom 中。我可以像这样禁用 **doSomething** 的执行:

```
<**plugin**>
    <**groupId**>org.example</**groupId**>
    <**artifactId**>example-plugin</**artifactId**>
    <**executions**>
        <**execution**>
            <**id**>doSomething</**id**>
            <**phase**>none</**phase**>
        </**execution**>   
     </**executions**>
</**plugin**>
```

# 插件配置继承

说到父 pom，有些情况下我想改变或添加父插件的配置。如果我要改变单个元素的值，这很容易。

例如，我在这里向 **doSomething** 的父执行添加了一个配置元素:

```
<**plugin**>
    <**groupId**>org.example</**groupId**>
    <**artifactId**>example-plugin</**artifactId**>
    <**executions**>
        <**execution**>
            <**id**>doSomething</**id**>
            <**configuration>**
                <**newValue**>otherValue</**newValue**>
            </**configuration**>
        </**execution**>   
     </**executions**>
</**plugin**>
```

这里父配置保持不变。如果我想覆盖父配置，我可以遵循相同的策略。

```
<**plugin**>
    <**groupId**>org.example</**groupId**>
    <**artifactId**>example-plugin</**artifactId**>
    <**executions**>
        <**execution**>
            <**id**>doSomething</**id**>
            <**configuration>**
                <**localValue**>childValue</**localValue**>
            </**configuration**>
        </**execution**>   
     </**executions**>
</**plugin**>
```

这里我用**子值**覆盖了**本地值**的父值。

> 注意:您只能在同一范围内覆盖元素

这意味着我不能在子 pom 中为 localValue 提供一个全局配置元素，并期望它覆盖所有父执行配置中的值。如果我想覆盖所有父执行配置中的值，我需要在我的子 pom 中列出每个父执行，并在每个子 POM 中提供一个配置。

# 插件配置继承:列表和映射

有时配置提供列表或地图元素。maven 的默认行为是尝试*合并*这些定义。

假设您的父级具有以下带有列表和地图元素的配置:

```
<**plugin**>
    <**groupId**>org.example</**groupId**>
    <**artifactId**>example-plugin</**artifactId**>
    <**version**>1.0.0</**version**>
    <**configuration**>
        <**globalValue**>someValue<**/globalValue**>
    <**/configuration**>
    <**executions**>
        <**execution**>
            <**id**>doSomething</**id**>
            <**phase**>generate-sources</**phase**>
            <**goals**>
                <**goal**>runIt</**goal**>
            </**goals**>
            <**configuration>**
                <**listValues**>
                  <**value**>hello</**value>** </**listValues**>
                <**mapValues**>
                  <**key1**>value1</**key1>** <**key2**>value2</**key2>** </**mapValues>** </**configuration**>
        </**execution**> 
     </**executions**>
</**plugin**>
```

## 我如何向列表中添加元素？

将以下属性添加到列表元素中:

```
combine.children=”append”
```

## 我该如何覆盖地图？

将以下属性添加到地图元素中:

```
combine.self="override"
```

例如配置:

```
<**plugin**>
    <**groupId**>org.example</**groupId**>
    <**artifactId**>example-plugin</**artifactId**>
    <**version**>1.0.0</**version**>
    <**configuration**>
        <**globalValue**>someValue<**/globalValue**>
    <**/configuration**>
    <**executions**>
        <**execution**>
            <**id**>doSomething</**id**>
            <**phase**>generate-sources</**phase**>
            <**goals**>
                <**goal**>runIt</**goal**>
            </**goals**>
            <**configuration>**
                <**listValues combine.children="append"**>
                  <**value**>world</**value>** </**listValues**>
                <**mapValues combine.self="override"**>
                  <**key3**>value1</**key3>** </**mapValues>** </**configuration**>
        </**execution**> 
     </**executions**>
</**plugin**>
```

将导致以下效果:

```
<**plugin**>
    <**groupId**>org.example</**groupId**>
    <**artifactId**>example-plugin</**artifactId**>
    <**version**>1.0.0</**version**>
    <**configuration**>
        <**globalValue**>someValue<**/globalValue**>
    <**/configuration**>
    <**executions**>
        <**execution**>
            <**id**>doSomething</**id**>
            <**phase**>generate-sources</**phase**>
            <**goals**>
                <**goal**>runIt</**goal**>
            </**goals**>
            <**configuration>**
                <**listValues combine.children="append"**>
                  <**value**>hello</**value>** <**value**>world</**value>** </**listValues**>
                <**mapValues combine.self="override"**>
                  <**key3**>value1</**key3>** </**mapValues>** </**configuration**>
        </**execution**> 
     </**executions**>
</**plugin**>
```

## 进一步阅读

*   [Maven 文档](https://maven.apache.org/pom.html#Plugins)

# Maven BOM

当您构建一个拥有多个不同模块的库时，您通常会希望所有相同版本的模块一起使用。你可以依靠消费者来做这件事，或者你可以通过创建一个 BOM 来帮助他们。

传统上，如果您想为一个特定的依赖项或一组依赖项定义一个版本，您应该在父 pom 中设置一个 dependencyManagement 块。然而，父 pom 使用继承模型，当您想要使用不同的库时，很快就会遇到问题。

maven 材料清单或 BOM 是一种特殊的包装，允许您组成*依赖管理*模块。

要创建一个 BOM，只需创建一个 maven 模块来打包 **pom** 并像平常一样设置 *dependencyManagement* 块:

```
*<?***xml version="1.0" encoding="UTF-8"***?>* <**project 
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd"**>
    <**modelVersion**>4.0.0</**modelVersion**>

    <**groupId**>io.github.efenglu.examples</**groupId**>
    <**artifactId**>examples-bom</**artifactId**>
    <**version**>1.0.0</**version**>
    <**packaging**>pom</**packaging**>

    <**properties**>
        <**examples.version**>${project.version}</**examples.version**>
    </**properties**>

    <**dependencyManagement**>
        <**dependencies**>
            <**dependency**>
                <**groupId**>io.github.efenglu.examples</**groupId**>
                <**artifactId**>example-1</**artifactId**>
                <**version**>${examples.version}</**version**>
            </**dependency**>
            <**dependency**>
                <**groupId**>io.github.efenglu.examples</**groupId**>
                <**artifactId**>example-2</**artifactId**>
                <**version**>${examples.version}</**version**>
            </**dependency**>... </**dependencies**>
    </**dependencyManagement**>

</**project**>
```

然后，为了使用 BOM，将以下内容添加到您自己的 *dependencyManagement* 块中:

```
<**dependencyManagement**>
    <**dependencies**>
<**dependency**>
            <**groupId**>io.github.efenglu.examples</**groupId**>
            <**artifactId**>examples-bom</**artifactId**>
            <**version**>1.0.0</**version**>
            <**type**>pom</**type**>
            <**scope**>import</**scope**>
        </**dependency**>
...
    </**dependencies**>
</**dependencyManagement**>
```

注意类型是 **pom** ，范围是 **import** 。这将导致 BOM 的 *dependencyManagement* 块中的所有依赖项被导入到您的 *dependencyManagement* 块中。

## 进一步阅读

*   梅文·博姆

# 设置版本

设置单个 pom 的版本是微不足道的。只需更改 version 元素中的值。然而，在有父项目和多个模块的大型项目中，手动更改版本可能会很繁琐并且容易出错。幸运的是，maven 有一个工具可以让修改版本变得非常简单，即使是在大型项目中。

运行以下命令:

```
mvn versions:set
```

Maven 随后会提示您想要将项目更改为哪个版本。如果这是一个父 pom，所有子 POM 的版本都会相应地更新。

## 进一步阅读

*   [Maven 版本插件](https://www.mojohaus.org/versions-maven-plugin/)

# 有效 POM

我如何更好地了解 maven 将要做什么，或者它为什么要做什么？

最好的方法是生成有效的 pom。

有效的 pom 是在替换了所有属性、继承了所有配置之后，maven 如何看待您的 pom 文件的完整视图。以及必要时合并或附加的配置。

这在调试插件配置时非常有用，应该是您解决 maven 配置问题的首选。

要生成有效的 pom，只需运行:

```
mvn help:effective-pom
```

## 延伸阅读:

*   [Maven 帮助插件](https://maven.apache.org/plugins/maven-help-plugin/usage.html)