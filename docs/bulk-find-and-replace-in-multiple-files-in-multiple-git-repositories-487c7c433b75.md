# 在多个 Git 存储库中的多个文件中批量查找和替换

> 原文：<https://itnext.io/bulk-find-and-replace-in-multiple-files-in-multiple-git-repositories-487c7c433b75?source=collection_archive---------5----------------------->

![](img/2cd5a488054e985aa71ca41c40fe7950.png)

上周，我需要为多种环境在多个 Git 存储库中更改我们的 AWS CloudFormation 模板中的一些配置。(有时候我们需要做类似那样的改变。)所以我创建了一个简单的 Python 脚本来进行这些更改。

[](https://github.com/omerkarabacak/bulk-find-and-replace-in-git-repositories) [## omerkarabacak/在 git 中批量查找并替换存储库

### 当您想要批量更改多个 git 存储库中多个文件的多个部分时，这个脚本非常有用…

github.com](https://github.com/omerkarabacak/bulk-find-and-replace-in-git-repositories) 

当您想要批量更改多个 git 存储库中多个文件的多个部分时，这个脚本非常有用。脚本从给定的基本分支创建一个新分支，并将更改提交到其中。也许推远程也可以添加一个小的变化。

在我们的项目中，我使用这个脚本来为多个微服务更改 CloudFormation 模板中的自动缩放配置。

# 如何运行脚本？

第一个克隆存储库

```
git clone [https://github.com/omerkarabacak/bulk-find-and-replace-in-git-repositories.git](https://github.com/omerkarabacak/bulk-find-and-replace-in-git-repositories.git)
```

# 跑步前要做什么？

在 config.json 文件中，更改这些；

*   在 **repository_list** 中添加新的分支名称和存储库
*   添加文本以查找和替换**查找 _ 和 _ 替换 _ 列表**
*   在**文件列表**中添加要检查的文件
*   更改提交消息**提交消息**
*   改变基础分支**基础分支**
*   如果需要，更改存储库目录**存储库 _ 目录**
*   更改作者姓名 **repository_author.name**
*   更改作者电子邮件**repository _ author . email**
*   如果你需要特定的东西，改变 SSH 密钥路径 **ssh_key**

你已经准备好了！

# 如何用 Docker 运行？

有两种选择。您可以使用 Dockerfile 构建您的 Docker 映像，或者使用 Docker Hub 中现成的公共 Docker 映像。

## 选项 1:构建自己的 Docker 映像并使用它

构建 Docker 映像:

```
docker build -t bulk-find-and-replace-in-git-repositories:1.0 .
```

使用构建的本地 Docker 映像运行:

```
docker run --rm -v $(pwd)/config.json:/app/config.json -v $(pwd)/repositories:/app/repositories -v ~/.ssh/id_rsa:/root/.ssh/id_rsa bulk-find-and-replace-in-git-repositories:1.0
```

## 选项 2:使用 Docker Hub 上托管的公共 Docker 映像

使用公共 Docker 图像运行:

```
docker run --rm -v $(pwd)/config.json:/app/config.json -v $(pwd)/repositories:/app/repositories -v ~/.ssh/id_rsa:/root/.ssh/id_rsa omerkarabacak/bulk-find-and-replace-in-git-repositories:1.0
```

# 如何用 Python 虚拟环境运行？

首先创建虚拟环境

```
python3 -m venv env
```

进入虚拟环境

```
source env/bin/activate
```

安装依赖项

```
pip install -r requirements.txt
```

运行脚本

```
python findandreplace.py
```

## 哪个 Python 版本？

用 3.6.8 测试

# **更新一:**

我已经修改了脚本，使其更具弹性，并将配置部分移到了 json 文件中，还添加了 Dockerfile 作为 Docker 映像运行。请查看存储库中更新的 README.md。