# 从堆栈跟踪中删除 GOPATH

> 原文：<https://itnext.io/trim-gopath-from-stack-trace-88b7402c8b47?source=collection_archive---------4----------------------->

既然你已经开始写 Go 了，那么你很可能对栈跟踪很熟悉。

```
panic: panic from librarygoroutine 1 [running]:
github.com/aerokite/build/pkg/lib.Panic()
 /Users/mir/go/src/github.com/aerokite/build/pkg/lib/lib.go:5 +0x39
github.com/aerokite/build/pkg.Middle()
 /Users/mir/go/src/github.com/aerokite/build/pkg/pkg.go:7 +0x20
main.main()
 /Users/mir/go/src/github.com/aerokite/build/main.go:6 +0x20
```

在大多数情况下，堆栈跟踪提供了足够的信息来确定死机(在本例中)发生的位置。它显示了函数调用链和带有调用函数的行号的源文件。

```
/Users/mir/go/src/github.com/aerokite/build/pkg/lib/lib.go:5 +0x39
/Users/mir/go/src/github.com/aerokite/build/pkg/pkg.go:7 +0x20
/Users/mir/go/src/github.com/aerokite/build/main.go:6 +0x20
```

源文件路径包括您可能不想在生产中使用的`$GOPATH`。因此，在为生产构建二进制文件时，我们可以从源文件路径中删除这个`$GOPATH`部分。

![](img/95e6bf6168372a80c8c824a4c26f0a5b.png)

来源:谷歌图片

## 示例代码

```
// $GOPATH/src/github.com/aerokite/build/main.go
package main
import "github.com/aerokite/build/pkg"
func main() {
    pkg.Middle()
}// $GOPATH/src/github.com/aerokite/build/pkg/pkg.go
package pkg
import "github.com/aerokite/build/pkg/lib"
func Middle() {
    lib.Panic()
}// $GOPATH/src/github.com/aerokite/build/pkg/lib/lib.go
package lib
func Panic() {
    panic("panic from library")
}
```

构建源代码并运行

```
**$** go build -o trace .
**$** ./trace
panic: panic from librarygoroutine 1 [running]:
github.com/aerokite/build/pkg/lib.Panic()
 /Users/mir/go/src/github.com/aerokite/build/pkg/lib/lib.go:5 +0x39
github.com/aerokite/build/pkg.Middle()
 /Users/mir/go/src/github.com/aerokite/build/pkg/pkg.go:7 +0x20
main.main()
 /Users/mir/go/src/github.com/aerokite/build/main.go:6 +0x20
```

这里，堆栈跟踪显示源文件路径中的`$GOPATH`。如果你想从源文件路径中删除`$GOPATH`，重新构建如下

```
**$** go build -o trace *-gcflags "all=-trimpath=$GOPATH"* .
**$** ./trace
panic: panic from librarygoroutine 1 [running]:
github.com/aerokite/build/pkg/lib.Panic()
 src/github.com/aerokite/build/pkg/lib/lib.go:5 +0x39
github.com/aerokite/build/pkg.Middle()
 src/github.com/aerokite/build/pkg/pkg.go:7 +0x20
main.main()
 src/github.com/aerokite/build/main.go:6 +0x20
```

在这个堆栈跟踪中，`/Users/mir/go/`被从源文件路径中删除。

标志`-trimpath`用于告诉*开始编译*以从堆栈跟踪中的源文件路径修剪`$GOPATH`。

```
go tool compile --help | grep -A 1 trimpath
  -trimpath prefix
     remove prefix from recorded source file paths
```

这就是你如何构建你的二进制文件，堆栈跟踪不会在源文件路径中打印出`$GOPATH`。