# 颤振最佳实践—第 4 部分

> 原文：<https://itnext.io/flutter-best-practices-part-4-709e7bceabf?source=collection_archive---------1----------------------->

![](img/fb9f7f623926b5b65dcdb475e7356093.png)

颤振最佳实践—第 4 部分

*以下是成为专业 flutter 开发者的 5 个新的最佳实践*

*本系列的其他部分链接在本文末尾提供。*

# 1.对未修改的变量使用 final/const 关键字

“final”意味着单次赋值:final 变量或字段必须有一个初始值设定项。一旦赋值，最终变量的值就不能更改。final 修改变量。

如果在编译时不知道值，final 应该用在 const 之上，它会在运行时被计算/抓取。

如果你有一个 const 集合，里面的所有东西都在 const 中。如果你有一个最终的收藏，里面的一切都不是最终的。

有更多的差异和更多的理解的例子。

```
// Don’tString firstName = “John”
int a = 1// Dofinal String firstName = “John”
const int a = 1//or const a = 1
```

# 2.不要使用“+”来连接字符串，使用字符串插值

```
// Don't
final String firstName = "John";
final String text = firstName + " Doe"// Do 
final String firstName = "John";
final String text = "$firstName Doe"
```

# **3。不要创建一个 lambda，当一个撕裂将做**

如果我们有一个函数调用一个方法，这个方法的参数与传递给它的参数相同，那么您不需要手动将调用包装在 lambda 中。

```
List<String> names = []

// Don’t
names.forEach((name) {
  print(name);
});// Do
names.forEach(print);
```

# 4.使用异步/等待更可读的方式

```
// Don’t
Future<int> getUsersCount() {
  return getUsers().then((users) {
    return users?.length ?? 0;
  }).catchError((e) {
    log.error(e);
    return 0;
  });
}// Do
Future<int> getUsersCount() async {
  try {
    var users = await getActiveUser();
    return users?.length ?? 0;
  } catch (e) {
    log.error(e);
    return 0;
  }
}
```

# 5.使用 ListView.builder 创建具有相同视图的建筑列表。

*   ***Listview.builder*** 创建仅按需构建行视图的列表。
*   ***listview . builder***将屏幕外的行视图重新用于用户可见的视图。
*   ***默认列表视图*** 不会重用行并一次创建所有列表，如果列表太大，会导致性能问题。

# **前几集**

[](/flutter-best-practices-part-1-e89467ea4823) [## 颤振—最佳实践—第 1 部分

### 这是颤振最佳实践的第一部分…

itnext.io](/flutter-best-practices-part-1-e89467ea4823) [](/flutter-best-practices-part-2-e9e5c79ccb16) [## 颤振最佳实践—第 2 部分

### 让我们开始…

itnext.io](/flutter-best-practices-part-2-e9e5c79ccb16) [](/flutter-best-practices-part-3-747f1bfaec6b) [## 颤振最佳实践—第 3 部分

### 你可以在下面找到这个系列的前两部分

itnext.io](/flutter-best-practices-part-3-747f1bfaec6b) 

希望你喜欢这篇文章。请在评论中让我知道。

如果你觉得有用，请鼓掌。

感谢阅读。

[更多教程](https://coderzheaven.com/)

[视频教程](https://www.youtube.com/c/MobileProgrammer)