# 内联 if 语句的副作用

> 原文：<https://itnext.io/side-effects-of-inline-if-statements-17cf073c78a3?source=collection_archive---------5----------------------->

![](img/1984734f37ad429914037a7d02d50580.png)

照片由[克莱门特 H](https://unsplash.com/@clemhlrdt?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 在 [Unsplash](https://unsplash.com/s/photos/code?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄

代码格式化程序防止围绕代码风格的争论，帮助将拉/合并请求限制在有意义的代码注释中。团队既可以就格式配置达成一致，也可以使用像`prettier`这样非常固执己见的工具，这样就没有什么选项可以解释了。

虽然本文关注的是内联`if`语句，但是同样的要点也适用于其他缺失的规则。

## 内嵌示例

```
if (!isAllowed) throw new Error("You do not have permission!");
```

## 嵌套示例

```
if (!isAllowed) {
  throw new Error("You do not have permission!");
}
```

## 副作用

*   在一些调试器中放置断点是困难的或者不可能的。开发人员可以转换为 block 语句(如果不重新加载/重新编译应用程序，这可能并不总是可行)，在条件之外命中断点，或者创建条件断点——所有这些都是乏味的选项。
*   **在块中添加新的代码行很烦人。**如果添加的代码是临时的，有一个额外的步骤来恢复大括号。

```
if (!isAllowed) {
  console.log('User:', user.name);
  throw new Error("You do not have permission!");
}
```

*   代码更难阅读。

> “你的拇指会学习。”—史蒂夫·乔布斯

*这句话是对一名记者抱怨 iPhone 上的无键(数字)键盘的回应。*

同样的理念也适用于可读代码。当阅读格式一致的知识库时，大脑最终会发现它是可读的，不管其风格是否独特。

如果代码库的其余部分没有内联语句，开发人员可能需要暂停来解析这种偏差。

# 结论

对同事好一点，不要偏离存储库其他部分的风格。