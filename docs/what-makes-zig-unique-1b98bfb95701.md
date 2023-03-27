# 是什么让 Zig 编程语言独一无二？

> 原文：<https://itnext.io/what-makes-zig-unique-1b98bfb95701?source=collection_archive---------1----------------------->

## Zig 允许您在编译期间运行代码。有什么含义？

![](img/d9472dddbb2ed08c21d4bd018b59b4c2.png)

Zig 吉祥物 Zero the Ziguana

20 世纪 60 年代，Lisp 编程语言开创了编译时计算。编译时计算意味着你以后编译的代码不仅仅是你写下来的。你后来编译的也是你的代码“写”出来的代码。代码生成代码。虽然在动态类型语言(如 Lisp 和 Julia)中这是一个常见的特性，但在静态类型系统编程语言(如 C)中却很少见。

因此，在讨论静态类型语言的编译时计算时，我决定选择 Zig 作为示例语言。为什么不是一种更广为人知的语言？Zig 非常接近 C，并且选择了以编译时计算为中心的语言。对于 C++来说，它更像是一个附加的东西。你可以很好地编写 C++代码，而无需有意识地进行编译时计算。Zig 翻转了脚本，使编译时计算成为该语言最核心、最受支持的特性之一。这就是为什么 Zig 是学习这个概念的好语言。在 Zig 世界中，我们称之为编译时计算`comptime`，来自于用来标记编译时需要运行的代码或者编译时已知的变量的关键字。

在编译期间运行 Zig 代码的能力允许 Zig 开发人员编写泛型代码和进行元编程，而不需要任何对泛型或模板的显式支持。

让我带您看一些代码示例，以便更好地解释整个想法是什么以及为什么它很重要。考虑下面的简单函数来找出两个值`a`和`b`中的最大值。如果没有泛型或`comptime`代码，我们将需要硬连接这样一个函数来操作特定的变量类型，比如在 Zig 中被称为`i32`的 32 位整数。

```
**fn** maximum(a: i32, b: i32) i32 {
    var result: i32 = undefined;

    **if** (a > b) {
        result = a;
    } **else** {
        result = b;
    }

    **return** result;
}
```

通常，Zig 中的可执行程序会有一个`main`函数，就像 C/C++程序一样。从那里我们可以调用我们的`maximum`函数。在下一个代码示例中，不要太关注我们如何获取`stdout`或者为什么我们需要用`try`关键字作为`print`函数调用的前缀。后者与 Zig 错误处理有关，我们不会在本文中涉及。

```
**pub** **fn** main() !void {
    const stdout = std.io.getStdOut().writer();

    const a = 10;
    const b = 5;

    const biggest = maximum(a, b);

    **try** stdout.print("Max of {} and {} is {}\n", 
                     .{ a, b, biggest });
}
```

显然，给出的解决方案相当有限。`maximum`只对 32 位整数进行运算。c 程序员对这个问题非常熟悉。在 C 程序员的世界里，C 预处理器宏来拯救他们。然而，安德鲁·凯利将 Zig 设计成不依赖 C 风格的宏。事实上，Zig 存在的全部原因是，Andrew 只是想用 C 语言编程，但没有像宏这样不好的部分。`comptime`的出现正是为了取代 C 宏。

让我们来看看这个问题的 Zig 解决方案。我们将在 Zig 中定义一个通用的`maxiumum`函数。`i32`类型参数将被替换为`anytype`和`@TypeOf(a)`。在调用`maximum`函数时，`anytype`将采用所提供的参数类型。请记住，我们不是在处理动态编程语言。相反，Zig 将为每种情况编译不同的`maximum`变体，其中`maximum`是用一组不同的参数类型调用的。`a`和`b`的类型仍然在编译时确定，而不是在运行时确定。

虽然可以在编译时确定输入参数的类型，但是对于变量或返回类型来说，这样做要复杂得多。您不能声明返回类型是`anytype`，因为在调用位置无法确定具体的类型。相反，我们使用一个编译器内在函数`@TypeOf`，它在编译时运行以产生返回类型。`@TypeOf(a)`在编译时评估为`a`参数的类型。我们使用同样的技巧来指定`result`变量的类型。

