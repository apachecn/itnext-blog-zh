# 使用 webpack、react、redux、router、saga 和 postcss 的快速现代前端设置，第 1 部分

> 原文：<https://itnext.io/fast-and-modern-front-end-setup-with-webpack-react-redux-router-saga-and-postcss-1360e9715d17?source=collection_archive---------2----------------------->

![](img/b09a23ab61b80f7a0f50c1ebb74d55a0.png)

注意:webpack 4 已经发布并经过测试，这个设置有点过时，你可以做得更好！！！

> 这是一个系列的第一部分，你可以在以下链接中阅读其他部分:
> - [第二部分](/fast-and-modern-front-end-setup-with-webpack-react-redux-router-saga-and-postcss-part-2-9ae7ad4e7cb2)-[第三部分](/fast-and-modern-front-end-setup-with-webpack-react-redux-router-saga-and-postcss-part-3-27ac4bc3f969)

如果你只是想开始使用设置

```
git clone [https://github.com/bojanaleksa/react-redux-router-saga-material-postcss-webpack](https://github.com/bojanaleksa/react-redux-router-saga-material-postcss-webpack)
```

然后`npm start`在 src/ folder 中编写代码。

# 现在一步一步来

首先，您需要安装 npm 和节点。这不言而喻，但是，我说了。

打开你的终端。我用的是 ubuntu，所以如果你看到一些你不知道的系统命令，那是 ubuntu，无论你用什么都可以查一下。

然后，创建一个新的文件夹，进入并`npm init -y`。我们势如破竹！！！

好的，首先我们将安装 webpack。

```
npm i -D webpack
```

Webpack 是一个模块捆绑器。这意味着，它将使用我们在 webpack.config.js 文件中提供的配置，将一些文件作为输入，并创建其他文件作为输出。让我们继续创建这个文件:

```
touch webpack.config.js
```

我们在这里想要的是有一个文件夹，我们可以在那里写代码，使它可读和组织良好，另一个文件夹，压缩和优化版本的代码将去，为浏览器做好准备。许多优化都是通过 webpack 完成的，它将使最终用户体验更好(主要是在速度方面)。

让我们创建一个名为 src 的文件夹。在其中，创建一个名为 index.js 的文件。

```
mkdir src
cd src
touch index.js
```

在这个文件中，只需放任何 JS 代码，像一些`console.log(‘here we go’);`

## WEBPACK 基本版

现在，我们将让 webpack 获取该文件，并在公共文件夹中创建一个新文件(它也将为我们创建该文件夹)。打开 webpack.config.js 并将其放入:

```
const path = require('path');module.exports = {
    entry: "./src/index.js",
    output: {
        filename: 'bundle.js',
        path: path.resolve(__dirname, 'public')
    }
}
```

module.exports 告诉我们什么将被导出用于其他地方，这是我们的配置对象。第一行导入了一些东西(在本例中是路径)。因此，如你所见，我们可以将代码组织成模块，并随意导入和导出内容。

但是首先，这个路径模块是从哪里来的？也许你在安装 webpack 的时候注意到了一个 node_modules/文件夹。该文件夹包含其他人构建的您(或您想要的模块)使用的所有模块。Webpack 需要 path，因此 path 随它一起安装。你可以随时使用它。

所以，我们的对象有两个道具，入口和输出。非常简单，entry 是我们想要加载的文件(注意。/在位置字符串中)，输出需要文件名和路径。我们使用 path 模块的原因现在很清楚了:我们不想要一个绝对路径，我们希望它相对于我们在文件系统中的位置。

在我们启动 webpack 之前，让我们实际上把它做成一个脚本，这样我们就可以使用 npm 来完成这个任务。进入 package.json 并找到脚本。让它看起来像这样:

```
"scripts": {
    "start": "webpack"
  },
```

现在回到终端，运行`npm start`。该命令运行 webpack，webpack 读取配置文件，创建公共文件夹并填充它。查看 bundle.js 并找到我们的 console.log。这看起来只是添加了更多的代码，但它是值得的。

为了更好地理解 webpack 导入，在 index.js 旁边创建另一个文件，并将其命名为 test.js。

```
export default 5;
```

现在打开 index.js 并:

