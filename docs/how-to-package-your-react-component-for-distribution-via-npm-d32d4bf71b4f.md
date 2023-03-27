# 如何打包 React 组件以便通过 NPM 分销

> 原文：<https://itnext.io/how-to-package-your-react-component-for-distribution-via-npm-d32d4bf71b4f?source=collection_archive---------0----------------------->

![](img/33ebd57f70328ebadae739f6909af91e.png)

"几个不同颜色的热气球飞上了古老的天空."由[斯文·舍尔梅尔](https://unsplash.com/@sveninho?utm_source=medium&utm_medium=referral)上[的 Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)

我编写了一个 React 组件，使用 Babel 进行传输，使用 Webpack 进行捆绑和构建。我想通过 NPM 在另一个应用程序中使用它。我的 NPM 发布包需要包括组件行为，风格和图像。那么，将我的 React 组件打包以便通过 NPM 销售有多难呢？一个小时左右的工作对吗？

因为下面的减速带，我花了很长时间才弄明白。
–对版本冲突做出反应
–如何从组件中捆绑和使用样式
–如何包含和捆绑图像

以下是步骤。

# 1.使包 NPM 可发布

`npm init`

在`package.json`中，确保填写了这些字段:

```
package.json{
    "name": "myUnflappableComponent",
    "version": "0.0.29",
    "main": "dist/index.js",
    "publishConfig": {
       "access": "restricted"
    },
    ...
}
```

# 2.不要捆绑反应。使用父级的 REACT 和 REACT-DOM。

在`package.json`中，在项目的`peerDependencies`中添加`React`和`react-dom`(并从`dependencies`中移除，但添加到 devDependencies 进行开发)

```
..."peerDependencies": {      
    "react": ">=15.0.1",
    "react-dom": ">=15.0.1"
},
 "devDependencies": {      
    "react": ">=15.0.1",      
    "react-dom": ">=15.0.1"  
},  
...
```

在 webpack 配置中，创建一个 UMD 包

```
...
module.exports = {  
    ...  
    output: {      
        path: path.join(__dirname, './dist'),      
        filename: 'myUnflappableComponent.js',      
        library: libraryName,      
        libraryTarget: 'umd',      
        publicPath: '/dist/',      
        umdNamedDefine: true  
    },  
    plugins: {...}, 
    module: {...},  
    resolve: {...},  
    externals: {...} 
}
```

最重要的是，不要捆绑反应

```
module.exports = {  
    output: {...},  
    plugins: {...},  
    module: {...},  
    resolve: {      
        alias: {          
            'react': path.resolve(__dirname, './node_modules/react'),
          'react-dom': path.resolve(__dirname, './node_modules/react-dom'),      
        }  
    },  
    externals: {      
        // Don't bundle react or react-dom      
        react: {          
            commonjs: "react",          
            commonjs2: "react",          
            amd: "React",          
            root: "React"      
        },      
        "react-dom": {          
            commonjs: "react-dom",          
            commonjs2: "react-dom",          
            amd: "ReactDOM",          
            root: "ReactDOM"      
        }  
    } 
}
```

# 3.设置您的。NPMIGNORE 文件

如果你不设置一个. npmignore 文件，npm 使用你的`.gitignore`文件，不好的事情就会发生。允许空的`.npmignore`文件。这是我的样子:

```
webpack.local.config.js
    webpack.production.config.js
    .eslintrc
    .gitignore
```

# 4.将“预发布”脚本添加到您的`PACKAGE.JSON`

发布前生成。

```
"scripts": {
     "prepublish": "rm -rf ./dist && npm run build",
    ...
}
```

# 5.提取你的 CSS 文件以供使用

我们使用 SCSS 文件作为我们的风格。这些被编译成 css，由 Webpack 提取出来。

安装以下组件:

```
npm install --save-dev extract-text-webpack-plugin node-sass style-loader css-loader sass-loader
```

更新你的`webpack.config`

```
const ExtractTextPlugin = require('extract-text-webpack-plugin');module.exports = {
    ...
    plugins:[
        new ExtractTextPlugin({
            filename: 'myUnflappableComponent.css',
        }),
    ],
    module:{
        rules:[
            {
                test: /\.*css$/,
                use : ExtractTextPlugin.extract({
                    fallback : 'style-loader',
                    use : [
                        'css-loader',
                        'sass-loader'
                    ]
                })
            },
            ....
        ]
    }
}
```

# 6.CSS 中的图像

您在组件中包含图像的方式将决定您的组件的消费者是否会得到它们。

我使用[内容属性](https://developer.mozilla.org/en-US/docs/Web/CSS/content)将它们包含在 css 文件中。例如

```
.mySky{
    width: 20px;
    height: 20px;
    content: url('../assets/images/thunderSky.png');
}
```

# 7.确保您的图像在组件外部可用

我遇到的问题是发布的 CSS 文件中图像的相对路径被打乱了。经过大量的搜索，[这篇文章](https://shakacode.gitbooks.io/react-on-rails/content/docs/additional-reading/rails-assets-relative-paths.html)(也在下面的链接中)帮了大忙。

安装以下组件:

```
npm install --save-dev file-loader url-loader
```

像这样更新你的`webpack.config`:

```
module.exports = {
    ...
    module: {
        rules: [
            {
                test: /\.(png|svg|jpg|gif)$/,
                use: [
                    {
                        loader: 'url-loader',
                        options:{
                            fallback: "file-loader",
                            name: "[name][md5:hash].[ext]",
                            outputPath: 'assets/',
                            publicPath: '/assets/'
                        }
                    }
                ]
            },
            ...
            resolve: {
                alias:{
                    ...
                    'assets': path.resolve(__dirname, 'assets')
                }
            }
        ]
    }
}
```

# 销售和推荐人:

1.  [如何在 npm 上发布您的包(关于](https://hackernoon.com/how-to-publish-your-package-on-npm-7fc1f5aae600) `[package.json](https://hackernoon.com/how-to-publish-your-package-on-npm-7fc1f5aae600)` [)](https://hackernoon.com/how-to-publish-your-package-on-npm-7fc1f5aae600)
2.  [向 NPM 发布测试版](https://maxisam.github.io/2017/03/29/publish-beta-to-npm/)
3.  [通过 webpack 导出图像(](https://shakacode.gitbooks.io/react-on-rails/content/docs/additional-reading/rails-assets-relative-paths.html) `[webpack.config.js](https://shakacode.gitbooks.io/react-on-rails/content/docs/additional-reading/rails-assets-relative-paths.html)` [)](https://shakacode.gitbooks.io/react-on-rails/content/docs/additional-reading/rails-assets-relative-paths.html)
4.  我的完整 webpack 配置:

```
const webpack = require('webpack');
const ExtractTextPlugin = require('extract-text-webpack-plugin');
const pkg = require('./package.json');
const path = require('path');const libraryName= pkg.name;module.exports = {
    entry: path.join(__dirname, "./src/index.js"),
    output: {
        path: path.join(__dirname, './dist'),
        filename: 'myUnflappableComponent.js',
        library: libraryName,
        libraryTarget: 'umd',
        publicPath: '/dist/',
        umdNamedDefine: true
    },
    plugins: [
        new ExtractTextPlugin({
            filename: 'myUnflappableComponent.css',
        }),
    ],
    node: {
      net: 'empty',
      tls: 'empty',
      dns: 'empty'
    },
    module: {
        rules : [
            {
            test: /\.(png|svg|jpg|gif)$/,
            use: [
                {
                    loader: 'url-loader',
                    options:{
                        fallback: "file-loader",
                        name: "[name][md5:hash].[ext]",
                        outputPath: 'assets/',
                        publicPath: '/assets/'
                    }
                }    
            ]
        },
        {
            test: /\.*css$/,
            use : ExtractTextPlugin.extract({
                fallback : 'style-loader',
                use : [
                    'css-loader',
                    'sass-loader'
                ]
            })
        },
        {
            test: /\.(js|jsx)$/,
            use: ["babel-loader"],
            include: path.resolve(__dirname, "src"),
            exclude: /node_modules/,
        },
        {
            test: /\.(eot|ttf|woff|woff2)$/,
            use: ["file-loader"],
        },
        {
            test: /\.(pdf|doc|zip)$/,
            use: ["file-loader"],
        }]
    },
    resolve: { 
        alias: { 
            'react': path.resolve(__dirname, './node_modules/react') ,
            'react-dom': path.resolve(__dirname, './node_modules/react-dom'),
            'assets': path.resolve(__dirname, 'assets')
        } 
    },
    externals: {
        // Don't bundle react or react-dom
        react: {
            commonjs: "react",
            commonjs2: "react",
            amd: "React",
            root: "React"
        },
        "react-dom": {
            commonjs: "react-dom",
            commonjs2: "react-dom",
            amd: "ReactDOM",
            root: "ReactDOM"
        }
    }
};
```