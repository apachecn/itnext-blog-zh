# 定制 gRPC 生成的代码

> 原文：<https://itnext.io/customizing-grpc-generated-code-5909a2551ca1?source=collection_archive---------3----------------------->

gRPC 是一个强大的框架，在服务之间提供快速有效的通信。gRPC 是什么以及如何使用它超出了本文的范围。如果您有兴趣开始使用 gRPC，我建议您阅读位于[此处](https://grpc.io/docs/guides/)的 gRPC 文档。

gRPC 的核心组件是客户机/服务器应用程序的代码生成。该代码由协议编译器从*中生成。proto* 文件。虽然生成的代码遵循了一些好的实践，比如不可变对象和构建器模式，但是对于生成的代码，您可能喜欢也可能不喜欢其他的决定。

从表面上看，gRPC 似乎是我们的完美解决方案。但是我们遇到了几个问题，不是直接与 gRPC 或 Protobuf 有关，而是与在生成的代码中特别做出的选择有关。

我们试图使用 gRPC 将现有的 Java 应用程序迁移到微服务。不幸的是，我们现有的代码在我们的域对象中明智地使用了 *null* 。(这是否是个好主意无关紧要。这就是我们所拥有的，我们没有时间去重构它。)从表面上看，这似乎不是一个问题，直到我们开始使用 gRPC 生成的代码。

这里有一个简单的例子。假设我有一个现有的 Java 域对象雇员:

```
@Value
**public class** Employee {
    String **name**;
    BigDecimal **salary**;
    BigDecimal **bonus**;
}
```

在我们现有的模型中，如果员工没有奖金，我们会将字段*留空*。

这个域对象的原型定义如下:

```
import **"google/type/money.proto"**;

message Employee {
    string name = 1;
    google.type.Money salary = 2;
    google.type.Money bonus = 3;
}
```

现在，我们的域对象有了一个 protobuf 模型。让我们开始使用它。作为产品化阶段，我们采用的模式是在原始域对象和新的原型对象之间构建代理接口和转换器。这允许我们在很少甚至没有重构的情况下将现有代码转换为使用 protobuf。

示例:

```
**public class** EmployeeTransformer {

    **public** Employee toEmployee(EmployeeProto employeeProto) {
        **return** Employee.*newBuilder*()
                .name(employeeProto.getName())
                .salary(toBigDecimal(employeeProto.getSalary()))
                .bonus(toBigDecimal(employeeProto.getBonus()))
                .build();
    }

    **public** EmployeeProto toEmployeeProto(Employee employee) {
        **return** EmployeeProto.*newBuilder*()
                .setSalary(toMoney(employee.getSalary()))
                .setBonus(toMoney(employee.getBonus()))
                .build();
    } ...
}
```

看起来不错，对吧？

作为优秀的开发人员，我们应该编写一些测试用例:

```
**def "test toEmployeeProro"**(EmployeeProto input, Employee expected) {
    **given**:
    EmployeeTransformer transformer = **new** EmployeeTransformer()

    **expect**:
    transformer.toEmployeeProto(expected) == input

    **where**:
    input                                | expected
    *createEmployeeProto*(**"bob"**, 45, 100)  | *createEmployee*(**"bob"**, 45, 100)
    *createEmployeeProto*(**"bob"**, 45, 0)    | *createEmployee*(**"bob"**, 45, 0)
    *createEmployeeProto*(**"bob"**, 45, **null**) | *createEmployee*(**"bob"**, 45, **null**)
}
```

让我们运行这个规范，一切都很好吧？不对！

```
Condition failed with Exception:transformer.toEmployeeProto(expected) == input
|           |               |
|           |               Employee(name=bob, salary=45, bonus=null)
|           java.lang.NullPointerException
com.efenglu.jprotoc.example.EmployeeTransformer@7c83dc97
```

嗯，太奇怪了。

让我们看看，当奖金为空值时，我们得到了一个 NPE，因为我们用空值调用了集合。

这是因为:

## Protobuf 不允许空值

它有也没有。Protobuf 不允许你设置一个字段为空。它将在带有空值的调用集上抛出一个 NPE。

那么，我们的另一个规格呢:

```
**def "test toEmployee"**(EmployeeProto input, Employee expected) {
    **given**:
    EmployeeTransformer transformer = **new** EmployeeTransformer()

    **expect**:
    transformer.toEmployee(input) == expected

    **where**:
    input                                | expected
    *createEmployeeProto*(**"bob"**, 45, 100)  | *createEmployee*(**"bob"**, 45, 100)
    *createEmployeeProto*(**"bob"**, 45, 0)    | *createEmployee*(**"bob"**, 45, 0)
    *createEmployeeProto*(**"bob"**, 45, **null**) | *createEmployee*(**"bob"**, 45, **null**)
}
```

这怎么公平:

```
Condition not satisfied:transformer.toEmployee(input) == expected
|           |          |      |  |
|           |          |      |  Employee(name=bob, salary=45, bonus=null)
|           |          |      false
|           |          name: "bob"
|           |          salary {
|           |            currency_code: "USD"
|           |            units: 45
|           |          }
|           Employee(name=bob, salary=45, bonus=0)
com.efenglu.jprotoc.example.EmployeeTransformer@198b6731
```

我们期望得到一个空值，但得到的是 0。为什么？

## Protobuf 不允许空值

当没有设置字段时，Protobuf 将返回一个默认对象。因此，该值为 0，而不是空值。

![](img/e195f3e215ce277bae38a0dd56a3514c.png)

[https://xkcd.com/386/](https://xkcd.com/386/)

同样，本文并不打算讨论使用空值/非空值等的优点。让我们向前看，想出如何处理我们已经失去的牌。

1.  我们有一个期望空值的域模型
2.  我们有一个不允许空值的传输框架

从表面上看，这似乎是一个棘手的问题。但不完全是。虽然 protobuf 不直接支持 null 上的概念，但它确实提供了一个*has*概念。 *has* boolean 意在告诉你一个值是否已经被设置。我们可以用这个来*传送*我们的空值概念。

让我们修改代码，使用哈希函数，避免 NPE 函数。

```
**public class** EmployeeTransformerHas {

    **public** Employee toEmployee(EmployeeProto employeeProto) {
        **final** Employee.EmployeeBuilder builder = Employee.*newBuilder*()
                .name(employeeProto.getName());
        **if** (employeeProto.hasSalary()) {
            builder.salary(toBigDecimal(employeeProto.getSalary()));
        }
        **if** (employeeProto.hasBonus()) {
            builder.bonus(toBigDecimal(employeeProto.getBonus()));
        }
        **return** builder.build();
    }

    **public** EmployeeProto toEmployeeProto(Employee employee) {
        **final** EmployeeProto.Builder builder = EmployeeProto.*newBuilder*()
                .setName(employee.getName());
        **if** (employee.getSalary() != **null**) {
            builder.setSalary(toMoney(employee.getSalary()));
        }
        **if** (employee.getBonus() != **null**) {
            builder.setBonus(toMoney(employee.getBonus()));
        }
        **return** builder.build();
    }

    ...
}
```

虽然我们的测试现在通过了，但是 transformer 代码非常难看。

此外，虽然 transformer 为现有的域对象提供了帮助，但我们更希望我们的客户端能够本地使用 protobuf。一旦我们的程序员开始使用代码，他们将不得不手工编写所有这些检查。如果我们能以某种方式支持 set 上的 null 和 get 上的 null 就好了。

进入协议插件。

# 协议插件扩展点

Protoc 有一个扩展点机制，允许你在生成代码的同时插入代码。这将允许我们添加具有我们想要的行为的助手/实用工具方法。

首先是一些坏消息。

如果不直接为 protoc 编译器编辑原始 C 代码，您就不能更改任何现有的生成代码。所以不能直接把 set 方法改成接受 null，或者把 get 方法改成返回 null。

您**可以**做的是向生成的类添加新方法。比如添加一个 *setOrClear* 方法和一个 *nullGet* 方法。

对于我们的项目，我们决定添加一个 *setOrClear* 和一个 *optionalGet* 方法。

## setOrClear

为生成的接受空值的构建器提供 setOrClear 方法。

*   如果提供的值为 null，构建器将在相关字段上调用 clear。
*   如果提供的值为非空，构建器将使用提供的值在关联字段上调用 set。这避免了用空值调用 set 时通常会引发的 NPE。

示例:

```
**public** Builder setOrClearBonus(com.google.type.Money value) {
   **if** (value != **null**) {
     **return** setBonus(value);
   } **else** {
     clearBonus();
   }
   **return this**;
}
```

## optionalGet

添加一个方法 *optional{Field}* ，该方法将在一个 [java.util.Optional](https://docs.oracle.com/javase/8/docs/api/java/util/Optional.html) 对象中包装一个消息类型的所有非原始字段。

*   当*有{字段}* 返回假时，可选为空
*   当*有{Field}* 返回 true 时，可选包含调用 *get{Field}的值。*

示例:

```
**public** java.util.Optional<com.google.type.Money> optionalBonus() {
   **if** (hasBonus()) {
     **return** java.util.Optional.*of*(getBonus());
   } **else** {
     **return** java.util.Optional.*empty*();
   }
}
```

通过组合 *setOrClear* 和 *optional* ，我们觉得这是两全其美的选择。我们可以设置空值，但仍然提供非空返回类型。

## 但是…

其他一些坏消息。这种方法要求开发人员调用正确的方法。 *set* 仍然抛出 NPE，get 仍然返回一个默认值。这并不理想，但是对于那些习惯了预期的 Protobuf 行为的人来说也是有用的。

同样，这些新方法只适用于你编译的原型。所以如果你重用别人预先打包好的原型定义，你就不会得到这些新方法。

# 创建插件

我们如何编写这个神奇的插件。你可以用 C 写它，并使用一些现有的协议库代码。然而，我们是一家 Java 商店，维护 C 代码并不是一个真正的选择。

谢天谢地，protoc 实际上并不关心你的插件是用什么写的。protoc 所做的就是调用插件可执行文件，并传入一个描述我们想要生成的 protobuf 的 protobuf 字符串。

很 meta，也很酷。

Salesforce 提供了一个 maven 插件来帮助我们。这允许我们用 Java 编写我们的插件作为一个常规的 maven 工件。

完整代码库的链接在文章末尾，但是让我们先关注几个部分。

## 步骤 1:设置 maven 模块

这里没什么特别的。我们只需要一个普通的 maven jar 模块。请确保我们依赖 Salesforce jprotoc

```
<**dependency**>
    <**groupId**>com.salesforce.servicelibs</**groupId**>
    <**artifactId**>jprotoc</**artifactId**>
    <**version**>0.9.0</**version**>
</**dependency**>
```

## 步骤 2:创建您的扩展点主类

正如我们前面提到的，protoc 将所有扩展都视为可执行程序，因此我们需要创建一个具有标准主入口点的 Java 类:

```
**public class** JProtoc **extends** com.salesforce.jprotoc.Generator {

    **public static void** main(String[] args) {
        com.salesforce.jprotoc.ProtocPlugin.*generate*(**new** JProtoc());
    }
    ...
}
```

到目前为止够简单了。

接下来覆盖超级*生成*的方法:

## 产生

*   我们正在返回一个文件对象流。这些代表由协议生成的文件
*   我们通过检查要生成的原型是否在要生成的文件列表中来过滤它。您将获得所有的原型文件，甚至是导入的文件，所以一定要过滤掉这些文件。(除非您想为它们生成代码)
*   然后我们调用另一个方法在 *handleProtoFile* 中生成我们的代码

## handleProtoFile

这里没什么特别的。请注意，java 包可能来自两个不同的位置。要么隐式等于 proto 包，要么显式带有 proto 选项。

## handleMessageType

现在我们要进入正题了。

首先构建我们将要添加代码的类的文件名。

这在这里并不正确，因为我假设您使用的是多文件选项。如果没有这一点，所有的原型类和内部类都应该被区别对待。

然后，我们调用其他子方法来为我们插入的代码构建字符串。这没什么特别的，只是在小胡子模板中插入值。详情请查看完整代码。

这里真正的问题是我们如何生成文件对象。

注意我们是如何设置文件名、想要插入的内容和**插入点的。**

插入点是注释中的一个特殊字符串，它出现在协议生成的代码中。有几个不同的插入点。我们使用的是 **builder_scope** 和 **class_scope** 插入点。

示例:

```
*// @@protoc_insertion_point(class_scope:com.efenglu.protobuf.EmployeeProto)*
```

## **构建者 _ 范围**

在内部生成器类的末尾插入代码。这是我们想要添加我们的 *setOrClear* 方法的地方。

## **class_scope**

在消息类型的 java 类的末尾插入代码。这是我们想要添加我们的 *optionalGet* 方法的地方。

存在其他插入点。打开生成的 proto 文件，搜索*protocol _ insertion _ point*，看看有什么可用的。

## 第三步:使用插件

使用新插件非常容易。

使用 maven proto Buffers 插件配置 Maven 项目以构建 Proto 文件:

 [## Maven 协议缓冲插件-简介

### Maven 协议缓冲插件使用协议缓冲编译器(protoco)工具从。原型…

www.xolstice.or](https://www.xolstice.org/protobuf-maven-plugin/) 

下面是我们如何挂接新插件:

*   os-maven-plugin:帮助我们为构建系统下载正确的协议二进制文件
*   protobuf-maven-plugin:挂接协议二进制文件和我们的自定义协议插件

注意*protocol plugin*元素。这里我们已经连接了我们的协议插件。我们为插件指定 maven 工件，并在工件中指定可执行类。 [protobuf-maven-plugin](https://www.xolstice.org/protobuf-maven-plugin/compile-mojo.html) 有很好的文档更详细地举例说明了它的用法。

这样我们就有了新的自定义方法。

# 以巨大的力量…

使用这种方法，您基本上可以将任何想要的东西添加到生成的原型代码中。但是在这样做之前，有几件事情需要考虑。

*   这只添加到您的代码中
*   这不是人们所期望的
*   你能用别的方法做它吗？

我希望这有所帮助。使用 protoc plugin 框架还有几个其他的挑战，比如从 proto 文件中添加注释，处理、*映射*和*重复*类型、字段名称的冲突。但这应该能让你开始。

有关本文引用的完整源代码，请查看 Github repo:

[](https://github.com/efenglu/jprotoc) [## 埃丰卢/jprotoc

### 用 Java 编写的协议插件的示例用例。通过在…上创建一个帐户，为 efenglu/jprotoc 开发做出贡献

github.com](https://github.com/efenglu/jprotoc)