# 面向 Go 程序员的 1.18 版本更新

> 原文：<https://itnext.io/version-1-18-refresh-for-go-programmers-f1b0fcfa3b4a?source=collection_archive---------0----------------------->

## 离开围棋编程有一段时间了吗？让我们看看你可能已经忘记的东西。

![](img/da6dbe2d3ccde4222b2cf00e78096ab0.png)

https://github.com/egonelbre/gophers[埃贡·厄尔布尔的艺术作品](https://github.com/egonelbre/gophers)

我在不同的语言间跳跃，当我回到一门我已经有一段时间没用过的语言时，有些东西我没有忘记，而其他的东西更容易忘记。这是你在围棋中可能会感到困惑的事情，因为它们在过去几年里发生了很大的变化。

主题故事:[刷新你的 Swift 和 Cocoa 编程技能](https://erik-engheim.medium.com/refreshing-your-swift-and-cocoa-programming-skills-f80b041fd24a)

以下是我喜欢谈论的一些话题:

*   `GOROOT`和`GOPATH`环境变量。还需要它们吗？
*   模块和包
*   命令行工具—构建、获取依赖项和运行测试
*   使用指针时的陷阱
*   记录
*   错误处理
*   枚举
*   在 Go 二进制文件中嵌入数据文件

## GOROOT 和 GOPATH 的区别，我们还需要它们吗？

出于某种原因，我总是把 Go 环境变量相互混淆，而且自从引入模块以来，我总是忘记是否需要它们。

*   `GOROOT` -你的 Go SDK 的位置。如果您安装了多个版本的 Go，这将非常有用。
*   `GOPATH` -个人工作区的根目录。在 Linux 和 macOS 上默认为`~/go`。

您可以自己设置这些变量。在我的 Mac 上`GOROOT`没有设置任何东西，Go 工作正常。你不需要`GOPATH`指向你开发代码的目录。我把我的 go 代码放在`~/Dev/go`里，但是你需要一个地方来安装第三方 Go 工具和包。这就是`GOPATH`的用武之地。我将我的`GOPATH`设置为`~/.go` in:

```
# Added to my ~/.config/fish/config.fish
set -x GOPATH $HOME/.go
```

我在前面加了一个点，让它看不见(只在 Unix 系统上有效)。原因是您很少需要进入`GOPATH`目录，因为您将自己的代码分开保存。您可以使用`tree`命令查看内容。

```
❯ tree -L 1 $GOPATH
~/.go
├── bin
├── pkg
└── src
```

您应该将`$GOPATH/bin`添加到您的`PATH`中，以确保 Go 二进制文件可以被您的 shell 发现并运行。下面是我如何配置我的 fish shell 来查找 go 二进制文件:

```
# To run home-made go commands and installed go programs
set -x PATH $PATH /usr/local/go/bin $GOPATH/bin
```

## Go v1.18 中的命令行工具和模块

Go 中模块的引入极大地改变了我们在 Go 中使用命令行工具和环境变量的方式。它降低了环境变量的重要性，如`GOPATH`。让我们看看我如何构建我的`rocket` Go 项目的不同部分。为了创建这个项目，我将执行以下操作:

```
❯ cd $GOPATH/src
❯ mkdir rocket
❯ cd rocket
❯ go mod init github.com/ordovician/rocket
```

一旦填充了源代码文件，它将类似于下面的清单:

```
❯ tree -L 2 rocket/
rocket/
├── README.md
├── go.mod
├── go.sum
├── cmd
│   ├── launcher
│   └── server
├── engine
│   ├── engine.go
│   ├── engine_example_test.go
│   ├── merlin_engine.go
│   └── rutherford_engine.go
├── math
│   └── math.go
├── physics
│   ├── equations.go
│   ├── motion.go
│   ├── rigidbody.go
│   └── rigidbody_test.go
├── propulsion.go
├── stagedrocket.go
├── stagedrocket_test.go
├── tank
│   ├── flexitank.go
│   ├── tank.go
│   └── tanks_test.go
├── types.go
└── util_test.go
```

顶层的源代码如`propulsion.go`和`stagedrocket.go`属于`rocket`包。这反映在它们在目录层次结构中的位置以及源代码中。我的`stagedrocket.go`文件的开头是这样的:

```
package rocket

import (
	. "github.com/ordovician/rocket/engine"
	. "github.com/ordovician/rocket/physics"
	. "github.com/ordovician/rocket/tank"
)

type MultiStaged struct {
	payload Rocket
	Propulsion
}
```

而`rocket`下的每个子目录定义了不同的包。下面是从`flexitank.go`文件开始的:

```
package tank

import (
	. "github.com/ordovician/rocket/physics"
)

// A tank with flexible size
type FlexiTank struct {
	DryMass    Kg
	TotalMass  Kg
	propellant Kg
}
```

模块和包之间有重叠。`github.com/ordovician/rocket`是包含`rocket`包的模块。但这不是它包含的唯一的包。例如，它还包含`physics`、`tank`和`math`包。

## cmd 中的命令行工具

`rocket/cmd`目录很有趣，因为每个目录都包含创建可执行文件的文件。这要求`main`功能存在。但是，您不能在任何包中拥有`main`功能。它需要在`main`包中。这就是为什么我们正在构建的每个命令在`rocket/cmd`下都有一个单独的目录。

## 构建包和命令行工具

我们可以用`go build`命令构建整个包、单个文件、子包或命令行工具。当创建一个模块时，你要为它指定一个名字，这个名字应该反映出你要把这个模块放到什么地方。例如，我的`rocket`模块用`go mod init`命令命名为`github.com/ordovician/rocket`,因为那是我将存储包的 git 库。因为这是模块的实际名称，所以在构建、运行或测试时通常使用它。你会写:

```
❯ go build github.com/ordovician/rocket
```

注意这个**只有当你在一个有`go.mod`文件的目录中时**才起作用，这个文件实际上定义了这个模块。这个构建命令不构建不是`rocket`的包，比如我们在`cmd`子目录中的命令行工具。要构建特定的命令，我们必须指定完整的包路径。导入 Go 包时使用的相同路径:

```
❯ go build github.com/ordovician/rocket/cmd/server
```

当然，每次都这样写完整路径可能会很尴尬。幸运的是，我们可以使用相对路径。你必须从`./`开始，因为 Go 会把`cmd/server`或`/cmd/server`当作一个包名，而不是一个本地定义的包。

```
❯ go build .              # build rocket package
❯ go build ./engine/      # build engine package
❯ go build ./cmd/server   # build server binary
```

这种方法也适用于`test`和`run`命令。让我们看看在`_test.go`文件中定义的运行测试(缩短的输出):

```
❯ go test -v ./tank
=== RUN   TestMakeMediumTank
--- PASS: TestMakeMediumTank (0.00s)
=== RUN   TestMakeLargeTank
--- PASS: TestMakeLargeTank (0.00s)
=== RUN   ExampleMediumTank_Consume
--- PASS: ExampleMediumTank_Consume (0.00s)
PASS
ok  	github.com/ordovician/rocket/tank	(cached)

❯ go test -v .
=== RUN   TestStageSeparation
--- PASS: TestStageSeparation (0.00s)
=== RUN   TestThrustTooLowForGravity
--- PASS: TestThrustTooLowForGravity (0.00s)
=== RUN   TestPlentyPowerful
--- PASS: TestPlentyPowerful (0.00s)
=== RUN   TestCompareWithNewtonMotionEquations
--- PASS: TestCompareWithNewtonMotionEquations (0.00s)
PASS
ok  	github.com/ordovician/rocket	(cached)
```

要运行单独的测试，添加`-run`标志。

```
❯ go test -v -run TestStageSeparation
=== RUN   TestStageSeparation
--- PASS: TestStageSeparation (0.00s)
PASS
ok  	github.com/ordovician/rocket	0.116s
```

这也适用于子包。

```
❯ go test -v -run TestMakeLargeTank ./tank
=== RUN   TestMakeLargeTank
--- PASS: TestMakeLargeTank (0.00s)
PASS
ok  	github.com/ordovician/rocket/tank	0.164s
```

我们可以以类似的方式使用`go run`命令。如果你在不断地改变和开发一个可执行文件，那么做一个先构建后运行的两步过程是很笨拙的。最好将编译和运行合并到一个步骤中。这里我将展示一个来自我的`cryptools`模块的例子，它包含了一些用于加密和解密的小命令。在这个例子中，我生成了一个以 base64 格式存储的 16 字节长的加密和解密密钥。

```
❯ go run ./cmd/generate -keylen 16 -encoding base64 key.txt
```

这相当于以下两步过程:

```
❯ go build ./cmd/generate
❯ ./generate -keylen 16 -encoding base64 key.txt
```

## 安装 Go 模块和工具

在过去，你可以使用`go get`来获得任何你想安装的 Go 工具。在 modern Go 中，我们只使用`go get`来安装你开发的模块的依赖项。假设你正在开发一个依赖于 GTK GUI 工具包的医疗应用程序。首先，我将创建项目:

```
❯ mkdir medical
❯ cd medical
❯ go mod init github.com/ordovician/medical
```

接下来，我添加 GTK 依赖项:

```
❯ go get github.com/gotk3/gotk3
go: downloading github.com/gotk3/gotk3 v0.6.1
go: added github.com/gotk3/gotk3 v0.6.1
```

您可以看到,`go.mod`文件被修改以包含这个依赖关系:

```
❯ cat go.mod
module github.com/ordovician/medical

go 1.18

require github.com/gotk3/gotk3 v0.6.1 // indirect
```

对于很多老的 go 项目，你会看到 README 文件说你应该写`go get`来安装它们，但是这不再适用了。相反，您使用`go install`并且您必须用一个`@`符号指定您想要的版本。使用`@latest`获取最新信息。我习惯使用 TextMate 编辑器，这意味着我需要安装`gocode`才能在我的编辑器中完成命令。我是这样做的:

```
❯ go install github.com/stamblerre/gocode@latest
```

但是你可能想安装许多其他有趣的 Go 工具。拥有一个调试器是有用的。Delve 是目前最受欢迎的围棋:

```
❯ go install github.com/go-delve/delve/cmd/dlv@latest
```

如果你想安装一个特定的版本，你可以写:

```
❯ go install github.com/go-delve/delve/cmd/dlv@v1.7.3
```

## 删除和添加所需的依赖项

您可能希望根据模块实际使用的内容来删除或添加依赖项。为此，您可以使用`go mod tidy`。它删除了任何源代码中没有使用的依赖项，实际上添加了您需要的依赖项。了解更多信息，请访问:

```
❯ go help mod tidy
```

## 使用指针的陷阱

我在休息一段时间后重新开始编程时犯的第一个错误是错误地使用指针，所以这里有一个警告和提醒来帮助你避免同样的问题。我专门建造了你们所说的竞技场分配器。这是一种快速分配相同大小的对象的方法。它对于像二叉树这样的东西很有用，因为 Go 垃圾收集器做得不如 Java。

竞技场分配器跟踪空闲内存块的列表。每当你释放一个块，它会被放回到这个列表中。这段代码使用了 Go 泛型，因此它也是对 Go 泛型的一次有益更新:

```
type Arena[T any] struct {
	blocks Stack[*T]  // NOTE: Stack is a collection I made
}

func (arena *Arena[T]) Alloc() *T {
	if arena.blocks.IsEmpty() {
		var blocks [8]T
		for i, _ := range blocks {
			arena.blocks.Push(&blocks[i])
		}
	}
	b, _ := arena.blocks.Top()
	arena.blocks.Pop()

	return b
}

func (arena *Arena[T]) Free(block *T) {
	if block == nil {
		panic("Cannot free nil pointer")
	}

	arena.blocks.Push(block)
}
```

这个变体是有效的，但是在我最初的变体中，我写的 for-loop 是错误的:

```
// DON'T do this, block will be a copy of blocks[i]
var blocks [8]T
for i, block := range blocks {
	arena.blocks.Push(&block)
}
```

这有什么不好？那不是更优雅吗？这里的问题是 Go 没有引用。`block`不是对元素`blocks[i]`的引用，而是该元素的副本。因此，`&block`将是这个副本的地址。因为每个`blocks[i]`元素在迭代时被复制到相同的内存位置，所以我最终将完全相同的地址推送到`arena.blocks`堆栈对象上。因此`Alloc`方法总是返回完全相同的对象。调试和发现每当我修改一个返回的对象时，所有其他分配的对象都会被修改，这真的令人困惑。

## 记录

作为一个经常编写 Julia 代码的人，我必须说我对登录 Go 并不感到兴奋。Julia 自带内置日志记录，它有日志记录级别，使用 LISP，就像宏一样，这使得编写日志记录代码变得非常方便。Go 的内置日志有点过于简单，而以我个人的浅见，像 [Zap](https://github.com/uber-go/zap) 这样的流行第三方日志工具并不容易设置，也不容易使用。Zap 最基本的用法很简单:

```
logger, _ := zap.NewProduction()
defer logger.Sync() // flushes buffer, if any
sugar := logger.Sugar()
sugar.Infow("failed to fetch URL",
  // Structured context as loosely typed key-value pairs.
  "url", url,
  "attempt", 3,
  "backoff", time.Second,
)
sugar.Infof("Failed to fetch URL: %s", url)
```

但是，我发现一旦我想做更复杂的事情，我不太明白代码在做什么，文档也没有很好地解释。但是对于任何认真编写大型应用程序的人来说，你应该投资学习 Zap，因为它给了你结构化的日志。结构化日志记录是以结构化的格式给出日志记录数据，因此可以很容易地被工具分析。

如果您不愿意花大量时间学习错综复杂的日志技术，那么我认为内置日志框架更好。你只需要学会如何定制它。下面是我如何用内置的 Julia 日志系统模拟日志级别。

```
var (
	DebugLog *log.Logger
	WarnLog  *log.Logger
	InfoLog  *log.Logger
	ErrorLog *log.Logger
)

// Called first in any package
func init() {
	file, err := os.OpenFile("logs.txt", os.O_APPEND|os.O_CREATE|os.O_WRONLY, 0666)
	if err != nil {
		log.Fatal(err)
	}

	flags := log.Ldate|log.Ltime|log.Lshortfile

	DebugLog = log.New(io.Discard, "DEBUG: ", flags)
	InfoLog  = log.New(io.Discard, "INFO: ", flags)
	WarnLog  = log.New(file, "WARNING: ", flags)
	ErrorLog = log.New(file, "ERROR: ", flags)
}

func main() {
	InfoLog.Println("Starting up")
    WarnLog.Println("Shutting down")
}
```

在这个设置中，我正在记录一个名为`logs.txt`的文件。创建记录器的函数如下所示:

```
func New(out io.Writer, prefix string, flag int) *Logger
```

第一个参数`out`表示将日志输出发送到哪里。你可能会注意到，对于`WarnLog`和`ErrorLog`，我将它设置为`file`，这是指我的`logs.txt`文件，而其他的都设置为`io.Discard`。`io.Discard`有点像`dev/null`，任何送到那里的东西都会消失。这是禁用日志记录的简单方法。所以在我的例子中，`InfoLog`和`DebugLog`已经被禁用。

## Go 中的错误处理

虽然在 Go 中处理错误往往只是返回一个错误对象，但是有很多关于如何做的模式，记住这些模式是很有用的。

在 Java、C#和 Python 等许多编程语言中，我们倾向于为每个错误创建全新的类型。Go 非常简单，所以 Go 中的不同错误往往只是简单的实例值。以下是一些常见输入/输出错误的定义:

```
// End of File. When you cannot read more
var EOF = errors.New("EOF")

// ErrUnexpectedEOF means that EOF was encountered in the
// middle of reading a fixed-size block or data structure.
var ErrUnexpectedEOF = errors.New("unexpected EOF")
```

请记住，`errors.New`总是返回一个新的对象，所以您必须与特定的`EOF`实例进行比较，以查看您是否到达了文件的末尾。您还需要在自定义代码中实际返回这个值。这里我从一个 TCP/IP 套接字读入一个缓冲区，并得到读取的字节数`n`和任何可能的错误`err`。因为错误可能是[嵌套对象](https://pkg.go.dev/errors)，所以应该使用`errors.Is`进行比较，而不是使用`err == io.EOF`。

```
n, err := socket.Read(buffer)
if errors.Is(err, io.EOF) {
	break
}
```

您应该熟悉[错误](https://pkg.go.dev/errors)包中的所有这些错误函数:

```
func As(err error, target any) bool
func Is(err, target error) bool
func New(text string) error
func Unwrap(err error) error
```

使用带有`%w`格式化程序的`fmt.Errorf`函数包装错误。

```
newerr := fmt.Errorf("Stuff went wrong because %w", err)
```

假设已经嵌套了几个错误对象，并且在嵌套深处的某个地方有一个`fs.PathError`错误，您希望获得导致该错误的`Path`。你可以用`As`方法做到这一点:

```
var perr *fs.PathError
if errors.As(err, &perr) {
	fmt.Println(perr.Path)
}
```

通常，您可以使用`fmt.Errorf`返回错误对象，但有时您需要创建带有特殊字段的自定义错误对象。对于匹配`error`接口的对象，它只需要实现`Error()`方法，返回描述错误的字符串。

```
type MyError struct {
	When time.Time
	What string
}

func (e MyError) Error() string {
	return fmt.Sprintf("%v: %v", e.When, e.What)
}
```

## 围棋中的枚举

与 errors 非常相似，enums 在 Go 中非常简单。事实上，并不真正支持特殊的枚举类型。相反，您需要处理的是一组关于常量的常见实践和约定。这里有一个为一些对象定义“枚举”的例子，这些对象是我在模拟 Unix 命令时创建的，这些命令在执行后必须返回一个结果。

```
//go:generate stringer -type=ExitCode

type ExitCode int

const (
	Ok      ExitCode = iota // normal result
	Failure                 // Command could not complete task
	Quit                    // request to quit shell
)
```

请注意`OK`、`Failure`和`Quit`不是被定义为简单的整数值，而是自定义的`ExitCode`类型。为什么这么做？这样，就更容易避免意外地传递超出范围的常规整数。使用`iota`这些常量得到值 0、1 和 2。除此之外，我们不要其他价值。

这不是一个万无一失的系统，因为有人可以只写`ExitCode(42)`并将其作为参数传递。就我个人而言，我不认为这是个问题。关键是不要意外地传递无效值。

## 满足纵梁接口

想要一个文本字符串版本的枚举值是很常见的。您可以将`String()`方法添加到`Exit`代码中，从而满足`Stringer`接口。

```
type Stringer interface {
	String() string
}
```

更方便的方法是使用与 Go 捆绑在一起的 [stringer 命令](https://pkg.go.dev/golang.org/x/tools/cmd/stringer)来自动生成它。一个简单的方法是:

```
❯ stringer -type=ExitCode
```

它将在当前目录中查找定义`ExitCode`类型的文件，并查看与之相关的常量以添加一个`String()`方法。它被放在`exitcode_string.go`文件中。所以只是名称的小写版本加上`_string.go`。

该命令并不总是有效。我的代码有些问题。它有助于将枚举放在一个单独的文件中，并显式指定该文件名。

```
❯ stringer -type=ExitCode
stringer: internal error: package "bytes" without types was imported from "github.com/ordovician/mainframe"

# Fix error by specifying source code file directly
❯ stringer -type=ExitCode exitcode.go
```

为每种枚举类型调用 [stringer](https://pkg.go.dev/golang.org/x/tools/cmd/stringer) 可能会让人难以理解。我们当然可以制作类似于`Makefile`的东西，但是 Go 的哲学是尽量避免使用复杂的构建脚本。Go 解决方案是使用`go generate`命令行工具。您将想要运行的命令放在源代码文件中。这就是为什么您会看到我们的枚举定义以:

```
//go:generate stringer -type=ExitCode
type ExitCode int
```

Go 惯例是以`go:`和`json:`开头的注释可以被各种 Go 工具读取。您可以通过 JSON 封送和解封看到这一点。

```
type User struct {
    Name string `json:"full_name"`
    Age int `json:"age,omitempty"`
    Active bool `json:"-"`
    lastLoginAt string
}
```

如果你从命令行运行`go generate`，它将访问所有 Go 源代码文件并运行`//go:generate`之后的命令。这样，我们可以在代码受到影响的情况下继续构建指令，并避免复杂的构建系统与要构建的源代码脱节。您可以将一个单独的源代码文件移动到另一个项目中，它将携带如何构建它需要的相关工件的指令。

## 使用非整数枚举

你的枚举不一定是数字。他们可以是任何东西。这里我用的是绳子。当它们表示来自用户或文件的字符串输入时，这是很有用的。

```
type Encoding string

const (
	PEM    Encoding = "pem"
	Hex    Encoding = "hex"
	Base32 Encoding = "base32"
	Base64 Encoding = "base64"
)
```

当然，您将需要验证您所读取的内容是否在有效范围内。

## 在 Go 二进制文件中嵌入数据文件

要制作自包含的二进制文件，您可以轻松地四处移动，将数据与它们捆绑在一起是很有用的。这通常意味着将它硬编码到源代码中，但这可能会变得混乱、丑陋和不切实际。

```
import "embed"

//go:embed data
var storage embed.FS
```

这段代码允许我从我的包源代码中的一个目录读取文件，如下所示:

```
data/
├── agents.aes
└── launchcodes.txt
```

您可以使用`storage`变量来访问嵌入的数据，就像访问普通文件一样。

```
file, _ := storage.Open("data/launchcodes.txt")
text, _ := io.ReadAll(file)
```

在以字符串形式嵌入文件的文件中，我们需要改变导入`embeded`的方式，因为如果代码中没有引用导入的包，Go 编译器会抱怨。它在注释字段中被引用不会被编译器发现。

```
import _ "embed"

//go:embed key.txt
var cryptKey string
```

破折号`_`只是让 Go 编译器闭嘴的一种方式。

# 摘要

为了便于参考，让我以更简洁的形式总结一下我在这里介绍的内容:

*   将`GOPATH`设置为`~/.go`，为已安装的第三方包和工具保留一个隐藏目录
*   保持你自己的项目在说`~/Dev/go`
*   在你的`PATH`中包含`$GOPATH/bin`，这样 Go 命令行工具就可以轻松运行。
*   `go mod init github.com/user/package` -初始化 Go 模块
*   `go mod tidy` -根据源代码使用的包添加和删除依赖项
*   `go build github.com/user/package/subpackage` -建立一个包或者`go build ./subpackage`
*   `go test -v -run TestFoobar ./subpackage` -运行子包`subpackage`中编写的特定测试`TestFoobar`
*   `go install github.com/user/tool@latest` -安装用 Go 编写的工具或命令
*   `io.Discard` -丢弃丢弃写入 writer 对象的数据。禁用记录器时很有用。
*   `//go:generate stringer -type=EnumName` -使 enum 实现`Stringer`接口
*   `//go:embed fileOrDir` -在二进制文件中嵌入文件或整个目录

下一次，我可能会尝试涵盖 Go 泛型中容易忘记或混淆的部分。许多人都知道这些特性，但是需要一个快速的提醒。