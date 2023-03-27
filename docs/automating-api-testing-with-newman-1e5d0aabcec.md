# 用 Newman 自动化 API 测试

> 原文：<https://itnext.io/automating-api-testing-with-newman-1e5d0aabcec?source=collection_archive---------1----------------------->

![](img/b7d5af4a8b092fa11f733a2eec64007e.png)

米克·豪普特在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上的照片

目前，我正在开发的产品有一个传统的质量保证(QA)团队，当工程师完成他们的工作后，它会进入 QA 队列等待..有些时候任何人都不喜欢太长的时间，但这就是我们今天的过程。

我们一直在寻找加快向我们的平台交付特性和错误修复的方法。在过去的 6 个月里，我花了太多的时间试图弄清楚别人是如何做到一个月/一年发布成百上千个版本的。在那段时间里，我不认为我读过一篇关于团队如何实现高速度的文章，他们也有一个传统的 QA 周期，包括人工审查。

这是我们追求速度的一大进步！

# 你好，纽曼

后来有一天，我的一个同事发给我一个链接，链接指向[纽曼](https://github.com/postmanlabs/newman) GitHub 回购。纽曼是一个 CLI 工具，允许您运行邮差收集。它被构建为集成到 CI/CD 或任何其他您想要的脚本中。

许多团队成员已经使用 Postman 进行了一些测试，所以它看起来很适合。现在我们所需要的是邮差收集我们在平台中的每一个微服务。

顺便说一句，我不知道这是不是故意的，但如果你记得《宋飞正传》中的纽曼，是的，他实际上是一名邮递员！

# 邮递员收藏

我为平台的每一个 API 创建了一个集合，并进行适当的测试和数据验证。由于没有 Swagger/OpenAPI 文档，我们的许多 API 都花了一些时间。然而，对于我们在 Spring Boot 的新项目，我只需将 [Postman 的导入指向 OpenAPI](https://blog.postman.com/openapi-specification-postman-how-to/) docs URL，Postman 就会自动创建集合。当然，自动创建并没有增加测试，这个过程仍然需要完成。

请记住，在创建您的集合时，您可以使用变量，它们可以通过 CLI 传递给 Newman。

一旦您通过充分的测试创建了所有您想要的收藏，您将希望[将它们导出](https://learning.postman.com/docs/getting-started/importing-and-exporting-data/)为 v2.1 收藏(单击收藏名称旁边的 3 个点并选择导出)或通过 Postman's cloud 提供它们，以便您可以引用 Newman 的 URL。我选择了出口。

# 本地测试

你可以在本地机器上安装 NPM 和纽曼，或者直接跳到 docker 容器。我个人选择了一个容器，因为我知道我想要

如果你首先从码头路线开始，你需要一个纽曼码头形象。我使用了邮差团队、`postman/newman:alpine`发布的图片[，效果很好。这将允许您计算出通过 CI/CD 运行时需要的确切命令行参数。](https://github.com/postmanlabs/newman/tree/develop/docker/)

[纽曼医生](https://github.com/postmanlabs/newman#newman-run-collection-file-source-options)在这里很有帮助！我最终使用了全局变量和环境变量的组合。我开始考虑环境变量，认为它们是系统环境变量，非常适合通过 CI/CD 使用；然而，他们**不是**他们是邮递员环境，这是完全不同的！

我的集合需要登录到一个内部 auth 服务，并获得一个用于访问 API 的不记名令牌，所以我将用户名和密码作为全局变量传入，还传入了要运行的 URL。目前，在新容器部署到 k8s 后，我们会针对每个环境运行。

我最终决定与 Newman 一起进行本地测试的命令是:

`newman run --env-var "url=$URL" --global-var "UNAME=$API_USERNAME" --global-var "PASS=$API_PASS" <collection export name>.postman_collection.json`

# GitLab CI/CD 集成

手动运行很好，但是通过 CI/CD 管道自动运行更好！

由于我们想要运行大约 15 个不同的 api，并且每个 API 都在各自的项目中，所以我在 GitLab 中创建了一个 api_automation 项目，它将包含所有的 Postman 集合和运行它的 CI/CD。这样每个项目都可以触发 api_automation 项目，事情就会运行起来。

我在 GitLab 中使用 CI/CD 变量作为用户名/密码，存储在更高的级别，以供他人有限查看。

Newman 还能够将它的发现记录为 Junit 测试。这与 GitLab 配合得非常好，因此您可以在管道的“测试”部分看到它们。

我为我的 CI/CD 项目使用一个模板，所以我将下面的内容添加到我的模板中。我包括了整个部分，这样您可以看到我是如何在多种环境中使用它的:

```
.e2e: &e2e
  stage: test    # It only starts when the job in the build stage completes successfully.
  script:
    - collection=`echo $API_IMAGE_TAG|awk -F/ '{print $NF}'`
    - echo $collection
    - python3 /opt/gosecure/rancher.py --collection=$collection --new_version=$API_IMAGE_VER
    - newman run --env-var "url=$NEWMAN_URL" --global-var "UNAME=$API_USERNAME" --global-var "PASS=$API_PASS" --global-var $collection.postman_collection.json --reporters junit,cli --reporter-junit-export=./myreport.xml
  artifacts:
    reports:
      junit:
        - ./myreport.xml

end2end-test-dev:
  extends: .e2e
  variables:
    NEWMAN_URL: https://devinstance.dev.company.com/api
  rules:
    - if: $NEWMAN_ENV == "DEV"
      when: always
    - when: never

end2end-test-job:qa:
  extends: .e2e
  variables:
    NEWMAN_URL: https://qainstance.dev.company.com/api
  rules:
    - if: $NEWMAN_ENV == "QA"
      when: always
    - when: never
```

# 确保容器已部署

我立即发现的一个挑战是，在开始 API 收集之前，我需要确保新容器已经部署。我们使用牧场主为我们的 Kubernetes 编排。Rancher 有一个非常健壮的 API，文档不多，绝对的 API！

如果您注意到 CI 中有一个 python3 rancher.py 调用，这将确保在开始 Newman 作业之前实际部署了新容器。

我将这个脚本和 Python 一起包含在我的 Newman 容器中。是的，我知道，我知道把 Python 添加到一个节点容器是不好的，但是嘿，我很擅长 Python，不知道 JavaScript，所以我们在这里。

该脚本实际上非常简单，因为它只是轮询 Rancher API 并确保容器是“活动的”。我将新的容器版本和容器名称传递给脚本，然后它轮询 API，看是否有正确的容器版本正在运行并处于活动状态。

需要注意的一点是，如果您正在使用 Rancher，并且想要为这样的脚本设置一个 API 键，您将需要创建未划分的 API 键。Rancher 2.5 中有一个错误，如果您试图将 API 键的范围限定为单个集群，将使所有 API 调用返回无效。

Rancher 中另一个非常有用的东西是使用 UI 中的“在 API 中查看”选项。这将向您显示实际的 API 返回结果。如果您单击右上角的“编辑”按钮，您可以看到运行的 CURL 命令。这比牧场医生有用多了！

下面是 Rancher.py 脚本。

```
RANCHER_URL = "https://rancher.dev.company.com/v3/"

import requests
from requests.models import HTTPBasicAuth
import os
import time
import argparse

RANCHER_USER = "TOKEN HERE"
RANCHER_ACCESS = "KEY HERE"
HEADERS = {
    "Accept": 'application/json',
    'Content-Type': 'application/json'
}

login = requests.put(
    RANCHER_URL,
    auth= HTTPBasicAuth(RANCHER_USER,RANCHER_ACCESS)
)

def get_container_ver(container, version):
    time.sleep(5)
    try:
        results = requests.get(
            RANCHER_URL + "projects/XXXX:XXX/workloads/deployment:dev:{}".format(container),
            auth= HTTPBasicAuth(RANCHER_USER,RANCHER_ACCESS)
        ).json()
        print(results['state'])
        if results['state'] == "active":
            for cont in results['containers']:
                if cont['image'].split(':')[-1] != version:
                    return True
            return False
        else:
            return True
    except:
        return False

parser = argparse.ArgumentParser()
parser.add_argument("--container",dest='API_IMAGE', help = "The container to verify", required=False)
parser.add_argument("--collection",dest='cont', help = "The container to verify", required=True)
parser.add_argument("--new_version",dest='API_VER', help = "The new version to wait for", required=True)
args = parser.parse_args()

while get_container_ver(args.cont,args.API_VER):
    print("Container Not updated")
    time.sleep(10)
exit(0)
```

# 现在我们有了自动化的 API 测试

如果在开发人员构建并部署到我们的开发环境之后，有什么地方不正常，这可以立即告诉开发人员。我们将样本数据加载到我们所有的系统中，这样一组测试就可以用于任何地方的 API。

这种自动化已经极大地帮助了团队。现在我们的 QA 团队正在维护和改进 Postman 系列。这对我们来说是双赢！

希望你觉得这很有用，如果你有任何问题，请让我知道，如果你喜欢这篇文章，请鼓掌并分享！