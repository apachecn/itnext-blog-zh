# 物联网的三个最佳 SDK

> 原文：<https://itnext.io/three-of-the-best-sdks-for-the-internet-of-things-ffd31566eb33?source=collection_archive---------0----------------------->

![](img/c611a1795dabcd93183d66038c6b8d60.png)

被誉为当代最大的突破之一，物联网的连接方面几乎存在于我们遇到的任何“事物”中。 [Gartner 预测](http://www.gartner.com/newsroom/id/3165317)到 2020 年，将有 208 亿“物”连接到物联网。思科和麦肯锡全球研究所预测，物联网将在未来十年产生超过 10 万亿美元的收入。

连通性和兼容性是物联网面临的两大挑战。底层技术和现有的通信模式阻碍了物联网的发展。

VMware、亚马逊和微软正试图用自己的软件开发工具包(SDK)来协助物联网开发者。这些公司的物联网软件开发套件简述如下:

## [VMware Liota](https://github.com/vmware/liota)

Liota 于 2016 年发布，是一款开源 SDK，用于“构建安全的物联网网关数据和控制编排应用”它支持应用程序从设备收集数据，将数据传输到数据中心，并执行来自数据中心组件(DCC)的控制信号。

Liota 或“小物联网代理”是供应商中立的，这使得开发的应用程序可以跨多个网关工作，而不管供应商的变化。

它包含用于构建应用程序的库，这些应用程序连接和编排跨设备、网关和云的数据和控制流；包括品脱图书馆，使使用国际单位制单位和转换单位的代码。

Liota 托管在 [GitHub](https://github.com/vmware/liota) 上，用 Python 编写，可以部署在任何支持 Python 的网关平台上。Liota 包括以下几层:

*   板层—底层硬件
*   网关层—硬件加操作系统
*   事物层—连接的设备
*   转换器层—用于创建指标的表示
*   传输层——如用于连接数据中心的网络套接字
*   DCC(数据中心组件)层

Liota 旨在解决当前与采用物联网技术相关的集成和基础设施问题。8 月 23 日，VMware [宣布与 Bayshore Networks、Dell Inc .、Intwine Connect、Deloitte Digital、PTC Inc .和 V5 Systems 建立新的联盟](http://www.marketwired.com/press-release/vmware-announces-alliances-and-strategies-for-the-internet-of-things-nyse-vmw-2152656.htm),以弥合开发人员和 IT 之间的现有差距。

## **微软 Azure 物联网 SDK**

微软有三个主要的物联网 SDK，包括:设备 SDK、服务 SDK 和网关 SDK。

*   [**Azure 物联网设备 SDK**](https://github.com/Azure/azure-iot-sdks) 微软 Azure 物联网设备 SDK 支持将客户端设备连接到 [Azure 物联网中心](https://azure.microsoft.com/en-us/documentation/articles/iot-hub-what-is-iot-hub/)(一种托管服务，用于物联网设备和解决方案后端之间可靠、安全的双向通信)。Azure 物联网设备库包含源代码，有助于开发连接到 Azure 物联网中心服务并由其管理的应用程序。

    包含 C、Node.js、Java 等多种语言的 SDK。NET 和 Python。
*   [**Azure 物联网服务 SDK**](https://github.com/Azure/azure-iot-sdks)微软 Azure 物联网服务 SDK 包含简化构建直接与物联网中心交互以管理设备和安全的应用程序的源代码。
    它包含用于的 SDK。NET、Node.js 和 Java 语言。
*   [**Azure 物联网网关 SDK**](https://github.com/Azure/azure-iot-gateway-sdk)**—Beta** 微软 Azure 物联网网关 SDK 使开发人员能够创建和部署根据其特定需求定制的网关智能。它提供了简化网关应用程序开发的源代码，包括动态模块加载、配置和数据管道。

    借助 Azure 物联网网关 SDK，开发者可以在数据发送到云端之前对其进行优化和处理。此外，Gateway SDK 使他们能够将传统设备连接到 Azure 云，而不必替换现有的基础设施。

    微软 Azure 物联网网关 SDK 有几个优势，包括支持实时敏感的边缘逻辑以最大限度地减少延迟，通过在本地处理选定的数据来降低带宽成本，以及帮助实施安全和隐私约束。

## [**AWS 物联网设备 SDK**](https://aws.amazon.com/iot/sdk/)

AWS 物联网设备 SDK 简化了硬件设备与 [AWS 物联网](https://aws.amazon.com/iot/)的连接。它通过[设备网关](https://aws.amazon.com/iot/how-it-works/#gateway)和[设备影子](https://aws.amazon.com/iot/how-it-works/#shadow)为硬件设备提供增强功能，使其安全工作。

AWS 物联网设备 SDK 包括开源库，用于在不同的硬件平台上构建物联网产品和解决方案。Device SDK 支持以下平台:

*   [嵌入式 C](https://github.com/aws/aws-iot-device-sdk-embedded-C/) (专门为资源受限的设备设计——运行在微控制器和 RTOS 上)
*   [Node.js](https://github.com/aws/aws-iot-device-sdk-js/)
*   [Arduino Yún](https://github.com/aws/aws-iot-device-sdk-arduino-yun/)