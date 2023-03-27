# 使用 React & TypeScript & HOC 简化交互式 SVG

> 原文：<https://itnext.io/interactive-svg-made-easy-using-react-typescript-hoc-4dcc1ad9e918?source=collection_archive---------5----------------------->

![](img/1882b802a135ccd11af3e0c57c81d3aa.png)

TL；正如我在[上一篇文章](https://medium.com/@yofujimo/enlighten-your-react-components-with-hocs-from-recompose-47519f76d851)中所写的，简单的&无状态 React 组件可以使用高阶组件(hoc)转换成交互式组件。
在本文中，我将展示一个使用 React & TypeScript 的交互式 SVG 示例。如果您想先尝试一下，请转到 CodePen 示例。

# 拖动的状态管理

我们并不认为它是日常的，但是鼠标拖动是一种状态转换。当你拖动一个文件时，首先你要指向该文件，然后按下鼠标按钮，然后移动光标到目的地，最后释放按钮。

这个顺序可以描述如下。

```
MouseDown -> MouseMove -> ... -> MouseMove -> MouseUp
```

在整个序列中，应该通过将当前位置反映到 UI 组件来指示它。

这就是管理拖动的状态。

*   **isDown** :鼠标按钮按下时为真。
*   **posX/posY** :父 SVG 元素中的组件位置。
*   **screenX/screenY** :屏幕坐标中拖动开始的位置。

```
interface DraggableState {
  isDown: boolean;
  posX: number;
  posY: number;
  screenX: number;
  screenY: number;
}
```

# onMouseDown

当此事件被触发时，状态应该保持初始屏幕位置。因为鼠标运动与屏幕坐标相关，所以从 [MouseEvent](https://reactjs.org/docs/events.html#mouse-events) 中检索 screenX 和 screenY 属性是最健壮的方法之一。

```
 onMouseDown: (state: DraggableState) => (e: MouseEvent) => {
      return {
        ...state,
        isDown: true,
        screenX: e.screenX,
        screenY: e.screenY
      }
    },
```

# onMouseMove

onMouseDown 之后，每次触发此事件时都应更新组件位置。新位置可以从屏幕位置的差值计算出来。

```
 onMouseMove: (state: DraggableState) => (e: MouseEvent) => {
      if (!state.isDown) {
        return { ...state }
      }
      const shiftX = e.screenX - state.screenX;
      const shiftY = e.screenY - state.screenY; return {
        ...state,
        posX: state.posX + shiftX,
        posY: state.posY + shiftY,
        screenX: e.screenX,
        screenY: e.screenY,
      };
    },
```

注意:如果鼠标光标移动得太快，MouseMove 事件可能会发送到封装组件。

# onMouseUp

该事件用于完成拖动并重置状态。

```
 onMouseUp: (state: DraggableState, props: DraggableProps) => (e: MouseEvent) => {
      return { ...state, isDown: false, screenX: 0, screenY: 0 }
    },
```

# CodePen 示例

这里有一个例子。尝试拖动彩色圆圈。

注意，通过预先应用*enhancementwithdragable*，每个圆具有不同的状态。

# 结论

即使在 web 应用程序中，UI 交互也需要各种状态管理。通过使用 React & HOCs，SVG 标记和状态管理被清晰地分开，并可以以直观的方式结合起来。