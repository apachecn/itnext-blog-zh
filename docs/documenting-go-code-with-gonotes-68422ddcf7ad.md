# 用 Gonotes 记录 Go 代码

> 原文：<https://itnext.io/documenting-go-code-with-gonotes-68422ddcf7ad?source=collection_archive---------3----------------------->

编写代码时，总是欢迎大量的文档来指导未来的读者和贡献者(包括你自己)。文档在 Go 中如此重要，以至于这种语言被设计成一种标准化的方法来编写好的代码注释，这些注释可以自动生成到文档中(参见 [Go 团队关于文档的官方博客文章](https://blog.golang.org/godoc-documenting-go-code))。

有一些惊人的工具展示了 Go 文档的威力，比如 [godoc](https://godoc.org/golang.org/x/tools/cmd/godoc) 。godoc 的核心是标准库中一个名为 [doc](https://golang.org/pkg/go/doc) 的简单包。文档库提供了以结构化方式注释代码的注释概念。最初，notes 最初是作为一种记录 bug 的机制出现的:`// BUG(r)`。如今，在文档库中，bug 注释的概念已被弃用，取而代之的是更通用的符号标记。我喜欢把这些笔记称为`gonotes`。这些注释采用标记、UID 和正文的形式:

`MARKER(uid): note body`

标记标明了评论的类别。例如，您可能有一个关于描述一个已知 bug 的注释，在这种情况下，标记应该是`BUG`。你也可以用一个标记来标记一个 TODO 项目，这个标记应该是`TODO`。在这两种情况下，UID 可以是负责跟踪该问题的用户名。注释正文包含任何相关的详细信息。

以下是 Go 标准库中的几个 godoc 注释示例:

*   `[TODO(mundaym): merge with case 24.](https://github.com/golang/go/blob/9fa988547a778540eebfe0358536b7433efe6748/src/cmd/internal/obj/s390x/asmz.go#L3083)`
*   `[TODO(rogpeppe): re-enable this test in https://go-review.googlesource.com/#/c/5910/](https://github.com/golang/go/blob/38c561cb2caa8019e44059e2be71c909ceef30a6/src/encoding/xml/marshal_test.go#L1768)`
*   `[TODO(rsc): Decide which tests are enabled by default.](https://github.com/golang/go/blob/834d2244a0150d8ae29b587ed2193e81e552d601/src/cmd/go/internal/test/test.go#L505)`
*   `[BUG(brainman): This package is not implemented on Windows. As the syslog package is frozen, Windows users are encouraged to use a package outside of the standard library. For background, see https://golang.org/issue/1108.](https://github.com/golang/go/blob/f9027d61ab48154e4cb29c50e356a3f462840e01/src/log/syslog/doc.go#L19)`
*   `[BUG(akumar): This package is not implemented on Plan 9.](https://github.com/golang/go/blob/f9027d61ab48154e4cb29c50e356a3f462840e01/src/log/syslog/doc.go#L24)`
*   `[BUG(rsc): FieldByName and related functions consider struct field names to be equal if the names are equal, even if they are unexported names originating in different packages.](https://github.com/golang/go/blob/997d7a1893ae15df1438c46487dd69903f16c57f/src/reflect/type.go#L211)`

godoc 工具支持显示笔记。要显示注释，请为命令提供注释选项:

`godoc -notes=TODO go/doc`

该命令输出:

```
use 'godoc cmd/go/doc' for documentation on the go/doc command PACKAGE DOCUMENTATIONpackage doc
    import "go/doc" Package doc extracts source code documentation from a Go AST.// YATTA YATTA YATTATODOS Recognize "Type.Method" as a name. There may be exported methods of non-exported types that can be called
   because of exported values (consts, vars, or function results) of that
   type. Could determine if that is the case and then show those methods in
   an appropriate section. fix this
```

不幸的是，godoc 工具只显示当前包的注释。在一个视图中查看嵌套包是不可能的(我认为…)。为了允许解析项目空间，我写了一个名为 notes 的[小库，递归解析目录中的每个 Go 包并返回所有的 notes。此外，该库还有一个 CLI 工具前端，可用于输出 JSON:](https://github.com/pokstad/gomate/tree/master/notes)

```
$ go get -u github.com/pokstad/gomate/notes/notes
$ notes -output json $GOPATH/src/github.com/pokstad/gomate
{"ASSET":[{"Tag":"ASSET","UID":"markdown.css","Body":"used for styling the HTML\n","Loc":{"AbsPath":"/Users/paulokstad/go/src/github.com/pokstad/gomate/cmd/gomate/main.go","Line":23,"Column":2,"RelPath":"/Users/paulokstad/go/src/github.com/pokstad/gomate/cmd/gomate/main.go","Filename":"main.go","Excerpt":"ASSET(markdown.css): used for styling the HTML\n"}}],"TODO":[{"Tag":"TODO","UID":"1","Body":"add more tests using golden files\n","Loc":{"AbsPath":"/Users/paulokstad/go/src/github.com/pokstad/gomate/html/getdoc_test.go","Line":13,"Column":1,"RelPath":"/Users/paulokstad/go/src/github.com/pokstad/gomate/html/getdoc_test.go","Filename":"getdoc_test.go","Excerpt":"TODO(1): add more tests using golden files\n"}}]}
```

JSON 的格式非常漂亮，如下所示:

```
{
  "ASSET": [
    {
      "Tag": "ASSET",
      "UID": "markdown.css",
      "Body": "used for styling the HTML\n",
      "Loc": {
        "AbsPath": "/Users/paulokstad/go/src/github.com/pokstad/gomate/cmd/gomate/main.go",
        "Line": 23,
        "Column": 2,
        "RelPath": "/Users/paulokstad/go/src/github.com/pokstad/gomate/cmd/gomate/main.go",
        "Filename": "main.go",
        "Excerpt": "ASSET(markdown.css): used for styling the HTML\n"
      }
    }
  ],
  "TODO": [
    {
      "Tag": "TODO",
      "UID": "1",
      "Body": "add more tests using golden files\n",
      "Loc": {
        "AbsPath": "/Users/paulokstad/go/src/github.com/pokstad/gomate/html/getdoc_test.go",
        "Line": 13,
        "Column": 1,
        "RelPath": "/Users/paulokstad/go/src/github.com/pokstad/gomate/html/getdoc_test.go",
        "Filename": "getdoc_test.go",
        "Excerpt": "TODO(1): add more tests using golden files\n"
      }
    }
  ]
}
```

尝试一下，让我知道你的想法。

[*这篇文章最初出现在我的博客这里。*](http://pokstad.com/2018/07/26/gonotes.html)