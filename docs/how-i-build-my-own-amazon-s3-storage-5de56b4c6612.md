# 我如何构建自己的亚马逊 S3 存储

> 原文：<https://itnext.io/how-i-build-my-own-amazon-s3-storage-5de56b4c6612?source=collection_archive---------0----------------------->

最近我决定把我的副业转移到 Heroku。但是 Heroku 没有提供托管媒体文件的方法。你需要使用外部存储(亚马逊 S3 或类似的)。

不幸的是，没有免费的解决方案，我的亚马逊免费等级已经过期。所以我决定使用 Python/Django 创建我自己的[亚马逊 S3 式存储](https://github.com/slyapustin/django-s3-like-storage)。

嘿，这很容易实现。

![](img/f7d2162477dc1782e54a1d7cf4511777.png)

全栈后端开发人员

## TL；DR；

这里是 [Github 库](https://github.com/slyapustin/django-s3-like-storage)。如果你想要一个生产就绪的系统——使用 [MinIO](https://min.io?utm_source=lyapustin) 。

## 计划

我想有一个应用程序，将复制基本的亚马逊 S3 功能，所以我可以在我的 [Django 项目](https://github.com/slyapustin?tab=repositories&q=django)(通过`[django-storages](https://django-storages.readthedocs.io/en/latest/backends/amazon-S3.html)`)使用它。

我不需要它很复杂，我只需要亚马逊 S3 上的一小部分:

*   文件上传
*   提供公共文件
*   能够拥有多个拥有自己证书的[桶](https://docs.aws.amazon.com/AmazonS3/latest/dev/UsingBucket.html),所以我可以为我的大多数项目使用该服务

我想通过单个`docker-compose.yml`在 Docker 中运行它，这样我就可以将它部署到我的[数字海洋](https://m.do.co/c/08ce1ee690de) 5 美元的 25Gb 存储的 droplet 中。

# 开始吧！

应用程序模板应该非常简单。我不需要 Django 管理页面之外的 UI，因为我不想以 SAAS 的形式提供该解决方案，也不想与 Amazon 竞争。

## 视图

将只有一个视图，处理所有的请求。

我的服务将只处理以下格式的请求:

`/<bucket_name>/<content-path>`

我们需要处理这些方法:

*   `PUT` —向存储器发送新文件
*   `GET` —获取文件
*   `HEAD` —检查文件是否已经存在于存储器中

## 模型

我现在有两种型号:

*   `Bucket` —定义存储桶名称和凭证
*   `Blob` —存储用户上传和一些元数据(大小、内容类型)的模型

## 验证请求

我们需要验证用户请求，因此只有拥有正确凭证的用户才能将文件上传到我们的服务中。

这是最棘手的部分，因为我们需要计算头和内容散列，并用请求中收到的来验证它们。你可以在亚马逊网站的[上查看如何用 Python 做这件事的例子。](https://docs.aws.amazon.com/general/latest/gr/sigv4-signed-request-examples.html)

有一个由大卫·库斯伯特制作的包，所以我们不需要自己做那部分。

该包使用秘密密钥为接收到的请求内容和报头计算签名，并将其与请求中提供的签名进行比较。因此，只有拥有正确密钥的用户才能上传内容。

该软件包目前缺少的一项功能是处理 HTTPS 请求，这是一个特例。有一个 [PR 可用](https://github.com/dacut/python-aws-sig/pull/3)，来解决这个问题。现在，我们将使用该包的定制版本，我们可以直接从 Github 安装它:

```
pip install git+git://github.com/slyapustin/python-aws-sig.git#egg=awssig
```

每次我们收到`PUT`请求时，我们将检查该请求是否具有有效的签名，因为桶与以下内容相关联:

我们将把所有东西都存储在 Django 的`FileField`中，并附带一些元数据，我们可能需要这些元数据来在以后适当地提供内容:

我存储内容类型，以便为图像提供正确的内容类型，这样它们就可以直接插入到 HTML 页面中:

# 部署

我想有一个 Docker 组合配置，这样我就可以将一切部署到我的[数字海洋](https://m.do.co/c/08ce1ee690de) droplet。

几个注意事项:

*   我使用一个单独的`docker.env`文件，其中包含 Docker 部署所需的所有环境变量
*   Docker 卷指向本地文件系统，所以我没有任何开销。
*   我在我的服务器上使用 Nginx 作为反向代理，这个应用程序将监听从 Nginx 发送到端口`8005`的请求。您可能有不同的配置，所以请记住这一点。

# 怎么用那个？

在您将它部署到您的服务器并使用新的凭证创建 Bucket 之后，您可以在您的 Django 应用程序中使用它，就像通常的带有`django-storages`包的 AWS S3 存储一样。

以下是您应该提供的设置:

就是这样。

## 演示

这里有一个 [Django 分类的演示](https://github.com/slyapustin/django-classified-demo)项目，它使用了那个`[Django-S3-Like-Storage](https://github.com/slyapustin/django-s3-like-storage)`。