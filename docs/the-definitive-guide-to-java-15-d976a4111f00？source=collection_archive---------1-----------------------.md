# Java 15 终极指南

> 原文：<https://itnext.io/the-definitive-guide-to-java-15-d976a4111f00?source=collection_archive---------1----------------------->

## 全面介绍 Java 14 和 Java 15 中最有效的更新

![](img/6fb043dd7969404125b902b6811f6a0e.png)

米歇尔·勒恩斯在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 拍摄的照片

随着 Java SE 8 于 2014 年 3 月 18 日发布，Java 成为当今世界上应用最广泛的编程语言之一，在主要应用程序中占了 60%的使用量。

Java 14 和 Java 15 分别于 2020 年 3 月 17 日和 9 月 15 日发布，为开发者增加了更多功能。

让我们对每个版本的主要 JEP (Java 增强建议)有一个概述。

**Java 14** :实例的模式匹配( [JEP 305](https://openjdk.java.net/jeps/305) )、有用的 NullPointerExceptions([JEP 358](https://openjdk.java.net/jeps/358))、记录( [JEP 359](https://openjdk.java.net/jeps/359) )、开关表达式( [JEP 361](https://openjdk.java.net/jeps/361) )、文本块( [JEP 368 【T17)](https://openjdk.java.net/jeps/368)

**Java 15:** 密封类或接口( [JEP 360](https://openjdk.java.net/jeps/360) )

# Java 14

## 实例的模式匹配

每个 Java 程序员都应该知道用来定义块的*实例的旧语法。它预见了三个主要步骤:测试、新变量声明和转换(将 obj 转换为 String)*

```
if (obj instanceof String) {
    String s = (String) obj;
    // use s
}
```

istanceof 操作符现在扩展为接受类型测试模式，而不仅仅是类型。在下面的例子中， *YourClass y* 是类型测试模式。

正如您所注意到的，新的 instanceof 操作符允许您在一个步骤中测试、声明和转换变量。显然，您可以马上使用新声明的变量！

这种语法也简化了方法的实现。仔细看看上面代码片段中的 *equals()* 和 *multiply()* 方法。

## 有用的 NullPointerExceptions

当程序试图取消引用一个空引用时，JVM 抛出一个 NPE。基本上，这个增强希望通过精确描述哪个变量为空来提高 JVM 生成的 NPE 的可用性。现在，考虑我们有下面的赋值表达式:

```
a.i = 99
```

…并认为 *a* 为空。该执行将导致 JVM 打印出导致 NPE 的方法、文件名和行号。

```
Exception in thread "main" java.lang.NullPointerException
    at Program.main(Program.java:5)
```

随着 Java 14 和 JEP 358 的发布，消息将是:

```
Exception in thread "main" java.lang.NullPointerException: 
        Cannot assign field "i" because "a" is null
    at Program.main(Program.java:5)
```

在更复杂的陈述中:

```
a.b.c.i = 99Exception in thread "main" java.lang.NullPointerException: 
        Cannot read field "c" because "a.b" is null
    at Program.main(Program.java:5)
```

## 记录

大家都知道 Java 真的太啰嗦了。在我看来，记录是有史以来最好的增强之一，特别是对于那些仅仅是数据集合者或载体的类，它们获得了仅仅由名称和状态表示的极端简洁的级别。国家宣布记录的组成部分。

记录将隐含地定义:

*   状态描述的每个组成部分的私有最终字段
*   状态描述的每个组件的公共读取访问器方法
*   具有状态描述的相同签名的公共构造函数
*   equals()、hashCode()和 toString()方法的实现

哪些是对记录的限制？

*   记录不能扩展任何其他类
*   除了状态描述中的最终字段，记录不能声明实例字段
*   任何其他声明的字段必须是静态的
*   记录是最终的(它不能被其他类或记录扩展)并且不能是抽象的

最后，谈到注释:如果它们适用于记录组件、参数、字段或方法，那么它们可以用在记录组件上。

## 切换表达式

一般来说，switch 块允许应用程序根据运行时表达式的结果拥有多个可能的执行路径。

在 Java 14 中，switch 表达式得到了扩展，引入了许多有趣的新特性。

让我们只记下用来定义块表达式的旧方法:

```
switch(***expression***) {
  case **a**:
    *// code block*
    break;
  case **b**:
    *// code block*
    break;
  default:
    *// code block*
}
```

在 JEP 361 中，引入了一种新形式的交换机标签:“case a ->”。这意味着如果标签匹配，则只执行标签右边的代码。

先前代码片段的新版本变成了

```
switch(***expression***) {
  case **a** -> // code
  case **b** -> // code
  default -> // code
}
```

接下来，switch 语句已被扩展，因此可以用作表达式。在最普通的形式中，开关表达式看起来像:

```
T result = switch(expression) {
  case label1 -> expression1;
  case label2 -> expression2;
  default -> expression3;
}
```

**产生一个值**

每当需要一个完整的块而不是单行表达式时，就会引入一个新的关键字 **yield** 来产生一个值。前面的语句变成了:

```
T result = switch(***expr***) {
  case label1 -> expression1;
  case label2 -> expression2;
  default -> {
    expression3;
    expression4;
    ***yield*** result;}
```

牢记以下条款非常重要:

> *这两个语句中，* `*break*` *(带或不带标签)和* `*yield*` *，便于* `*switch*` *语句和* `*switch*` *表达式之间容易产生歧义:一个* `*switch*` *语句而不是一个* `*switch*` *表达式可以是一个* `*break*` *语句的目标；并且一个* `*switch*` *表达式而不是一个* `*switch*` *语句可以成为一个* `*yield*` *语句的目标。*

这意味着 yield 关键字仅用于从完整的块代码中返回值；必须赋给变量的结果。

所以，在最后，我想报告另一个新特性 switch 语句的例子，它被用作`syso call`中的表达式:

```
static void isEven(int k) {
    int e = k % 2;
    System.out.println(
        switch (e) {
            case  0 -> true;
            case  1 -> false;
            default -> "WTF";
        }
    );
}
```

## 文本块

欢迎，文字块！从现在开始，开发人员可以定义跨越几行源代码(例如 HTML、SQL 查询)的字符串，避免转义序列，从而增强可读性，比如 Python 中的。下面是一个例子:

```
String query = """
               SELECT `PERSON_ID`, `NAME` FROM `PERSONS`
               WHERE `AGE` > 35
               ORDER BY `PERSON_ID`, `NAME`;
               """;
```

# Java 15

## 密封的类或接口

密封类(或接口)是 JEP 360 在 Java 15 中引入的一个有趣特性。显然，继承的目的不仅仅是重用代码，甚至是对领域中存在的各种可能性进行建模。

密封的类或接口通过关键字*permit*限制了其他类或接口可以扩展或实现它们的集合。

举个例子，我们可以决定定义一个抽象类*动物*，并用*爬行动物*和*两栖动物*来对领域建模，如下面的代码片段*。*

我希望你喜欢阅读这篇文章！

谢谢你，❤

[](/springboot-liquibase-and-mariadb-b3f943c29370) [## SpringBoot，Liquibase 和 MariaDB

### 用 SpringBoot 中的 Liquibase 管理 MariaDB 数据库

itnext.io](/springboot-liquibase-and-mariadb-b3f943c29370) [](/dockerize-a-springboot-app-1755e49fe3d9) [## 将 SpringBoot 应用程序归档

### 让我们从 docker 图像和容器开始，让我们展示如何将 SpringBoot 应用程序 Docker 化并上传到 docker…

itnext.io](/dockerize-a-springboot-app-1755e49fe3d9)