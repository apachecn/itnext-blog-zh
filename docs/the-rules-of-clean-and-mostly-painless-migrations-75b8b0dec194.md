# 干净和(大部分)无痛的数据库迁移规则

> 原文：<https://itnext.io/the-rules-of-clean-and-mostly-painless-migrations-75b8b0dec194?source=collection_archive---------2----------------------->

![](img/3df3724754567f120d8f40135351a610.png)

照片由 [K B](https://unsplash.com/@kbrembo?utm_source=medium&utm_medium=referral) 在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄

*注意:这是数据库迁移系列文章的第 2 部分，共 2 部分。如果您已经知道什么是数据库迁移，以及为什么我们使用它们来管理模式，那么您可以继续阅读第 2 部分。如果您想了解我们为什么需要迁移以及我们为什么需要针对它们的“规则”，请查看第 1 部分:“什么是数据库迁移？”*T9*。*

以下是干净且(大部分)无痛的迁移需要遵循的规则。其中有些比其他的更重要，有些人可能觉得有些没必要。我鼓励你和你的团队讨论这些，并决定你自己的一套通用规则。有一套一致同意的规则会对你的过程有很大的改善。

1.  **所有现有代码在迁移前后都必须工作。**这对于零停机部署非常重要，因为零停机部署是逐步展开的。如果您需要删除列/表或重命名它们等，应该分阶段完成。首先部署可用于新旧模式的代码。然后部署迁移以删除/重命名。然后部署只适用于新模式的代码。通过这种方式，生产应用程序不需要任何停机时间，并且应用程序可以在迁移期间继续运行。
2.  **在不同的场景中多次测试您的迁移**，以确保它们具有弹性！用最新的数据库测试它们，用空数据库测试它们，用部分更新的数据库测试它们。我建议在您的 CI/CD 和测试脚本中这样做。一旦自动化，开发人员只需要在他们的本地数据库上进行测试，然后 CI 将在其他场景中进行验证。
3.  **应尽量减少长期运行的锁定迁移，并在美国工作时间之外部署。**如果可能的话，尝试将迁移查询分解成一系列更小的查询，以避免长时间运行的表、行和列的锁。如果迁移有可能锁定表或耗尽资源(如 CPU)，请确保在工作时间之外进行部署(并记住不同的美国时区。犹他州的下午 7 点是阿拉斯加的下午 5 点，夏威夷的下午 3 点。犹他州的早上 6 点是纽约的早上 8 点)。
4.  **所有的迁移都应该是幂等的**，这意味着它们可以运行多次而不会失败。如果它们会影响模式上的 NOOP，那么它们应该报告成功，而不对模式进行任何更改。` column _ 存在？`和`表 _ 存在吗？“你最好的朋友在这里吗？
5.  **避免在代码**中使用 `SELECT *` **，更喜欢显式请求列。这减少了返回新的(意外的)列而导致代码错误的情况。它还避免了添加新的非常昂贵的列的情况，这会导致数据库发送负担，并且应用程序代码无论如何也不会使用它。**

# 常见操作的步骤

随着数据库规模和复杂性的增长，以前不是问题的东西可能会变成问题。以下提示将帮助您安全地迁移现有的大型数据库。

其中许多步骤需要在不同的迁移/部署中完成。

# 桌子

## 创建表格

如果新表将有一个指向现有表的外键，您应该分两步进行迁移:

1.  添加带有外键的表，但不要使其成为必需的
2.  添加要求

## 删除表格

1.  删除对表的所有引用(应用程序代码和视图、函数等)并部署
2.  做个数据库备份以防万一
3.  删除*其他*表中对此表的任何外键引用
4.  放下桌子

## 重命名表格

尽可能避免这种情况。如果你必须这么做，试着安排一个维护/停工期。但是，您可以通过以下方式在不停机的情况下完成它:

1.  使用正确的名称和与旧表相同的模式创建新表
2.  在旧表上创建触发器，以便在插入和更新时更新新表
3.  将所有旧表格行复制到新表格
4.  仅使用新表部署应用程序版本
5.  扔掉旧桌子

# 列

## 添加列

1.  添加没有默认值且不包含任何约束的列
2.  回填现有记录中的列值
3.  如果需要默认值，请在列上设置默认值
4.  添加您需要的任何约束

## 更改列类型

如果旧类型对新类型是强制的，您可以就地更改它，但它通常会在此期间获得一个表上的锁，因此要考虑这种影响。最好是:

1.  添加名为“${column}_new”的新列
2.  设置触发器，以便在插入或更新时将任何旧列值写入新列
3.  用旧列中的所有现有值回填新列
4.  在事务中，将旧列重命名为“${column}_old”，将新列重命名为“${column}”
5.  删除旧列

## 删除列

1.  删除对该列的所有引用(应用程序代码和视图、函数等)并部署
2.  做个数据库备份以防万一
3.  删除该列