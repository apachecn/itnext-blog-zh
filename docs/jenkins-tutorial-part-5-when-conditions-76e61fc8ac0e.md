# 詹金斯教程—第 5 部分—当条件

> 原文：<https://itnext.io/jenkins-tutorial-part-5-when-conditions-76e61fc8ac0e?source=collection_archive---------0----------------------->

![](img/c518273176f996c36d381a98cf53ea9a.png)

在 Jenkins 教程系列的这篇文章中，我打算解释 Jenkins 管道中的条件。假设您希望在满足一个或多个条件时执行管道阶段。你怎么能这样做？答案是当条件具备时。那么，我们开始吧。

如果你对这个教程系列感兴趣， **STAR** ize 以下 GitHub repo。这个回购是我为本教程创建的一个特殊回购。

[](https://github.com/ssbostan/jenkins-tutorial) [## GitHub-ssbo stan/jenkins-教程:Jenkins 参考和管道代码示例

### Jenkins 参考和管道代码示例的 Jenkins 教程系列-GitHub-ssbo stan/Jenkins-教程…

github.com](https://github.com/ssbostan/jenkins-tutorial) 

# 詹金斯“何时”指令:

流水线阶段的执行可以用条件来控制。这些条件必须在每个阶段的`when`块中定义。Jenkins 支持一组重要的条件，可以定义这些条件来限制阶段执行。每个`when`块必须包含至少一个条件。如果在`when`块中声明了多个条件，那么对于正在执行的阶段，所有条件都应该返回 true。此外，使用嵌套条件可以定义更复杂的条件，下面将对此进行解释。

以上示例中的阶段**测试**仅在管道作业的第一次运行时运行，且仅运行一次。这个阶段不是从二号楼开始运行。

# 内置条件:

Jenkins 本地支持的条件称为内置条件。除了这些条件，一些插件可能会添加更多的条件。对于这种情况，请参见 Jenkins 插件文档。

**branch** 用给定的模式检查源代码分支名。如果分支名称与模式匹配，则执行阶段。您应该注意，这种情况仅适用于多分支管道。

下例中的**测试**阶段在分支名称以“release-”开始时执行，所有 ANT 样式的模式都被接受。

**buildingTag** 如果当前 git 提交有一个标签，则运行下面的阶段。这种情况受到一个未修复的错误的影响，如果您看到它不起作用，您应该手动设置 TAG_NAME 环境变量。

**changelog** 获取一个正则表达式，并将其与上一次 git 提交的消息进行匹配。如果日志消息与给定的模式匹配，则执行下面的阶段。您应该注意，此条件仅在多分支管道以及脚本来自 SCM repo 的管道中有效。在下面的示例中，当 git 提交消息包含“ **Test** ”字符串时，该阶段运行。

**变更集**按照给定的模式观察文件/目录的变化。它会看到最后一次 git 提交，如果有任何文件/目录发生了与给定模式相匹配的更改，就会执行这个阶段。

**环境**检查环境变量值。

**等于**如果实际值等于期望值，则运行阶段。例如，如果当前内部版本号为 1，以下条件将运行阶段。可以说，它只运行一次。

**表达式**获取一个 Groovy 语言表达式，如果表达式的值为 true，则运行下面的阶段。在字符串的情况下，包括“0”和“假”的所有值都返回**真**。如果您打算使用字符串作为表达式的一部分，您必须将值设置为 **null** ，以将其计算为 **false** 。

如果 TAG_NAME 变量与给定模式匹配，tag 运行舞台。如果模式为空，并且 TAG_NAME 变量存在，它将运行舞台。

**triggeredBy** 当当前构建被给定参数触发时，执行该阶段。此条件对于通知非常有用。

# 詹金斯复杂条件:

复杂条件通常是上面解释的一组条件。Jenkins 支持三种复杂/嵌套条件。`not`、`allOf`、`anyOf`是与条件结合使用的复杂条件。

**not** 执行嵌套条件为**假**的阶段。

如果所有嵌套条件都为**真**，则 **allOf** 执行该阶段。

**如果至少有一个嵌套条件为**真**，anyOf** 执行该阶段。

# 何时评估“When”指令？

默认情况下，`when`指令在`agent`、`input`和`options` 指令之后被评估。您可以在`when`块内用`beforeAgent`、`beforeInput`和`beforeOptions`改变这些。

```
when {
    beforeAgent true
    beforeInput true
    beforeOptions true
    // Write your conditions here...
}
```

Jenkins CI 是实现 CI/CD 管道的一个强大而丰富的工具。有时候，你可能会觉得很复杂，其实不然。你应该拥有日复一日的实践来巩固你的知识。如果您有任何问题，请在下面评论或在教程的 GitHub repo 上打开一个问题。

关注我的 LinkedIn[https://www.linkedin.com/in/ssbostan](https://www.linkedin.com/in/ssbostan)