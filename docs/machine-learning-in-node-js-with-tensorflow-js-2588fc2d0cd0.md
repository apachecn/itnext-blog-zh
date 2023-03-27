# 用 TensorFlow.js 在 Node.js 中进行机器学习

> 原文：<https://itnext.io/machine-learning-in-node-js-with-tensorflow-js-2588fc2d0cd0?source=collection_archive---------7----------------------->

[TensorFlow.js](https://js.tensorflow.org/) 是流行的开源库的新版本，它为 JavaScript 带来了深度学习。开发人员现在可以使用[高级库 API](https://js.tensorflow.org/api/0.12.0/) 来定义、训练和运行机器学习模型。

[预先训练的模型](https://github.com/tensorflow/tfjs-models/)意味着开发人员现在可以轻松地执行复杂的任务，如[视觉识别](https://emojiscavengerhunt.withgoogle.com/)、[生成音乐](https://magenta.tensorflow.org/demos/performance_rnn/index.html#2%7C2,0,1,0,1,1,0,1,0,1,0,1%7C1,1,1,1,1,1,1,1,1,1,1,1%7C1,1,1,1,1,1,1,1,1,1,1,1%7Cfalse)或[检测人类姿势](https://storage.googleapis.com/tfjs-models/demos/posenet/camera.html)只需几行 JavaScript。

作为 web 浏览器的前端库，最近的更新增加了对 Node.js 的实验性支持，这使得 TensorFlow.js 可以在后端 JavaScript 应用程序中使用，而不必使用 Python。

*读到关于这个库的内容，我想用一个简单的任务来测试一下……*🧐

> ***使用 TensorFlow.js 从 Node.js*** 使用 JavaScript 对图像进行视觉识别

不幸的是，提供的大多数[文档](https://js.tensorflow.org/#getting-started)和[示例代码](https://js.tensorflow.org/tutorials/webcam-transfer-learning.html)都在浏览器中使用这个库。[为简化加载和使用预训练模型而提供的项目实用程序](https://github.com/tensorflow/tfjs-models/tree/master/mobilenet)尚未扩展 Node.js 支持。完成这项工作的最终结果是我花了很多时间阅读库的 Typescript 源文件。👎

然而，经过几天的黑客，我设法得到了[这个完成的](https://gist.github.com/jthomas/145610bdeda2638d94fab9a397eb1f1d)！万岁！🤩🤩🤩

在深入研究代码之前，让我们先来概述一下不同的 TensorFlow 库。

# 张量流

[TensorFlow](https://www.tensorflow.org/) 是一个面向机器学习应用的开源软件库。TensorFlow 可用于实现神经网络和其他深度学习算法。

TensorFlow 于 2015 年 11 月由谷歌发布，最初是一个 [Python 库](https://www.tensorflow.org/api_docs/python/)。它使用基于 CPU 或 GPU 的计算来训练和评估机器学习模型。该库最初被设计为运行在具有昂贵 GPU 的高性能服务器上。

最近的更新扩展了该软件，使其可以在移动设备和网络浏览器等资源受限的环境中运行。

# TensorFlow Lite

[面向移动和嵌入式设备的轻量级版本库 Tensorflow Lite](https://www.tensorflow.org/mobile/tflite/) 于 2017 年 5 月发布。这伴随着一系列新的预训练深度学习模型，用于视觉识别任务，称为 [MobileNet](https://ai.googleblog.com/2017/06/mobilenets-open-source-models-for.html) 。MobileNet 模型设计用于在资源受限的环境中高效工作，如移动设备。

# TensorFlow.js

继 Tensorflow Lite 之后， [TensorFlow.js](https://medium.com/tensorflow/introducing-tensorflow-js-machine-learning-in-javascript-bf3eab376db) 于 2018 年 3 月公布。这个版本的库被设计为在浏览器中运行，构建在一个名为 [deeplearn.js](https://twitter.com/deeplearnjs) 的早期项目上。WebGL 提供了对库的 GPU 访问。开发人员使用 JavaScript API 来训练、加载和运行模型。

TensorFlow.js 最近被扩展为在 Node.js 上运行，使用了一个名为`tfjs-node`的[扩展库](https://github.com/tensorflow/tfjs-node)。

Node.js 扩展是一个 alpha 版本，仍在积极开发中。

## 将现有模型导入 TensorFlow.js

可以使用 TensorFlow.js 库执行现有的 TensorFlow 和 Keras 模型。模型需要在执行前使用此工具转换成新的格式[。Github](https://github.com/tensorflow/tfjs-converter) 上提供了用于图像分类、姿态检测和 k 近邻的预训练和转换模型[。](https://github.com/tensorflow/tfjs-models)

# 在 Node.js 中使用 TensorFlow.js

## 安装 TensorFlow 库

TensorFlow.js 可以从 [NPM 注册表](https://www.npmjs.com/)安装。

```
npm install @tensorflow/tfjs @tensorflow/tfjs-node
// or... 
npm install @tensorflow/tfjs @tensorflow/tfjs-node-gpu
```

这两个 Node.js 扩展都使用将按需编译的本机依赖项。

## 加载 TensorFlow 库

TensorFlow 的 [JavaScript API](https://js.tensorflow.org/api/0.12.0/) 从核心库暴露出来。启用 Node.js 支持的扩展模块不公开额外的 API。

```
const tf = require('@tensorflow/tfjs')
// Load the binding (CPU computation)
require('@tensorflow/tfjs-node')
// Or load the binding (GPU computation)
require('@tensorflow/tfjs-node-gpu')
```

## 加载张量流模型

TensorFlow.js 提供了一个 [NPM 库](https://github.com/tensorflow/tfjs-models) ( `tfjs-models`)，以方便加载预先训练好的&转换模型，用于[图像分类](https://github.com/tensorflow/tfjs-models/tree/master/mobilenet)、[姿态检测](https://github.com/tensorflow/tfjs-models/tree/master/posenet)和 [k 近邻](https://github.com/tensorflow/tfjs-models/tree/master/knn-classifier)。

用于图像分类的 [MobileNet 模型](https://github.com/tensorflow/tfjs-models/tree/master/mobilenet)是一个深度神经网络，被训练用于[识别 1000 个不同的类别](https://github.com/tensorflow/tfjs-models/blob/master/mobilenet/src/imagenet_classes.ts)。

在项目的自述文件中，下面的[示例代码](https://github.com/tensorflow/tfjs-models/tree/master/mobilenet#via-npm)用于加载模型。

```
import * as mobilenet from '@tensorflow-models/mobilenet';
// Load the model.
const model = await mobilenet.load();
```

**我遇到的第一个挑战是，这在 Node.js 上不起作用。**

`Error: browserHTTPRequest is not supported outside the web browser.`

查看[源代码](https://github.com/tensorflow/tfjs-models/blob/master/mobilenet/src/index.ts#L27),`mobilenet`库是底层`tf.Model`类的包装器。当调用`load()`方法时，它会自动从外部 HTTP 地址下载正确的模型文件，并实例化 TensorFlow 模型。

Node.js 扩展尚不支持动态检索模型的 HTTP 请求。相反，必须从文件系统中手动加载模型。

*在阅读了库的源代码后，我设法创建了一个解决方案……*

## 从文件系统加载模型

如果手工创建了`MobileNet`类，那么包含模型 HTTP 地址的自动生成的`path`变量可以用本地文件系统路径覆盖，而不是调用模块的`load`方法。完成这些后，调用类实例上的`load`方法将触发[文件系统加载器类](https://js.tensorflow.org/tutorials/model-save-load.html)，而不是尝试使用基于浏览器的 HTTP 加载器。

```
const path = "mobilenet/model.json"
const mn = new mobilenet.MobileNet(1, 1);
mn.path = `file://${path}`
await mn.load()
```

**厉害，管用！**

*但是模型文件是怎么来的呢？*

# MobileNet 型号

TensorFlow.js 的模型由两种文件类型组成，一种是存储在 JSON 中的模型配置文件，另一种是二进制格式的模型权重。模型权重通常被分割成多个文件，以便浏览器更好地缓存。

查看 MobileNet 型号的[自动加载代码](https://github.com/tensorflow/tfjs-models/blob/master/mobilenet/src/index.ts#L68-L76)，型号配置和重量碎片从该地址的公共存储桶中检索。

```
[https://storage.googleapis.com/tfjs-models/tfjs/mobilenet_v${version}_${alpha}_${size}/](https://storage.googleapis.com/tfjs-models/tfjs/mobilenet_v${version}_${alpha}_${size}/)
```

URL 中的模板参数指此处列出的[型号版本。每个版本的分类准确度结果也显示在该页面上。](https://github.com/tensorflow/models/blob/master/research/slim/nets/mobilenet_v1.md#pre-trained-models)

*根据* [*源代码*](https://github.com/tensorflow/tfjs-models/blob/master/mobilenet/src/index.ts#L36) *，使用* `*tensorflow-models/mobilenet*` *库只能加载 MobileNet v1 机型。*

HTTP 检索代码从这个位置加载`model.json`文件，然后递归地获取所有引用的模型权重碎片。这些文件的格式是`groupX-shard1of1`。

## 手动下载模型

通过检索模型配置文件、解析出引用的权重文件并手动下载每个权重文件，可以将所有模型文件保存到文件系统中。

我想使用 alpha 值为 1.0、图像大小为 224 像素的 MobileNet V1 模块。这给了我模型配置文件的[以下 URL](https://storage.googleapis.com/tfjs-models/tfjs/mobilenet_v1_1.0_224/model.json) 。

`[https://storage.googleapis.com/tfjs-models/tfjs/mobilenet_v1_1.0_224/model.json](https://storage.googleapis.com/tfjs-models/tfjs/mobilenet_v1_1.0_224/model.json)`

一旦这个文件被下载到本地，我就可以使用`[jq](https://stedolan.github.io/jq/)` [工具](https://stedolan.github.io/jq/)来解析所有的权重文件名。

```
$ cat model.json | jq -r ".weightsManifest[].paths[0]"
group1-shard1of1
group2-shard1of1
group3-shard1of1
...
```

使用`sed`工具，我可以用 HTTP URL 作为这些名称的前缀，为每个权重文件生成 URL。

```
$ cat model.json | jq -r ".weightsManifest[].paths[0]" | sed 's/^/https:\/\/storage.googleapis.com\/tfjs-models\/tfjs\/mobilenet_v1_1.0_224\//'https://storage.googleapis.com/tfjs-models/tfjs/mobilenet_v1_1.0_224/group1-shard1of1
https://storage.googleapis.com/tfjs-models/tfjs/mobilenet_v1_1.0_224/group2-shard1of1
https://storage.googleapis.com/tfjs-models/tfjs/mobilenet_v1_1.0_224/group3-shard1of1
...
```

使用`parallel`和`curl`命令，我可以将所有这些文件下载到我的本地目录。

```
cat model.json | jq -r ".weightsManifest[].paths[0]" | sed 's/^/https:\/\/storage.googleapis.com\/tfjs-models\/tfjs\/mobilenet_v1_1.0_224\//' | parallel curl -O
```

# 图像分类

[本示例代码](https://github.com/tensorflow/tfjs-models/tree/master/mobilenet#via-npm)由 TensorFlow.js 提供，用于演示返回图像的分类。

```
const img = document.getElementById('img');
// Classify the image.
const predictions = await model.classify(img);
```

**由于缺少 DOM，这在 Node.js 上不起作用。**

`classify` [方法](https://github.com/tensorflow/tfjs-models/blob/master/mobilenet/src/index.ts#L143-L155)接受大量的 DOM 元素(`canvas`、`video`、`image`)，并将自动从这些元素中检索图像字节并将其转换成一个`[tf.Tensor3D](https://js.tensorflow.org/api/latest/index.html#tensor3d)` [类](https://js.tensorflow.org/api/latest/index.html#tensor3d)，作为模型的输入。或者，可以直接传递`tf.Tensor3D`输入。

**比起尝试使用外部包来模拟 Node.js 中的 DOM 元素，我发现手动构造** `**tf.Tensor3D**` **更容易。**

## 从图像生成张量 3D

阅读用于将 DOM 元素转换为 Tensor3D 类的方法的[源代码](https://github.com/tensorflow/tfjs-core/blob/master/src/kernels/backend_cpu.ts#L126-L140)，以下输入参数用于生成 Tensor3D 类。

```
const values = new Int32Array(image.height * image.width * numChannels);
// fill pixels with pixel channel bytes from image
const outShape = [image.height, image.width, numChannels];
const input = tf.tensor3d(values, outShape, 'int32');
```

`pixels`是(Int32Array)类型的 2D 数组，包含每个像素的通道值的顺序列表。`numChannels`是每个像素通道值的数量。

## 为 JPEGs 创建输入值

`[jpeg-js](https://www.npmjs.com/package/jpeg-js)` [库](https://www.npmjs.com/package/jpeg-js)是 Node.js 的纯 javascript JPEG 编码器和解码器。使用该库可以提取每个像素的 RGB 值。

```
const pixels = jpeg.decode(buffer, true);
```

这将为每个像素(`width * height`)返回具有四个通道值(`RGBA`)的`Uint8Array`。MobileNet 模型仅使用三个颜色通道(`RGB`)进行分类，忽略 alpha 通道。此代码将四通道数组转换为正确的三通道版本。

```
const numChannels = 3;
const numPixels = image.width * image.height;
const values = new Int32Array(numPixels * numChannels);

for (let i = 0; i < numPixels; i++) {
  for (let channel = 0; channel < numChannels; ++channel) {
    values[i * numChannels + channel] = pixels[i * 4 + channel];
  }
}
```

## MobileNet 模型输入要求

正在使用的 [MobileNet 模型](https://github.com/tensorflow/models/blob/master/research/slim/nets/mobilenet_v1.md#mobilenet_v1)对宽和高 224 像素的图像进行分类。对于三个通道的每个像素值，输入张量必须包含介于-1 和 1 之间的浮点值。

不同尺寸图像的输入值需要在分类前重新调整大小。此外，来自 JPEG 解码器的像素值在范围*0–255*内，而不是 *-1 到 1* 。这些值也需要在分类前进行转换。

**TensorFlow.js 有库方法来使这个过程变得更容易，但是对我们来说幸运的是，** `**tfjs-models/mobilenet**` **库** [**自动处理**](https://github.com/tensorflow/tfjs-models/blob/master/mobilenet/src/index.ts#L103-L114) **这个问题！**👍

开发人员可以将类型为`int32`和不同维度的 Tensor3D 输入传递给`classify`方法，它会在分类之前将输入转换为正确的格式。也就是说没事可做…超级🕺🕺🕺.

## 获得预测

Tensorflow 中的 MobileNet 模型经过训练，可以识别来自 [ImageNet](http://image-net.org/) 数据集中[前 1000 个类](https://github.com/tensorflow/tfjs-models/blob/master/mobilenet/src/imagenet_classes.ts)的实体。模型输出这些实体中的每一个在被分类的图像中的概率。

*在* [*这个文件*](https://github.com/tensorflow/tfjs-models/blob/master/mobilenet/src/imagenet_classes.ts) *中可以找到正在使用的模型的训练类的完整列表。*

`tfjs-models/mobilenet`库在`MobileNet`类上公开了一个`classify`方法，从图像输入中返回概率最高的前 X 个类。

```
const predictions = await mn_model.classify(input, 10);
```

`predictions`是 X 个类别和概率的数组，格式如下。

```
{  className: 'panda',  probability: 0.9993536472320557 }
```

# 例子

学习了如何在 Node.js 上使用 TensorFlow.js 库和 MobileNet 模型之后，[这个脚本](https://gist.github.com/jthomas/145610bdeda2638d94fab9a397eb1f1d)将对作为命令行参数给出的图像进行分类。

# 源代码

*   将这个脚本文件和包描述符保存到本地文件。

# 测试它

*   按照上面的说明将模型文件下载到一个`mobilenet`目录下。
*   使用 NPM 安装项目依赖项
*   下载一个样本 JPEG 文件进行分类

```
wget http://bit.ly/2JYSal9 -O panda.jpg
```

![](img/c2324451ad931312a207502de28653c0.png)

*   使用模型文件和输入图像作为参数运行脚本。

```
node script.js mobilenet/model.json panda.jpg
```

**如果一切正常，下面的输出应该会打印到控制台。**

```
classification results: [ {  className: 'giant panda, panda, panda bear, coon bear',  probability: 0.9993536472320557 } ]
```

该图像以 99.93%的概率被正确分类为包含熊猫！🐼🐼🐼

# 结论

TensorFlow.js 为 JavaScript 开发者带来了深度学习的力量。通过 TensorFlow.js 库使用预先训练的模型，可以用最少的工作和代码轻松扩展 JavaScript 应用程序，完成复杂的机器学习任务。

TensorFlow.js 已经作为基于浏览器的库发布，现在已经扩展到在 Node.js 上工作，尽管并非所有的工具和实用程序都支持新的运行时。经过几天的研究，我能够使用 MobileNet 模型库对本地文件中的图像进行视觉识别。

让它在 Node.js 运行时工作意味着我现在开始下一个想法……让它在一个无服务器函数中运行！很快回来阅读我与 TensorFlow.js 的下一次冒险。👋

*原载于 2018 年 8 月 7 日*[*jamesthom . as*](http://jamesthom.as/blog/2018/08/07/machine-learning-in-node-dot-js-with-tensorflow-dot-js/)*。*