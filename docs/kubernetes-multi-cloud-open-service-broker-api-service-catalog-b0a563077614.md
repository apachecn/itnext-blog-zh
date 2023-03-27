# Kubernetes 多云—开放服务代理 API 和服务目录

> 原文：<https://itnext.io/kubernetes-multi-cloud-open-service-broker-api-service-catalog-b0a563077614?source=collection_archive---------3----------------------->

![](img/a34c96c557f1c506297e52ff7e4990c4.png)

使用开放服务代理 API 和 Kubernetes 服务目录的多云 Kubernetes

简单的说，多云策略就是使用两种或两种以上的云计算服务。通过这种策略，用户可以从不同的云平台中选择最佳的服务，并将它们结合起来，为您的企业创建最佳的解决方案。 [2018 年云状况调查](https://www.rightscale.com/blog/cloud-industry-insights/cloud-computing-trends-2018-state-cloud-survey)显示，81%的企业使用多种云。

Kubernetes 被广泛认为是部署和操作容器化应用程序的最佳方式。此外，Kubernetes 的一个关键价值主张是，它可以帮助云提供商实现功能标准化。如果用户希望将不同平台上的资源绑定到 Kubernetes 应用程序，这将为他提供丰富的优势，同时不构建相同的本地应用程序，该怎么办？您将什么平台放在哪里，以便以最有效的方式获得最大的价值？Azure MysQL 服务用于 MysQL 还是 Amazon RDS 用于 PostgreSQL？AWS SQS 还是 Azure 的 AQS？或者想尝试语音界面？莱克斯接了电话。您有 18 种 AWS 服务可供选择！

开放服务代理 API 是一个行业范围的努力，通过一个简单和安全的机制来满足消费云服务的需求。该项目的参与者来自 Google、IBM、Pivotal、Red Hat、SAP 和许多其他领先的云公司。

# 服务目录和服务代理

服务目录为开发人员、ISV 和 SaaS 供应商提供了一种方式来供应服务实例，这些实例可以由应用程序以标准方式使用。Service Catalog 是一个扩展 API，支持 Kubernetes 集群中运行的应用轻松使用外部管理的软件产品，例如云提供商*提供的数据存储服务。*服务代理是开源[开放服务代理(OSB) API](http://www.openservicebrokerapi.org/) 的实现，它简化了向运行在 Kubernetes 上的应用交付特定于云平台的服务。

![](img/f68795bf11b0f3922f3a00d0cf46f178.png)

Kubernetes 架构上的服务目录

使用服务目录体系结构，应用程序开发人员能够从目录中请求服务，而不必担心如何以及在哪里提供服务。一旦服务可用，他们就可以将服务绑定到他们的应用程序，并使用任何相关的凭证。这些供应和绑定请求是向服务代理(AWS 服务代理、Azure 服务代理、GCP 服务代理等)发出的。)，它们在服务目录中注册，充当运行在其他平台和应用程序上的服务之间的粘合代码。

在 Kubernetes 上，服务目录由 catalog-apiserver 和 catalog-controller-manager 组成，部署为 Kubernetes 部署，如下所示:

![](img/48f6f6890234f95835453f3d76f1baa2.png)

Kubernetes 上的服务目录

假设一个应用程序开发人员希望在本地 Kubernetes 集群上运行一个 web 应用程序，并使用 Azure MySQL 作为数据库，使用 AWS S3 作为存储后端，并且完全不知道如何操作这些应用程序(将这些服务与应用程序粘合在一起)，Service Broker 可以帮助实现后端服务与应用程序本身的无缝集成。

![](img/b92ca9f2bc5fd3ff05f53e5a11a5e4a7.png)

成分

服务代理提供许多服务类(例如，如果 Azure 代理部署在集群中，Azure MySQL 将是这些服务类之一。类似云服务的东西)，每个服务类将构成众多的服务计划(例如，Azure MySQL-Basic、Azure MySQL-Production_Grade 等。类似风味/计划的东西)。

![](img/e7c36e6c720c17cf36b327a856f20454.png)

# Kubernetes 上的典型工作流程

![](img/03b86a725f283ac5e93b2f1cf8b22aa3.png)

Kubernetes 上的工作流

# 面向 Azure 的开放服务代理

在 Microsoft Azure 公共云中提供托管服务的兼容 API 服务器。微软一直是服务目录的积极贡献者，该服务目录使 Kubernetes 运营商能够利用 Azure 平台提供的云原生服务。

下面的信息显示了使用 Azure 的 OSBA 将运行在 Kubernetes 本地集群上的 Wordpress 应用程序连接到 Azure 的 MySQL 服务的示例工作流

![](img/46227a4dd2ed6867919a31e67df70900.png)

演示集群

Open Service Broker for Azure 作为 Pod/Deployment 部署在 Kubernetes 上，并使用 Redis 作为其状态的后备存储。

![](img/94620f9da025fce116590ae407145d04.png)![](img/2062f4690e091f921801cbc7c42c372e.png)

创建 Service Broker 部署时，应提供具有所需凭据的用户帐户。

![](img/d78002fd89c8149c405c603bdf6ab3cb.png)

如上所示，“svcat”是一个服务目录的命令行工具，一旦 Azure 的 OSBA 可用，它会列出所有可用的服务类。如上所述，每个服务类别将包括多个服务计划。例如,“azure-mysql”服务类将包括如下所示的计划:

![](img/ac7afa8d62b65dfbce32860437e85c58.png)

服务目录提供的服务计划

用户可以根据需求选择方案。Open Service Broker for Azure 使用一个服务主体代表用户提供 Azure 资源，或者在这种情况下提供一个可信的媒介，由代理无缝地部署服务。应创建一个资源组(RG ),将 OSBA 创建的一组资产分组到逻辑组中，以便于甚至自动进行资源调配。

![](img/dc85fb860990a761f055447f57e447ce.png)

用于授权服务目录和代理创建资源的服务主体

![](img/428f7e4bfb8479b035400512c110d659.png)

Azure 上的资源组

部署了一个 Wordpress 应用程序，默认情况下，它利用 Open Service Broker For Azure 为 MySQL 数据库提供一个 [Azure 数据库，供 Wordpress 服务器使用。](https://azure.microsoft.com/en-us/services/mysql/)

![](img/53875553bf05d2b2965e97b7cc31788b.png)

一旦部署了包含“ServiceInstance”和“ServiceBinding”资源的应用程序，就会在用户 Azure 帐户上请求 MySQL 资源。

![](img/075adc34cf1e8ad655440f0aedf2c597.png)

服务实例和服务绑定

这将在用户 Azure 帐户上启动服务部署，如下所示:

![](img/463fe4814603bbad63574cd7b51f2057.png)

由于上面已经声明了 RG 和 SP，所有资源都被分组到指定的资源组下，如下所示:

![](img/4b92ed85240ce50821c1551326692117.png)

在 Azure 上创建的 MySQL 用户帐户

![](img/1fe2a0a0ae6a677d6b297fac7207119d.png)

如下所示，Azure 的 MySQL 服务中将创建一个 DB 条目，供 Wordpress 使用，所有的流水线操作都是无缝完成的。

![](img/34c7051b726f00a9eab1a83ada846590.png)

DB Entry-Wordpress 在用户 Kubernetes 集群上的应用

通过使用上面的服务器名称登录到 MySQL 服务来验证连接性，我们可以看到本地运行的 Wordpress 应用程序正在将数据写入 Azure MySQL 服务:

![](img/2b0ded53bb51313e13819d742d035fc5.png)

Kubernetes 上的 Wordpress 应用程序在 Azure 上编写 MySQL

# 面向 AWS 的开放式服务代理

代理直接在应用程序平台中提供 AWS 服务的简单集成将 AWS 服务直接集成到 Kubernetes 上运行的应用程序中。

以下信息显示了在本地集群中运行的 Kubernetes pod 使用 AWS S3 进行存储的示例工作流。这需要在 Kubernetes 集群上运行服务目录。

![](img/eafed7b7e81d4049809b00e93f70175f.png)

AWS Servicebroker —演示集群

AWS Service Broker 先决条件:

1.  用于跟踪状态的 DynamoDB 表。

![](img/49daf3003ddcbaed46bcd753bbe2d056.png)

用于服务代理状态跟踪的 AWS DynamoDB

2.一个 S3 存储桶，用于存储代理用来管理 AWS 服务的模板。

![](img/e0cbfe0ff71e5ca7643beffa928e4e52.png)

存储 Service Broker 使用的云信息模板的 S3 存储桶

3.一个 IAM 角色，拥有与 DynamoDB 表、S3 桶和 AWS CloudFormation 交互的权限。

![](img/ed2b02fcc0675e5e54d82c57630e4c7a.png)

IAM 用于授权和访问

所有这些先决条件都可以使用官方 AWS 资源提供的云形成模板来创建。

![](img/be04d6461f71a6df688d7045df89a757.png)

用于设置 Service Broker 先决条件的 AWS CloudFormation 模板

AWS Service Broker 部署为 Kubernetes 部署:

![](img/b6f0de5ba4c1dec06ac3c2d5c98f0e54.png)

在 Kubernetes 上部署 AWS 服务代理

一旦部署了相同的服务，目录就会同步，用户应该能够看到代理支持的所有服务类。

![](img/0af907947d965eb34dfa23e243befd96.png)

通过 Kubernetes CRD 的集群服务代理

![](img/b52ae7968b42636c763279091358482a.png)

由 Kubernetes 上的 AWS 服务代理提供的服务类别

部署了一个示例 web 应用程序，默认情况下，它使用 S3 作为后端存储，随机写入 S3 存储桶。

![](img/e34e39f7f27778892e302081e1b58d25.png)

带有 S3 后端的演示应用程序

随着上述应用程序的创建，AWS Cloudformation 服务开始使用上述 IAM 配置为应用程序创建 S3 桶。

![](img/9f425e6103698998e2aa15024fbe5be0.png)

AWS 上的资源创建

如上所示，无缝创建了一个 S3 存储桶和一个日志存储桶来满足应用程序存储的需求。

![](img/7dbb6d7528a7ce6ff907f81091569ec8.png)

AWS 上的应用程序存储-S3

将在 Kubernetes 名称空间上创建一个对象服务实例，并选择服务类，服务目录与 Service Broker 交互，将信息同步到本地环境，如下所示:

![](img/437a9dd85836d759c041e0c9ec9b5c74.png)

Kubernetes 上的服务实例对象

服务实例信息将被写入 DynamoDB 表之一，以跟踪状态:

![](img/d5b05485408ca0cdf1001090c8f46e11.png)

包含服务实例信息的 Dynamo 表

访问在 AWS 上创建的 S3 资源的身份验证绑定由服务绑定提供:

![](img/3e0ebbb48c974b8f25196ef1062683b1.png)

Kubernetes 上的服务绑定

由于机密对象没有提供给上面创建的应用程序容器，对 S3 资源的访问将被禁止，如下所示:

![](img/f2607c23ccb8e070df7d4bb0565ae836.png)

Kubernetes 资源访问机密

收集的所有秘密都可以提供给上面使用清单中的 container_spec 创建的应用程序，如下所示:

![](img/286c47bcedfa72e1965a3d3e0091303e.png)

一旦上述凭证/秘密被提供给应用程序，应用程序就可以写入 AWS 上的 S3 桶。快速测试连通性:

![](img/84108a61a3bff214b37564651f048283.png)![](img/8dde942c455a04c37860ea5126e98032.png)

Kubernetes 在 S3 自动气象站上的应用

如上所示，Kubernetes 本地集群上的应用程序可以写入 AWS 上的 S3 存储桶。

服务目录提供了强大的抽象，使外部服务在运行于多样化环境中的 Kubernetes 集群中可用。这些服务通常是特定云平台的第三方托管云产品，其中完整的抽象更容易掩盖集成的复杂性。服务目录和服务代理不仅限于公共云平台，还可以用于多种内部服务。另一方面，开发人员可以专注于应用程序，而无需管理复杂的服务部署。