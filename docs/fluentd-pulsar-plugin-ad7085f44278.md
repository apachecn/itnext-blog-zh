# Fluentd Pulsar 插件

> 原文：<https://itnext.io/fluentd-pulsar-plugin-ad7085f44278?source=collection_archive---------1----------------------->

![](img/3133b4b21e22cbddbdb2f0e8a7a0e530.png)

照片由[德鲁·比默](https://unsplash.com/@drew_beamer?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄

对于那些碰巧读过[脉冲星或 Kakfa 的人来说？](/pulsar-or-kafka-and-the-lessons-from-doing-our-own-testing-a0180dfc4582)你会知道我们最近决定双脚跳进脉冲星。我们的部分数据流使用 Fluentd 将数据路由到许多地方，包括 Grafana Loki([Loki to the rescue](/loki-to-the-rescue-7c3cc0989a7))和一些将数据推送到 Pulsar 进行一些额外测试的微服务。这些都工作了，但是还有一个服务需要维护。

我们首先想到的是，有人已经为 Fluentd 创建了一个 Pulsar 插件。我们找不到！我们确实发现了一些 Fluentbit 插件，但是它们没有被设置为处理多个实例(将数据推送到多个主题)。我们甚至发现了几个毫无用处的空插件。我们试图修改其中一个插件，并能够让它处理多个实例，我们认为生活很好，直到我们尝试了一些测试，其中 Fluentbit 无法连接到 pulsar 集群，这导致了 Fluentbit 本身的 SIGTERM，我们认为它来自 pulsar go 库。所以这些都是彻头彻尾的失败。

在我们测试 Kafka 和 Strimzi 时，它有一个名为 Bridge 的组件，这是 Kafka 的一个 REST API，允许您推送事件。谢天谢地，Pulsar 最近也实现了类似的功能；Pulsar 只让你充当发行商，但这正是我们所需要的！请记住，这个特性是刚刚添加到 2.10.0 版本中的。

那么两个不认识 Ruby 的家伙该怎么办呢？好吧，从当前不稳定的代码中取出一些，并使其服从我们的意愿。这包括一个格式化程序和输出插件。

## 格式化程序

我们面临的第一个挑战是通过 REST API 将数据转换成 Pulsar 期望的正确格式。

因为我们处理的所有数据都是 JSON，所以我们采用了 formatter_json.rb 并对其进行了修改，使其适用于正确的格式，并保存为 formatter_json_pulsar.rb。您将会注意到，唯一真正的区别是将当前消息放在 message 中:{}

## 输出插件

为了实际发送到 Pulsar REST API，我们需要稍微修改一下输出，将所有消息作为有效载荷数组下的一个数组，然后将它们发送到 Fluentd HTTP 输入。

## 包括新的插件

我们使用 Bitnami Fluentd 容器映像，以便将这两个文件添加到/opt/bitbnami/fluentd/plugins 目录中。这可以通过创建一个新的容器(我们的路径)或者通过 docker-compose 文件来完成。如果您走容器路线，您将只需要一个复制命令来把文件放到插件目录中。

## 流动配置

既然我们已经将新的插件添加到了 Fluentd 容器中，我们可以修改我们的 Fluentd 配置来包含新的 http_pulsar 输出。默认情况下，输出使用格式化程序，因此您不需要显式设置它。

注意:Pulsar REST API 只对已经创建的主题有效，它不会创建新的主题。

这个配置将通过样本输入生成一些样本数据，并输出到 pulsar。当然你需要适应你的环境，但是会给你一个它能做什么的想法。HTTP 输出中的所有标准配置选项都在这里，除了 content_type，它已经被硬编码，因为 Pulsar 只接受 Application/json

# 摘要

希望你觉得这有点用！我花了很长时间寻找一个可以工作的 Pulsar 插件，但是每个插件都有这样或那样的问题。我们已经做了大量的测试，还没有发现弱点，但是我确信这个解决方案中存在一个弱点(像所有软件一样！).

感谢您的阅读，如果您喜欢这篇文章或我的其他文章，请鼓掌并关注！