```
import test from './test';
console.log(test);
```

运行`npm start`test . js 文件被包含在 public/bundle.js 中，并与 index.js 内容捆绑在一起，这就是导入/导出的工作方式。您将只有一个主文件 index.js，并导入您需要的内容。Webpack 将为您优化导入，所有内容都将保存在一个文件中供浏览器使用。

还要注意，webpack 实际上并不运行这些文件，所以在捆绑之后，您可能会在浏览器中看到错误。

现在删除 test.js 并运行`npm start`。检查错误。红色部分告诉您试图导入一个不存在的模块的确切位置。如果你拼错或忘记了，这很方便。

好了，关于 webpack，有很多内容要讲。添加此行

```
context: path.resolve(__dirname, 'src')
```

作为我们对象中的下一个属性，并从条目字符串中删除`/src`。现在看起来是这样的:

```
const path = require('path');module.exports = {
    entry: "./index.js",
    output: {
        filename: 'bundle.js',
        path: path.resolve(__dirname, 'public')
    },
    context: path.resolve(__dirname, 'src')
}
```

这一行添加了包发生的上下文。对我们来说，它是 src/文件夹。

## 超文本标记语言

每个好的前端都需要一个名为 index.html 的 html 文件。我们计划在 src/文件夹中完成所有工作，让我们在那里创建一个。像这样填充它:

```
<!DOCTYPE html>
<html lang="en">
<head>
  <title>Basic Setup</title> <meta charset="utf-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1"><link rel="shortcut icon" type="image/x-icon" href="/favicon.ico"/></head>
<body>
<div id="app"></div>
</body>
</html>
```

正如你注意到的，我们不加载 js，也不会加载 css。Webpack 会帮我们做到这一点。现在，webpack 还需要将这个文件复制到 public/文件夹中。为此，我们将使用 webpack 的一些插件。Webpack 有很多插件，它们扩展了基本功能。让我们安装我们需要的东西

```
npm i -D html-webpack-plugin
```

在我们的 webpack.config.js 中，加载 path 模块下的插件

```
const HtmlWebpackPlugin = require('html-webpack-plugin');
```

在配置对象中，添加一个插件数组，并在那里使用我们的插件。现在看起来都是这样的

```
const path = require('path');const HtmlWebpackPlugin = require('html-webpack-plugin');module.exports = {
    entry: "./index.js",
    output: {
        filename: 'bundle.js',
        path: path.resolve(__dirname, 'public')
    },
    context: path.resolve(__dirname, 'src'),
    plugins: [
     new HtmlWebpackPlugin({
            template: 'index.html'
        })
    ]
}
```

我们为 index.html 定义了模板。现在再次运行`npm start`，在 public/文件夹中查找。注意，我们现在将 bundle.js 加载到 index.html 的 public/ folder 中。如我所说，让 webpack 做它的工作。

很好，现在我们有了一个可以在浏览器中运行的文件。如果这样做，您会注意到它运行 index.js. Console 中的代码，以确保这一点。

## 开发服务器

接下来我们需要一个开发服务器。如果我们每次修改东西都要运行`npm start`并在浏览器中打开文件，那就糟了。最好让 webpack 也这么做。Webpack 有个东西叫 webpack-dev-server，我们来安装一下:

```
npm i -D webpack-dev-server
```

