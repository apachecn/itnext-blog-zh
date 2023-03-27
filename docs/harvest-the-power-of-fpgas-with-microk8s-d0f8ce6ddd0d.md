# 用 MicroK8s 收获 FPGAs 的力量

> 原文：<https://itnext.io/harvest-the-power-of-fpgas-with-microk8s-d0f8ce6ddd0d?source=collection_archive---------5----------------------->

![](img/dea96713fa1417744f3f528e5d922195.png)

一场伟大的比赛！

[FPGA](https://en.wikipedia.org/wiki/Field-programmable_gate_array)一直在我的待办事项清单上。想到你正在处理的代码可以通过在硬件中运行来加速，总是令人兴奋的。多么天真…

事实是，在 FPGAs 上运行代码本身就是一门艺术。相当复杂。您需要投入时间和精力让所有工具正确对齐，并获得现场可编程门阵列的强大功能。

谢天谢地，在 T4 的伟大的人们在这里帮助我们。凭借其经过深思熟虑的精简工具集，他们减轻了集成、存储和交付硬件加速代码的负担。

自然，InAccel 和 MicroK8s 是绝配！我们欢迎他们加入 MicroK8s 生态系统。以下是如何让你一瞥我们为你做的事情有多简单:

```
sudo snap install microk8s --classic --channel=latest/edge
sudo microk8s status --wait-ready
sudo microk8s enable inaccel 
```

inaccel 附加组件目前在`latest/edge`频道提供，并将包含在 MicroK8s 的下一个稳定版本 *v1.23* 中。阅读更多关于[我们的文档](https://microk8s.io/docs/addon-inaccel)。

玩得开心！

# 链接

[](https://en.wikipedia.org/wiki/Field-programmable_gate_array) [## 现场可编程门阵列-维基百科

### 现场可编程门阵列(FPGA)是一种集成电路，旨在由客户或设计者进行配置…

en.wikipedia.org](https://en.wikipedia.org/wiki/Field-programmable_gate_array) [](https://inaccel.com/) [## 应用加速变得简单

### InAccel 提供了让您开始使用硬件加速所需的所有工具。它汇集了版本…

inaccel.com](https://inaccel.com/)  [## micro k8s-Addon:in Accel | micro k8s

### 主页:来自 MicroK8s 的 https://inaccel.com/版本:1.23+支持的 arch: amd64 InAccel 允许您构建、运送和…

microk8s.io](https://microk8s.io/docs/addon-inaccel) [](https://github.com/ubuntu/microk8s) [## GitHub-Ubuntu/micro k8s:micro k8s 是一个小型、快速、单包的 Kubernetes，面向开发者、IoT…

### 单包完全兼容的轻量级 Kubernetes，可在 42 种风格的 Linux 上运行。完美适用于:典范力量…

github.com](https://github.com/ubuntu/microk8s)