# 如何从 GitHub 提交历史中删除敏感文件？

> 原文：<https://itnext.io/how-to-remove-sensitive-files-from-github-commit-history-638a3f291f74?source=collection_archive---------2----------------------->

![](img/7672299ce9ada4ff120240a1193443db.png)

照片由[扬西·敏](https://unsplash.com/@yancymin?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄

当我学习 CI/CD 的时候，我不小心把我的数据库配置文件提交到了我的 GitHub 帐户，让它工作。

只有在我这样做之后，我才发现环境变量可以用于像这样敏感的事情。所以我很快删除了数据库配置文件，但我仍然有一个问题。这是一个开源项目，所以每个人仍然可以在提交历史中看到我完整的配置文件。

幸运的是，有一种方法可以去除它。你还不能用 GitHub desktop 做到这一点，所以你需要使用命令行。转到您的分支并执行以下命令(确保添加您的文件路径)

```
git filter-branch --force --index-filter \"git rm --cached --ignore-unmatch INSERT_PATH_TO_FILE_HERE" \--prune-empty --tag-name-filter cat -- --all
```

一旦你有了这个，你可以在你的 GitHub 桌面上检查历史，你会看到这个文件不再存在于你的任何提交历史中。但是，你仍然会在 GitHub 网站上看到它。简单的推拉不会在这里切断它。如果您尝试这样做，您将看到以下错误: ***无法合并该存储库中不相关的历史。***

要正确执行此操作，您需要使用以下命令进行强制推送

```
git push origin --force --all
```

这就对了。您的敏感文件将从您的提交历史记录中完全删除，您再也不用担心人们会发现它们！

感谢您的阅读！

推特:@tadaspetra

YouTube:[https://www.youtube.com/tadaspetra](https://www.youtube.com/amateurcoder)