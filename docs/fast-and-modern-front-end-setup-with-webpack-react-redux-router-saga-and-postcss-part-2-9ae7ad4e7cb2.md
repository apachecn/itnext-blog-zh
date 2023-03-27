# 使用 webpack、react、redux、router、saga 和 postcss 的快速现代前端设置，第 2 部分

> 原文：<https://itnext.io/fast-and-modern-front-end-setup-with-webpack-react-redux-router-saga-and-postcss-part-2-9ae7ad4e7cb2?source=collection_archive---------0----------------------->

![](img/620171a7d5b7ea51511c27079b95bdaf.png)

这是设置的第二部分。有不止一个部分，仅仅是因为它包含了大量的文本。

[*点击这里在 LinkedIn*](https://www.linkedin.com/cws/share?url=https%3A%2F%2Fitnext.io%2Ffast-and-modern-front-end-setup-with-webpack-react-redux-router-saga-and-postcss-part-2–9ae7ad4e7cb2) 上分享这篇文章

[这里的](/fast-and-modern-front-end-setup-with-webpack-react-redux-router-saga-and-postcss-1360e9715d17)是第一部分。

首先，我们忘了添加 commonChunksPlugin。这个插件的作用是删除重复内容。显然，它会将找到的副本放在一个单独的包中。这个插件包含在 webpack 中，所以我们只需要导入 webpack，不需要下载:

```
const webpack = require('webpack');
```

并在插件中实现。最终的 webpack.config.js 如下所示:

```
const path = require('path');
const webpack = require('webpack');const CleanWebpackPlugin = require('clean-webpack-plugin');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const ExtractTextPlugin = require('extract-text-webpack-plugin');
const extractPlugin = new ExtractTextPlugin({
    filename: './style.css'
});module.exports = {
    entry: "./index.js",
    output: {
        // CHANGED LINE        
        filename: '[name].bundle.js',
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
        extractPlugin,
        // NEW LINES
        new webpack.optimize.CommonsChunkPlugin({
            name: 'common'
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
            test: /\.scss$/,
            use: extractPlugin.extract({
             use: ["css-loader", "sass-loader", "postcss-loader"],
             fallback: 'style-loader'
            })
        }, {
         test: /\.js$/,
         use: {
          loader: 'babel-loader',
          options: {
           presets: ['env', 'stage-0', 'react']
          }
         }
        }]
    }
}
```

另请注意，我们添加了一个[名称]。这是因为 commonChunksPlugin 将生成一个新的输出文件。现在你可以运行`npm run build`来检查一下。

在 public/文件夹中，您现在会注意到 main.bundle.js 和 common.bundle.js。这使得我们的最终输出更小，这意味着最终用户:D 的加载时间更快

然而，如果我们运行`npm start`，我们会注意到两件事:第一，浏览器中页面的加载时间比我们开始时慢，第二，控制台输出中有额外的一行通知我们有东西被取消优化。回过头来看，我们发现我们所做的一些事情意味着以过程本身为代价来优化最终输出。我们可以通过拆分配置文件来解决这个问题，并为开发和生产提供单独的配置。这将证明对以后的其他事情是有用的。

因此，我们将首先在 webpack.config 文件旁边创建一个 webpack.prod.config.js 文件。

主文件将是开发配置文件，另一个文件将从中加载并为生产优化。让我们从生产文件开始:

```
var webpack = require("webpack");module.exports = require('./webpack.config.js');delete module.exports.devtool;module.exports.plugins.push(
    new webpack.DefinePlugin({
        'process.env.NODE_ENV': JSON.stringify('production')
    })
);module.exports.plugins.push(
    new webpack.optimize.UglifyJsPlugin({
        comments: false,
        warnings: false
    })
);
```

所以，首先我们加载 webpack 以便能够使用 DefinePlugin 和 UglifyJSPlugin。接下来，我们加载主文件。我们将在这里更改一些设置，但保留主要的设置。第一个变化，删除 devtool 对象，我们在生产中不需要它。然后，我们将另一个插件 DefinePlugin 添加到插件数组中。这个允许我们设置一些全局变量。但这并不意味着它们对浏览器是全局的，只是你可以在代码中使用它们。当我们为开发和生产制作前端配置文件时，这将非常方便。

最后稍微改一下丑化。如果您只是运行`npm run build-prod`，您会注意到文件输出是单行的，非常不可读。这个过程被称为丑化，它使文件更小，以便更快地上传到用户的电脑。Webpack 在生产中会自动丑化，但我们会更改这两个标志。

不要高兴，因为我们还有更多的事情要做，这只是基本的配置。我们将通过转到 package.json 来调用该文件，并在脚本中更改一行:

```
"build-prod": "webpack -p --config webpack.prod.config.js"
```

现在你可以运行`npm start build-prod`来确保它正常工作。

很好，现在来解决前面提到的两个问题。让我们首先将 commonChunksPlugin 转移到生产中，它会减慢我们的开发速度:

```
module.exports.plugins.push(
    new webpack.optimize.CommonsChunkPlugin({
        name: 'common'
    })
);
```

在产品配置中添加这一行，并从主配置中删除插件。运行`npm start`来看看它是否更快，也许改变 index.js 中的一些东西来测试一下。

很好，但更重要的是，我们目前处理/node_modules/文件夹中所有被调用的文件。这个文件夹是加载外部库的地方。我喜欢重新处理这些文件，因为你可以很容易地找到一个未经优化的库，但我们不需要在开发中这样做。我们将通过在任何测试下添加它来避免这种情况:在加载器中:

```
exclude: /node_modules/,
```

并再次运行`npm start`: d .并将其添加到生产中，添加以下内容

```
module.exports.module.rules.forEach(rule => {
    delete rule.exclude;
    return rule;
});
```

现在，生产将采取和优化一切，而开发优化工作。仍有一些空间进行其他优化，可能会将其他插件传递给 prod，但对我来说，速度并没有任何差异，所以这是我停下来的地方。

这里是帮助你的配置文件

```
const path = require('path');
const webpack = require('webpack');const CleanWebpackPlugin = require('clean-webpack-plugin');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const ExtractTextPlugin = require('extract-text-webpack-plugin');
const extractPlugin = new ExtractTextPlugin({
    filename: './style.css'
});module.exports = {
    entry: "./index.js",
    output: {
        filename: '[name].bundle.js',
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
        extractPlugin
    ],
    module: {
        rules: [{
            test: /\.(jpg|png|gif|svg)$/,
            exclude: /node_modules/,
            use: [
            {
                loader: 'file-loader',
                options: {
                    name: '[name].[ext]',
                    outputPath: './assets/',
                }
            }]
        }, {
            test: /\.scss$/,
            exclude: /node_modules/,
            use: extractPlugin.extract({
                use: ["css-loader", "sass-loader", "postcss-loader"],
                fallback: 'style-loader'
            })
        }, {
            test: /\.js$/,
            exclude: /node_modules/,
            use: {
                loader: 'babel-loader',
                options: {
                    presets: ['env', 'stage-0', 'react']
                }
            }
        }]
    }
}
```

和 webpack.prod.config.js

```
var webpack = require("webpack");const ExtractTextPlugin = require('extract-text-webpack-plugin');
const extractPlugin = new ExtractTextPlugin({
    filename: './style.css'
});module.exports = require('./webpack.config.js');delete module.exports.devtool;module.exports.plugins.push(
    new webpack.DefinePlugin({
        'process.env.NODE_ENV': JSON.stringify('production')
    })
);module.exports.plugins.push(
    new webpack.optimize.CommonsChunkPlugin({
        name: 'common'
    })
);module.exports.plugins.push(extractPlugin);module.exports.module.rules.forEach(rule => {
    delete rule.exclude;
    return rule;
});module.exports.plugins.push(
    new webpack.optimize.UglifyJsPlugin({
        comments: false,
        warnings: false
    })
);
```

！！！搞定了。！！

希望我们没有忘记什么:d .请记住，根据您的需要，还有其他插件可能有用，但对于标题中有完整设置的文章来说，这已经是太多的 webpack 了。做出反应。

## 反应

入口文件是 index.js，所以我们将在这里加载 react。React 需要一个装载的地方。幸运的是，我们已经在 index.html 用 id="app "制作了一个 div，看看吧。接下来，因为我忘了我们还没有做到这一点，去终端，并得到反应

```
npm i -S react react-dom prop-types
```

这是我们的 index.js 文件:

```
import "babel-polyfill";
import React from "react";
import ReactDOM from "react-dom";ReactDOM.render(
  <div>
    Hello world!
  </div>,
  document.getElementById('app')
);
```

所以，首先我们要装载巴别塔填充物。这件事到此为止，我们结束了。接下来，react 和 react-dom 将我们的 react 应用程序加载到 html。请注意，在 render 函数的第二行中，我们获取了#app 元素。现在运行`npm start`，看看你好世界！在浏览器中。

太好了，我们现在有反应了。我们还得到了 prop-types，这在代码中有更多人的时候很有用，因为它告诉他们组件期望什么样的 prop。

为了真正利用 react 的优势，让我们在 src/文件夹中创建一个组件/文件夹。我们将把组件放在这里，并将主组件命名为 App.js。

```
import React from 'react';const App = props => <div>Our main component</div>export default App;
```

在每个 react 组件中，我们都必须导入 react。const App 行是我们的元素，最后一行导出它，正如我们已经看到的。我们现在可以在 index.js 文件中导入这个组件，这样我们就可以用 components/ folder 中组织的组件来构建整个应用程序:

```
import "babel-polyfill";
import React from "react";
import ReactDOM from "react-dom";import App from './components/App';ReactDOM.render(
  <App />,
  document.getElementById('app')
);
```

如果您已经在运行`npm start`，您应该会立即看到变化。您也可以在我们的 App.js 中更改文本，以查看它重新加载并向您显示结果。

注意，以防有人在这里询问热重装:这是很棒的东西，但我发现它实际上减慢了初始编码。只有当应用程序变得更大时，它才显示出它的价值，所以我不把它放在主设置中。

## 材料-用户界面

让我们不要重新发明轮子，使用已经构建和测试过的组件材料——ui 给我们。

```
npm i -S material-ui
```

有一个更新的版本叫做 next，但它仍然是测试版。对于这个版本，我们需要将所有东西封装在一个主题中，我们将在 index.js 文件中这样做:

```
import "babel-polyfill";
import React from "react";
import ReactDOM from "react-dom";import MuiThemeProvider from 'material-ui/styles/MuiThemeProvider';import App from './components/App';ReactDOM.render(
  <MuiThemeProvider>
    <App />
  </MuiThemeProvider>,
  document.getElementById('app')
);
```

我们的应用程序现在有了一个主题，我们可以在 App.js 文件中使用任何材质元素:

```
import React from 'react';
import RaisedButton from 'material-ui/RaisedButton';const App = props => <div>
 <RaisedButton label="CLICK ME" />
</div>export default App;
```

我们会在页面上看到一个按钮。

## 风格

有两种主要的方式来组织样式。有些人分别为每个组件设计和加载样式，有一些 postcss 插件可以帮助完成这个任务。我喜欢尽可能地使用集群和组，所以我只加载一个样式文件，并让它加载其余的，这样就迫使自己也尝试重用这些类。我们将在 index.js 中添加一行

```
import './styles/main.scss';
```

我们可以在这个文件中导入 Roboto 字体，因为它在 material-ui 中使用:

```
[@import](http://twitter.com/import) url([http://fonts.googleapis.com/css?family=Roboto](http://fonts.googleapis.com/css?family=Roboto));
```

我通常只是作为一个导入程序使用这个文件，只有导入行，然后按照我想要的方式组织文件夹。

## 状态管理

当用户与你的应用程序交互时，事情一件接一件地发生。当前的情况可以表示为一个 json 对象，我们可以称之为状态。状态封装了位置(url)、当前用户、用户点击了什么等等。我们想把状态从视图中分离出来。我们将让 react 处理视图(它将如何显示)并让 redux 处理状态(将显示什么)。如果这有点不清楚，让我们通过让它工作来解释它:

```
npm i -S redux react-redux
```

让我们首先更改我们的 App.js 文件，看看我们将如何使用 redux:

```
import React from 'react';
import PropTypes from 'prop-types';import RaisedButton from 'material-ui/RaisedButton';const App = props => <div>
 <RaisedButton label={props.buttonText} onClick={props.onClick} />
</div>App.propTypes = {
 buttonText: PropTypes.string.isRequired,
 onClick: PropTypes.func.isRequired
}App.defaultProps = {
 buttonText: 'defaultText',
 onClick: () => console.log('default click action')
}export default App;
```

为了展示我们的目标，我们在这里看到了一些变化。首先，我们导入了 prop-types 模块，并为组件定义了道具类型。通过这种方式，我们允许下一个人(或者我们自己，当我们忘记了这段代码，需要继续工作的时候)快速了解需要传递给组件的道具。正如我们所看到的，我们需要一个字符串 buttonText 和一个函数 onClick，它们都是必需的。如果我们不提供它们，浏览器会显示一个错误，你可以测试一下。

接下来，我们使用按钮中的道具对象，如您所见。最后，我们将定义默认道具，保存并签出我们的应用程序。您将在浏览器中看到一个带有默认文本的按钮，当您单击它时，控制台将记录我们的文本。

这种分离现在清楚地表明，我们希望让我们的组件显示事物，但是实际的文本和功能以及诸如此类的东西将在其他地方定义。将来，我们都可以更容易地找到一个突然出现的 bug，重用组件或修改它们，而不需要接触应用程序的其他部分。

现在让我们在 src/中创建一个名为 containers/的新文件夹。这个文件夹将充当 react 组件和状态之间的中间人。我们稍后会看到这种状态在哪里，以及如何利用它。在其中，我们将制作一个文件 App.js:

```
import { connect } from 'react-redux'
import TheComponent from '../components/App';const mapStateToProps = (state, ownProps) => {
    return {
     buttonText: state.text
    }
}const mapDispatchToProps = (dispatch, ownProps) => {
    return {
        onClick: () => {
          dispatch({type: 'BASIC_ACTION', text: 'new text'})
        }
    }
}const App = connect(
    mapStateToProps,
    mapDispatchToProps
)(TheComponent)export default App;
```

好了，我们现在看到，我们将使用它把道具传递给组件。这允许我们制作几个使用相同组件的不同容器，但是我们也可以使用具有不同组件的相同容器。在这个例子中，我们实际上导出了预加载了组件的容器，但是您不需要这样做。

在 mapStateToProps 中，我们将字符串、数字、数组和对象传递给组件。在 mapDispatchToProps 中，我们将函数传递给组件。通过将它们传递给 connect()函数，我们将它们连接到状态对象。想象这个状态对象是这样的:`{text: ‘some text’}`。我们可以获取这个属性，并在我们的组件中使用它，但我们也可以在 onClick()函数中更新它(稍后显示，需要更多的工作:D)。我们还可以在其他组件中使用和更新状态对象。这意味着，我们只有一个信息来源，每当发生变化，应用程序中的所有组件都会收到通知，并做出相应的反应。

这看起来很棒。我们正在慢慢地分离和优化我们的应用程序，现在我们需要理解这个 dispatch()函数是如何更新状态的。为此，我们使用一种叫做减压器的东西。reducer 是一个函数，它从 dispatch()函数中获取数据，并决定如何创建新的状态。让我们创建一个名为 reducers/的文件夹，并将 index.js 放入其中:

```
import { combineReducers } from "redux";
import basicReducer from './basicReducer';export const reducers = combineReducers({
  text: basicReducer
});
```

我们将使用它作为主减速器，并在其中装载我们的减速器。reducer 结构定义了状态树结构，所以现在 state.text 将是 basicReducer 返回的任何内容。让我们将 basicReducer.js 放在同一个文件夹中:

```
const basicReducer = (state = 'some text', action) => {
 switch(action.type) {
  case 'BASIC_ACTION': return action.text
  default: return state
 }
}export default basicReducer;
```

我们在这里看到，我们需要告诉 reducer 返回任何情况下的状态，除非 action.type(强制属性)匹配某个字符串。稍后我们将在一个单独的文件中提取这些常量。所以现在，每当我们从容器中调用 dispatch()时，reducer 都会检查类型是否匹配。如果不是，保持现状。如果是，返回另一个状态，在这种情况下，新的状态变成 action.text。在我们单击按钮后，状态将被更新为{text: 'new text'}，这个属性将被传递回组件，组件将依次显示这个文本。暂时不要测试，因为我们还没有设置 redux。

这里还要知道的一件事是，我们决不能改变 reducer 中的状态对象，只能用它来创建和传递一个新对象。减压器是纯函数，也就是说没有副作用。这带来了很多好处。例如，应用程序存储每个州的信息，这意味着我们可以很容易地通过用户操作来找到用户遇到的错误。

让我们现在设置 redux，转到 index.js

```
import "babel-polyfill";
import React from "react";
import ReactDOM from "react-dom";import { Provider } from 'react-redux';
import { createStore } from 'redux';
import { reducers } from './reducers/index';import './styles/main.scss';import MuiThemeProvider from 'material-ui/styles/MuiThemeProvider';import App from './containers/App';let store = createStore(reducers);ReactDOM.render(
 <Provider store={store}>
    <MuiThemeProvider>
      <App />
    </MuiThemeProvider>
   </Provider>,
  document.getElementById('app')
);
```

我们添加的内容:我们现在从 redux 导入 redux 和 createStore，以及我们的 reducers 列表。存储保存状态对象。我们用我们的 reducers 创建它，并将整个应用程序封装在<provider>组件中，以允许任何组件访问状态对象。注意这个文件中的另一个变化:我们现在从。/containers/App，而不是。/components/App，我们可以`npm start`我们的 App:)。</provider>