```
**fn** maximum(a: **anytype**, b: **anytype**) @TypeOf(a) {
    **var** result: @TypeOf(a) = undefined;

    **if** (a > b) {
        result = a;
    } **else** {
        result = b;
    }

    **return** result;
}
```

虽然这种解决方案是一种改进，但它有许多问题:

1.  没有什么能阻止你用非数字的值调用`maximum`。
2.  如果`b`是较大的值，它可能包含一个比类型`@TypeOf(a)`所能容纳的位更多的值。

为了检查`a`和`b`的类型是否正确，我们可以创建一个在编译时运行的函数来检查类型是否为数字。让我们定义一个函数`assertNumber`，用一个参数`T`代表一个类型而不是一个值。参数定义前面有关键字`comptime`，告诉编译器参数*在编译时必须已知*。

还要注意 switch-case 语句。在 Zig 中，switch-case 可以返回值。我们打开类型参数`T`。如果`T`匹配一个数字类型，switch-case 语句返回`true`，它被赋给`is_num`变量。否则，我们默认使用`else`关键字返回`false`。

```
**fn** assertNumber(comptime T: type) void {
    const is_num = **switch** (T) {
        i8, i16, i32, i64 => true,
        u8, u16, u32, u64 => true,
        comptime_int, comptime_float => true,
        f16, f32, f64 => true,
        **else** => false,
    };

    **if** (!is_num) {
        @compileError("Inputs must be numbers");
    }
}

// testing function
**pub** **fn** main() !void {
    assertNumber(bool);
}
```

对这个函数定义特别感兴趣的是编译器内部函数`@compileError`。它用于向用户发送编译器错误。在这个代码示例中，我提供了一个非数字类型作为`assertNumber`的参数。`bool`具体来说。如果您尝试编译这个程序，您将会得到以下错误消息:

```
assert-number.zig:11:9: error: Inputs must be numbers
        @compileError("Inputs must be numbers");
        ^
assert-number.zig:17:17: note: called from here
    assertNumber(bool);
                ^
assert-number.zig:16:21: note: called from here
pub fn main() !void {
```

换句话说，我们可以以这样的方式编写代码，当用户试图编译无效代码时，我们可以给用户一个有用的错误消息。

我们可以使用`assertNumber`来检查`maximum`函数的输入。为了确保返回类型足够大，我们将要求两个输入都是相同的类型。

```
**fn** maximum(a: **anytype**, b: **anytype**) @TypeOf(a) {
    const A = @TypeOf(a);
    const B = @TypeOf(b);

    assertNumber(A);
    assertNumber(B);

    var result: @TypeOf(a) = undefined;

    **if** (A != B) {
        @compileError("Inputs must be of the same type");
    }

    **if** (a > b) {
        result = a;
    } **else** {
        result = b;
    }

    **return** result;
}
```

当`maximum`在运行时被调用时，所有的编译时代码都已经运行并被它们的结果所取代。

当前的解决方案不能解决我们最初天真的解决方案的所有问题。我们被迫将`a`和`b`参数设为同一类型。如果我们希望同时允许有符号的 8 位和有符号的 32 位整数参数呢？在 Zig 中，这将是类型为`i8`和`i32`的参数。在这种情况下，我们必须确保返回类型是`i32`。我们当前的解决方案不能做到这一点。我们需要的是一个在编译时运行的函数，比较`a`和`b`的类型，并返回具有最高位长的类型。

为了实现这一点，我们将制作一些函数:

*   `nbits`计算一种类型的位数的函数`T`
*   `largestType`功能选择`A`和`B`两种类型中最大的

请注意，在下一个代码示例中，我们如何用`comptime`标记类型参数，以告诉 Zig 这些输入必须在编译时已知。我们使用`@typeInfo`编译器内部函数，它在编译时返回一个复合对象`info`，它描述了一个类型:这个类型是有符号的还是无符号的？用多少位来表示类型？

```
**fn** nbits(comptime T: type) i8 {
    **return** **switch** (@typeInfo(T)) {
        .Float => |info| info.bits,
        .Int => |info| info.bits,
        **else** => 64,
    };
}

**fn** largestType(**comptime** A: type, comptime B: type) type {
    **if** (nbits(A) > nbits(B)) {
        **return** A;
    } **else** {
        **return** B;
    }
}

**fn** maximum(a: anytype, b: anytype) largestType(@TypeOf(a),
                                               @TypeOf(b)) {
    var result: @TypeOf(a) = undefined;

    **if** (a > b) {
        result = a;
    } **else** {
        result = b;
    }

    **return** result;
}
```

