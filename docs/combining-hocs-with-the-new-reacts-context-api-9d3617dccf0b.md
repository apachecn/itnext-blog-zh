# 将 HOCs 与新 React 的上下文 API 相结合

> 原文：<https://itnext.io/combining-hocs-with-the-new-reacts-context-api-9d3617dccf0b?source=collection_archive---------0----------------------->

React 最近发布了一个次要版本(16.3)，其中包括一个大肆宣传的上下文 API 的“稳定”版本。它被大肆宣传的原因是 React 让开发人员可以提前接触到 API 的“测试”版本。它现在得到了他们团队的官方支持，并被强烈推荐使用。

# 但是，什么是上下文 API 呢？

如果你不知道**什么是**上下文 API，我推荐你阅读 [React 的官方文档](https://reactjs.org/docs/context.html)或者这篇由 [Kent C. Dodds](https://medium.com/u/db72389e89d8?source=post_page-----9d3617dccf0b--------------------------------) 撰写的[文章](https://medium.com/dailyjs/reacts-%EF%B8%8F-new-context-api-70c9fe01596b)。

![](img/0e25a1e3629868d21b77c02eb07ffd63.png)

# 为什么要使用它的灵感

大约 12 个月前，我开始写我想在 [Askable](https://www.askable.com/) 使用的结构的初稿。根据我以前的经验，我尝试使用 React 和 Redux。React 听起来像是一个显而易见的选择，但 Redux 不是。

不要误会，Redux 是一个很神奇的工具。然而，我总是觉得我必须写太多的代码来实现我想要的。编写动作、归约器、连接器等等不是一件容易的事情(特别是如果你有一个嵌套结构的状态)。

最后，我决定不使用 Redux，并检查一些替代品，因为我想快速交付特性。

## 问题是

我面临的主要问题是找到一种简单的方法，在不同的文件中使用公共组件，而不必将它们导入到每个文件中。

例如，您有一个名为`Header.js`的组件，并且想要调用一个名为`modalPricing.js`的模态组件。把`modalPricing.js`文件直接包含到你的`Header.js`里问题不大。但是，如果您需要将同一个文件调用到您的`Footer.js`中呢？你会导入同样的文件吗？如果你不得不在其他地方调用同一个模态呢？

如果您采用这种解决方案，您将会在 DOM 中出现同一个模态窗口的多个实例。您可能知道，这不是一个好的做法。

## 解决方案

我可以使用 Redux Provider 并传递 props 来打开/关闭模态。然而，正如我之前提到的，我不想走这条路。此外，我不想在我的项目中包含另一个依赖项。

这就是当我看到 React 团队计划支持开箱即用的上下文 API 时，我感到兴奋的主要原因。理论上，这将解决我的问题。

然后我决定使用“就这么做”的方法:)

# 将上下文 API 与 HOC 一起使用

在使用新 API 一段时间后，包装组件的过度使用困扰着我。将一个上下文包装到您的组件中没什么大不了的，但是如果您有多个上下文呢？在到达组件之前，您会包装多个上下文吗？它可以工作，但是开发人员的体验会受到影响。

例如，假设您有三个上下文——auth context、AppContext 和 ComponentsContext。你会把你的组件包装成三个不同的上下文来得到你想要的吗？我不这么认为。

我为这个问题找到的解决方案是结合新的上下文 API 和 HOC 的([高阶组件文档](https://reactjs.org/docs/higher-order-components.html))。

所以，我们来看看怎么实现。:)

你可以在这里查看这篇文章的知识库: [Github 上下文 Api](https://github.com/xicovarisco/context-api-with-hocs)

## 1 —创建您的 React 提供程序

*appProvider.js*

```
import React, { Component } from 'react';
import { PricingModal, Snackbar } from 'src/components/common';// Create a new context for the app
export const AppContext = React.createContext('app');// Creates a provider Component
class AppProvider extends Component {
    constructor(props) {
        super(props); this.state = {
            openPricingModal: false
        }; this.onOpenPricingModal = this.onOpenPricingModal.bind(this);
    } onOpenPricingModal() {
        this.setState({ openPricingModal: true });
    } render() {
        return (
            <**AppContext.Provider**
                value={{
                    state: this.state,
                    **onOpenPricingModal**: this.onOpenPricingModal
                }}
            >
                {this.props.children}
                <PricingModal
                    open={this.state.openPricingModal}
                    onClose={() => this.setState({ openPricingModal: false })}
                />
           ** </AppContext.Provider>**
        );
    }
}export default AppProvider;
```

## 2-为您的上下文创建一个特设

*withAppContext.js*

```
import React from 'react';
import { AppContext } from 'src/components/common';export function withAppContext(Component) {
    return function WrapperComponent(props) {
        return (
            <AppContext.Consumer>
                {state => <Component {...props} context={state} />}
            </AppContext.Consumer>
        );
    };
}
```

## 3 —将根组件包装到提供程序中

*app.js*

```
import React, { Component } from 'react';
import { AppProvider } from 'src/components/common';class App extends Component {
    render() {
        return (
            **<AppProvider>**
                {/* Your component here */}
            **</AppProvider>**
        );
    }
}export default App;
```

## 4-在组件上使用上下文 HOC

*header.js*

```
import React, { Component } from 'react';
import { withAppContext } from 'src/components/HOCS';class Header extends Component {
    render() {
        return (
            <div className="headerComponent">
                <a
                    className="pricingContainer"
                    onClick={**this.props.context.onOpenPricingModal**}
                >
                    <p className="buy">
                        Pricing. <strong>save up to 27%</strong>
                    </p>
                </a>
            </div>
        );
    }
}export default withAppContext(Header);
```

# 结论

将 HOC 与新的 React 的上下文 API 结合使用，可以为您提供灵活性、可伸缩性和良好的开发人员体验。您可以根据需要创建任意多的上下文，并将它们封装到您需要使用的组件中。

如果不使用 Redux，这是保持代码简洁和不多次导入同一个组件的简单方法。

在 [Askable](https://askable.com) ，我们在生产中的所有项目中广泛使用这种模式，我们团队之间的接受度很高。

[Askable](http://askable.com) 是一个寻找你的**完美**参与者的平台，最初是作为 [Orange Digital](https://www.orangedigital.com.au) 的衍生产品开发的。在澳大利亚运行用户测试。跳转到我们的[登陆页面](https://au.askable.com/)或我们的[网站](http://askable.com/)了解更多详情。