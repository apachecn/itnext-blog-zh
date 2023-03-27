# 用 Kotlin 将 PyTorch 浮点张量转换为 Android RGBA 位图

> 原文：<https://itnext.io/converting-pytorch-float-tensor-to-android-rgba-bitmap-with-kotlin-ffd4602a16b6?source=collection_archive---------6----------------------->

PyTorch 最近发布了 1.3.0 版本，这是他们的第一个 Android 版本，允许你在智能手机上运行模型推理。这套功能还不是很完整，实现也有一些小错误，但是对于在您的设备上使用 DNNs 来说，这是一个很好的、很有帮助的步骤。

![](img/006c450607ac744cf480c7960923958e.png)

克里斯·里德在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

一个明显缺失的功能是将模型(如 UNet)的输出从张量转换回位图以显示给用户。我在 Kotlin 中实现了这个函数，你可以在这里找到它的要点:[https://gist . github . com/Phillies/830 f 52 b 0 BF 592 c 32 fc 507669694 f 66d 8](https://gist.github.com/phillies/830f52b0bf592c32fc507669694f66d8)

该函数接受一个浮点数组，并将其转换为 RGBA 位图，将最小的浮点值映射为 0，最大的浮点值映射为 255，反之亦然。

该函数需要一个`floatArray`作为主参数，它可以通过`myTensor.dataAsFloatArray`从张量中获得，并且应该是一个形状为`[height, width]`的 2D 张量。确保你的模型呈现出这种形状。只要确保 N 和 C 为 1，标准 PyTorch NCHW 形状就可以了。`alpha`参数设置阿尔法通道的值，`reverseScale`定义最小浮点值是否设置为 0 ( `false`)或 255 ( `true`)。

生成的位图是 RGBA 格式的，可以很容易地用 ImageView 显示出来。希望这对你们有些帮助:-)