# 学习 Dart #5:在 30 秒内读写文件

> 原文：<https://itnext.io/learn-dart-5-read-and-write-files-in-under-30-seconds-9adbf9351ef2?source=collection_archive---------0----------------------->

## 了解如何在这个紧凑的指南中快速读写文件

![](img/cb67bef26bead9fc878df8c23a0c54d7.png)

在今天的快速技巧中，我们将了解在 Dart 中读写文件是多么容易。我们将使用‘dart:io’库的`File`对象来实现这一点。

接下来，用 stagehand 生成**控制台全**项目:

```
$ mkdir my_app && cd my_app
$ **stagehand console-full**
```

查看以下视频中的解决方案:

以下是完整的解决方案:

```
**import 'dart:io';**

main() async {
  var **file** = **File**('data.txt');
  var contents;

  if (**await file.exists()**) {
    // Read file
    contents = **await file.readAsString()**;
    print(contents);

    // Write file
    var fileCopy = **await File('data-copy.txt').writeAsString(contents)**;
    print(await fileCopy.exists());
    print(await fileCopy.length());
  }
}
```

以及[源代码](http://bit.ly/2qpAXKt)。

# 扩展示例

## 1.如何更好地指定文件路径

通过将依赖项添加到您的`pubspec.yaml`文件来安装`path`包:

```
dependencies:
  path: ^1.6.2
```

并且在你的终端里运行`pub get`。

将它导入到`bin/main.dart`的顶部，并修改路径如下:

```
import 'dart:io';
**import 'package:path/path.dart';**

void main() async {
  var pathToFile = **join(dirname(Platform.script.toFilePath()), '..', 'data.txt')**;
  var file = File(pathToFile);
  ..
  ..
}
```

## 2.如何逐行读取文件

我们可以创建一个读取流来处理文件:

```
**import 'dart:convert';** // Contains the `Utf8Decoder` and `LineSplitter` stream transformers
..
..
  if (await file.exists()) {
    // Read file
    **contents = StringBuffer();**
    **var contentStream = file.openRead()**;

    **contentStream
      .transform**(Utf8Decoder())
      **.transform**(LineSplitter())
      **.listen**((String line) => **contents.write(line)**, // Add line to our StringBuffer object
        onDone: () => print(contents.toString()), // Call toString() method to receive the complete data
        onError: (e) => print('[Problems]: $e'));
    ..
    ..
  }
..
..
```

一个很好的用例是处理包含环境变量的文件，如下所示:

```
PORT=8080
API_KEY=lkjsk453lkslfkl5
API_USER=jermaineo
```

请在评论中留下您的反馈，并让我知道您希望看到接下来演示什么！

**订阅**[**Youtube 频道**](https://www.youtube.com/channel/UCHSRZk4k6e-hqIXBBM4b2iA?view_as=subscriber) **了解 Dart** 上即将推出的视频。谢谢！

喜欢，分享和关注我😍有关 Dart 的更多内容。

延伸阅读:

1.  [文件类](https://api.dartlang.org/stable/2.0.0/dart-io/File-class.html)
2.  [**免费飞镖截屏在 Egghead.io**](https://dev.to/graphicbeacon/Free%20Dart%20Screencasts%20on%20Egghead.io)