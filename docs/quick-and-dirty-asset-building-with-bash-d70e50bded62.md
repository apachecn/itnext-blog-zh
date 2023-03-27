# 使用 bash 快速构建脏资产

> 原文：<https://itnext.io/quick-and-dirty-asset-building-with-bash-d70e50bded62?source=collection_archive---------2----------------------->

有时，您只想发布少量的 JavaScript 和 CSS 资产，这些资产是您编写的某些软件的附件(可能是用另一种语言编写的)。我所看到的 JavaScript 构建系统没有一个是特别简单或直观的——它们都有一个痛苦的学习曲线，和/或令人困惑的文档。其中一些要求你按照他们的*方式*工作，这可能不适合你的实践。

这篇文章将带你经历一个快速而肮脏的方法，仅仅运送你的资产和它们所有的依赖关系。这将让你深入了解一个完整的构建系统是做什么的，同时让你可以马上开始你的项目，而不用去想那些东西。

如果你真的想要一个构建系统，那么我推荐你看看这篇文章，虽然已经有几年的历史了，但是它会给你一些好的建议。

# 我们需要的东西

我们将在 [bash](https://www.gnu.org/software/bash/) 中这样做，如果你在一个类似 unix 的系统上，就可以使用它。如果你用的是 Windows，我希望这些概念能被翻译，尽管你只能靠自己。

此外，我们将需要[节点](https://nodejs.org/en/)，它将允许我们在命令行上运行 JavaScript 文件。

我们还需要一个小型的。我用的是[中的`r.js`要求](http://requirejs.org/)，但是如果你喜欢的话，你也可以用其他东西，比如[丑陋的](https://github.com/mishoo/UglifyJS)。我使用`r.js`是因为它将缩小 JS 和 CSS(而且，它碰巧在内部使用了 uglify)。

# 剧本

让我们按照脚本的方式进行操作，并在操作过程中查看每一部分。

首先，声明这是一个 bash 脚本，并设置指向 Node 和`r.js`实例的变量:

```
#!/usr/bin/env bash
NODEJS="nodejs"
R="/path/to/r.js"
```

接下来声明一些我们将在接下来的脚本中使用几次的便利变量。这让我们有机会思考什么是输入和输出:

*   一个输出目录，所有已编译的资源都将在这里结束
*   您的工作代码和 CSS 的源目录
*   你自己的代码和 CSS 编译后的文件名
*   代码依赖项的 JS 和 CSS 包的文件名
*   记录一些构建信息的文件(不是必需的，但是很好的做法)

```
OUT="release"
SRC="src"
COMPILED_JS=$OUT/myapp.min.js
DEPENDENCIES_JS=$OUT/myapp.dependencies.js
COMPILED_CSS=$OUT/myapp.min.css
DEPENDENCIES_CSS=$OUT/myapp.dependencies.css
BUILD_INFO=$OUT/build.txt
```

我们的构建将分两步进行:

1.  个人资产的缩减
2.  所有资产的串联

由于这种两阶段方法，我们将需要两个中间暂存区，在连接之前存储精简的代码:

```
MINIFIED_JS=$OUT/minified_js
MINIFIED_CSS=$OUT/minified_css
```

要开始真正的工作，从清除任何旧的构建，并为新的构建重置我们的目录结构这一枯燥但必要的步骤开始:

```
rm -r $OUT
mkdir $OUT
mkdir $MINIFIED_JS
mkdir $MINIFIED_CSS
```

## 构建项目 JavaScript 源代码

可能最有趣的一点是构建您编写的实际 JS。首先，我们将整个 JS 树编译到上面创建的`MINIFIED_JS`目录中:

```
$NODEJS $R -o appDir=$SRC/js baseDir=. dir=$MINIFIED_JS
```

这个命令需要一些解包。首先，变量`$NODEJS`和`$R`指向我们的节点可执行文件和我们的`r.js`文件，它们只是要求节点执行那个脚本。`r.js`然后取 3 个参数，前缀为`-o`:

*   `appDir` —您的 JavaScript 的源目录
*   `baseDir` —指定源的基目录，应该用`.`设置为当前目录。
*   `dir` —精简代码的输出目录

当这个命令运行时，整个树将被遍历，所有文件被单独缩小，并放入`dir`中的一个相同的树结构中。

由于这个过程没有连接所有这些文件，我们需要自己做，这个过程冗长得令人讨厌。

我们将所有的源文件按照正确的顺序显式地`cat`放入最终的`COMPILED_JS`文件中，这是根据它们之间的相互依赖关系(依赖关系在列表中的位置高于依赖关系)，因此:

```
cat $MINIFIED_JS/myapp.js *<*(echo) \
    $MINIFIED_JS/extension.js *<*(echo) \
    $MINIFIED_JS/custom.js *<*(echo) \
    *>* $COMPILED_JS
```

现在你已经有了你写的 JS 文件，并且保存在一个文件中。

## 组装 JavaScript 依赖项

您的项目可能依赖于一些外部依赖项。例如，如果你像我一样，你将使用 [jQuery](https://jquery.com/) ，也许还有 [ChartJS](http://www.chartjs.org/) ，如果所有这些依赖项都可以在一个包中交付给用户，那将会很方便。我们可以使用`cat`以与上面完全相同的方式来完成:

```
cat /path/to/jquery-1.12.4.min.js *<*(echo) \
    /path/to/Chart.min.js *<*(echo) \
    *> $*DEPENDENCIES_JS
```

请注意，我们并没有试图缩小这些资产——我们已经在使用那些缩小版本的项目了。

## 构建项目 CSS

在`r.js`中，CSS 编译的工作方式与 JS 编译略有不同。例如，由于某种原因，你不能缩小整个树，你必须一个文件一个文件地缩小。我们处理每个文件的命令看起来像这样:

```
$NODEJS $R -o cssIn=$SRC/css/myapp.css out=$MINIFIED_CSS/myapp.css baseUrl=.
```

我们再次通过 Node 执行`r.js`脚本，这次我们的参数是:

*   `cssIn` —要缩小的 CSS 文件
*   `out` —结果输出文件
*   `baseUrl` —任何 URL 都将从其展开的基本 URL。我们可以在这里使用`.`。

最后，和以前一样，只需`cat`将所有生成的 CSS 文件放在一起:

```
cat $MINIFIED_CSS/myapp.css *<*(echo) \
    $MINIFIED_CSS/theme.css <(echo) \
    *>* $COMPILED_CSS
```

## 组装 CSS 依赖项

就像 JS 一样，您可能有 CSS 依赖项(比如 [Bootstrap](https://getbootstrap.com/) )，并且您会希望像捆绑 JS 依赖项一样捆绑它们。同样，这只是`cat`(确保您的文件处于正确的顺序):

```
cat /path/to/bootstrap.min.css *<*(echo) \
    /path/to/other.min.css <(echo) \
    > $DEPENDENCIES_CSS
```

## 写一些构建信息

我们已经完成了构建的实际重要部分，但是记住您最后一次运行您的构建总是一个好主意，特别是如果您正在为发布而构建。我们可以编写一个非常快速的构建信息文件，如下所示:

```
echo "Build $(date -u +"%Y-%m-%dT%H:%M:%SZ")" *>* $BUILD_INFO
```

这将在包含构建时间戳的`release`目录的根目录中输出我们的`build.txt`文件。

在本质上，JavaScript(和 CSS)资产构建系统本质上是在做这个脚本所做的事情，尽管复杂程度更高。例如，一些工具将支持将您的 JavaScript 编译成 ES 的早期版本，这种方法不包括在内。其他工具将监视您的源目录，并在文件改变时自动构建。此外，还有客户端的缓存破坏功能，更不用说整个世界的依赖性管理和项目的构建时插件了。

我的脚本并不试图处理这些事情，尝试这样做是疯狂的。如果你没有时间或者没有意愿去实现一个完整的构建系统，从阅读这里可以清楚地看到，用例是一些文件的小型库。好好享受吧！

*Richard 是软件开发咨询公司*[*Cottage Labs*](https://cottagelabs.com)*的创始人和高级合伙人，专门研究数据生命周期的各个方面。他偶尔会在推特上发*[*@ Richard _ d _ Jones*](https://twitter.com/richard_d_jones)

PS。从这篇文章中把整个文件分成几个部分复制可能会很烦人，所以这里有一个模板脚本供你使用:

```
#!/usr/bin/env bash

*# set the paths to our executables* NODEJS="nodejs"
R="/path/to/r.js"

*# define variables for the various paths we'll use later* OUT="release"
SRC="src"
COMPILED_JS=$OUT/myapp.min.js
DEPENDENCIES_JS=$OUT/myapp.dependencies.js
COMPILED_CSS=$OUT/myapp.min.css
DEPENDENCIES_CSS=$OUT/myapp.dependencies.css
BUILD_INFO=$OUT/build.txt

*# set our staging area directories* MINIFIED_JS=$OUT/minified_js
MINIFIED_CSS=$OUT/minified_css

*# tear down and re-build the output directory* rm -r $OUT
mkdir $OUT
mkdir $MINIFIED_JS
mkdir $MINIFIED_CSS

*# minify the entire js code tree* $NODEJS $R -o appDir=$SRC/js baseDir=. dir=$MINIFIED_JS

*# cat our code into the final compiled file* cat $MINIFIED_JS/file1.js *<*(echo) \
    $MINIFIED_JS/file2.js *<*(echo) \
    *> $COMPILED_JS

# cat our dependencies into a single file* cat vendor/dependency1.min.js *<*(echo) \
    vendor/dependency2.min.js *<*(echo) \
    *> $DEPENDENCIES_JS

# build our CSS files* $NODEJS $R -o cssIn=$SRC/css/file1.css out=$MINIFIED_CSS/file1.css baseUrl=.
$NODEJS $R -o cssIn=$SRC/css/file2.css out=$MINIFIED_CSS/file2.css baseUrl=.

*# cat our CSS into a single file* cat $MINIFIED_CSS/file1.css *<*(echo) \
    $MINIFIED_CSS/file2.css *<*(echo) \
    *> $COMPILED_CSS

# cat our CSS dependencies into a single file* cat vendor/dependency1.min.css *<*(echo) \
    vendor/dependency2.min.css *<*(echo) \
    *> $DEPENDENCIES_CSS* echo "Build $(date -u +"%Y-%m-%dT%H:%M:%SZ")" *> $BUILD_INFO*
```