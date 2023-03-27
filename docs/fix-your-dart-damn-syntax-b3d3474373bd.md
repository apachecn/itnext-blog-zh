# 修正你该死的 Dart 语法

> 原文：<https://itnext.io/fix-your-dart-damn-syntax-b3d3474373bd?source=collection_archive---------2----------------------->

![](img/8ca3917a54f29d569fd6b94462b097b4.png)

## 飞镖/扑击新手或假人的备忘单

当我检查其他项目时，有一件事经常困扰我，那就是我们大多数人不遵守 Dart 语法规则

我知道你可能来自另一种语言背景，但你现在使用 Dart，Dart 做一些不同的事情。

实际上，Dart 文档完美地解释了一切，但是我们大多数人都懒得去阅读整个文档。所以我决定为我们这些懒虫做一个总结。

希望对你有帮助！

## 文件夹/文件

```
lower_snake_case**NOT**
FolderName
fileName
file-name
```

## 班

```
UpperCamelCase
```

## 功能

```
lowerCamelCase
```

## 变量

```
lowerCamelCase
```

## 延长

```
UpperCamelCase
```

## 混合蛋白

```
UpperCamelCase
```

## **常数**

```
CAPITALIZE_EVERY_DAMN_LETTER // NOlowerCamelCase // yes
```

## 枚举

```
enum Name { ENUM, NAME } // WRONG!!enum Name { enum, name } // RIGHT!!
```

## 首选使用`_`、`__`等。对于未使用的回调参数常量名称

```
// IF YOU WON'T USE DON'T MENTION ITfutureOfVoid.then((unusedParameter) => print('Operation complete.'));futureOfVoid.then((_) => print('Operation complete.'));
```

## 更喜欢使用插值来构成字符串和值。

```
// GOOD BOY
'Hello, $name! You are ${year - birth} years old.';// BAD BOY
'Hello, ' + name + '! You are ' + (year - birth).toString() + ' y...';
```

## 避免使用不必要的 getters 和 setters

```
// GOOD
class Box {
  var contents;
}// BAD
class Box {
  var _contents;
  get contents => _contents;
  set contents(value) {
    _contents = value;
  }
}
```

## 到处写每一个该死的类型

```
add(a,b) => a + b; // DAMN WRONGint add(int a, int b) => a + b;  // HELL YEAHBUTfinal List<String> users = <String>[];  // THAT'S OVERKILLfinal List<String> users = []; // GREAT
final users = <String>[]; // WONDERFUL
```

## 请停止使用新的关键字，它是史前的

```
// I'm old dude
new Container();// I'm a brand new energetic open-minded sexy young dude
Container();
```

抱歉，如果我有点咄咄逼人，但请立即修复您的代码，否则我会找到你。此外，我在想，如果我及时遇到新的沉船，我会添加更多的提示，所以请小心。

# 参考

【https://dart.dev/guides/language/effective-dart 

 [## 镖用棉绒

### 该列表是根据我们的来源自动生成的。规则被组织成熟悉的规则组。此外，规则可以是…

dart-lang.github.io](https://dart-lang.github.io/linter/lints/index.html) 

# 感谢您的阅读！

如果你喜欢这篇文章，请点击👏按钮(你知道你可以升到 50 吗？)