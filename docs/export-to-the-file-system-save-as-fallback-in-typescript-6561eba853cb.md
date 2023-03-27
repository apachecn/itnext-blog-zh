# 导出到文件系统(另存为…)+TypeScript 中的回退

> 原文：<https://itnext.io/export-to-the-file-system-save-as-fallback-in-typescript-6561eba853cb?source=collection_archive---------0----------------------->

## 如何使用新的文件系统访问 API 和针对不兼容浏览器的回退将文件保存到用户的本地设备。

![](img/bc038dcfb8814983df583a5723d716ac.png)

[伊万·迪亚兹](https://unsplash.com/@ivvndiaz?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)在 [Unsplash](https://unsplash.com/?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上的照片

在几乎每个 web 应用程序中，我最终都重复使用相同的模式以 JavaScript 将数据导出到文件系统——即使用[文件系统访问 API](https://developer.mozilla.org/en-US/docs/Web/API/File_System_Access_API) 和一个很好的旧“下载”特性作为后备的解决方案。我认为写一篇关于它的文章作为文档是值得的😉。

# 介绍

文件系统访问 API 允许在您的浏览器中进行读取、写入和文件管理。它使开发人员能够构建强大的 web 应用程序，与用户本地设备上的文件进行交互。

web.dev 团队有一个[教程](https://web.dev/file-system-access/)介绍并强调了所有的特性。​

它是一个相对较新的 API，因此还没有被所有的浏览器厂商采用。​

例如，我们使用的一个关键功能——`showSaveFilePicker`打开一个对话框，选择将写入用户本地驱动器的文件的目的地——只有 Edge、Chrome 和 Opera 支持(2022 年 2 月——来源[可以使用](https://caniuse.com/?search=showSaveFilePicker))。

# 入门指南

一般来说，我使用打字稿。这个解决方案也提供了类型安全。这就是为什么它需要首先安装文件系统访问 API 的类型定义。

```
npm i -D @types/wicg-file-system-access
```

# 亲自动手

为了导出文件，我使用了一个`Blob`——也就是我想要导出的文件的内容——和一个`filename`。我创建了一个功能，可以保存到用户的本地设备，并可以在我的应用程序中使用。

```
export const save = (data: {blob: Blob, filename: string}) => {
    if ('showSaveFilePicker' in window) {
        return exportNativeFileSystem(data);
    }

    return download(data);
};
```

# 文件系统访问 API —另存为

​

上述功能测试`showSaveFilePicker`在`window`对象中是否可用——即检查浏览器是否支持文件系统访问 API。​

为了用新的 API 保存文件，我们首先以“保存”模式向用户显示一个对话框。使用它，用户可以选择保存文件的位置。一旦设置了路径，文件就可以有效地写入本地驱动器。

```
const exportNativeFileSystem =
        async ({blob, filename}: {blob: Blob, filename: string}) => {
    const fileHandle: FileSystemFileHandle =
        await getNewFileHandle({filename});

    if (!fileHandle) {
        throw new Error('Cannot access filesystem');
    }

    await writeFile({fileHandle, blob});
};
```

在许多情况下，我希望我的应用程序建议一个默认的文件名。这可以通过设置`suggestedName`来实现。此外，我还通过提供 mime 类型和相关的文件扩展名来确定可以选择的文件类型。

```
const getNewFileHandle = 
    ({filename}: {filename: string}): Promise<FileSystemFileHandle> => {
  const opts: SaveFilePickerOptions = {
    suggestedName: filename,
    types: [
      {
        description: 'Markdown file',
        accept: {
          'text/plain': ['.md']
        }
      }
    ]
  };

  return showSaveFilePicker(opts);
};
```

最后，可以使用 API 的另一个函数`writeFile`有效地编写文件。它使用我之前请求的文件句柄来知道将数据导出到文件系统的哪里。

```
const writeFile = 
    async ({fileHandle, blob}: {fileHandle: FileSystemFileHandle, blob: Blob}) => {
  const writer = await fileHandle.createWritable();
  await writer.write(blob);
  await writer.close();
};
```

# 回退—下载

作为后备，我在 DOM 中添加了一个可以自动点击的临时锚元素。为了将文件导出到用户的默认下载文件夹，我提供了一个对象作为`blob`的 URL。

```
const download = async ({filename, blob}: {filename: string; blob: Blob}) => {
  const a: HTMLAnchorElement = document.createElement('a');
  a.style.display = 'none';
  document.body.appendChild(a);

  const url: string = window.URL.createObjectURL(blob);

  a.href = url;
  a.download = `${filename}.md`;

  a.click();

  window.URL.revokeObjectURL(url);
  a.parentElement?.removeChild(a);
};
```

# 获取代码

你可以在我最近发布在 GitHub 上的 Chrome 插件中找到本文中的所有代码👉 [save-utils.ts](https://github.com/papyrs/markdown-plugin/blob/main/src/plugin/utils/save.utils.ts)

# 摘要

这是一个相当短的帖子，我希望至少有点娱乐性🤪。如果你想深入挖掘文件系统访问 API，我再次建议你看看 [web.dev](https://web.dev/file-system-access/) 团队的精彩帖子。​

到无限远处
大卫