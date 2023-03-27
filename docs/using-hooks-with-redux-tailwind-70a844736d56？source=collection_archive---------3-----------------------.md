# 使用带有 Redux +顺风的挂钩

> 原文：<https://itnext.io/using-hooks-with-redux-tailwind-70a844736d56?source=collection_archive---------3----------------------->

react 组件的新`hooks`提案看起来很棒，语法看起来非常干净，可读性也很好。上周末，我决定试一试，开发一个简单的[预算应用](https://budget.bleext.com/)。

![](img/8a4c31cc229b4ca2b5c3858644c6f469.png)

使用 React + Redux + Hooks + Tailwind 的预算应用程序

为了加快速度，我决定使用 Create React 应用程序在几分钟内启动并运行。考虑到我在周末写代码的时间非常有限，我决定用 tailwind 做样式，这是我一直想用的工具。

# 集成顺风 CSS

将 tailwind 与 CRA 集成非常容易，只需安装这个库并创建几个脚本来编译它。

```
$ yarn add tailwindcss --dev
$ yarn add postcss-cli --dev
$ yarn add autoprefixer --dev
```

一旦准备好依赖项，就需要像这样初始化 tailwind:

```
$ ./node_modules/.bin/tailwind init
```

这将在根目录下创建一个配置文件。您可以在这里配置[颜色](https://github.com/crysfel/budgeting-app/blob/master/tailwind.config.js#L46)、[大小](https://github.com/crysfel/budgeting-app/blob/master/tailwind.config.js#L645)、[字体](https://github.com/crysfel/budgeting-app/blob/master/tailwind.config.js#L195)等。在我的例子中，我离开默认配置仅仅是因为我赶时间(我只能在周末当我的宝宝睡觉的时候编码😅).但是会回来调整它并使[最终束变得更小](https://tailwindcss.com/docs/controlling-file-size)。

我们还需要配置 postcss，我们基本上需要添加 tailwind 作为插件。

```
const tailwindcss = require('tailwindcss');module.exports = {
  plugins: [
    tailwindcss('./tailwind.js'),
  ],
};
```

鉴于我使用的是 CRA，我没有对 webpack 配置的完全访问权(除非[我们破解了](https://daveceddia.com/customize-create-react-app-webpack-without-ejecting/)，否则我们可以使用 webpack [加载器来代替 postcss 用于顺风](https://tailwindcss.com/docs/installation)。

为了在 React 中使用 tailwind，我们需要创建一个 css 文件并导入预检和实用程序。文件可以在任何地方定义，[在我的例子中](https://github.com/crysfel/budgeting-app/blob/master/src/theme/base.css)在`src/theme/base.css`下

```
@tailwind preflight;
@tailwind utilities;/* Your custom CSS here */
```

我们需要做的最后一件事是编译 css 并在 react 应用程序中使用它。我们需要在`package.json`中添加[以下任务](https://github.com/crysfel/budgeting-app/blob/master/package.json#L11-L12)

```
"build:css": "postcss src/theme/base.css -o src/index.css",
"watch:css": "postcss src/theme/base.css -o src/index.css -w"
```

第一项任务是获取基本 css 文件并将其编译成`src/index.css`，这是我们需要导入 react 应用程序的文件。

第二个任务是可选的，如果您想观察对基本 css 文件的任何更改，这对开发很有用。

现在我们需要在使用它之前构建 css，我们可以手动完成它，但我认为在启动 webpack 之前完成它会更容易。

```
"start": "npm run watch:css && react-scripts start",
```

最后，我们需要将编译后的文件导入 React 组件。

```
import React from 'react';
import ReactDOM from 'react-dom';
import { StoreProvider } from 'redux-react-hook';
import App from './containers/App';
import store from './store';
import { setAppUpdated } from './store/modules/app/actions';
import * as serviceWorker from './serviceWorker';import './index.css'; // <---- Here!!ReactDOM.render(
  <StoreProvider value={store}>
    <App />
  </StoreProvider>,
  document.getElementById('root')
);// If you want your app to work offline and load faster, you can change
// unregister() to register() below. Note this comes with some pitfalls.
// Learn more about service workers: [http://bit.ly/CRA-PWA](http://bit.ly/CRA-PWA)
serviceWorker.register({
  onUpdate: () => store.dispatch(setAppUpdated()),
});
```

就这样，现在我们可以在 react 组件中使用 tailwind 了！不用写任何 css 就能构建 ui，真的很牛逼！

看看我的面板组件，超级容易搭建！我不用写一行 CSS 代码:

```
import React from 'react';export default function Panel({ children, title }) {
  return (
    <div className="bg-white py-8 px-4 border-t-4 border-orange"
      {title && <h3 className="mb-4">{title}</h3> }
      {children}
    </div>
 );
}
```

![](img/05ba0b6ef41cf28de760aa2f4574d4a6.png)

一个顺风建造的面板组件！

# 在 Redux 中使用 React 钩子

关于如何为 redux 实现钩子已经有了一个[讨论](https://github.com/reduxjs/react-redux/issues/1063)，确实是非常有趣的方法。然而，目前官方的 redux 项目还不支持钩子。

有几个项目可以帮助你用钩子访问 redux 状态，我最喜欢的一个是 [redux-react-hook](https://github.com/ianobermiller/redux-react-hook) 。我喜欢它，因为它通过定义一个映射函数让你访问状态，类似于我们在`connect`中使用的，它也让你访问`dispatch`方法来触发动作。

我们需要做的第一件事是删除官方的 [redux provider](https://github.com/crysfel/budgeting-app/blob/master/src/index.js) 并在这个库中使用这个 provider。

```
import { StoreProvider } from 'redux-react-hook';
import store from './store';ReactDOM.render(
  <StoreProvider value={store}>
    <App />
  </StoreProvider>,
  document.getElementById('root')
);
```

为了访问 React 组件内部的状态，您需要使用`useMappedState`函数，它接收一个映射函数并返回一个包含您需要的数据的对象。

```
import React from 'react';
import { useMappedState } from 'redux-react-hook';
import { getLatestGroupedByDate } from 'store/modules/transactions/selectors';const mapState = state => ({
  latest: getLatestGroupedByDate(state),
});export default function Dashboard() {
  const { latest } = useMappedState(mapState); return (
    <span>{latest.length} Transactions</span>
  );
}
```

`mapState`函数和我们在`connect`中使用的完全一样，在 react 组件之外声明这个函数很重要，否则它会在每次渲染时被创建。

如果我们需要在`mapState`中使用`props`，那么我们可以将它移动到组件中，但是我们必须使用`useCallback`来记忆它并避免在每次渲染时重新创建它。

通过调用`useMappedState`，我们可以从 redux 存储中检索我们需要的数据！使用钩子的语法看起来非常简洁，易于阅读和理解。我们不再需要 hoc 了🎉！

现在，为了分派动作，我们需要做的就是调用钩子`useDispatch`，它将从存储中返回分派方法！从那里你基本上可以发射任何你想要的动作，例如:

```
import { useDispatch } from 'redux-react-hook';export default function AddTransaction({ isExpense }) {
  const [amount, setAmount] = useState(0);
  const dispatch = useDispatch();
  const saveTransaction = useCallback(
    dispatch(postTransaction({ amount }))
  , [amount]); return (
    <button onClick={saveTransaction}>Save</button>
  );
}
```

这里重要的是使用了`useCallback`，这是为了避免在每次渲染时创建内部动作，只有在`amount`更新时才会创建。

# 结论

通过使用钩子，你可以看到访问状态和分派动作是多么容易。我真的很喜欢 hooks 玩 redux 的方式，我认为这是向前迈出的一步，降低了新开发的门槛。

用顺风真的很牛逼！我喜欢不用写任何 css 就能构建一个应用程序，真的很棒！

你可以看到[运行](https://budget.bleext.com/)的应用程序，如果你对代码感兴趣，你应该[检查我的回购](https://github.com/crysfel/budgeting-app)。