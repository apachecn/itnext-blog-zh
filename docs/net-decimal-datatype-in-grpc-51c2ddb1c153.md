# 。gRPC 中的. NET 十进制数据类型

> 原文：<https://itnext.io/net-decimal-datatype-in-grpc-51c2ddb1c153?source=collection_archive---------3----------------------->

## 中的十进制数据类型的完整工作示例的分步指南。网络 gRPC

![](img/a73f807515bee8c8a7534360e07407f2.png)

图片由[格尔德·奥特曼](https://pixabay.com/users/geralt-9301/?utm_source=link-attribution&utm_medium=referral&utm_campaign=image&utm_content=5594879)来自[皮克斯拜](https://pixabay.com/?utm_source=link-attribution&utm_medium=referral&utm_campaign=image&utm_content=5594879)

如果你查看协议缓冲区(protobuf) [语言指南，](https://developers.google.com/protocol-buffers/docs/proto#scalar)你会看到它只支持**浮点型**和**双精度型**实数型[。在大多数情况下，他们是好的。但是，如果您的部分数据包含货币值，它们就不是理想的选择，因为它们不精确。](https://en.wikipedia.org/wiki/Real_number)

```
> 3.99d * 5
19.950000000000003
```

我们预计是 19.95，但经过计算后是 19.950000000000003，这就是为什么它们对货币价值没有好处。

> **为什么我的数字，比如 0.1 + 0.2，加起来不是一个很好的整数 0.3，而是我得到一个奇怪的结果，比如 0.3000000000000004？**
> 
> 因为在内部，计算机使用的是一种格式([二进制](https://floating-point-gui.de/formats/binary/) [浮点](https://floating-point-gui.de/formats/fp/))，根本无法准确表示 0.1、0.2 或者 0.3 *这样的数字*。
> 
> 当代码被编译或解释时，你的“0.1”已经被舍入到该格式中最接近的数字，这导致一个小的[舍入误差](https://floating-point-gui.de/errors/rounding/)，甚至在计算发生之前。
> ——[https://floating-point-gui.de/basic/](https://floating-point-gui.de/basic/)

C#十进制数据类型[提供 28–29 位数精度](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/floating-point-numeric-types#characteristics-of-the-floating-point-types)。作为示例，请看下面的代码:

```
> decimal d1 = 3.99M;
> d1 * 5
19.95
```

## 在 gRPC 中为实现十进制。网

Protobuf 不支持任何类似 decimal 的东西，但是我们可以为它实现自定义类型！

> *💡对于这个例子，我在中使用 grpc 的模板项目。净 5。您可以通过在您想要的终端/命令行中执行* `*dotnet new grpc*` *来创建模板项目。*

创建一个新的`protobuf`文件，在您的 Protos 目录中将其命名为`customTypes.proto`,并将以下代码放入其中。

```
syntax = "proto3";

option csharp_namespace = "YourNameSpace";

package customTypes;

// Example: 12345.6789 -> { units = 12345, nanos = 678900000 }
message DecimalValue {

  // Whole units part of the amount
  int64 units = 1;

  // Nano units of the amount (10^-9)
  // Must be same sign as units
  sfixed32 nanos = 2;
}
```

打开项目的`csproj`文件，确保创建的文件配置如下:

```
<ItemGroup>
    <Protobuf Include="Protos\greet.proto" GrpcServices="Server"  ProtoRoot="Protos\"/>
  </ItemGroup>

  <ItemGroup>
    <Protobuf Include="Protos\customTypes.proto" GrpcServices="Both" ProtoRoot="Protos\"/>
  </ItemGroup>
```

*   💡这实际上说明了。NET 编译和生成。NET 代码从您的 Protobuf 文件。
*   属性很重要，确保它指向你保存原型文件的地方。

现在，在您想要使用十进制值的服务原型文件中，导入`customTypes.proto`并使用如下的`DecimalValue`类型:

```
syntax = "proto3";

option csharp_namespace = "YourNameSpace";

package greet;

import "customTypes.proto";

// The greeting service definition.
service Greeter {
  // Sends a greeting
  rpc SayHello (HelloRequest) returns (HelloReply);
}

// The request message containing the user's name.
message HelloRequest {
  string name = 1;
}

// The response message containing the greetings.
message HelloReply {
  string message = 1;
  customTypes.DecimalValue Value = 2;
}
```

之前我说过。NET Build 会把你的原型文件编译成。NET 代码。这个编译器的好处是它为它生成了一个分部类[,正因为如此，很容易扩展这个类型。](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/partial-classes-and-methods)

您可以在这里看到部分生成的代码:

从 Protobuf 生成的代码

最后，您需要创建一个分部类，并将其命名为我们刚刚创建的 CustomType:

```
namespace YourNameSpace
{
    public partial class DecimalValue
    {
        private const decimal NanoFactor = 1_000_000_000;
        public DecimalValue(long units, int nanos)
        {
            Units = units;
            Nanos = nanos;
        }

        public static implicit operator decimal(DecimalValue grpcDecimal)
        {
            return grpcDecimal.Units + grpcDecimal.Nanos / NanoFactor;
        }

        public static implicit operator DecimalValue(decimal value)
        {
            var units = decimal.ToInt64(value);
            var nanos = decimal.ToInt32((value - units) * NanoFactor);
            return new DecimalValue(units, nanos);
        }
    }
}
```

用这个类中定义的`implicit`运算符，。NET 会自动将`DecimalValue`转换为`Decimal`和 wise-verse。例如，在这个实例中，`3.99M * 5`的十进制值被分配给一个`DecimalValue`属性。

![](img/fa06c0725b47bcbfe5c2b82eb8dbc9e5.png)

## 结论

gRPC 和 protbuf 在性能和功能方面都非常出色，并且具有不可思议的。NET 支持，您可以更灵活地根据自己的需要扩展服务。

如果你想要这个项目的完整工作示例，请随意克隆我的回购与以下地址:

[https://github.com/0x414c49/dotnet-grpc-decimal-example](https://github.com/0x414c49/dotnet-grpc-decimal-example)