在 package.json 里面，把`“start”: “webpack”`改成`“start”: “webpack-dev-server”`。现在再次运行`npm start`，它会告诉你在 URL:[http://localhost:8080/](http://localhost:8080/)打开浏览器。你可以在这里看到你的项目。在 index.js 中更改一些内容，然后返回浏览器。它会立即更新。现在你可以工作并检查你的进度，中间没有任何步骤。那真是太好了。

如果要停止服务器，只需在终端中按 ctrl-c 即可。制作另一个脚本，只构建应用程序而不运行它，我们以后会需要它。现在 package.json 脚本部分如下所示:

```
"scripts": {
    "start": "webpack-dev-server",
    "build": "webpack",
    "build-prod": "webpack -p"
  },
```

如果您运行`npm run build`，您将看到与之前相同的内容。但是 build-prod 脚本有-p 标志，它告诉 webpack 为生产进行优化。运行`npm run build-prod`并检查 bundle.js。现在它已经被压缩并为浏览器做了更好的优化。但是，我们将使用开发模式来开发，因为它更容易。

让我们做得更好，让 webpack 为我们打开浏览器标签。在 webpack.config.js 中添加以下属性:

```
devServer: {
   contentBase: path.resolve(__dirname, 'public/assets'),
   stats: 'errors-only',
   open: true,
   port: 8080,
   compress: true
},
```

运行`npm start`，欣赏它如何在你的浏览器中自己打开一个标签。我们在这里所做的是，我们为开发服务器添加了一个配置，告诉它提供来自/public/assets 的图像，只显示错误，打开特定的端口，gzip 压缩输出。

## 静态资产(图像……)

我们希望能够服务静态资产，比如图像。让我们在 src/中创建一个资产/文件夹，并在其中放置一张图片。我们应该能够在 index.js 中得到这样的图像:

```
import img from './assets/image.png';
console.log(img);
```

但是如果我们运行`npm run build`我们会得到一个错误。这是因为不知道这是一个静态文件，它会尝试分析它。让我们告诉 webpack 在这里做什么。我们需要一种叫做装载机的东西。webpack 中的加载器是一个用于预处理某些文件(如图像)的工具。让我们安装它:

```
npm i -D file-loader
```

现在用靠近底部的行扩展 webpack.config.js。粘贴整个文件:

```
const path = require('path');const HtmlWebpackPlugin = require('html-webpack-plugin');module.exports = {
    entry: "./index.js",
    output: {
        filename: 'bundle.js',
        path: path.resolve(__dirname, 'public')
    },
    context: path.resolve(__dirname, 'src'),
    devServer: {
        contentBase: path.resolve(__dirname, 'public/assets'),
        stats: 'errors-only',
        open: true,
        port: 8080,
        compress: true
    },
    plugins: [
        new HtmlWebpackPlugin({
            template: 'index.html'
        })
    ],
    module: {
        rules: [{
            test: /\.(jpg|png|gif|svg)$/,
            use: [
            {
                loader: 'file-loader',
                options: {
                    name: '[name].[ext]',
                    outputPath: './assets/',
                }
            }]
        }]
    }
}
```

您可以看到，我们在规则道具中添加了一个模块道具。这就是 webpack3 的组织方式，不用担心。这里的要点是，rules 是一个数组，你向它传递加载器。每个加载器都需要一个测试来检查它是否应该在文件上运行，以及一个带有选项的实际加载器模块。我们在这里看到，我们希望文件加载器接管图像文件。运行`npm run build`并检查公共/文件夹。那里会有一个资产文件夹，里面有我们的图像。如果图像被导入到代码中的某个地方，Webpack 将只在这里复制图像。您也可以运行`npm start`，因为我们为静态文件配置了 assets 文件夹。

## 清理网络包

现在，让我们说你改变或删除一个图像，或一些其他文件。Webpack 之前已经复制过了。现在你不用了，它还在，占地方。将 public/文件夹复制到生产服务器会更容易，而不用担心额外的文件。这有一个插件:

```
npm i -D clean-webpack-plugin
```

在 webpack.config.js 中设置它

```
const CleanWebpackPlugin = require('clean-webpack-plugin');
...
plugins: [
new CleanWebpackPlugin(['public']),
...
]
```

我们告诉这个插件在它重建的时候删除公共文件夹，这样只有我们实际使用的文件夹才能进入。如果你担心开发速度，开发服务器实际上并不创建文件夹，正如你将看到的。运行`npm start`你会注意到 public/ folder 不在这里，但是我们可以加载所有的图像和文件。这是为了使开发更快，这样代码中的每一个变化都会立即显示在浏览器中。

但是如果您运行`npm run build`，您将再次看到 public/ folder，这显示了开发和生产之间的区别。

## 添加 CSS 文件

我们希望能够将样式与代码的其余部分分开。我们将通过一种允许我们在任何需要的地方加载. css 文件的方式来实现。

让我们在 src/ folder 中做一个名为 styles 的文件夹，在里面放一个 main.css 文件。在文件中放一些 css，像这样

```
body {
 background: red;
}
```

我们也可以在 index.js 中导入这个文件

```
import './styles/main.css';
```

注意文件的路径。现在运行`npm start`，你会得到一个错误:“你可能需要一个合适的加载程序来处理这种文件类型。”很好，我们知道该怎么做了，让我们从终端添加 css 加载器

```
npm i -D css-loader
```

并将加载器规则添加到规则属性内的 webpack.config.js 中

```
const path = require('path');const CleanWebpackPlugin = require('clean-webpack-plugin');
const HtmlWebpackPlugin = require('html-webpack-plugin');module.exports = {
    entry: "./index.js",
    output: {
        filename: 'bundle.js',
        path: path.resolve(__dirname, 'public')
    },
    context: path.resolve(__dirname, 'src'),
    devServer: {
        contentBase: path.resolve(__dirname, 'public/assets'),
        stats: 'errors-only',
        open: true,
        port: 8080,
        compress: true
    },
    plugins: [
     new CleanWebpackPlugin(['public']),
        new HtmlWebpackPlugin({
            template: 'index.html'
        })
    ],
    module: {
        rules: [{
            test: /\.(jpg|png|gif|svg)$/,
            use: [
            {
                loader: 'file-loader',
                options: {
                    name: '[name].[ext]',
                    outputPath: './assets/',
                }
            }]
        }, {//HERE ARE THE NEW LINES
         test: /\.css$/,
         use: "css-loader"
        }]
    }
}
```

如果我们运行`npm start`，我们的代码将运行并打开浏览器。但是背景不是红色的。这是因为我们现在有了一个加载器来处理 css 代码，但是它不能单独存在于 js 文件中。我们需要一种方法，以一种普通的方式将它连接到代码上。回到终端:

```
npm i -D style-loader
```

这个加载器实际上将样式加载到系统中。让我们编辑配置:

```
...{
   test: /\.css$/,
   use: ["style-loader", "css-loader"]
}
```

好的，首先注意我们第一次是如何为 use:属性使用一个对象的，然后是一个字符串，现在是一个数组。是不是很好听:d。

还要注意。我们在这里传递的 css 文件将由两个加载器处理，但最后一个加载器先处理，所以我们首先处理 css 代码，然后用 style-loader 连接它。如果我们现在运行`npm start`，我们的背景将是红色的。但是如果我们运行`npm run build`，你会注意到。css 文件无处可寻。这是因为 css 代码被 js 插入了一个<样式的>标签。最好能有一个单独的文件供浏览器加载，这就是我们样式设置的第三部分:

```
npm i -D extract-text-webpack-plugin
```

顾名思义，插件会将任何文本提取到一个单独的文件中。在 webpack.config.js 中，我们添加了这个插件

```
const path = require('path');const CleanWebpackPlugin = require('clean-webpack-plugin');
const HtmlWebpackPlugin = require('html-webpack-plugin');
// NEW LINE
const ExtractTextPlugin = require('extract-text-webpack-plugin');module.exports = {
    entry: "./index.js",
    output: {
        filename: 'bundle.js',
        path: path.resolve(__dirname, 'public')
    },
    context: path.resolve(__dirname, 'src'),
    devServer: {
        contentBase: path.resolve(__dirname, 'public/assets'),
        stats: 'errors-only',
        open: true,
        port: 8080,
        compress: true
    },
    plugins: [
     new CleanWebpackPlugin(['public']),
        new HtmlWebpackPlugin({
            template: 'index.html'
        }),
        // NEW LINE
        new ExtractTextPlugin({
         filename: './style.css'
        })
    ],
    module: {
        rules: [{
            test: /\.(jpg|png|gif|svg)$/,
            use: [
            {
                loader: 'file-loader',
                options: {
                    name: '[name].[ext]',
                    outputPath: './assets/',
                }
            }]
        }, {
         test: /\.css$/,
         use: ["style-loader", "css-loader"]
        }]
    }
}
```

好了，我们导入了插件，并将其添加到插件数组中，指定了输出文件，然后我们运行`npm run build`。恐怖的是，public/ folder 中没有 css 文件。我们实际上需要一个加载器来告诉 webpack 它的文件进入了插件，所以我们将稍微重写一下配置。

```
const path = require('path');const CleanWebpackPlugin = require('clean-webpack-plugin');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const ExtractTextPlugin = require('extract-text-webpack-plugin');
// WE MOVED THE INSTANCE HERE, SO WE CAN USE IT
const extractPlugin = new ExtractTextPlugin({
    filename: './style.css'
});module.exports = {
    entry: "./index.js",
    output: {
        filename: 'bundle.js',
        path: path.resolve(__dirname, 'public')
    },
    context: path.resolve(__dirname, 'src'),
    devServer: {
        contentBase: path.resolve(__dirname, 'public/assets'),
        stats: 'errors-only',
        open: true,
        port: 8080,
        compress: true
    },
    plugins: [
        new CleanWebpackPlugin(['public']),
        new HtmlWebpackPlugin({
            template: 'index.html'
        }),
        // WE JUST PASS IT HERE
        extractPlugin
    ],
    module: {
        rules: [{
            test: /\.(jpg|png|gif|svg)$/,
            use: [
            {
                loader: 'file-loader',
                options: {
                    name: '[name].[ext]',
                    outputPath: './assets/',
                }
            }]
        }, {
            test: /\.css$/,
            // AND WE USE IT HERE
            use: extractPlugin.extract({
             use: ["css-loader"],
             fallback: 'style-loader'
            })
        }]
    }
}
```

我们首先实例化了插件，这意味着如果需要，我们可以在其他地方使用另一个实例。请注意加载器中的配置:我们只使用 css 加载器，但是样式加载器只是一个后备，这很好。运行`npm run build`并检查公共/文件夹。现在我们的 style.css 文件就在那里，它在 index.html 文件中为我们链接。全部完成:d。

## 一些额外的 CSS 东西

让我们为 css 添加两个更好的加载器:scss 和 postcss。您可以了解它们并利用它们的优势，但是让我们快速安装这些东西

```
npm i -D postcss-loader sass-loader node-sass autoprefixer cssnano postcss-import postcss-cssnext
```

我们将使用。scss 扩展名，因此将 main.css 重命名为 main.scss，并更改 index.js 中的 import 命令以匹配更改后的名称。在主文件夹(webpack.config.js 所在的位置)中创建一个 postcss.config.js 文件，并将它放入

```
module.exports = {
  plugins: {
    'autoprefixer': {},
    'cssnano': {},
    'postcss-import': {},
    'postcss-cssnext': {}
  }
}
```

我们定义了 postcss 要使用的插件。这里最重要的是 autoprefixer，它会在需要的地方自动添加供应商前缀，比如 webkit 和 moz。

我们现在将这两个加载器添加到我们的 webpack 配置中，与 css-loader 放在一起:

```
use: ["css-loader", "sass-loader", "postcss-loader"],
```

而且我们可以随意运行测试:)

