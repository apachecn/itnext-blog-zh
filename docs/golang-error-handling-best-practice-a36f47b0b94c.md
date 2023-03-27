# Golang 错误处理—2020 年最佳实践

> 原文：<https://itnext.io/golang-error-handling-best-practice-a36f47b0b94c?source=collection_archive---------0----------------------->

![](img/7a50d554f820bd10b2094b8121014f00.png)

Golang 有很多优点，它的受欢迎程度说明了这一点。但是 Go 1 中的错误处理不是很高效，我们在日常开发中要写很多冗长、不方便的代码。

有几种开源解决方案可以解决这个问题。与此同时，Go 团队正在从语言和标准库两个方面对此进行改进。

今天，我们将分析常见问题，比较解决方案，并展示目前的最佳实践(go 1.13)。

先下结论:我的偏好是`github.com/pkg/errors`。原因将在下面详细解释。

# 问题

在 Go 中编程时，我们需要检查返回的错误并处理它，一个最简单的例子是这样的:

```
import (
   "database/sql"
   "fmt"
)

func foo() error {
   return sql.ErrNoRows
}

func bar() error {
   return foo()
}

func main() {
   err := bar()
   if **err != nil** {
      fmt.Printf("got err, %+v\n", err)
   }
}
//Outputs:
// got err, sql: no rows in result set
```

有时我们需要根据不同类型的错误进行不同的处理:

```
import (
   "database/sql"
   "fmt"
)

func foo() error {
   return sql.ErrNoRows
}

func bar() error {
   return foo()
}

func main() {
   err := bar()
   if **err == sql.ErrNoRows** {
      fmt.Printf("data not found, %+v\n", err)
      return
   }
   if err != nil {
      // Unknown error
   }
}
//Outputs:
// data not found, sql: no rows in result set
```

在实践中，我们经常在错误返回之前添加额外的上下文。上下文将帮助呼叫者理解正在发生什么。例如，我们可以重写函数`foo`:

```
func foo() error {
   return fmt.Errorf("foo err, %v", sql.ErrNoRows)
}
```

那么`err == sql.ErrNoRows`条件将变为假。此外，当返回错误时，调用堆栈会被丢弃，这是最重要的诊断信息。我们需要一个更灵活的方法来处理这些问题。

# 解决方法

有几种方法可以解决这个问题。用这些库包装错误将保持根错误和完整调用堆栈的可访问性

# 1.github.com/pkg/errors

