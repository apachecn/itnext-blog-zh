# 如何构建一个简单的 Vue CLI 插件

> 原文：<https://itnext.io/how-to-build-a-simple-vue-cli-plugin-a2e1323de1a0?source=collection_archive---------1----------------------->

![](img/3618ce233d15563ee87dab29952766e3.png)

如果你正在使用 Vue 框架，你可能已经知道什么是 Vue CLI。这是一个用于快速 Vue.js 开发的完整系统，提供项目脚手架和原型。

cli 的一个重要部分是 CLI 插件。他们可以修改内部 webpack 配置，并向`vue-cli-service`注入命令。一个很好的例子是一个`@vue/cli-plugin-typescript`:当你调用它时，它会给你的项目添加一个`tsconfig.json`并将`App.vue`改为有类型，所以你不需要手动操作。

插件非常有用，现在有很多插件用于不同的情况——[graph QL+Apollo](https://github.com/Akryum/vue-apollo)支持,[electronic builder](https://github.com/nklayman/vue-cli-plugin-electron-builder)添加 UI 库，如 [Vuetify](https://github.com/vuetifyjs/vue-cli-plugin-vuetify) 或 [ElementUI](https://github.com/ElementUI/vue-cli-plugin-element) …但是如果你想为某个特定的库安装一个插件，而它并不存在呢？这是我的案子😅…我决定自己动手建造。

在本文中，我们将构建一个 [vue-cli-plugin-rx](https://github.com/NataliaTepluhina/vue-cli-plugin-rx) 。它允许我们向我们的项目添加一个`vue-rx`库，并在我们的 Vue 应用程序中获得 RxJS 支持。

# 🎛️ Vue-cli 插件结构

首先，什么是 CLI 插件？只是一个有一定结构的 npm 包。关于[文档](https://cli.vuejs.org/dev-guide/plugin-dev.html#core-concepts)，它**必须**有一个服务插件作为它的主要导出，**可以**有额外的特性，如生成器和提示文件。

> 现在还不清楚什么是服务插件或生成器，但是不用担心——稍后会解释的！

当然，像任何 npm 包一样，CLI 插件的根文件夹中必须有一个`package.json`，最好有一个带描述的`README.md`。

所以，让我们从下面的结构开始:

```
.
├── README.md
├── index.js      # service plugin
└── package.json
```

现在让我们看看可选部分。一个**生成器**可以将额外的依赖项或字段注入到`package.json`中，并将文件添加到项目中。我们需要它吗？当然，我们希望将`rxjs`和`vue-rx`添加为我们的依赖项！更确切地说，如果用户想在插件安装期间添加组件，我们想创建一些示例组件。因此，我们需要添加`generator.js`或`generator/index.js`。我更喜欢第二种方式。现在结构看起来像这样:

```
.
├── README.md
├── index.js      # service plugin
├── generator
│   └── index.js  # generator
└── package.json
```

还有一点要添加的是一个提示文件:我希望我的插件询问用户是否想要一个示例组件。我们需要在根文件夹中有一个`prompts.js`文件来实现这个行为。因此，现在的结构如下所示:

```
├── README.md
├── index.js      # service plugin
├── generator
│   └── index.js  # generator
├── prompts.js    # prompts file
└── package.json
```

# ⚙️服务插件

服务插件应该导出一个接收两个参数的函数:一个 PluginAPI 实例和一个包含项目本地选项的对象。它可以扩展/修改不同环境的内部 webpack 配置，并向`vue-cli-service`注入额外的命令。让我们思考一下:我们是想以某种方式改变 webpack 配置还是创建一个额外的 npm 任务？答案是**否**，如果有必要，我们只想添加一些依赖项和示例组件。所以我们在`index.js`中需要改变的是:

```
module.exports = (api, opts) => {}
```

如果您的插件需要更改 webpack 配置，请阅读官方 Vue CLI 文档中的[这一节](https://cli.vuejs.org/dev-guide/plugin-dev.html#service-plugin)。

# 🛠️通过生成器添加依赖项

如上所述，CLI 插件生成器帮助我们添加依赖项和更改项目文件。所以，我们需要做的第一步是让我们的插件增加两个依赖项:`rxjs`和`vue-rx`:

```
module.exports = (api, options, rootOptions) => {
  api.extendPackage({
    dependencies: {
      'rxjs': '^6.3.3',
      'vue-rx': '^6.0.1',
    },
  });
}
```

生成器应该导出一个接收三个参数的函数:一个 GeneratorAPI 实例，生成器选项，如果用户使用某个预置创建一个项目，整个预置将作为第三个参数传递。

`api.extendPackage`方法扩展了项目的`package.json`。嵌套字段是深度合并的，除非您将`{ merge: false }`作为参数传递。在我们的例子中，我们向`dependencies`部分添加了两个依赖项。

现在我们需要改变一个`main.js`文件。为了让 RxJS 在 Vue 组件内部工作，我们需要导入一个`VueRx`并调用`Vue.use(VueRx)`

首先，让我们创建一个要添加到主文件中的字符串:

```
let rxLines = `\nimport VueRx from 'vue-rx';\n\nVue.use(VueRx);`;
```

现在我们将使用`api.onCreateComplete` hook。当文件被写入磁盘时，它被调用。

```
api.onCreateComplete(() => {
    // inject to main.js
    const fs = require('fs');
    const ext = api.hasPlugin('typescript') ? 'ts' : 'js';
    const mainPath = api.resolve(`./src/main.${ext}`);
};
```

这里我们寻找主文件:如果它是一个 TypeScript 项目，它将是一个`main.ts`，否则它将是一个`main.js`文件。`fs`这里是一个文件系统。

现在让我们改变文件内容

```
api.onCreateComplete(() => {
    // inject to main.js
    const fs = require('fs');
    const ext = api.hasPlugin('typescript') ? 'ts' : 'js';
    const mainPath = api.resolve(`./src/main.${ext}`); // get content
    let contentMain = fs.readFileSync(mainPath, { encoding: 'utf-8' });
    const lines = contentMain.split(/\r?\n/g).reverse(); // inject import
    const lastImportIndex = lines.findIndex(line => line.match(/^import/));
    lines[lastImportIndex] += rxLines; // modify app
    contentMain = lines.reverse().join('\n');
    fs.writeFileSync(mainPath, contentMain, { encoding: 'utf-8' });
  });
};
```

这里发生了什么事？我们正在读取主文件的内容，将它分成几行并恢复它们的顺序。然后，我们搜索带有`import`语句的第一行(这将是第二次还原后的最后一行),并在那里添加我们的`rxLines`。在这之后，我们反转行的数组并保存文件。

# 💻在本地测试 cli 插件

让我们在`package.json`中添加一些关于我们插件的信息，并试着安装到本地进行测试。最重要的信息通常是插件名称和版本(将插件发布到 npm 时需要这些字段)，但是可以随意添加更多信息！`package.json`字段的完整列表可以在[这里](https://docs.npmjs.com/files/package.json)找到。以下是我的文件:

```
{
  "name": "vue-cli-plugin-rx",
  "version": "0.1.5",
  "description": "Vue-cli 3 plugin for adding RxJS support to project using vue-rx",
  "main": "index.js",
  "repository": {
    "type": "git",
    "url": "git+https://github.com/NataliaTepluhina/vue-cli-plugin-rx.git"
  },
  "keywords": [
    "vue",
    "vue-cli",
    "rxjs",
    "vue-rx"
  ],
  "author": "Natalia Tepluhina <my@mail.com>",
  "license": "MIT",
  "homepage": "https://github.com/NataliaTepluhina/vue-cli-plugin-rx#readme"
}
```

现在是时候检查我们的插件是如何工作的了！为此，让我们创建一个简单的基于 vue-cli 的项目:

```
vue create test-app
```

转到项目文件夹，安装我们新创建的插件:

```
cd test-app
npm install --save-dev file:/full/path/to/your/plugin
```

插件安装后，您需要调用它:

```
vue invoke vue-cli-plugin-rx
```

现在，如果您尝试检查`main.js`文件，您可以看到它发生了变化:

```
import Vue from 'vue'
import App from './App.vue'
import VueRx from 'vue-rx';Vue.use(VueRx);
```

同样，你可以在你的测试应用`package.json`的`devDependencies`部分找到你的插件。

# 📂使用生成器创建新组件

太好了，一个插件工作了！是时候扩展一下它的功能，让它能够创建一个示例组件了。生成器 API 为此使用了一个`render`方法。

首先，让我们创建这个示例组件。它应该是位于项目`src/components`文件夹中的`.vue`文件。在`generator`文件夹中创建一个`template`文件夹，然后在其中模仿整个结构:

![](img/a6ba0206291bb92d5b6afbc3076d4cc7.png)

您的示例组件应该是…嗯，只是一个 Vue 单文件组件！在本文中，我不会深入 RxJS 的解释，但是我创建了一个简单的基于 RxJS 的点击计数器，它有两个按钮:

![](img/d249b0b12d0fe9149c81bf8c4dd37ee8.png)

它的源代码可以在这里找到[。](https://github.com/NataliaTepluhina/vue-cli-plugin-rx/blob/master/generator/template/src/components/RxExample.vue)

现在我们需要指示插件在调用时呈现这个组件。为此，让我们将这段代码添加到`generator/index.js`:

```
api.render('./template', {
  ...options,
});
```

这将渲染整个`template`文件夹。现在，当插件被调用时，一个新的`RxExample.vue`将被添加到`src/components`文件夹中。

> 我决定不覆盖一个 `*App.vue*` *，让用户自己添加一个示例组件。但是，您可以替换部分现有文件，参见文档* 中的 [*示例*](https://cli.vuejs.org/dev-guide/plugin-dev.html#generator)

# ⌨️用提示处理用户选择

如果用户不想要一个示例组件呢？让用户在插件安装过程中决定这一点不是很明智吗？这就是提示存在的原因！

之前我们已经在插件根文件夹中创建了`prompts.js`文件。这个文件应该包含一个问题数组:每个问题都是一个具有特定字段集的对象，如`name`、`message`、`choices`、`type`等。

> 名称很重要:我们稍后将在 generator 中使用它来创建一个呈现示例组件的条件！

提示可以有[不同的类型](https://github.com/SBoudrias/Inquirer.js#prompt-types)，但是在 CLI 中使用最广泛的是`checkbox`和`confirm`。我们将使用后者来创建一个带有是/否答案的问题。

所以，我们把我们的提示码加到`prompts.js`里吧！

```
module.exports = [
  {
    name: `addExample`,
    type: 'confirm',
    message: 'Add example component to components folder?',
    default: false,
  },
];
```

我们有一个`addExample`提示，询问用户是否想要添加一个组件到组件文件夹。默认答案是`No`。

让我们回到生成器文件并做一些修正。将`api.render`调用替换为

```
if (options.addExample) {
    api.render('./template', {
      ...options,
    });
}
```

我们正在检查`addExample`是否有肯定的答案，如果有，组件将被渲染。

> 不要忘记在每次修改后重新安装并测试你的插件！

# 📦公之于众！

重要提示:在发布你的插件之前，请检查它的名字是否与模式`vue-cli-plugin-<YOUR-PLUGIN-NAME>`匹配。这允许你的插件被`@vue/cli-service`发现并通过`vue add`安装。

我还希望我的插件在 Vue CLI UI 中有一个漂亮的外观，所以我给`package.json`添加了描述、标签和库名，并创建了一个徽标。Logo 图片应该命名为`logo.png`，放在插件根文件夹中。因此，我的插件在 UI 插件列表中看起来是这样的:

![](img/6098406612aa2a15dac94ee04074f259.png)

现在我们准备出版了。你需要注册一个 npmjs.com，显然你应该安装 npm。

要发布一个插件，进入插件根文件夹，在终端输入`npm publish`。瞧，你刚刚发布了一个 npm 包！

这时你应该可以用`vue add`命令安装插件了。试试看！

当然，本文中描述的插件是非常基础的，但我希望我的说明能帮助某人开始 cli 插件开发。

你缺少哪种命令行界面插件？请在评论中分享你的想法🤗