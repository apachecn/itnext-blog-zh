# 构建 FFmpeg WebAssembly 版本(= FFmpeg . wasm):part . 5 FFmpeg . wasm v 0.3—js 之前和直播(已过时)

> 原文：<https://itnext.io/build-ffmpeg-webassembly-version-ffmpeg-js-part-5-ffmpeg-js-v0-3-pre-js-and-live-streaming-c1498939a74c?source=collection_archive---------4----------------------->

![](img/8c0c378d4f8e771bf56ebf20e4685f63.png)

以前的故事:[构建 FFmpeg WebAssembly 版本(= FFmpeg . js):part . 4 FFmpeg . js v 0.2—Web Worker 和 Libx264](/build-ffmpeg-webassembly-version-ffmpeg-js-part-4-ffmpeg-js-v0-2-web-worker-and-libx264-d0596f1beb4e)

在这一部分，您将学习:

1.  使用`--pre-js`重新定义模块中的功能
2.  使用带网络摄像头的`ffmpeg.js`

# 使用`--pre-js`重新定义模块中的功能

FFmpeg 有大量的输出，它包含了重要的信息，比如视频的元数据，编码器/解码器的输出和任务的进度。默认情况下，这些信息是从原生 C 库`printf` / `sprintf`中打印出来的，虽然你可以在 DevTools 中看到，但不能用 JavaScript 操作。

幸运的是，在 Emscripten 中，我们可以用`--pre-js`或`--post-js`重新定义[的一些默认函数](https://emscripten.org/docs/api_reference/module.html)的行为。对于上面的场景，我们需要重新定义的函数是`Module['printErr']`(因为 FFmpeg 的输出使用了`stderr`)并用`--pre-js`添加到我们的 ffmpeg.js 中。

以下是我学到一些经验，供你参考:

*   在`--pre-js`中只能使用 ES5 语法(无箭头函数，const，let)
*   您需要添加额外的宏来防止您的代码被闭包编译器移除(这里我不使用闭包编译器)

下面是 ffmpeg.js 中使用的电流`prepend.js`:

我们实际上定义了一个新的函数`setLogger`来注册我们的自定义记录器函数，并重新定义了两个现有的函数`print`和`printErr`。有了这个 prepend.js，现在我们可以轻松地操作 FFmpeg 的输出消息，并开发更多的功能(如进度条)。

在构建脚本中添加`--pre-js`很简单(第 54 行):

现在我们在模块对象中有了`setLogger`，我们可以在初始化后使用它(第 13 行):

# 使用带网络摄像头的`ffmpeg.js`

在这里，我想描述如何使用 ffmpeg 与直播流，这里我们使用网络摄像头为例，但大多数情况下应该有类似的工作流程。

基本工作流程是:

1.  使用 MediaRecorder API 将流保存到 Blob
2.  将 Blob 转换为 Uint8Array 数据
3.  使用 ffmpeg.js 对 Uint8Array 数据进行代码转换

步骤 1 .使用`getUserMedia`访问网络摄像头(需要 https 协议)

```
<video id="webcam" width="320px" height="180px"></video>
<script>
const webcam = document.getElementById('webcam');
(async () => {
  webcam.srcObject = await navigator.mediaDevices.getUserMedia({ video: true, audio: true });
  await webcam.play();
})();
</script>
```

步骤 2 使用`MediaRecorder`记录组块

```
const startRecording = () => {
  const rec = new MediaRecorder(webcam.srcObject);
  const chunks = [];
  rec.ondataavailable = e => chunks.push(e.data);
  rec.onstop = async () => {
    transcode(new Uint8Array(await (new Blob(chunks)).arrayBuffer()));
  };
  rec.start();
};
```

步骤 3:重复使用前一部分中的代码转换进行代码转换。

查看下面的 CodePen 以获得完整代码和现场演示:

在第 5 部分中，我们已经学习了如何使用`--pre-js`来重新定义/扩展模块的功能，并介绍了一个如何在直播场景中使用 ffmpeg 的例子。

对于第 6 部分，我们将深入研究文件系统:[构建 FFmpeg WebAssembly 版本(= ffmpeg.js):第 6 部分深入研究文件系统](https://medium.com/@jeromewus/build-ffmpeg-webassembly-version-ffmpeg-js-part-6-a-deep-dive-into-file-system-56eba10067ca)

存储库:

*   ffmpeg-core . js:[https://github.com/ffmpegjs/FFmpeg](https://github.com/ffmpegjs/FFmpeg)
*   ffmpeg . js:[https://github.com/ffmpegjs/ffmpeg.js](https://github.com/ffmpegjs/ffmpeg.js)