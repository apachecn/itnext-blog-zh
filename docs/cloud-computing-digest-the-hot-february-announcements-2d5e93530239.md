# 云计算文摘:热门的二月公告

> 原文：<https://itnext.io/cloud-computing-digest-the-hot-february-announcements-2d5e93530239?source=collection_archive---------1----------------------->

欢迎来到[每周云文摘](https://oleksiidzhulai.com/blog/2019-02-09-cloud-computing-digest-the-latest-february-announcements/)！尽管今年年初，我们已经收到了来自领先提供商和云原生社区的许多公告，所以让我们回顾一下下面最令人兴奋的公告。

![](img/1feeae4f3db71e1ad08af9d19a0c923e.png)

# 自动警报系统

[推出 AWS 备份](https://aws.amazon.com/blogs/aws/aws-backup-automate-and-centrally-manage-your-backups/)一体式解决方案，支持 EBS、S3、EFS、迪纳摩、RDS 和存储网关卷，这也使其能够用于本地部署。

[AWS Worklink](https://aws.amazon.com/blogs/aws/amazon-worklink-secure-one-click-mobile-access-to-internal-websites-and-applications/) 是一项新服务，它最大限度地减少了向企业用户公开内部网络应用程序的麻烦。主要特点是集成认证和移动设备支持。

网络负载平衡器[获得了非常需要的 TLS 终止支持](https://aws.amazon.com/blogs/aws/new-tls-termination-for-network-load-balancers/)，这允许从您的后端卸载这项工作。此外，NLB 通过 ACM 支持自动证书部署。

站点到站点 VPN [获得对 IKEv2](https://aws.amazon.com/about-aws/whats-new/2019/02/aws-site-to-site-vpn-now-supports-ikev2/) 的支持，这增加了兼容网络设备的列表并简化了多云集成，例如，应该不需要启动 EC2 来建立与 Azure VPN 网关的 IPsec 隧道。

AWS Ops Automator，获得对 EC2 垂直缩放的[支持。以前自定义脚本也是可能的，但是，开箱即用并与 ALB 集成使我们的生活更加简单。](https://aws.amazon.com/blogs/architecture/aws-ops-automator-v2-features-vertical-scaling-preview/)

# GCP

Cloud Pub/Sub [获得了一个重放特性](https://cloud.google.com/blog/products/data-analytics/reliable-streaming-pipeline-development-with-cloud-pubsub-replay)，这带来了一个令人兴奋的可能性，可以捕捉和“倒带”在调试和修复 bug 期间导致错误的流量

位于 NoSQL 的 Cloud Firestore 基于 Firebase DB 的服务[宣布正式上市](https://cloud.google.com/blog/products/databases/announcing-cloud-firestore-general-availability-and-updates)。

Google [为 App Engine Flexible 增加了 WebSockets beta 支持](https://cloud.google.com/blog/products/application-development/introducing-websockets-support-for-app-engine-flexible-environment)，从而支持高度交互前端经常需要的流和全双工通信，并允许避免轮询开销。

# 蔚蓝的

Azure Storage 的帐户故障转移[现已在公开预览](https://azure.microsoft.com/en-us/blog/account-failover-now-in-public-preview-for-azure-storage/)中，允许将所有存储帐户数据复制到辅助区域，然后制作主区域。

[成本管理](https://azure.microsoft.com/en-us/blog/azure-cost-management-now-general-availability-for-enterprise-agreements-and-more/)，Azure 的云成本分析和预算服务，现在已经普遍提供给企业使用。

# 云原生

Rancher 2.2 增加了[多集群应用支持](https://rancher.com/press/multi-cluster-apps/)，不仅允许管理多个 Kubernetes 集群，还允许在它们之上作为单个实体整体部署一个应用。

Linux 基金会[推出新的 LF Edge](https://www.linuxfoundation.org/press-release/2019/01/the-linux-foundation-launches-new-lf-edge-to-establish-a-unified-open-source-framework-for-the-edge/) 为 Edge 建立统一的开源框架

# 本周阅读

CNCF 案例研究:[优步如何监管 4000 个微服务](https://www.cncf.io/blog/2019/02/05/how-uber-monitors-4000-microservices/)

Alex Tcherniakhovsky 著[用云 KMS 加密 Kubernetes](https://cloud.google.com/blog/products/containers-kubernetes/exploring-container-security-encrypting-kubernetes-secrets-with-cloud-kms) 秘密

贾纳基拉姆·MSV 关于[用 AWS 应用 Mesh 和 EKS 部署金丝雀](https://thenewstack.io/perform-canary-deployments-with-aws-app-mesh-on-amazon-eks/)

# 想要获取更新吗？考虑[订阅](https://oleksiidzhulai.com/)，后会有期！