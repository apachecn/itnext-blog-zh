# 故事书:技巧和窍门

> 原文：<https://itnext.io/storybookjs-tips-tricks-785bc39aacbe?source=collection_archive---------1----------------------->

## 将 Markdown 文件导入为 Docs only 页面，使用 CDN 加载依赖关系和排序故事。

![](img/f8062f7d3a337cba70ef6fb8dbbc2a85.png)

照片由[弗洛伦西亚·维亚达纳](https://unsplash.com/@florenciaviadana?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)在 [Unsplash](https://unsplash.com/s/photos/books?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 拍摄

我只是把 DeckDeckGo 的[文档](https://docs.deckdeckgo.com)移植到了 [StorybookJS](https://storybook.js.org/) 中。除了能够简化构建和测试之外，我还喜欢将它用于文档目的，因为它允许我集成由 [StencilJS](https://stenciljs.com/) 自动生成的 README.md 文件。代码和文档之间的对练是最好的，你不这样认为吗？

以下是我在这个过程中重复使用或发现的一些技巧和诀窍。

# 将降价文件导入 StorybookJS

StencilJS 的一个特别酷的特性是，它从代码的注释中自动生成 markdown 中的`readme.md`文件。这不是很棒吗？

我觉得是。因此，为了进一步推进概念，我设置了 StorybookJS 来导入这些 Markdown 文件作为 Docs only 页面。这样，文档就保留了下来，并尽可能地按照代码进行编辑，然后交付给最终用户，没有任何中断。

## 元

在编写 StorybookJS 问题的时候 [#11981](https://github.com/storybookjs/storybook/issues/11981) 列出各种各样的解决方案，包括使用`transcludeMarkdown`设置或 raw 加载器。如果下面的解决方案不适合你，尝试其中一个。

## 限制

尽管它工作得很好，我还是没能突出显示页面中显示的和从 Markdown 文件导入的代码块。我相应地评论了这个问题。

如果你设法解决了这个问题，让我现在或者，更好的是，在 GitHub 上给我发一个[拉请求](https://github.com/deckgo/deckdeckgo/)😉。

## 解决办法

我使用 HTML 版本的故事书，我在`.js`文件中处理我的故事，例如在`Text.stories.js`文件中，我记录了一段接受背景颜色作为参数的段落。

```
export default {
  title: 'Components/Text',
  argTypes: {
    bg: {control: 'color'}
  }
};

export const Text = ({bg}) => {
  return `<p style="background: ${bg};">
    Hello World
  </p>`;
};

Text.args = {
  bg: '#FF6900'
};
```

根据 StorybookJS 的说法，我们可以在组件级替换 DocsPage 模板，用 MDX 文档或定制组件展示我们自己的文档。这就是为什么，在我们的故事旁边，我们创建了一个新文件`Text.mdx`，我们将它导入并作为`page`提供给我们的故事。

```
import {Doc} from './Text.mdx';

export default {
  title: 'Components/Text',
  parameters: {
    docs: {
      page: Doc
    }
  },
  argTypes: {
    bg: {control: 'color'}
  }
};

export const Text = ({bg}) => {
  return `<p style="background: ${bg};">
    Hello World
  </p>`;
};

Text.args = {
  bg: '#FF6900'
};
```

最后，在我们的`.mdx`文件中，我们导入我们的`README.md`文件(或者任何其他的降价文件),并且，我们使用基本的故事书`Description`模块，将文档页面与定制文档重新混合。

```
import {Description} from '@storybook/addon-docs/blocks';

import readme from './readme.md';

export const Doc = () => <Description markdown={readme} />;
```

就这样，降价文件被整合为 StorybookJS 🥳.的文档页面

## 使用 CDN 加载依赖项

不确定是否有人会有这样的需求，但是如果像我一样，您需要从 CDN 加载依赖项，这里有一个技巧:将您的`script`添加到`./storybook/preview-head.html`。它会用你的故事来评价。

同样，如果你想为你的组件定义一些`style`或者加载一个特定的 Google 字体，你也可以修改同一个文件。

一些例子摘自我的[preview-head.html](https://github.com/deckgo/deckdeckgo/blob/main/docs/.storybook/preview-head.html)文件:

```
<link rel="preconnect" href="https://fonts.gstatic.com" />
<link href="https://fonts.googleapis.com/css2?family=JetBrains+Mono&display=swap" rel="stylesheet" />

<script type="module" src="https://unpkg.com/@deckdeckgo/color@latest/dist/deckdeckgo-color/deckdeckgo-color.esm.js"></script><style>
  pre:not(.prismjs) > div {
     box-shadow: none;
     margin: 25px 0;
  }
</style>
```

## 分类故事

使用属性`storySort`可以在`./storybook/preview.js`中定义故事的特定顺序。每一章都必须作为`string`提供，它们的故事列表作为`array`。

```
import theme from './theme';

export const parameters= {
  actions: {argTypesRegex: '^on[A-Z].*'},
  options: {
    storySort: {
      order: [
        'Introduction',
        ['Introduction', 'Getting Started'],
        'Edit',
        ['HTML', 'Lazy Loading']
      ]
    }
  },
  docs: {
    theme
  }
};
```

名称应与故事中提供的`title`相匹配。

用`MDX`使用`meta`:

```
import {Meta} from '@storybook/addon-docs/blocks';

<Meta title="Introduction/Getting Started" />

*# Getting started*
```

用`JS`通过默认`title`:

```
export default {
  title: 'Components/Lazy Image',
  argTypes: {
    imgSrc: {control: 'text'}
  }
};
```

# 摘要

[stencil js](https://stenciljs.com/)+[storybook js](https://storybook.js.org/)=牛逼💪

到无限和更远的地方！

大卫

你可以在推特上或我的网站上找到我。

为您的下一张幻灯片尝试一下 [DeckDeckGo](https://deckdeckgo.com/) 🤟。

[![](img/14fb201662569938ef9ae2beeaadafc7.png)](https://deckdeckgo.com)