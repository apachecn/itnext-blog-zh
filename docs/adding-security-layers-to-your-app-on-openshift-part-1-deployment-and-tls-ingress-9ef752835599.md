# 在 OpenShift 上为您的应用添加安全层—第 1 部分:部署和 TLS 入口

> 原文：<https://itnext.io/adding-security-layers-to-your-app-on-openshift-part-1-deployment-and-tls-ingress-9ef752835599?source=collection_archive---------1----------------------->

今天，保护你的应用程序是必须的，但是如果你没有在一开始就考虑它，那么在不修改代码的情况下引入它是很困难的。幸运的是，像 [Kubernetes](http://kubernetes.io) 这样的容器和平台带来的新的云原生模式提供了简单的方法来解决安全问题，而无需接触代码。

在这个博客系列中，我们将经历许多阶段来保护现有的`fruits-catalog`应用程序。一步一步来，我们将添加新的安全层，以实现最先进的安全应用。我们将使用 [OpenShift](http://www.openshift.com) 、 [Keycloak](http://www.keycloak.org) 和 [Hashicorp Vault](http://vaultproject.io) 的高级功能来增强不同的安全方面，并且只通过修改配置或添加新代码的轻量级部分(不修改现有代码)。

![](img/bcf6ac4486caa824e59fffd7e1ed83cd.png)

第 1 部分—在 OpenShift 上部署 TLS 入口

这篇文章是一个更大的系列文章的一部分，讲述了如何给现有的应用程序增加安全层。目前公布的其他部分有:

*   *第 1 部分——部署和 TLS 入口(本文)*
*   [*第二部分——使用 Keycloak 的认证和授权*](https://medium.com/@lbroudoux/adding-security-layers-to-your-app-on-openshift-part-2-8320018bcdd1)
*   [*第三部分——金库秘密管理*](https://medium.com/@lbroudoux/adding-security-layers-to-your-app-on-openshift-part-3-secret-management-with-vault-8efd4ec29ec4)
*   [*第四部分——带金库的动态秘密*](https://medium.com/@lbroudoux/adding-security-layers-to-your-app-on-openshift-part-4-dynamic-secrets-with-vault-b5fe1fc7709b)
*   [*第 5 部分——与 Istio*](https://medium.com/@lbroudoux/adding-security-layers-to-your-app-on-openshift-part-5-mutual-tls-with-istio-a8800c2e4df4) 的相互 TLS
*   [*第 6 部分—使用保管库和证书管理器的 PKI 服务*](https://medium.com/@lbroudoux/adding-security-layers-to-your-app-on-openshift-part-6-pki-as-a-service-with-vault-and-cert-e6dbbe7028c7)

每篇文章都遵循相同的结构以保持易懂:我们想要达到什么？如何应用？怎么查？它是如何工作的？在结束之前。这个博客系列的源代码和技术资源位于我的 GitHub 库:[https://github.com/lbroudoux/secured-fruits-catalog-k8s](https://github.com/lbroudoux/secured-fruits-catalog-k8s)

# 介绍

`fruits-catalog`应用程序是一个非常简单的应用程序，由 3 个组件组成:一个 [Angular](http://angular.io) 前端，一个 [Spring Boot](https://spring.io/projects/spring-boot) 后端和一个 [MongoDB](https://www.mongodb.com) 数据存储。相当经典！起初，应用程序并没有被认为是在容器中运行的。如果您的机器上运行着 MongoDB 数据库，那么您可以通过克隆存储库并启动应用程序来轻松地进行测试:

```
$ git clone [https://github.com/lbroudoux/secured-fruits-catalog-k8s](https://github.com/lbroudoux/secured-fruits-catalog-k8s)
[...]
$ cd [secured-fruits-catalog-k8s](https://github.com/lbroudoux/secured-fruits-catalog-k8s)
$ mvn package -Pprod
[...]
$ java -jar target/fruits-catalog-1.0.0-SNAPSHOT.jar
[...]
2019-06-21 16:43:13.933  INFO 29503 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2019-06-21 16:43:13.935  INFO 29503 --- [           main] c.g.l.f.c.FruitsCatalogApplication       : Started FruitsCatalogApplication in 8.216 seconds (JVM running for 8.963)
```

一旦应用程序完成启动，你可以将浏览器指向`[http://localhost:8080](http://localhost:8080)`来检查结果，你应该会看到一个漂亮的 GUI 来显示你的目录(现在是空的)。可以开始加`apple, orange, grape, banana, cherry, lemon`等等……(尊重案例看图标)

![](img/0df931fb595eb57f4ccaae093747ba72.png)

> 顺便说一下，感谢 [Freepik](https://www.flaticon.com/authors/freepik) 为 nice[Creative Commons BY 3.0](http://creativecommons.org/licenses/by/3.0/)授权图标！

为了为应用程序部署这一系列增强，您需要一个 Kubernetes 集群。为了便于使用，我们实际上选择了 [OpenShift](http://www.openshift.com) 发行版。

> OpenShift **是一个** Kubernetes 发行版，所以每个 Kubernetes API 和命令行工具都可以使用。即使为了节省时间，我把重点放在了 OpenShift 上，但是每个概念和附加功能都可以应用到普通的 Kube 安装上。

# 我们想在这部分实现什么？

在第一部分中——作为热身——我们将简单地看看如何将该应用程序作为容器轻松部署到 OpenShift Kubernetes 发行版上。我们还将看到如何使用 Kubernetes secrets，并在进入集群时使用 [TLS](https://en.wikipedia.org/wiki/Transport_Layer_Security) 来保护传输层。

基本上，我们希望拥有以下架构，如下图所示:

![](img/74048a209278bd4ceb0a0eefbb76ed9e.png)

从安全性的角度来看，第一次转换的好处可以总结为:

*   作为容器部署的组件可以利用底层主机系统的多租户、隔离和资源密集化特性。请看一下[集装箱安全的十层](https://www.redhat.com/en/resources/container-security-openshift-cloud-devops-whitepaper)白皮书，
*   Kubernetes / OpenShift 允许对向其他服务和外部世界公开的内容进行细粒度控制。部署单元(又名 Pod)不再是公开的和可直接寻址的，
*   数据库凭据作为机密进行管理，独立于应用程序部署。他们可以使用强大的 [RBAC 模型](https://docs.openshift.com/container-platform/3.11/architecture/additional_concepts/authorization.html)由专门的操作人员观看/编辑，
*   在 TLS 的支持下，通过 OpenShift [路由](https://docs.openshift.com/container-platform/3.11/dev_guide/routes.html)控制对外部世界的暴露。

# 如何应用？

假设您有某种 OpenShift 集群实例正在运行并且可用。根据您的环境和需求，此实例可以采用多种形式:

*   在您的站点或您的云实例上完全成熟的 OpenShift 集群，参见如何[在您的站点上安装 open shift](https://docs.openshift.com/container-platform/3.11/install_config/index.html)，
*   Red Hat Container 开发工具包，了解如何[开始使用 CDK](http://developers.redhat.com/products/cdk/get-started/) ，
*   笔记本电脑上的轻量级 Minishift，请参见 [Minishift 项目页面](https://github.com/minishift/minishift)。

登录到 OpenShift 环境后，开始为我们的组件创建一个新项目(Kubernetes 名称空间++)。

```
$ oc new-project fruits-catalog --display-name="Fruits Catalog"
```

然后，从您的存储库克隆的根开始，在您的项目中创建新元素。首先，在放松了一些约束之后，部署一个新的 MongoDB 数据库:

```
$ oc adm policy add-scc-to-user anyuid -z default -n fruits-catalog
$ oc adm policy add-scc-to-user privileged -z default -n fruits-catalog
$ oc new-app mongodb-persistent --name=mongodb -p DATABASE_SERVICE_NAME=mongodb -p MONGODB_DATABASE=sampledb -l app=fruits-catalog -n fruits-catalog
```

> 因此，现在放松一些约束并允许`*anyuid*`和`*privileged*`并不是强制性的，我们将在第 5 部分中需要它来进行高级调整。

OpenShift 上的 MongoDB 部署会自动为您创建一个新的 Kubernetes secret，使用 admin、用户名和密码来连接数据库。然后，只需使用以下命令将我们的应用程序作为新的 OpenShift `DeploymentConfig`部署到您的集群上:

```
$ mvn fabric8:deploy -Popenshift
[...]
```

在一切部署之前等待几分钟——您可以从控制台或从`oc logs`命令检查日志。之后，在检查 pod 和路线时，您应该会看到类似的内容:

```
$ oc get pods -n fruits-catalog                                                                                         NAME                                   READY     STATUS      RESTARTS   AGE
fruits-catalog-1-xx7nd                 1/1       Running     0          1h
fruits-catalog-s2i-1-build             0/1       Completed   0          1h
mongodb-1-t85wm                        1/1       Running     0          1h$ oc get routes                                                                                                                          NAME             HOST/PORT                                                  PATH      SERVICES         PORT      TERMINATION   WILDCARD
fruits-catalog   fruits-catalog-fruits-catalog-2.apps.x.x.x.x.nip.io             fruits-catalog   8080                  None
```

哇！所有这些对象都是用一个简单的命令创建的！这就是 [Maven Fabric8](https://maven.fabric8.io/) 插件的神奇之处，它使用一些约定(你可以用片段覆盖)为你创建所有这些资源。您可以看到，默认路由是在没有终端的情况下创建的。最后，您必须修补路由，以便添加边缘 TLS 终端:

```
$ oc patch route/fruits-catalog --type=json -p '[{"op":"add", "path":"/spec/tls", "value":{"termination":"edge"}}]' -n fruits-catalog
```

现在，您应该已经为第一部分做好了一切准备！

# 检查它是否按预期工作

只需使用之前升级的路由的 URL 打开浏览器窗口，并检查应用程序是否按预期运行。应用程序现在使用 TLS 提供服务，您可以访问证书的详细信息。

![](img/9ff153200f11be4a2666c1e392f43d05.png)

我们在上面的截图中发现了一些错误，因为我的安装使用了一个定制的证书颁发机构来生成自签名证书，但是你可以很容易地配置 OpenShift 来使用外部提供者，比如 [Let'sEncrypt](https://letsencrypt.org/) 。

最后，我们看到我们的应用程序正在与我们的后端对话，而没有对它进行配置。嗯，事实并非如此…事情并没有神奇到那种地步。看看回购中的`[src/main/figure8/deployment.yml](https://github.com/lbroudoux/secured-fruits-catalog-k8s/blob/master/src/main/fabric8/deployment.yml#L34)`片段。您会注意到我们已经做好了准备，以便我们的应用程序能够从 Kubernetes secret 中的凭证检索并构建一个连接字符串。

```
env:
  - name: SPRING_DATA_MONGODB_USER
    valueFrom:
      secretKeyRef:
        key: database-user
        name: mongodb
  - name: SPRING_DATA_MONGODB_PASSWORD
    valueFrom:
      secretKeyRef:
        key: database-password
        name: mongodb
  - name: SPRING_DATA_MONGODB_URI
    value: mongodb://${SPRING_DATA_MONGODB_USER}:${SPRING_DATA_MONGODB_PASSWORD}@mongodb/sampledb
```

还记得这个秘密是在部署 MongoDB pod 时创建的吗？然后，即使我们不知道实际值，我们现在也能够引用和使用它。`SPRING_DATA_*`环境变量是从这些值中创建的，这些典型的变量是应用程序在启动时创建连接时所熟知的。

# 结论和下一步

在第一部分中，我们看到了使用简单的 Maven Fabric8 插件如何帮助我们轻松地将应用程序部署到 OpenShift Kubernetes 发行版。将我们的应用程序部署为容器使我们能够利用平台服务的优势，如应用程序的秘密管理。此外，它允许与主机系统的有效隔离、资源控制和多租户。最后，它允许我们通过 TLS 轻松地控制我们的应用程序的公开，而不必修改它、配置 Java 密钥库、信任库或这类东西。

在这个小小的热身之后，我们将进入正题:我们将看到如何使用 Keycloak 为我们的应用程序添加身份验证和授权。敬请期待第二部分！