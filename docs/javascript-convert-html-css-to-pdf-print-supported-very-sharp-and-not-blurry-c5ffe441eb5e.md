# Javascript:将 HTML + CSS 转换成 PDF。几秒钟内打印 HTML

> 原文：<https://itnext.io/javascript-convert-html-css-to-pdf-print-supported-very-sharp-and-not-blurry-c5ffe441eb5e?source=collection_archive---------0----------------------->

![](img/a55830efee8cf3eba105480d47a5fd2c.png)

这可能是你的 HTML

我目前正在做的一个个人项目涉及到将一个样式化的 HTML 节点树转换成客户端可打印的 PDF 资产。

我花了很长时间寻找最终的解决方案，经历了很多磨难。

我将与您分享我的解决方案，我相信它非常容易使用，并且我将讨论其他方法的优缺点。

只需两分钟即可实现。

# 设置

> ***仅供参考:*** *我使用 React，所以我使用* `*npm / yarn*` *来添加库，使用 ES6* `*import*` *语句。不使用那些包管理器，你可以使用一个简单的* `*<script></script>*` *标签来集成库。*

它将带你:

*   两个运行在客户端的 Javascript 库。
*   Javascript 的 5 行实际代码

它要求:

*   基本的 Javascript 知识
*   **jsPDF:**
*   **html2canvas :** `yarn add html2canvas`

# 小片

流程如下:

*   您想要转换成 PDF 的 HTML 节点树首先使用`html2canvas`**转换成画布(第 4 行)**
*   然后，**使用 A4 格式创建空的 PDF 结构。顺便说一下，我们告诉 jsPDF 使用`mm`作为下一个操作的单位(第 5 行)**
*   差不多完成了，我们**使用`canvas.toDataURL('image/png')`(第 6 行)把画布变成 PNG 图像**。
*   然后，该 P **NG 图像被粘贴到坐标`(0,0)`处的空 PDF** 中，并在`(211,298)`处调整大小(第 6 行)。A4 幅面宽 210 毫米，高 297 毫米，所以我在粘贴图像时多使用了一毫米，以避免边缘出现白条。
*   最后，我们提示浏览器使用`pdf.save(filename)`(第 7 行)下载新生成的 PDF。

另一个函数将`quality`作为参数，从 1(默认质量)开始，允许您获得具有 SVG 外观的更清晰的 PDF。

这个`quality`只是用来首先把 HTML 节点树变成画布的尺度。它越大，PDF 就越清晰和沉重(在我的一个复杂节点结构上使用`quality = 4`产生 60 MB 的 PDF，而`quality = 1`产生 800 KB 的 PDF)。

# 考虑

这项技术可以让你用 5 行代码在几秒钟内打印出基于 HTML 的 pdf。

它已经在多个浏览器上进行了测试，截至 2018 年 8 月 1 日，使用我的 Chrome 浏览器没有遇到任何问题。

尽管如此，请考虑以下几点:

*   这是一个 PDF 格式的图像。不是实际的文本。
*   这些生成的 PDF 文件不允许您选择/复制/粘贴文本，因为其中没有文本。超链接等也是如此。
*   随着质量的提高，您的 pdf 可能会变得非常沉重。

**仍然:**

*   非常容易实现
*   既不是代码重组，也不是调整。
*   在客户端像 charm 一样工作，利用了两个久经考验的库(jsPDF 和 html2canvas)。
*   使用 3 或 4 作为`quality`修饰符，你的 pdf 看起来会像 SVG 文件。超清晰无模糊。
*   没有多少库允许直接的 HTML + CSS 到 PDF 的转换。

# 感谢阅读

告诉我它是否像救了我的命一样救了你的命。

查看我的最新文章:

[](https://dmware.fr/disable-categories-tags-and-author-pages-on-your-wordpress-website/) [## 禁用 WordPress 网站上的类别、标签和作者页面

### 我在网上找不到任何简单的超快解决方案，所以这是我的。它将带你:它假设…

dmware.fr](https://dmware.fr/disable-categories-tags-and-author-pages-on-your-wordpress-website/) 

欢迎致电 **david.mellul@outlook.fr ☕️😃**

![](img/964621a3b2a0bdf3a7f1510106bd3b41.png)

“白色马克杯中的卡布奇诺，木质桌子上的白色泡沫艺术”作者[吴仪](https://unsplash.com/@takeshi2?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上