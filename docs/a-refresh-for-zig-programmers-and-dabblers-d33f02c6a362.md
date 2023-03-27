# Zig 程序员和业余爱好者的更新

> 原文：<https://itnext.io/a-refresh-for-zig-programmers-and-dabblers-d33f02c6a362?source=collection_archive---------0----------------------->

## 重温 Zig 编程语言时容易记住的事情

![](img/cd12d88967e2374ebf40355d774abfa8.png)

大约两年前，我尝试了一下 Zig 编程语言，并写了一些关于它的文章。今天，我再次启动了一个旧的 Zig 项目，并写下我重返 Zig 的经历。在我看来，这是一个有用的测试，因为它有助于识别可能会绊倒初学者的东西，并有助于评估语言的持久力。任何一种你可以在长时间休息后不费吹灰之力就能继续使用的语言，都比一种每次你想使用时都需要大量努力和时间投入的语言更有价值。

这篇文章将是一个章节的集合，描述我不得不刷新的概念的随机分类，或者我在尝试刷新我的 Zig 知识时所面临的挑战。我假设你们中的大多数人能够很容易地理解 if 语句、while 循环和定义函数。这里的重点是各种可能会让你犯错的角落案例或概念。

## 编译和运行 Zig 程序

编译单个文件和构建整个项目之间的区别是我已经忘记的。这样做很简单。让我们看一个例子，源代码文件位于`src`子目录中。您可以通过以下方式编译或直接运行`main.zig`源代码文件:

```
❯ zig build-exe src/main.zig
❯ zig run src/main.zig
```

请记住，就像 Go 编程语言一样，大多数时候您实际上并不需要复杂的构建设置。`main.zig`文件可以指向许多其他 Zig 源代码文件，这些文件又可以指向其他文件。下面是一个将`colors.zig`文件包含在`main.zig`文件中的例子:

```
const colors =  @import("colors.zig");
```

对于像我这样的老 C/C++开发人员来说，这是相当激进的。事实上，当我第一次写这一节时，我解释了`build-exe`,就好像它只是编译一个 C/C++源代码文件，该文件以后需要与所有相关源代码文件的目标代码相链接。然而，这将是一种误解。没有单独的链接阶段，你需要与 Zig 执行。使用`build-exe`,原则上你可以构建一个由数百个源代码文件组成的完整项目。

那么，你为什么需要一个`build.zig`文件呢？好处是它允许你建立多个可执行文件，库和类似的构建。您还可以定义安装过程，并定义要编译的标志。因此，虽然您可以在很多时候避免类似于`Makefile`的事情，但是在 Zig 中使用一个定义您的构建设置的文件仍然是有益的。与 C/C++代码的 makefiles 不同，您不需要学习新的语言。`build.zig`文件包含常规 Zig 代码。它只有一个特定的要求:它必须包含一个名为`build`的函数，其签名如下:

```
const Builder = @import("std").build.Builder
pub fn build(b: *Builder) void
```

