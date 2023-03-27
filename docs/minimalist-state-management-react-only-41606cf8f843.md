# 极简状态管理。(反应)

> 原文：<https://itnext.io/minimalist-state-management-react-only-41606cf8f843?source=collection_archive---------4----------------------->

![](img/5e09314be9ca1bc4b747cb1b75edcbba.png)

16.3 版 react 引入了新的上下文 API。在我看来，这个新特性对于中小型应用的状态管理来说已经足够好了。最近我写了一个小项目，其中我使用上下文作为前端的主要数据源。在这篇文章中，我想分享我学到的知识和方法。

## 新 API

让我们快速提醒自己新的 API。

```
const Context = React.createContext(initialData)
```

创建新的上下文。可能有多个包含不同数据的上下文。

```
<Context.Provider value={data}/>;
```

接受“值”属性。当数据更改时，将重新呈现所有相关的使用者。

```
<Context.Consumer/>
```

有权访问提供商的数据。消费者可以通过两种方式访问数据:

**1:**

```
class Modal extends React.Component {
    static contextType = AppContext;
    //…
}class Cmp extends Component {render() {
     console.log(this.context);
     //...}
}
```

**2:**

```
render() {
  return (
      <Consumer>
        {data => <div>{data.title}</div>}
      </Consumer>
   )
}
```

## 新旧 API 之间的差异

在旧的 API PureComponents 和 Components 中，当道具或状态改变时，实现的 shouldComponentUpdate 会重新呈现。React 不考虑上下文的值。这种行为会导致上下文中的数据过时。
下面是一个使用旧 API 的例子:

更改标题组件以扩展 PureComponent。多次单击增量。<title>不会重新呈现，但上下文值已更改。</p></div><div class="ab cl kw kx hu ky" role="separator"><span class="kz bw bk la lb lc"/><span class="kz bw bk la lb lc"/><span class="kz bw bk la lb"/></div><div class="ij ik il im in"><h1 id="ce97" class="mr le iq bd lf ms mt mu li mv mw mx ll my mz na lo nb nc nd lr ne nf ng lu nh bi translated">最小状态管理</h1><p id="bc0d" class="pw-post-body-paragraph jy jz iq ka b kb lw kd ke kf lx kh ki kj ly kl km kn lz kp kq kr ma kt ku kv ij bi translated">如前所述，我在项目中使用了新的 React Context API 进行状态管理。为什么我没有用 redux？<br/>首先，对于小型项目，减少过度冗余。请记住—操作、减少器、e2e 通信的 redux-thunk、connect()、组合减少器。我喜欢只用一个 React 来构建应用的想法。一开始，我将所有数据和更新程序放在一个文件存储中。</p><figure class="mb mc md me gt jr"><div class="bz fp l di"><div class="mp mq l"/></div></figure><p id="f84e" class="pw-post-body-paragraph jy jz iq ka b kb kc kd ke kf kg kh ki kj kk kl km kn ko kp kq kr ks kt ku kv ij bi translated">这样的实现满足了我的所有需求。但是在某些时候，我意识到有 300 多行代码已经足够了。<br/>所以我决定根据数据的类型——用户、信息、产品和主题——来划分数据。我将尽可能保持我的例子简单。首先，我将所有用户数据从存储状态转移到单独的类:</p><figure class="mb mc md me gt jr"><div class="bz fp l di"><div class="mp mq l"/></div></figure><p id="223d" class="pw-post-body-paragraph jy jz iq ka b kb kc kd ke kf kg kh ki kj kk kl km kn ko kp kq kr ks kt ku kv ij bi translated">每个模型接收两个重要参数。getState 该方法返回商店组件的“this.state”。<br/> rootUpdater —是 Store 组件的“this.setState”方法。然后我修改了我的商店组件:</p><figure class="mb mc md me gt jr"><div class="bz fp l di"><div class="mp mq l"/></div></figure><p id="f16c" class="pw-post-body-paragraph jy jz iq ka b kb kc kd ke kf kg kh ki kj kk kl km kn ko kp kq kr ks kt ku kv ij bi translated">想象这样一种情况，其中一个模型需要根据另一个模型内部的值进行一些计算。绕过 this.getState 方法，每个模型都可以访问所有数据树。<br/>这里有一个完整的例子:</p><figure class="mb mc md me gt jr"><div class="bz fp l di"><div class="mp mq l"/></div></figure><p id="a496" class="pw-post-body-paragraph jy jz iq ka b kb kc kd ke kf kg kh ki kj kk kl km kn ko kp kq kr ks kt ku kv ij bi translated">记住这个的每个调用。_rootUpdater 将重新呈现每个连接的消费者。考虑使用多个上下文来避免不必要的重新渲染。</p></div></div> </body> </html></title>