上面代码示例中的 switch 语句可能不太明显。让我澄清一下。从`@typeInfo(T)`返回的类型是类型`std.builtin.TypeInfo`，它是一个*联合类型*。联合类型有点像结构。它们有多个字段，但是这些字段共享内存。因此，我们需要弄清楚哪个字段实际上正在使用。开关盒允许我们确定当前使用的是`.Int`还是`.Float`字段。Zig 使用`|info|`语法来展开值。在这种情况下，我们展开描述类型的结构。

`info`对象可以是`TypeInfo.Int`或`TypeInfo.Float`类型，但是两种`struct`类型都有一个`bits`字段。

在我们修改过的`maximum`函数中，我们没有明确指定返回值。相反，我们调用`largestType`函数，该函数返回我们想要用作`maximum`的返回类型的类型。我知道这听起来很奇怪，但它确实有效，因为 Zig 编译器可以确定`largestType`函数调用只依赖于编译时已知的信息。编译器会根据每个被调用的地方生成`maximum`的多个变体。每个版本都将使用不同的输入和输出类型进行编译。

# 使用编译时代码实现泛型

为了展示 Zig `comptime`有多强大，我将向您展示如何使用它来实现泛型。这里我们实现了一个`minimum`函数，对于习惯于泛型或基于模板的编程的开发人员来说，这个函数看起来更熟悉。一个关键的区别是类型参数`T`是作为常规参数提供的。C++、Java 和 C#开发人员会通过编写类似于`minimum<i8>(x, y)`的东西来调用这个函数，而 Zig 开发人员编写的是`minimum(i8, x, y)`。

```
**fn** minimum(**comptime** T: type, a: T, b: T) T {
    assertNumber(T);

    var result: T = undefined;
    **if** (a < b) {
        result = a;
    } **else** {
        result = b;
    }

    **return** result;
}
```

在 C++、Java、C++和 Swift 等语言中，通常可以通过查看输入参数来推断类型。使用 Zig，这样的类型推断是不可能的，因为参数`T`是作为常规参数提供的，因此不能得到特殊处理。虽然这个限制是`comptime`优于泛型的一个缺点，但好处是`comptime`在使用上更加灵活。

我们可以使用`comptime`代码来定义泛型类型。我将用一个简单的 2D 向量类来演示，它用来表示力、速度或位置等东西。

## 2D 向量类型

我们将从称为`Vector2D`的非通用复合类型的定义开始，该复合类型用于表示平面中的位移。两个字段`dx`和`dy`用于保存沿 x 轴和 y 轴的位移。每个字段的默认值都设置为零。我们得到了`sub`和`add`成员函数来处理减法和加法向量。

`format`功能很好用。您可以将此函数添加到任何结构中，以允许通过`print`函数显示结构。它给出了复合类型的默认文本表示。

```
const Vector2D = **struct** {
    dx: f64 = 0,
    dy: f64 = 0,

    **fn** sub(u: Vector2D, v: Vector2D) Vector2D {
        **return** Vector2D { 
			.dx = u.dx - v.dx,
			.dy = u.dy - v.dy
		};
    }

    **fn** add(u: Vector2D, v: Vector2D) Vector2D {
        **return** Vector2D { 
			.dx = u.dx + v.dx, 
			.dy = u.dy + v.dy 
		};
    }

    **pub** **fn** format(
        v: Vector2D,
        **comptime** _: []const u8,
        _: std.fmt.FormatOptions,
        writer: anytype,
    ) !void {
        **try** writer.print("Vector2D({}, {})", .{ v.dx, v.dy });
    }
};
```

我们可以使用这个`Vector2D`定义来定义一个`main`函数。注意我们如何使用`{}`作为`Vector2D`值的占位符。感谢`format`功能，`print`将知道在`{}`占位符处插入什么文本。