## JS 的东西

本文的最后一部分太长了，src/ folder 的设置将在下一篇文章中介绍。

我们想要现代的 js 语法、react 和所有那些花哨的东西。为此我们需要巴贝尔。Babel 就像一个加载器(出于 webpack 的目的，但官方上它是一个 transpiler)。

```
npm i -D babel-core babel-loader
```

我们添加一个新规则来通过所有。js 文件到 babel-loader

```
{
    test: /\.js$/,
    use: {
        loader: 'babel-loader',
        options: {
          presets: []
        }
    }
}
```

在 options prop 中，我们需要定义我们希望使用的预置。预设是一组“将这个转换为那个”的规则，因此我们可以选择我们需要的规则。

```
npm i -D babel-polyfill babel-preset-env babel-preset-react babel-preset-stage-0
```

在某些浏览器中，需要 polyfill 预置来处理缺失的承诺，我们需要将它导入 index.js 文件以使其工作。

```
import "babel-polyfill";
```

其他的我们将作为预置加载到 webpack.config.js 中

```
presets: ['env', 'stage-0', 'react']
```

在下一篇文章中，您将看到我们如何从这些预置中获益。

点击[此处](https://medium.com/@bojanaleksa/fast-and-modern-front-end-setup-with-webpack-react-redux-router-saga-and-postcss-part-2-9ae7ad4e7cb2)进入第二部分。

我希望你喜欢你所读的，请留下一些反馈，并为:D·:D·:D 鼓掌