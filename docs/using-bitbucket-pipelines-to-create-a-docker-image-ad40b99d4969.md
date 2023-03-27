# 使用位桶管道创建 Docker 映像

> 原文：<https://itnext.io/using-bitbucket-pipelines-to-create-a-docker-image-ad40b99d4969?source=collection_archive---------1----------------------->

使用管道创建 Docker 映像并将新创建的 Docker 映像推送到 AWS ECR 的示例。

**场景:**您正在使用一个 AWS 容器服务，并且需要构建一个 Docker 映像来部署它。

首先，我们将在项目的根目录下创建一个空白文件 bitbucket-pipelines.yml，并复制到下面的模板中。

模板细分:

*   在*图像下的第 1 行，*我们定义了每个构建步骤将使用的默认图像。
*   第 4 行在*定义下，*我们定义我们的构建步骤和它们的逻辑。
*   在*管道下的第 6 行，*我们定义何时触发我们的构建步骤。

从这里我们可以添加我们的第一个构建步骤和触发器。我们的第一个构建步骤将处理图像的构建并将其推送到 AWS ECR。接下来，我们将设置我们的触发器，这样当变更被推/合并到主分支时，它将运行构建映像步骤。

构建步骤分解(第 6 行到第 18 行):

*   第 6 行是显示在 Bitbucket 中的用户友好名称。
*   第 7 行定义了 Bitbucket 服务器上缓存的依赖项，以减少构建时间。
*   第 9 行定义了构建步骤中使用的服务，这些服务运行在独立但链接的容器中。
*   第 11 行定义了为这个构建步骤运行的命令。

构建步骤脚本分解(从第 12 行开始):

*   第 12 行是我们制作图像 URI 的地方，包括它所在的路径和标签。
*   第 14 行我们运行“aws ecr get-login”来检索带有正确凭证的 docker 登录命令，然后使用 eval 来运行返回的命令。
*   第 16 行构建 Docker 图像，同时标记它。
*   最后，第 18 行将新创建的图像推送到 AWS ECR。

此时，你可能会想等等变量 DOCKER_IMAGE_URL 和 BITBUCKET_BUILD_NUMBER 是在哪里定义的？

Bitbucket pipelines 提供了一组默认变量，这些变量以 Bitbucket 开头，这使得很容易将它们与用户定义的变量区分开来。另一方面，DOCKER_IMAGE_URL 需要在 Bitbucket 中与其他 3 个变量一起定义，这可以通过以下方式完成:

1.  转到 Bitbucket 中的存储库
2.  点击左侧第二个菜单栏中的*设置*
3.  点击管道标题下的*储存库变量*

4 个用户定义的变量:

现在，您已经完成了所有这些工作，您可以继续添加额外的构建步骤来简化构建和部署工作流。

# 奖金:

作为奖励，我包含了一个示例管道文件，它将:

*   运行 composer install(针对 PHP)。
*   建立码头工人形象。
*   上传一个豆茎神器。
*   部署到试运行和生产环境。

下面是我的示例库的链接。

[](https://github.com/zimosworld/beanstalk-single-php-container) [## zimos world/beanstalk-单 PHP-容器

### 使用 Terraform 和 Bitbucket 管道的 Beanstalk 设置示例

github.com](https://github.com/zimosworld/beanstalk-single-php-container)