```
**pub** **fn** main() !void {
    const stdout = std.io.getStdOut().writer();

    const u = Vector2D{ .dx = 5, .dy = 4 };
    const v = Vector2D{ .dx = 10, .dy = 2 };

    **try** stdout.print("Adding u and v gives {}\n", .{u.add(v)});
    **try** stdout.print("Subtracting u from v gives {}\n", 
                     .{v.sub(u)});
}
```

运行这个程序，我们得到以下预期输出:

```
Adding u and v gives Vector2D(15, 6)
Subtracting u from v gives Vector2D(5, -2)
```

当前解决方案的问题是，它被硬连线到具有 64 位浮点位移值的 2D 向量。如果我们想使用整数或不同的位长呢？这样做需要在我们当前的解决方案中复制代码。让我们看看`comptime`如何帮助我们避免代码重复。

## 通用 2D 向量类型

为了能够为给定的字段类型`T`快速生成一个`Vector2D`类型，我们将定义一个函数`Vector2D(T: type)`，它将类型参数`T`作为输入，并返回一个在该类型上参数化的 2D 向量类型。因此，我们正在处理类型为`type`的值。保存类型的变量属于类型`type`。在编译时，Zig 允许我们返回或接受像结构这样的类型作为参数。我们在下面的代码中利用了这种能力。该函数返回一个`struct`类型定义。

```
**fn** Vector2D(**comptime** T: **type**) **type** {
    **return** struct {
        dx: T = 0,
        dy: T = 0,

        **fn** sub(u: Vector2D(T), v: Vector2D(T)) Vector2D(T) {
            return Vector2D(T) { 
				.dx = u.dx - v.dx, 
				.dy = u.dy - v.dy 
			};
        }

        **fn** add(u: Vector2D(T), v: Vector2D(T)) Vector2D(T) {
            return Vector2D(T) { 
				.dx = u.dx + v.dx, 
				.dy = u.dy + v.dy
			};
        }

        **pub** **fn** format(
            v: Vector2D(T),
            comptime _: []const u8,
            _: std.fmt.FormatOptions,
            writer: anytype,
        ) !void {
            **try** writer.print("Vector2D({d}, {d})", .{ v.dx, v.dy });
        }
    };
}
```

让我们看一个使用我们创建的`Vector2D`函数的例子。该函数允许我们按需剔除新的 2D 向量类型。在编译期间，`Vector2D(i8)`调用将被一个实际的 2D 向量类型所取代，该向量类型将`dx`和`dy`字段作为 8 位有符号整数值。因此，在代码被调用后，这些泛型类型就消失了。代码只包含具体类型。

```
**pub** **fn** main() !void {
    const stdout = std.io.getStdOut().writer();

    const u = Vector2D(i8){ .dx = 5, .dy = 4 };
    const v = Vector2D(i8){ .dx = 10, .dy = 2 };

    **try** stdout.print("Adding u and v gives {}\n", .{u.add(v)});
    **try** stdout.print("Subtracting u from v gives {}\n", 
                     .{v.sub(u)});
}
```

# 结束语

我发现 Zig `comptime`解决方案的美妙之处在于开发人员自己可以创建所有这些很酷的特性，比如特别的泛型类型。语言设计者从来不需要显式地设计语言来拥有模板。安德鲁·凯利只是想找到一种替代 C 宏的方法，而所有这些东西都是副作用。我非常喜欢动态类型语言，比如 Lisp、Julia 和 Lua。我喜欢动态语言的很多原因是我们在元编程方面获得的灵活性。多亏了`comptime`，Zig 将这一点带入了静态类型的世界。

我做了大约 15 年的 C++开发人员，尽管我使用的语言非常复杂，但我还是被迫写了这么多重复的样板代码，这一直困扰着我。具有强大功能和灵活性的简单语言一直吸引着我。Zig 做了我认为几乎不可能的事情，即在静态类型语言中实现简单、灵活和强大。Zig 是一种类似于 C 的非常低级的语言，这使得它能够具有这种程度的灵活性更加令人瞩目。

这也许是你在使用 Zig 时需要不断提醒自己的事情之一:它只是类固醇上的一种 C。你不是在用高级语言工作。Zig 没有提供闭包、自动内存管理、继承，甚至接口。它是一种基本的语言，但`comptime`经常可以与你的感知进行游戏，因为它非常强大。