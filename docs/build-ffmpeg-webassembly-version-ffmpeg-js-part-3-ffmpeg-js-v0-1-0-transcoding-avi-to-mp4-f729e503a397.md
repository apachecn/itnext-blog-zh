# 构建 FFmpeg WebAssembly 版本(= FFmpeg . wasm):part . 3 FFmpeg . wasm v 0.1—将 avi 代码转换为 mp4

> 原文：<https://itnext.io/build-ffmpeg-webassembly-version-ffmpeg-js-part-3-ffmpeg-js-v0-1-0-transcoding-avi-to-mp4-f729e503a397?source=collection_archive---------4----------------------->

![](img/8c0c378d4f8e771bf56ebf20e4685f63.png)

> 2020/9 更新:调整段落结构，使其更具可读性。

以前的故事:[构建 FFmpeg WebAssembly 版本(= ffmpeg.wasm): Part.2 用 Emscripten 编译](https://medium.com/@jeromewus/build-ffmpeg-webassembly-version-ffmpeg-js-part-2-compile-with-emscripten-4c581e8c9a16)

在这一部分，您将学习:

1.  使用优化的参数构建 FFmpeg 的库版本。
2.  与 ffmpeg.wasm 互动
3.  管理脚本文件系统。
4.  开发具有转码功能的 ffmpeg.wasm v0.1。

# 使用优化的参数构建 FFmpeg 的库版本。

在第 3 部分中，我们的目标是创建一个基本的 ffmpeg.wasm v0.1 来将 avi 转码为 mp4。由于我们在第 2 部分只创建了 FFmpeg 的基本版本，现在我们需要用一些参数进一步优化。

1.  `-O3`:优化代码并减少代码大小(从 30 MB 减少到 15 MB)(更多详情[此处](https://emscripten.org/docs/optimizing/Optimizing-Code.html))
2.  `-s PROXY_TO_PTHREAD=1`:让我们的程序在使用 pthread 时有反应(更多细节[这里](https://emscripten.org/docs/porting/pthreads.html#additional-flags))
3.  `-o wasm/dist/ffmpeg-core.js`:将 ffmpeg.js 重命名为 ffmpeg-core.js
    (从这里我们称之为 ffmpeg-core.js，因为我们将创建一个 ffmpeg.js 库来包装 ffmpeg-core.js 并提供用户友好的 API。)
4.  `-s EXPORTED_FUNCTIONS="[_main, _proxy_main]"`:将 main()和 proxy_main()(由 PROXY_TO_PTHREAD 添加)C 函数导出到 JavaScript 世界
5.  `-s EXTRA_EXPORTED_RUNTIME_METHODS="[FS, cwrap, setValue, writeAsciiToMemory]"`:用于操作函数、文件系统和指针的额外函数，查看与代码和 [preamble.js](https://emscripten.org/docs/api_reference/preamble.js.html) 交互的[了解更多详情。](https://emscripten.org/docs/porting/connecting_cpp_and_javascript/Interacting-with-code.html)

> 有关这些参数的更多细节，您可以查看 emscripten github 存储库中的 [src/settings.js](https://github.com/emscripten-core/emscripten/blob/1.39.18/src/settings.js) 。

有了这些新的论点，让我们更新我们的`build.sh`:

接下来我们试着和 ffmpeg.wasm 互动一下。

# 与 ffmpeg.wasm 互动

为了确保我们的 ffmpeg.wasm 正常工作，让我们尝试在 ffmpeg.wasm 中实现以下命令:

```
$ ffmpeg -hide_banner
```

使用`-hide_banner`参数，ffmpeg 隐藏了关于其版本和构建参数的细节，典型的输出如下所示:

```
Hyper fast Audio and Video encoder
usage: ffmpeg [options] [[infile options] -i infile]... {[outfile options] outfile}...Use -h to get full help or, even better, run 'man ffmpeg'
```

首先，让我们用下面的代码创建一个名为`ffmpeg.js`的文件:

上面代码的执行需要 Node 中的额外参数。JS:

```
$ node --experimental-wasm-threads --experimental-wasm-bulk-memory ffmpeg.js
```

功能说明:

*   `onRuntimeInitialized`:由于 WebAssembly 需要一些时间来启动，你需要等待这个函数被调用后才能使用这个库。
*   `cwrap`:JavaScript 世界中 C 函数的包装函数。这里我们将 proxy_main() / main()函数包装在`fftools/ffmpeg.c`中。函数签名是`int main(int argc, char **argv)`，很明显`int`映射到`number`，由于`char **argv`是 C 语言中的指针，我们也可以将其映射到`number`。

然后我们需要把参数传递给它。`$ ffmpeg -hide_banner`的等价自变量是`main(2, ["./ffmpeg", "-hide_banner"])`。第一个参数很简单，但是我们如何传递一个字符串数组呢？让我们将问题分解为两部分:

1.  我们需要将 JavaScript 中的 string 转换成 C 中的 char 数组
2.  我们需要将 JavaScript 中的数字数组转换成 C 中的指针数组

第一部分比较容易，因为我们有一个来自 Emscripten 的名为`writeAsciiToMemory()`的实用函数来帮助我们，下面是一个使用该函数的示例:

第二部分比较复杂，我们需要用 C 语言创建一个 32 位整数的指针数组，因为指针是 32 位整数。我们需要在这里使用`setValue`来创建我们需要的数组:

合并上面的所有片段，现在我们可以与 ffmpeg.wasm 交互并生成预期的结果:

现在，我们可以轻松地与 ffmpeg.wasm 进行交互，但如何将视频文件传递给它呢？这是下一节的重点:文件系统。

# 管理脚本文件系统。

在 Emscripten 中，有一个虚拟文件系统来支持 C 中的标准文件读/写，因此我们需要在将参数传递给 ffmpeg.wasm 之前将视频文件写入这个文件系统。

> 在[文件系统 API](https://emscripten.org/docs/api_reference/Filesystem-API.html) 中找到更多细节。

很多时候，完成任务只需要 2 个 FS 函数:`FS.writeFile()`和`FS.readFile()`。

对于所有写入或读取文件系统的数据，它必须是 JavaScript 中的 Uint8Array 类型，记住在使用数据之前要进行类型转换。

对于本教程，我们先用一个名为`flame.avi`(这里可以下载[)的文件，用`fs.readFileSync()`读取，用`FS.writeFile()`写入 Emscripten 文件系统。](https://github.com/ffmpegwasm/testdata/raw/master/flame.avi)

# 开发具有转码功能的 ffmpeg.wasm v0.1。

现在我们能够将参数传递给 ffmpeg.wasm 并将文件保存到文件系统，让我们将它们组合在一起，让我们的 ffmpeg.wasm v0.1 工作起来。

我们需要注意的最后一个细节是，上面的`ffmpeg()`实际上是异步运行的，所以为了获得输出文件，我们需要使用一个`setInterval()`来解析日志文件，以了解转换是否完成。

综上所述，现在我们有了第一个 ffmpeg.wasm，可以将 avi 文件转码为 mp4 文件，不会出现任何问题:

你可以访问这里的知识库，详细了解它是如何工作的:[https://github.com/ffmpegwasm/FFmpeg/tree/n4.3.1-p3](https://github.com/ffmpegwasm/FFmpeg/tree/n4.3.1-p3)

并且可以在这里随意下载构建神器:[https://github.com/ffmpegwasm/FFmpeg/releases/tag/n4.3.1-p3](https://github.com/ffmpegwasm/FFmpeg/releases/tag/n4.3.1-p3)

请注意，目前它只是 Node.js 版本，但我们将在[构建 FFmpeg WebAssembly 版本(= FFmpeg . wasm):part . 4 FFmpeg . wasm v 0.2-添加 Libx264](https://medium.com/@jeromewus/build-ffmpeg-webassembly-version-ffmpeg-js-part-4-ffmpeg-js-v0-2-web-worker-and-libx264-d0596f1beb4e) 中开发一个浏览器版本

期待在第四部分见到你😃