从[戴夫·切尼](https://dave.cheney.net/)开始，这个库有 3 个关键方法:

1.  `Wrap`用于包装底层错误，添加上下文文本信息，附加调用栈。一般来说，它用于包装其他人(标准库或第三方库)对 API 的调用。
2.  `WithMessage`用于向底层错误添加上下文文本信息，而不附加调用堆栈。仅对“包装错误”应用此方法。注意:不要重复`Wrap`，它会记录冗余调用栈
3.  `Cause`方法用于确定潜在误差

用`github.com/pkg/errors`重写上面的例子:

```
import (
   "database/sql"
   "fmt"

   "github.com/pkg/errors"
)

func foo() error {
   return **errors.Wrap**(sql.ErrNoRows, "foo failed")
}

func bar() error {
   return **errors.WithMessage**(foo(), "bar failed")
}

func main() {
   err := bar()
   if **errors.Cause**(err) == sql.ErrNoRows {
      fmt.Printf("data not found, **%v**\n", err)
      fmt.Printf("**%+v**\n", err)
      return
   }
   if err != nil {
      // unknown error
   }
}/*Output:data not found, bar failed: foo failed: sql: no rows in result set
sql: no rows in result set
foo failed
main.foo
    /usr/three/main.go:11
main.bar
    /usr/three/main.go:15
main.main
    /usr/three/main.go:19
runtime.main
    ...*/
```

如果我们使用`%v`作为格式参数，我们将得到一个单行输出字符串，按照调用堆栈的顺序包含所有上下文文本。如果将格式参数改为`%+v`，我们将得到完整的调用栈。

如果你想简单地用附加调用栈包装错误，不需要额外的上下文文本，那么使用`WithStack`

```
func foo() error {
   return errors.WithStack(sql.ErrNoRows)
}
```

注意:当使用`Wrap`、`WithMessage`或`WithStack`时，如果 err 参数为 nil，则返回 nil，这意味着我们在调用方法之前不需要检查`err != nil`条件。保持代码简单

# 2.golang.org/x/xerrors

在听取了来自社区的反馈后，Go 团队发布了一个[提案](https://go.googlesource.com/proposal/+/master/design/29934-error-values.md)来简化 Go 2 中的错误处理。Go 核心团队成员 [Russ Cox](https://research.swtch.com/) 在`golang.org/x/xerrors`部分实施了该提议。它用类似于`github.com/pkg/errors`的方法解决了同样的问题，引入了一个格式动词`: %w`，并使用方法`Is`来确定底层错误。

```
import (
   "database/sql"
   "fmt"

   "golang.org/x/xerrors"
)

func bar() error {
   if err := foo(); err != nil {
      return **xerrors.Errorf**("bar failed**: %w**", foo())
   }
   return nil
}

func foo() error {
   return **xerrors.Errorf**("foo failed**: %w**", sql.ErrNoRows)
}

func main() {
   err := bar()
   if **xerrors.Is**(err, sql.ErrNoRows) {
      fmt.Printf("data not found, %v\n", err)
      fmt.Printf("%+v\n", err)
      return
   }
   if err != nil {
      // unknown error
   }
}
/* Outputs:data not found, bar failed: foo failed: sql: no rows in result set
bar failed:
    main.bar
        /usr/four/main.go:12
  - foo failed:
    main.foo
        /usr/four/main.go:18
  - sql: no rows in result set
*/
```

与`github.com/pkg/errors`相比，它有几个缺点:

1.  用格式参数`: %w`替换`Wrap`。看起来简化了代码，但是这种方法**失去了编译时检查**。如果`: %w`不是格式字符串的尾部(`e.g., "foo : %w bar"`)，或者缺少冒号(`e.g., "foo %w"`)，或者冒号和百分号之间缺少空格(`e.g., "foo:%w"`)，换行会失败，不会有任何警告
2.  更严重的是打`xerrors.Errorf`之前还要检查病情`err != nil` 。这实际上根本没有简化开发人员的工作

# 3.Go 1.13 中内置的错误包装支持

从 Go 1.13 开始，`xerrors`的部分(不是全部)特性已经集成到标准库中。它继承了`xerrors`的所有缺点，贡献了一个额外的 one☹️.因此，我建议目前不要使用它

```
import (
   "database/sql"
   "errors"
   "fmt"
)

func bar() error {
   if err := foo(); err != nil {
      return **fmt.Errorf**("bar failed**: %w**", foo())
   }
   return nil
}

func foo() error {
   return **fmt.Errorf**("foo failed**: %w**", sql.ErrNoRows)
}

func main() {
   err := bar()
   if **errors.Is**(err, sql.ErrNoRows) {
      fmt.Printf("data not found,  %+v\n", err)
      return
   }
   if err != nil {
      // unknown error
   }
}
/* Outputs:
data not found,  bar failed: foo failed: sql: no rows in result set
*/
```

类似于`xerrors`版本。但是它 [**不支持调用栈输出**](https://github.com/golang/go/issues/29934#issuecomment-489682919) 。而且根据[官方说法](https://github.com/golang/go/issues/34349)来看，这个是没有时间表的。所以目前`github.com/pkg/errors`是更好的选择

# 总结

通过以上比较，我相信你已经做出了选择。让我说清楚。我的选择顺序是 1> 2> 3

1.  如果你用的是`github.com/pkg/errors`，那就留着吧。目前没有比这更好的解决方案了
2.  如果你已经大量使用`golang.org/x/xerrors`，不要匆忙切换到内置解决方案，那不值得

平心而论，围棋自诞生以来，在大多数方面已经相当成熟和健壮。在其演变过程中很少出现犹豫和摇摆。但是错误处理是个例外。

别说广受抱怨的`if err != nil`，就连它的改进路线图都是那么有争议。实际上 Go 团队[已经调整了](https://github.com/golang/go/issues/32437#issuecomment-512035919)一个[提案](https://github.com/golang/go/issues/32437)由于压倒性的反对。

幸运的是，Go 团队比以前更愿意听取社区的意见，他们甚至专门针对这个问题建立了一个[反馈页面](https://github.com/golang/go/wiki/Go2ErrorHandlingFeedback)。我相信我们最终会找到更好的解决方案

# 最后一个音符

虽然我们讨论了如何高效、优雅地包装错误，但是像其他技术一样，它应该只在适当的地方使用。不应将其视为一般原则。

为什么？当用户开始使用`errors.Cause(err, sql.ErrNoRows)`或`xerrors.Is (err, sql.ErrNoRows)`时，意味着`sql.ErrNoRows`作为一个实现细节被暴露给了外界，成为 API 的一部分。

如果你正在用一些库进行应用程序开发，这是可以接受的。

但是，如果您正在定义一些公共 API，这个问题就变得特别重要。也许更好的方法是定义一个基本的错误类型，然后从中派生出不同的错误实例，并附上错误代码

参见:[掌握钢丝](https://medium.com/@dche423/mastering-wire-f1226717bbac)