我们将看到带有默认文本的按钮，该文本来自我们的基本 reducer 中定义的状态。转到 basicReducer.js 并更改默认文本，然后看到按钮 update。太好了，现在我们从状态对象那里得到了我们的道具。

此外，当我们单击按钮时，文本会发生变化。让我们再来一遍这个循环:

1.  我们单击组件中的按钮
2.  组件调用容器中的函数
3.  容器调度一个动作，并定义了一个类型(强制的)，如果需要的话还会调度一些数据，在本例中是新的文本
4.  reducers 监听调度，当一个动作类型匹配一个定义的字符串时，它们返回定义的新状态。
5.  store 更新状态，并通知所有监听状态的组件
6.  容器现在看到状态发生了变化，并将新的道具发送到按钮组件
7.  组件将使用新的道具重新渲染

好了，状态管理起作用了，但是我们将通过在一个单独的文件中提取所有那些动作类型来更好地组织它。在我们的 src/文件夹中创建操作/文件夹。在其中，创建一个 index.js 文件来保存我们的常量:

```
export default {
 BASIC_ACTION:'BASIC_ACTION' 
}
```

我们将在 dispatch 和 reducer 中使用它，因此字符串中的任何更改都可以包含在内。组件/应用程序:

```
import { connect } from 'react-redux';
import actions from '../actions/';import TheComponent from '../components/App';const mapStateToProps = (state, ownProps) => {
    return {
     buttonText: state.text
    }
}const mapDispatchToProps = (dispatch, ownProps) => {
    return {
        onClick: () => {
          dispatch({type: actions.BASIC_ACTION, text: 'new text'})
        }
    }
}const App = connect(
    mapStateToProps,
    mapDispatchToProps
)(TheComponent)export default App;
```

我们导入整个动作集，然后使用我们想要的类型。我发现导入所有动作更容易，因为你经常需要更改它们或复制文件，我不再需要考虑导入了。请注意 dispatch()函数中的变化。

在 basicReducer.js 中:

```
import actions from '../actions';const basicReducer = (state = 'some other text', action) => {
 switch(action.type) {
  case actions.BASIC_ACTION: return action.text
  default: return state
 }
}export default basicReducer;
```

此外，通过导入整个 actions 对象，我们可以很容易地扩展 reducer。

在开始时，状态管理可能看起来像是一种矫枉过正的行为，而且很容易如此，您需要定义是否需要 redux。但是我发现，对状态树进行适当的维护意味着可以很容易地将逻辑分离到树的不同部分。例如，如果您的状态对象有用户属性和朋友属性，并且用户注销，您可以分派一个动作，然后让两个不同的 reducers 分别对其做出反应，从而将用户属性设置为{}，将朋友属性设置为[]。这使得国家管理变得轻而易举。

再说一次，这太长了。老实说，当我开始时，我没有意识到我们需要覆盖多少地面，但我被迫再次切割。希望对你有帮助，我一定会尽快完成第三部分。

期待掌声:D·:D·:D