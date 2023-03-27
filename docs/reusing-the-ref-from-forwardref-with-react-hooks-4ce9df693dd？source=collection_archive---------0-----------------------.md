# 使用 React 挂钩重用“forwardRef”中的“ref”

> 原文：<https://itnext.io/reusing-the-ref-from-forwardref-with-react-hooks-4ce9df693dd?source=collection_archive---------0----------------------->

![](img/0393935df7d9083c0864c9449377e15c.png)

照片由[尼克·费因斯](https://unsplash.com/@jannerboy62?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄

我想，我们都已经开始使用 React v16 的新功能了。其中之一是转发`refs`到我们的组件的新方法。常见的语法是:

```
const FancyButton = React.forwardRef((props, ref) => (
  <button ref={ref} className="FancyButton">
    {props.children}
  </button>
))

// You can now get a ref directly to the DOM button:
const ref = React.createRef()
<FancyButton ref={ref}>Click me!</FancyButton>
```

# 问题

很简单，对吧？

但是，如果您需要在 forwardable 组件中重用`ref`来满足您的需求，该怎么办呢？这个问题并不像想象的那样容易解决。

假设我们有一个在组件`MeasuredInput`中使用的`input`。我们需要使用`ref`来测量那个`input`的尺寸，但是我们也希望`MeasuredInput`组件将引用转发给子`input`组件。所以让我们写一些代码:

```
const MeasuredInput = forwardRef((props, ref) => {
    // measure here dimensions of <input /> return (
        <input ref={ref} />
    )
})
```

因此，现在要实现测量，我们需要使用`ref`。让我们这样做而不考虑`ref`转发:

```
const MeasuredInput = props => {
    const ref = React.useRef(null) React.useLayoutEffect(() => {
        const rect = ref.current.getBoundingClientRect()
        console.log('Input dimensions:', rect.width, rect.height)
    }, [ref]) return (
        <input ref={ref} />
    )
}
```

如果我们想从`MeasuredInput`的外部转发`ref`会发生什么？— 🤷‍♂️

# 解决办法

`ref`物体就是我们从 React 文档[https://reactjs.org/docs/hooks-reference.html#useref](https://reactjs.org/docs/hooks-reference.html#useref)中知道的具有`{ current: null }`形状的物体。而`current`只是对一个 DOM 节点的引用。因此，理想情况下，我们希望在`MeasuredInput`中重用这个来自`forwardRef`的引用。像这样:

```
const MeasuredInput = React.forwardRef((props, ref) => {
    const innerRef = React.useRef(ref)// set ref as an initial value React.useLayoutEffect(() => {
        const rect = innerRef.current.getBoundingClientRect()
        console.log('Input dimensions:', rect.width, rect.height)
    }, [ref]) return (
        <input ref={innerRef} />
    )
})
```

但遗憾的是它并不这样工作:)来自外部的`ref`停留在`{ current: undefined }`中，并没有用`input`的引用值进行更新。

为了解决这个问题，我们需要为`ref`编写一些手动更新函数，并合并这些引用以使用单个引用值:

```
function useCombinedRefs(...refs) {
  const targetRef = React.useRef()

  React.useEffect(() => {
    refs.forEach(ref => {
      if (!ref) return

      if (typeof ref === 'function') {
        ref(targetRef.current)
      } else {
        ref.current = targetRef.current
      }
    })
  }, [refs])

  return targetRef
}const MeasuredInput = React.forwardRef((props, ref) => {
    const innerRef = React.useRef(null)
    const combinedRef = useCombinedRefs(ref, innerRef) React.useLayoutEffect(() => {
        const rect = combinedRef.current.getBoundingClientRect()
        console.log('Input dimensions:', rect.width, rect.height)
    }, [ref]) return (
        <input ref={combinedRef} />
    )
})
```

这将完全按照预期工作。老实说，我不认为它应该停留一段时间，因为解决方案相当大，并且肯定不容易理解和维护。所以希望 React 的维护者们早日找到更好的 API。

尽情享受吧！