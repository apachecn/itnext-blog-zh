# 通过一致地应用这些模式和规则来驯服大型 React 项目

> 原文：<https://itnext.io/tame-large-react-projects-by-consistently-applying-these-patterns-and-rules-b7c70baf5105?source=collection_archive---------0----------------------->

![](img/e14aa091fc320221b757feca13fa4b0a.png)

就像沙子一样，你可以把不同种类的组件放进不同的桶里。([照片由 Don DeBold](https://www.flickr.com/photos/ddebold/15991919514) 拍摄，并在[许可](https://creativecommons.org/licenses/by/2.0/)条款下使用)

## 维护和处理纯粹和简单的 vs 显示，以及自我管理的组件

作为团队的一员，我最近一直在做一个项目，构建一个 React 组件库，用于许多相关的企业环境中。作为该过程的一部分，我们发现根据组件从哪里获得数据来对组件进行分类是有意义的。

我们还确定了一些规则，帮助我们保持组件的一致性和可预测性。

# 组件类型

React 组件可以从几个地方获取数据:

1.  组件从`props`获取**其所有**数据，或者
2.  它从`props`获得一些数据，从其他地方获得一些`context`、`state`、`app-state`等

进一步缩小范围，我们发现，在仅从`props`获取数据的组件集合中，一些可以被视为*纯*组件，但是我们也有*可记忆*组件，以及*不可记忆*组件，并且不是所有的*可记忆*组件都一定是*纯*。

## 纯成分

一个*纯*组件是一个当提供相同的`props`时总是呈现相同输出的组件。*纯*组件不从除了它们的`props`之外的任何地方拉数据，它们不管理任何内部`state`，并且，如果`props`是简单的，组件可以被*记忆*以确保它仅在其`props`改变时才重新呈现。*纯*成分无副作用。

例如:

```
import { memo } from 'react'
import { string } from 'prop-types'const Wylde = ({ city }) => (
  <div>
    As pure as {city} snow
  </div>
)
Wylde.propTypes = { city: string.isRequired }export default memo(Wylde)
```

## 我说的“简单”是什么意思？

React `[memo](https://reactjs.org/docs/react-api.html#reactmemo)`函数将一个*纯*组件作为输入，并返回一个相对于`props`被*缓存*的组件，如果所提供的`prop`值自上次渲染后没有改变，则阻止其重新渲染。这是一个很好的优化，但经常被误用。

默认情况下，`memo`的问题在于，使用简单的`===`等式来比较`props`。因此，如果你的`prop`是一个函数、数组、对象，或者基本上是任何不能通过`===`进行比较的东西，那么`memo`仍然允许组件重新渲染。

## 添加杂质

你可以提供第二个参数，一个`arePropsEqual`函数给`memo`函数来覆盖默认行为，但是如果你的比较逻辑比你的渲染代码更复杂，那么这只是浪费时间。如果你的`arePropsEqual`函数忽略了任何一个`props`，那么你的组件就不再是纯粹的*。*

以下面的组件为例:

```
import { memo } from 'react'
import { string, func } from 'prop-types'const Clicky = ({ label, onClick }) => (
  <button onClick={onClick}>{label}</button>
)
Clicky.propTypes = {
  label: string.isRequired,
  onClick: func.isRequired
}const arePropsEqual = (prev, next) =>
  prev.label === next.labelexport default memo(Clicky, arePropsEqual)
```

这只会在`label`改变时重新渲染，并且会忽略`onClick`函数中的任何差异。这可能会产生意想不到的后果。它不再是一个纯粹的*组件，因为如果你提供一个不同的`onClick`函数，你仍然会得到传递给`button`的旧函数，因为组件不会重新呈现。*

同样的，对*来说，把一个数组、函数或对象当作道具的组件是没有意义的。*

```
import { memo } from 'react'
import { arrayOf, string } from 'prop-types'const Listy = ({ items }) => (
  <ul>
    {items.map(item => (
      <li key={item}>{item}</li>
    )}
  </ul>
)
Listy.propTypes = { items: arrayOf(string).isRequired }const arePropsEqual = (prev, next) =>
  prev.items.reduce((acc, elem, i) =>
    prev.items[i] === next.items[i]
      ? acc
      : [...acc, elem], []).length === 0export default memo(Listy, arePropsEqual)
```

如果列表长于几个条目，那么`Listy`组件可能会花更多的时间来比较数组，而不是重新呈现列表。然而它仍然是一个纯粹的组件；只是一个低效的。长的对象列表不应该被传递给一个 *memoised* 函数。事实上，我认为如果一个组件的任何属性不是布尔值、字符串或数字，你就不应该费心去记忆它。

我应用以下分类:组件是

1.  *简单*如果是*记忆*纯组件，
2.  *如果是*纯*而不是*记忆*则显示*，否则
3.  如果它从`props`以外的地方获取数据，则它是*自管理的*。

# 组件文件结构

构建组件库时，以一致的方式对组件的组成部分进行分组非常重要。该库的细节取决于组件是简单的、*显示的*还是自管理的*。*

## 简单组件

```
components/Simple/
  index.js
  index.spec.js
  Simple.js
  Simple.spec.js
  Simple.stories.js
  config.js
  utils.js
  utils.spec.js
```

一个简单的*组件的`index.js`文件将被*

```
export { default } from './Simple'
```

> 规则:避免从一个文件中导出多个组件。

`index.spec.js`文件将测试底层组件正在被导出。

```
import Index from '.'
import Simple from './Simple'describe('components/Simple', () => {
  it('is the same component', () => {
    expect(Index).toBe(Simple)
  })
})
```

实际的`Simple.js`必须是一个*纯*，*由简单的`props`组成的*组件。

```
import { memo } from 'react'
import { bool, number, string } from 'prop-types'const Simple = ({ small, count, label }) => small
  ? (
      <div><span>{count}</span></div>
    )
  : (
      <div>
        <label>{label}</label>
        <span>{count}</span>
      </div>
    )Simple.propTypes = {
  small: bool,
  count: number.isRequired,
  label: string.isRequired
}
Simple.defaultProps = { small: false }export default memo(Simple)
```

你会注意到我在上面默认了`small`到`false`。原因是我一直应用的一条规则，简单来说就是:

> 规则:所有`boolean`道具必须可选，默认为`false`。

让你可以像这样使用组件

```
<Simple count={c} label={l} />
```

如果你想让它显示`count`和`label`，或者

```
<Simple small count={c} label={l} />
```

如果你想让它只显示`count`。

`Simple.spec.js`简单地测试，给定一个合理的输入范围，它将提供有效的输出。

```
import { render } from '@testing-library/react'import Simple from './Simple'const label = 'some test label'describe.each`
  small        | count
  ${undefined} | ${0}
  ${false}     | ${1}
  ${true}      | ${2}
`('components/Simple/Simple',
  ({ small, count }) => {
    let component beforeAll(() => {
      component = render(<Simple
        small={small}
        count={count}
        label={label}
      />)
    }) it('rendered okay', () => {
      expect(component.toFragment()).toMatchSnapshot()
    })
 })
```

`Simple.stories.js`提供了单独演示组件的[故事书](https://storybook.js.org)故事。这通常既用于组件的文档，也用于组件开发和维护时的用户验收测试。

`config.js`文件是可选的，用于包含任何组件特定的配置，同样`utils.js`也是可选的，用于保存组件可能需要的任何*纯*功能。*简单的*组件不会有任何`hooks`。组件的故事书`stories`可能也会访问`config`和`utils`。

## 显示组件

一个*显示*组件的文件夹结构本质上与一个*简单*组件相同，只是它可能包括一个可选的`shapes.js`文件。

```
components/Display/
  index.js
  index.spec.js
  Display.js
  Display.spec.js
  Display.stories.js
  config.js
  shapes.js
  utils.js
  utils.spec.js
```

在我们的`shapes.js`中，我们将只导出我们正在展示的东西的形状。

```
import { string } from 'prop-types'export const thingShape = {
  id: string.isRequired,
  text: string.isRequired,
  title: string.isRequired
}
```

在这种情况下，`Display.js`可能类似于

```
import { Fragment } from 'react'
import { arrayOf, func, shape } from 'prop-types'import { thingShape } from './shapes'const Display = ({ things, handleClick }) => (
  <dl>
    {things.map(({ title, text, id }) => (
      <Fragment key={id}>
       <dt>
         <button
           type="text"
           onClick={handleClick(id)}
           data-testid={id}
         >
           {title}
         </button>
       </dt>
       <dd>{text}</dd>
      </Fragment>
     )}
  </dl>
)
Display.propTypes = {
  things: arrayOf(shape(thingShape)).isRequired,
  handleClick: func.isRequired // id => evt => { }
}export default Display
```

虽然这仍然是一个纯粹的组件，但由于它被提供了一个对象数组和一个函数作为`props`，所以不容易被*记忆。*

像以前一样，我们将像`Display.spec.js`一样测试它:

```
import { render, screen, fireEvent } from '@testing-library/react'import Display from './Display'const makeThing = (id, title, text) = ({ id, title, text })
const theThings = [
  makeThing('1', 'One', 'This is the one'),
  makeThing('2', 'Two', 'This is number two'),
  makeThing('3', 'Three', 'The magic number')
]describe.each`
  things
  ${[]}
  ${theThings}
`('components/Display/Display',
  ({ things }) => {
    const onClick = jest.fn()
    const handleClick = jest.fn().mockReturnValue(onClick)
    let component beforeAll(() => {
      component = render(<Display
        things={things}
        handleClick={handleClick}
      />)
      const firstButton = screen.getByTestId('1')
      fireEvent.click(firstButton)
    }) it('used the click handler', () => {
      expect(handleClick).toHaveBeenCalled()
    }) it('rendered okay', () => {
      expect(component.toFragment()).toMatchSnapshot()
    }) it('clicking worked as expected', () => {
      expect(onClick).toHaveBeenCalled()
    })
 })
```

同样，*显示*组件是*纯*组件，因此不会有任何挂钩，但是它们可以引用*纯*实用函数和不可变的配置数据。*显示*组件会有故事书故事，可以引用本地`utils`或`config`，本地`utils`只是*纯*功能，易于测试。

## 自我管理的组件

最后一类组件不是纯粹的组件。他们可能会跟踪一些内部状态，或者他们可能会从外部应用程序状态(如 Redux store 或 React 上下文)引入状态。他们可能会使用像`useEffect`这样的钩子，使得组件在安装和卸载时触发副作用。

```
components/SelfManaged/
  index.js
  index.spec.js
  index.stories.js
  SelfManaged.js
  SelfManaged.spec.js
  SelfManaged.stories.js
  config.js
  hooks.js
  hooks.spec.js
  shapes.js
  utils.js
  utils.spec.js
  wrappers.js
```

在*自管理的*组件中，最好将它们分离成进行外部数据交互的位，然后该位将数据向下传递给一个*纯*组件。在我们的例子中，假设我们有一个组件，它被提供了`Thing`的`id`，当它被挂载时，它加载`Thing`(使用一个钩子)，将加载的数据向下传递到*纯* `SelfManaged.js`。

这就引出了我的下一条规则。

> 规则:所有没有作为`prop`提供的数据必须来自一个钩子。

通过遵循这条规则，你可以保持组件本身非常简单，并避免任何复杂性。我们这样做的原因是，通常，从事组件工作的开发人员真的不想考虑数据来自哪里。请参阅我关于[前端服务层](/what-are-front-end-service-layers-4dba95db21bb)的文章了解更多信息。

所以我们的`index.js`会是这样的:

```
import { string } from 'prop-types'
import { useThing } from './hooks'import PureSelfManaged from './SelfManaged'const SelfManaged = ({ id }) => {
  const thing = useThing(id) return thing ? <PureSelfManaged {...thing} /> : null
}
SelfManaged.propTypes = { id: string.isRequired }export default SelfManaged
```

我们将把`useThing`如何工作的任何细节放在一边，但是让我们假设它在内部从一个 API 或者一些异步的东西加载一些数据，所以最初`thing`的值是`undefined`或者`null`，直到它被实际的`thing`数据填充。

您会注意到，我们将`thing`数据作为其实际的`props`而不是`thing={thing}`传递给了`PureSelfManaged`组件。这有双重原因。首先，如果`thing`的属性很简单，那么我们可以*记忆*，其次，当我们为`SelfManged.stories.js`制作故事时，我们可以用他们自己的控件拼出单个的`props`，而不是只有一个道具。

因此，重用上面显示组件中的`shape.js`示例，我们可以将`SelfManaged.js`定义为:

```
import { memo } from 'react'import { thingShape } from './shapes'const SelfManaged = ({ id, title, text }) => (
  <>
    <dt>
      {title}
      <tt>{id}</tt>
    </dt>
    <dd>{text}</dd>
  </>
)
SelfManaged.propTypes = { ...thingShape }export default memo(SelfManaged)
```

`index.stories.js`可能会使用 Storybook *loaders* 和*decorator*来预加载数据，并将组件包装在任何可能需要的上下文提供者中，以正确地允许`useThing`挂钩运行，以及将组件封装在组件可能期望的外部标签(在本例中为`<dl>…</dl>`)中的包装器。`SelfManaged.stories.js`将为`thing`的每个属性提供控件，并提供相同类型的包装器。为了保持美观和干燥，你可以把普通的包装放在一个共享的`wrappers.js`中。

## 污染性*纯*成分

*如果纯*组件在内部使用*自管理*组件，它们就会变得不纯，尽管这通常是不可避免的，但最好承认这一点，并像对待*自管理*组件一样对待这些组件，尤其是在为它们编写测试和故事时。

## 日期和时间敏感组件

假设有一个组件因为过期而突出显示了一个`date`。为了测试这一点，您需要为`today`提供一个值，否则一旦日期改变，您的快照测试就会失败。

```
import { string } from 'prop-types'
import { useDates } from 'hooks/dates' // omitted for brevity
import Date from 'components/Date' // assume this existsconst Due = ({ date, today }) => (
  const { isBefore } = useDates(today)

  return <Date date={date} highlight={!isBefore(date)} />
}
Due.propTypes = {
  date: string.isRequired, // Zulu Time string
  today: string // only supplied in tests
}
Due.defaultProps = {
  today: undefined
}export default Due
```

从表面上看，这看起来像是*可记忆的*，但这并不是因为`today`的值只在测试中提供，通常是`undefined`，这意味着它将默认为调用`useDates`钩子时`new Date()`返回的值，假设`useDates`钩子类似于:

```
const toDate = d => d
  ? typeof d === 'string'
    ? new Date(d)
    : d
  : new Date()export const useDates = (tDay) => {
  const today = toDate(tDay) const isBefore = d => toDate(d).getTime() < today.getTime() return { isBefore }
}
```

所以我的另一个原则是

> 规则:如果它比较日期或时间，那么总是提供`date`或`time`作为 [*祖鲁时间*](https://www.utctime.net/z-time-now) 字符串，并且总是提供一个值给`today`与之比较。

## 字符串可选时

想象一个简单的`Button`组件，它带有一个可选的`label`。

```
import { func, string } from 'prop-types'const Button = ({ label, onClick }) => (
  <button type="button" onClick={onClick}>{label}</button>
)
Button.propTypes = {
  label: string,
  onClick: func.isRequired
}
Button.defaultProps = { label: undefined }export default Button
```

将标签默认设置为 null 或''，这很有吸引力，但是这样做可能会引入错误，特别是当您将属性传递到组件树的更深处时。

> 规则:可选字符串`prop`的默认值必须始终为`undefined`。

# 最后

在小型项目中，您可以将组件放在单个文件中，跳过测试，不在故事书中记录组件，以及用各种钩子、实用程序和配置将组件弄乱，但是随着项目的进行，您会后悔的。

更好的做法是按照你想玩的方式来练习，并且从一开始就以一致和结构化的方式来构建你的组件。这意味着每个组件都有一个文件夹，既有一个`index.js`文件又有一个`Component.js`文件，即使是普通的组件也是如此。

## 组件类型

1.  *简单*如果是*记忆*纯*组件，*
2.  *如果是*纯*而不是*记忆*则显示*，否则
3.  如果它从`props`之外的地方获得数据，那么它就是*自管理的*。

## 组件规则。

1.  避免从一个文件中导出多个组件
2.  所有布尔值`props`必须是可选的，默认为`false`。
3.  所有没有作为`prop`提供的数据必须来自一个钩子。
4.  如果它比较日期或时间，那么总是提供`date`或`time`作为祖鲁语时间字符串，并且总是提供一个值给`today`与之比较。
5.  可选字符串`prop`的默认值必须始终为`undefined`。

# ⚡

# 链接

*   [https://reactjs.org/docs/react-api.html#reactmemo](https://reactjs.org/docs/react-api.html#reactmemo)
*   [https://storybook.js.org](https://storybook.js.org)
*   [https://it next . io/what-are-front-end-service-layers-4 DBA 95 db 21 bb](/what-are-front-end-service-layers-4dba95db21bb)
*   [https://www.utctime.net/z-time-now](https://www.utctime.net/z-time-now)

这个规则的例外是，当您将组件分组到`components/index.js`中，以便 Rollup 之类的工具在生成包时使用。

—

像这样但不是订户？你可以通过[davesag.medium.com](https://davesag.medium.com/membership)加入来支持作者。