# 停止在 React 中重新解决获取和管理 API 数据的问题

> 原文：<https://itnext.io/stop-re-solving-the-problems-of-fetching-and-managing-your-api-data-in-react-44e425a92ff8?source=collection_archive---------1----------------------->

请允许我向您介绍 [Jetset](https://github.com/DigitalGlobe/jetset/) ，这是一个库，它承诺将您所有的前端 RESTful API 逻辑抽象成简单、直观和可重用的模块。

# 问题

1.  已经[做了一些](https://loopback.io/) [工作](http://jsonapi.org/)来标准化后端的 API，但是前端开发人员继续解决并再次解决获取数据、缓存数据或以其他方式最小化重新获取以及管理相关 UI 状态的问题。

*   为了从 API 请求数据，你使用 superagent、fetch、同构-fetch、XMLHttpRequest、oboe 和/或这些之上的抽象吗？
*   对于 API 数据缓存和状态管理，你使用 Redux，React 组件状态，Flux，其他？
*   你会乐观地更新你的 UI 吗？如果是，您使用什么模式来恢复数据和更新 UI？

2.GraphQL + Relay 和类似的解决方案最终显示了巨大的发展前景，但是 a)配置可能非常复杂且不直观，b)不是每个人现在或很快都会转换现有的 API，以及 c)许多 API 需要支持许多不同类型的客户端，其中一些可能不太容易与 GraphQL 一起工作。在未来很长一段时间内，REST 将是实现 API 的常用方法。

# 一个解决方案

![](img/3db266f8c124b7d8f3a9eff33f0e92e6.png)

使用[喷射装置](https://github.com/DigitalGlobe/jetset/)。它抽象出了所有的获取、缓存和状态管理细节。它以安全和优化的方式将 API 数据直接注入到组件中，这样您就可以专注于构建视图。

几乎不需要任何配置，就可以获得给定资源的`create(), list(), get(), update(), and delete()`,作为道具直接传递给组件。例如:

```
import { apiDecorator } from 'jetset';const Api = apiDecorator({
  url: 'http://domain.com/api',
  users: '/users'
})const MyComponent = Api( props => {
  const users = props.users.list();
  return (
    <div>
      { users.data.map(({ data: user }) => <div>{ user.fullName }</div> )}
    </div>
  )
})
```

这里我们有一个高阶组件，它将您的 url 和资源转换成 props，这些 props 公开了获取、缓存和呈现数据的方法。

注意，我们在渲染时调用了`list()` **。第一次被调用时，它实际上触发了一次提取并缓存了数据，这样在随后的渲染中，它只是从缓存中读取数据，而不是再次提取。您不需要自己设置获取或触发操作或管理任何状态。尽管如果你需要的话，有很多方法可以连接到获取生命周期的不同阶段。更多信息见下文和[文档](https://github.com/DigitalGlobe/jetset/blob/master/docs/index.md)。**

默认情况下，所有 CRUD 操作都捆绑在一起:

```
export default Api(({ users }) =>
  <div>
    { users.list().data.map( user => (
      <div>
        <span>{ user.data.name }</span>

        <button onClick={() => user.update({ name: 'Pat' }) }>Rename to Pat</button>

        <button onClick={ user.delete }>Delete</button>

        <button onClick={() => users.get( user.data.id ) }>Get detail</button>
      </div>
    ))}

    <button onClick={() => users.create({ name: 'Chris' }) }>Create user named Chris</button>
  </div>
)
```

指示提取状态的关键帧助手:

```
const users = props.users.list();users.isPending ?
  <span>Loading...</span> :users.error ?
  <span>Error: {list.error.message}</span> :<div>
  { users.data.map( user => <div>{ user.data.name }</div> ) }
</div>
```

除了标准的创建、列表、获取、删除之外，定制您的路线:

```
const routes = {
  default: '/users',
  onError: error => localStorage.removeItem('some_token')
  getUserAlbums: id => ({ method: 'get', route: `/users/${id}/albums`, usesCache: true })
}

<Api ... users={{ routes }}>
```

还有更多！更多信息见[https://github.com/DigitalGlobe/jetset](https://github.com/DigitalGlobe/jetset)及其[更详细文件](https://github.com/DigitalGlobe/jetset/blob/master/docs/index.md)。