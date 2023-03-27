# GCP 命令行备忘单

> 原文：<https://itnext.io/gcp-command-line-cheatsheet-5e4434ca2c84?source=collection_archive---------1----------------------->

谷歌云平台(GCP)命令行工具(`gcloud`)的各种命令的备忘单。

随着我遇到更多的命令，我计划进一步扩展这个列表。

![](img/4293f4e0b8062a5cfb41c1c8d7ec9cac.png)

# 目录

1.  [**新建一个项目并将其设为默认**](#68b3)
2.  [**设置默认项目**](#d8b9)
3.  [**设置默认计算区域和区域**](#383e)
4.  [**禁用交互提示**](#05b7)
5.  [**列出当前 CLI 配置**](#1472)
6.  [**创建带有自定义子网的 VPC 网络**](#dc56)
7.  [**创建一个带有自动子网**](#d350) 的 VPC 网络
8.  [**获取一个 VPC 网络的所有子网**](#2e26)
9.  [**用特定的机器类型**](#7b06) 创建一个计算实例
10.  [**在特定的 VPC 网络和子网中创建一个计算实例**](#7219)
11.  [**使用特定的操作系统映像创建一个计算实例**](#8296)
12.  [**获取一个计算实例的 VPC 网络和子网**](#b0e3)
13.  [**获取所有计算实例的名称**](#1311)
14.  [**允许流量进入 VPC 网络**](#f7c4)
15.  [**创建一个区域静态 IP 地址**](#9609)
16.  [**创建全局静态 IP 地址**](#0e6f)

# 创建一个新项目并将其设置为默认项目

```
gcloud **projects** create black-butterfly-4450 \
  --name black-butterfly \
  --set-as-default
```

*   `black-butterfly-4450`是项目 ID(必须是全球唯一的)
*   `black-butterfly`是项目名称(在您的帐户中必须唯一)

> 如果省略`--name`，项目名称被设置为等于项目 ID。

# 设置默认项目

```
gcloud **config** set core/project black-butterfly-4450
```

> 您必须指定**项目 ID** (全局唯一的)而不是项目名称。

> 👉[配置设置](https://cloud.google.com/sdk/gcloud/reference/config/set)👈

# 设置默认计算区域和分区

```
gcloud **config** set compute/region europe-west6
gcloud **config** set compute/zone europe-west6-a
```

> 👉[区域和分区](https://cloud.google.com/compute/docs/regions-zones/#locations)👈

# 禁用交互式提示

```
gcloud **config** set core/disable_prompts 1
```

禁用所有交互式提示，例如，在删除资源时。

> 或者，您可以设置`CLOUDSDK_CORE_DISABLE_PROMPTS=1`环境变量，或者在单独的命令中使用`-q/--quiet`全局变量。

# 列出当前的 CLI 配置

```
gcloud **config** list
```

# 创建带有自定义子网的 VPC 网络

1.  创建没有任何子网的 VPC 网络:

```
gcloud **compute networks** create my-vpc --subnet-mode custom
```

> VPC 网络是全球性的。子网是区域性的。

2.手动创建子网:

```
gcloud **compute networks subnets** create my-subnet-1 \
  --network my-vpc \
  --range 10.240.0.0/24
```

# 创建一个带有自动子网的 VPC 网络

```
gcloud **compute networks** create my-vpc
```

在每个区域自动创建一个子网。

> 子网有一个`*/20` CIDR 范围(例如 10.128.0.0/20)。

# 获取 VPC 网络的所有子网

```
gcloud **compute networks subnets** list --filter="network:my-vpc"
```

> 👉[过滤语法](https://cloud.google.com/sdk/gcloud/reference/topic/filters)👈

# 创建具有特定机器类型的计算实例

```
gcloud **compute instances** create i1 --machine-type=n1-standard-2
```

> 👉[机器类型](https://cloud.google.com/compute/docs/machine-types)👈

*   默认机器类型是`[n1-standard-1](https://cloud.google.com/compute/docs/machine-types#n1_machine_types)` (1 个 CPU，3.75 GB 内存)
*   实例名参数可以重复使用，以创建多个实例

# 在特定的 VPC 网络和子网中创建计算实例

```
gcloud **compute instances** create i1 \
  --network my-vpc \
  --subnet my-subnet-1
```

*   默认 VPC 网络是`default`
*   如果`--network`被设置为具有“自定义”子网模式的 VPC 网络，则还必须指定`--subnet`
*   实例名参数可以重复使用，以创建多个实例

# 使用特定操作系统映像创建计算实例

```
gcloud c**ompute instances** create i1 \
  --image-family ubuntu-1804-lts \
  --image-project ubuntu-os-cloud
```

> 👉[图片](https://cloud.google.com/compute/docs/images)👈

*   默认图像系列是`debian-9`
*   用户使用`--image-family`(使用该系列的最新图像)或`--image`(具体图像)
*   `--image-project`作为`--image`和`--image-family`的名称空间(在多个项目中可能有多个同名的图像/图像族)

列出所有可用的图像(包括项目和族),包括:

```
gcloud **compute images** list
```

# 获取计算实例的 VPC 网络和子网

```
{
  gcloud **compute instances** describe i1 \
    --format "value(networkInterfaces.**network**)" |
    sed 's|.*/||'
  gcloud **compute instances** describe i1 \
    --format "value(networkInterfaces.**subnetwork**)" |
    sed 's|.*/||'
}
```

> 👉[格式语法](https://cloud.google.com/sdk/gcloud/reference/topic/formats)👈

# 获取所有计算实例的名称

```
gcloud **compute instances** list --format="value(name)"
```

> 👉[格式语法](https://cloud.google.com/sdk/gcloud/reference/topic/formats)👈

例如，可用于删除所有现有的计算实例:

```
gcloud **compute instances** delete \
  $(gcloud compute instances list --format="value(name)")
```

# 允许流量进入 VPC 网络

```
gcloud **compute firewall-rules** create my-vpc-allow-ssh-icmp \
  --network my-vpc \  
  --allow tcp:22,icmp \
  --source-ranges 0.0.0.0/0
```

> `0.0.0.0/0`是`--source-ranges`的默认值，可以省略。

这允许来自任何来源(例如，来自公共互联网)的传入 ICMP 和 SSH (TCP 端口 22)流量到达 VPC 网络中的任何实例。

创建此防火墙规则后，您可以:

*   VPC 网络中的 Ping 实例:`ping EXTERNAL_IP`
*   SSH 到 VPC 网络中的实例:`gcloud compute ssh i1`

> 请注意，新创建的 VPC 网络没有应用防火墙规则，实例根本无法访问(甚至无法从 VPC 网络内部访问)。您必须创建防火墙规则，以使计算实例可访问。

# 创建一个区域静态 IP 地址

```
gcloud **compute addresses** create addr-1 --region=europe-west6
```

*   **区域 IP 地址**可以附加到计算实例、区域负载平衡器等。与 IP 地址在同一个区域。
*   可以重复 name 参数来创建多个地址

> 必须指定`*--global*`或`*--region*`或**之一**。

# 创建一个全局静态 IP 地址

```
gcloud **compute addresses** create addr-1 --global
```

*   **全局 IP 地址**只能连接到全局 HTTPS、SSL 代理和 TCP 代理负载平衡器。
*   可以重复 name 参数来创建多个地址

> 必须指定`--global`或`--region`或**之一**。