# [错误修复，不适用]修复版权快照中的年份

> 原文：<https://itnext.io/bug-fix-n-a-fix-year-in-copyright-snapshot-571b7d75dd95?source=collection_archive---------6----------------------->

![](img/3c8281f65de1da115835adfe7d873519.png)

这种编程错误可以在任何地方出现，所以你最好小心点——照片摄于南澳大利亚的 Memory Cove。

## 这是我今年的第一次公关，也是我们所有人的一个教训。

我运行了`git pull upstream develop`，看到没有人对`xMas`做任何改动，我悄悄松了一口气。我跑`git push`只是为了确保我的叉子是同步的。在运行`git push`时，`package.json`文件有一个预处理器步骤，在推送之前链接和测试所有东西。

该死的测试失败了。

*为什么*测试失败了？

因为现在是 2021 年，而`Copyright`组件的快照仍然显示 2020 年，诅咒它的名字。

组件:

```
const Copyright = ({ holder }) => (
  <Typography>
    {'Copyright © ' + holder}
    {new Date().getFullYear()}.
  </Typography>
)Copyright.propTypes = {
  holder: string.isRequired
}
```

用`Jest`测试。

```
describe('components > Copyright', () => {
  const holder = 'test'
  let component beforeAll(() => {
    component = render(<Copyright holder={holder} />)
  }) it('rendered without crashing', () => {
    expect(component.asFragment()).toMatchSnapshot()
  })
})
```

这很容易确定。

第一个，也是最懒惰的，也是最正确的解决方案就是运行:

```
npm test -- -u
```

并重新生成快照，但这样做的后果是在一年后将问题变成别人的问题。

我们不是那样的人，对吧？

让我们重写`Copyright`组件，使其接受一个可选的日期字符串属性`today`。

```
const Copyright = ({ holder, today }) => (
  <Typography>
    {'Copyright © ' + holder}
    {new Date(today).getFullYear()}.
  </Typography>
)Copyright.propTypes = {
  holder: string.isRequired,
  today: string // zulu-time string only used in testing.
}Copyright.defaultProps = {
  today: undefined
}
```

并更新测试

```
describe('components > Copyright', () => {
  const holder = 'test'
  const today = '2021-01-03T22:12:18.217Z'
  let component beforeAll(() => {
    component = render(<Copyright *holder*={holder} *today*={today} />)
  }) it('rendered without crashing', () => {
    expect(component.asFragment()).toMatchSnapshot()
  })
})
```

如今，现在

```
npm test -- -u
```

强制更新快照。

如果没有可选的`today`字段，组件的行为将完全相同，因为如果`new Date`将`undefined`作为参数接收，它的行为将是正确的。

在`Copyright`组件中的日期每年只改变一次，所以直到我今天开始工作之前没有人注意到。

```
git commit -am "n/a added optional today to Copyright"
git flow feature publish fix-copyright-date
```

今年的第一次公关，这是一个美丽的晴天。

每当有处理日期的组件时就使用这种技术，否则，您可能会在时钟的每一秒都生成新的快照。

大家新年快乐。

—

像这样但不是订户？你可以通过[davesag.medium.com](https://davesag.medium.com/membership)加入来支持作者。