这意味着它必须是一个公共函数，接受一个指向`Builder`对象的指针，并且不返回任何内容。Zig 标准库文档更详细地解释了如何设置构建配置: [BuildMode](https://ziglang.org/documentation/0.9.1/#toc-Build-Mode) 。下面是一个简单的示例构建脚本:

```
const Builder = @import("std").build.Builder;

pub fn build(b: *Builder) void {
    const exe = b.addExecutable("example", "example.zig");
    exe.setBuildMode(b.standardReleaseOptions());
    b.default_step.dependOn(&exe.step);
}
```

一旦你得到了一个编译文件，你就可以做一些事情，比如编译可执行文件，编译并运行或者安装。

```
❯ zig build         # just build the project
❯ zig build run     # build project and run executable
❯ zig build install # install executable
❯ zig build uninstall
```

Ikrima 有一个很棒的，更详细的 Zig 构建系统指南。

## 文件是结构

关于 Zig，我忘记的第一件事是源代码文件充当结构。我在阅读一些 Zig 标准库代码时被这个问题绊倒了，这些代码使用了关键字`@This`来引用当前周围结构的类型。我们可以在`std/rand.zig`文件中看到这样的例子，这里我们有一个`SequentialPrng`类型的定义。我们用`@This()`来定义`Self`类型。

```
const SequentialPrng = struct {
    const Self = @This();
    next_value: u8,

    pub fn init() Self {
        return Self{
            .next_value = 0,
        };
    }

    pub fn random(self: *Self) Random {
        return Random.init(self, fill);
    }

    pub fn fill(self: *Self, buf: []u8) void {
        for (buf) |*b| {
            b.* = self.next_value;
        }
        self.next_value +%= 1;
    }
};
```

让我困惑的是在顶层的`std/mem/Allocator.zig`文件中找到下面一行。

```
const Allocator = @This();
```

我问自己，“周围的结构在哪里？”我开始思考一些疯狂的想法，比如在一个`struct`定义中导入文件。c 程序员有时会做类似的事情。然而，这并不是什么疯狂的事情。源代码文件中的所有顶级定义都是包含整个源代码文件的未命名结构中的字段。很容易忘记，在 Zig 中，结构无处不在，而且它们并没有真正命名。

你可能会问我为什么要在 Zig 源代码中挖掘。原因很简单，如果你想学习 Zig 编程，现在你必须习惯这一点。在我的例子中，我正在查看一个旧的代码片段，用于在我用 Zig 编写的名为 [Zacktron-33](https://github.com/ordovician/Zacktron-33) 的玩具汇编器解析器中分配一个字典来保存符号标签。

```
var labels = Dict(i16).init(allocator);
```

我想更好地理解这个字典分配是如何工作的，所以我点击了`init`函数来跳转到实现(你可以用 Zig 语言服务器 [ZLS](https://marketplace.visualstudio.com/items?itemName=AugusteRame.zls-vscode) 在 VS 代码中这样做)。这让我发现它采用了类型为`Allocator`的 at 参数，我点击它了解到它在`std/mem.zig`文件中被定义为:

```
pub const Allocator = @import("mem/Allocator.zig");
```

这意味着`Allocator`是一个在`std/mem/Allocator.zig`文件顶层保存所有定义的结构。在顶层，`Allocator.zig`文件定义了这些变量:

```
ptr: *anyopaque,
vtable: *const VTable,
```

这些就像 struct 字段一样工作，因为我们可以看到`init`函数如何返回由文件 struct(代表整个源代码文件的 struct)定义的 struct 的实例。

```
const Allocator = @This();

pub fn init(...) Allocator {   // edited out parameters
	// edit out code
    return .{
        .ptr = pointer,
        .vtable = &gen.vtable,
    };
}
```

## 函数的第一个参数得到特殊处理

注意如何将`add`函数视为`Vec2D`结构的方法。第一个参数`u`作为`self`参数。你可以看到`u.add(v)`是如何等同于`Vec2D.add(u, v)`的。

```
const std = @import("std");

const Vec2D = struct {
    dx: i32,
    dy: i32,

    fn add(u: Vec2D, v: Vec2D) Vec2D {
        return Vec2D {
            .dx = u.dx + v.dx,
            .dy = u.dy + v.dy,
        };
    }
};

pub fn main() anyerror!void {   
    const u = Vec2D { .dx = 3, .dy = 4 };
    const v = Vec2D { .dx = 2, .dy = 1 };

    const w = u.add(v);
    const z = Vec2D.add(u, v);

    std.log.info("u.add(v) == {d}", .{w});
    std.log.info("Vec2D.add(u, v) == {d}", .{z});
}
```

## Zig 没有接口

Zig 结构不能像 Go 结构那样实现接口。我注意到，就我个人而言，这个事实经常让我犯错。我努力尝试定义一个带`std.io.Writer`参数的函数，这样我就可以选择是将我编写的汇编器的输出发送给`stdout`还是发送给`std.io.getStdOut().writer()`获得的文件。我天真地写了一个这样的函数签名:

```
// Doens't work
const Writer = std.io.Writer;
fn assemble(allocator: Allocator, 
            file: File, 
            writer: Writer   // naive assumption about interfaces
   ) !void
```

这里突出的问题是`std.io.Writer`不是一个类型，而是一个函数。Zig 中的每个编写器都产生不同的类型，这意味着没有运行时方法来处理这个问题。在 Go 中通常使用接口的地方，Zig 方式是使用`anytype`类型。`std.io.Writer`函数仅用于创建表示作者的结构。但是每个结构都是不同的。您不能将它们视为某个`Writer`接口的子类型，因为 Zig 没有接口的概念。

```
fn assemble(allocator: Allocator, 
            file: File,
            writer: anytype) !void
```

编写器的类型必须在编译时已知。在这方面，Zig 感觉在某些方面有点像脚本语言。您可以通过添加各种编译时检查来检查并确保传递了正确的类型。

如果你查看 Zig 源代码，你会发现`anytime`经常被作家使用。例如，这就是如何实现`format`函数。我们可以向自定义数据类型添加一个`format`函数，以便`print`函数可以正确显示它。请注意，`writer`参数属于`anytype`类型。

```
const Vec2D = struct {
    dx: i32,
    dy: i32,

    fn add(u: Vec2D, v: Vec2D) Vec2D {
        return Vec2D {
            .dx = u.dx + v.dx,
            .dy = u.dy + v.dy,
        };
    }

    pub fn format(
        v: Vec2D,
        comptime fmt: []const u8,
        options: std.fmt.FormatOptions,
        writer: anytype,
    ) !void {
        try writer.print("Vec2D({d}, {d})", .{ v.dx, v.dy });

        // Make compiler shut up
        _ = fmt; _ = options;
    }
};
```

## Zig 中的函数参数是常量

因为当您在字典上调用`deinit`时，字典中的值和键不会自动释放，所以我尝试创建一个`releaseDict`函数来处理它。

```
const Dict = std.StringHashMap;

fn releaseDict(allocator: Allocator, dict:  Dict(i16)) void {
    var iter = dict.iterator();
    while (iter.next()) |entry|
        allocator.free(entry.key_ptr.*);
    dict.deinit();    
}
```

运行这段代码效果不好。相反，我得到了以下错误消息:

```
❯ zig test src/assembler.zig
./src/assembler.zig:213:5: error: expected type '*std.hash_map.HashMap([]const u8,i16,std.hash_map.StringContext,80)', found '*const std.hash_map.HashMap([]const u8,i16,std.hash_map.StringContext,80)'
    dict.deinit();
    ^
./src/assembler.zig:213:5: note: cast discards const qualifier
    dict.deinit();
```

如果你仔细观察，你会发现类型应该是`*HashMap`(指向`HashMap`的指针)，但是`deinit`却被赋予了一个`const *HashMap`。这种行为的原因是函数参数在 Zig 中都是作为常量传递的。来自 [Zig 语言参考](https://ziglang.org/documentation/0.9.1/#Pass-by-value-Parameters):

> 结构、联合和数组作为引用传递有时会更有效，因为根据大小的不同，副本的开销可能会很大。当这些类型作为参数传递时，Zig 可以选择复制并通过值传递，或者通过引用传递，无论哪种方式 Zig 决定会更快。**这在一定程度上是因为参数是不可变的**。

如果你看一下`deinit`的实现，你就会明白为什么这是一个问题:

```
pub fn deinit(self: *Self) void {
    self.unmanaged.deinit(self.allocator);
    self.* = undefined;
}
```

在最后一行，我们正在修改`self`指针。我们将其设置为`undefined`。如果`self`引用一个常量类型，这是不可能的，如果它是一个函数参数，就是这种情况。在 Zig 中，你经常会在其他语言使用`null`的地方使用`undefined`。指针在 Zig 中不能是`null`，除非是可选类型。在 Zig 中，我们可以将指针设置为`undefined`，但其含义与使用`null`不同。在调试模式下，未定义的数据被设置为`0xaa`，这允许 Zig 发现是否从未定义的存储器中读取。从`undefined`内存中读取是非法的。相比之下，检查变量是否为`null`是一件完全有效且合理的事情。我们可以使用`null`来终止一个链表或二叉树。然而，一旦内存被释放，你就再也不想访问那个变量了。

坦率地说，我发现`undefined`和`null`之间的区别非常聪明，我很惊讶没有更多的语言模仿这种想法。我注意到，在 Zig 中，错误地使用内存或忘记释放内存比我使用过的任何其他手动内存管理语言都要容易得多，比如 C/C++。

处理这个问题的正确方法是将`dict`作为指针传递。

```
fn releaseDict(allocator: Allocator, dict: *Dict(i16)) void {
    var iter = dict.iterator();
    while (iter.next()) |entry|
        allocator.free(entry.key_ptr.*);
    dict.deinit();    
}
```

虽然这种行为有点烦人，但我看到了参数为常量的好处。这种选择允许以最佳方式传递结构。编译器可以选择复制结构或通过引用传递结构。在 C++中，我们没有这种便利，它迫使你写类似于`const &Dict dict`的东西。对于基于模板的代码，这是次优的，因为您不知道您传递的类型有多大。如果是基元类型，比如整数，那么通过引用传递是低效的。

# Zig 资源

*   Ikrima 的游戏开发指南提供了一些很棒的 Zig 资源。
*   Zig 速成班——让有经验的程序员或任何想快速更新 Zig 的人参加。
*   Zig 中的 comp time—在 Zig 中编译时运行代码的大范围覆盖。赋予 Zig 开发者元编程能力。
*   [构建 Zig 代码](https://ikrima.dev/dev-notes/zig/zig-build/) —设置用于指定 Zig 项目的`build.zig`文件。
*   洛里斯·克罗的[编译时间代码](https://kristoff.it/blog/what-is-zig-comptime/)——包含了一个有趣的代码示例，展示了 Zig `comptime`如何用于在运行时删除一个组合的 for 循环和 switch 语句。他展示了如何将一系列操作应用于一些输入，并将其转化为一系列语句。
*   安德鲁·凯利的完美散列法 Zig 的创始人有一个使用编译时间代码创建完美散列函数的有趣例子。这是一个有点复杂的例子，但有助于真正打开你的思维，看看 Zig `comptime`功能允许你做什么。
*   Zig 接口 —更新为使用类似 Go 的胖指针。