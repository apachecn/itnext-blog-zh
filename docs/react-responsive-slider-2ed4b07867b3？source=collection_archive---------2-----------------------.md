# 反应响应滑块

> 原文：<https://itnext.io/react-responsive-slider-2ed4b07867b3?source=collection_archive---------2----------------------->

![](img/f7d9492e0ee37d6bef4e8461fe39a5e9.png)

## 介绍

在我的上一篇文章 [3 种在 React 应用中实现响应式设计的方法](https://medium.com/@jkeohan/3-ways-to-implement-responsive-design-in-your-react-app-bcb6ee7eb424)中，我讨论了几种在 React 应用中实现响应式设计的方法。这个应用的灵感来自 mars.nasa.gov 的[，在这篇文章中，我继续重建网站的其他部分，主要关注实现主页标题部分的底部滑块功能。](https://mars.nasa.gov/)

这是一个部署版本和 codesandbox:

app:【https://02nz9.csb.app/ 

code sandbox:[https://codesandbox.io/s/mars-slider-demo-02nz9](https://codesandbox.io/s/mars-slider-demo-02nz9)

## “BottomSlider”组件

创建一个组件来包含 slider 的所有元素和功能似乎是最实用的方法，尤其是因为我正在考虑将它包含在我想要构建的其他应用程序中。

因为我总是想让自己跟上最新最棒的反应，所以我决定加入钩子。我已经包含了 **useReducer** 来管理 **PREV/NEXT/RESET** 状态，包含了 **useEffect** 来在 window.innerWidth 值改变时启动 **RESET** 状态，包含了 **useRef** 来获取。支持滑块的光滑轨道元件。

这是基本结构。其他内容，如 reducer 的内部工作方式或 sliderController 对象，将在本文后面解释。

```
import React, { useReducer, useEffect, useRef } from  "react";
import './BottomSlider.css'function BottomSlider() { // useRef
 const slideTrackRef = useRef() // useReducer 
 const reducer = (state, action) => {
  ...rest of code
 }
 const [sliders, dispatch] = useReducer(reducer, sliderController) // useEffect
 useEffect( () => {
   dispatch('RESET')
 }, [windowWidth()]) ...rest of code
}
```

## 内容

内容是我从火星站点镜像的一组对象。为了便于组织，我将内容放入自己的名为 **BottomSliderData.js** 的文件中，并将其导入主 **BottomSlider.js** 组件。

```
// BottomSlider.js Component
import sliderContent from './BottomSliderData'// BottomSliderData.js
   export default[{
     title: 'Curiosity',
     value: '2440 SOLS ON MARS'
   },...additional objects]
```

## 呈现内容

因为我喜欢推断任何循环结构(。map)从组件**返回**语句**，**我选择创建一个名为 **renderSlickTrackSlides** 的变量，它存储映射到 **sliderContent** 数组和创建单个 slider 元素的结果。

```
const renderSlickTrackSlides = sliderContent.map( (d,i) => (
  <article className="slide" key={i}>
   <div className="image_and_description_container">
    <div className="readout">
     <div className="title">{d.title}</div>
     <div className="value">{d.value}</div>
    </div>
   </div>
   <span className="circle_plus"></span>
  </article>
))
```

的。 **slick-track** 将渲染那些元素，并且已经被分配了之前定义的**slide track ref**reference**。**

```
<div className="slick-track" ref={slideTrackRef}>  
   {renderSlickTracks}
</div>
```

**按钮被放置在**的两侧。slick-list** 元素，并被分配了直接从火星站点提取的背景图像。谢谢 NASA！**

```
<section className="dashboard">
 <div className="slide_container">
  <button className="slick-prev"></button>
  <div className="slick-list">
   <div className="slick-track" ref={slideTrackRef} >  
    {renderSlickTracks}
   </div>
   <button className="slick-next"</button>
  </div>
 </div>
</section>
```

## **添加事件侦听器**

**点击事件被添加到两个**。slick-prev** 和**。slick-next** 元素，并被配置为调用用户的**分派**方法，向其传递两个值之一: **"PREV"** 或 **"NEXT"** 。**

```
<section className="dashboard">
 <div className="slide_container">
  <button className="slick-prev" onClick={() => dispatch('PREV')}>
  </button>
   ...slick-list
  <button className="slick-next" onClick={() => dispatch('NEXT')}>
  </button>
 </div>
</section>
```

## **显示正确数量的幻灯片**

**响应式设计在这个项目中一直占据优先地位，所以我创建了 **windowWidth()** 函数，该函数返回 **window.innerWidth** 的当前值。**

****numOfVisibleSlides()** 函数调用 windowWidth()并返回一个值，该值等于基于特定断点当前正在播放的幻灯片的数量。**

```
const windowWidth = () => {
  return window.innerWidth;
};const numOfVisibleSides = () => {
  switch (true) {
   case windowWidth() <= 767 :
    return 1
   case windowWidth() <= 1023 :
    return 2
   default :
    return 3
  }
}
```

**创建了一个 **sliderController** 对象来跟踪以下属性:**

*   ****numOfVisibleSlides:** 当前窗口宽度下可见的幻灯片数量**
*   ****numOfSliders:**slider content 数组的当前长度**
*   ****translateValue:** 滑块的当前过渡值**

```
const sliderController = {
 numOfVisibleSlides: numOfVisibleSldies(),
 numOfSliders: sliderContent.length,
 translateValue: 0
}
```

## **使用用户管理下一个和上一个状态**

**对于更复杂的状态管理，useReducer 挂钩是一个很好的选择。如果您是 useReducer 的新手，那么额外的代码开销初看起来可能很大，但是，它是管理您的状态和应用程序逻辑的一个很好的工具，从而使它更加直观和可读。**

**useReducer 被传递了一个 **reducer** 函数，该函数被绑定到 **dispatch** 方法和绑定到 **sliders** 变量的 **sliderController** 数组。**

```
const [sliders, dispatch] = useReducer(reducer, sliderController)
```

****【下一步】**和**【上一步】**动作与当前分配给前进/后退按钮的事件监听器相关联。**“重置”**被添加以重置用户的用户状态，从而在 window.innerWidth 值改变时将滑块转换回其原始位置。**

```
function reducer(state,action) {
 switch(action) {
  case 'RESET' :
    return sliderController
  case 'PREV' :
   return handlePrev(state)
  case 'NEXT' :
   return handleNext(state)
  default :
   return state
 }
}
```

**useReducer 利用 switch 语句，因为它比试图读取一系列 if/else if/else 语句更直观。**

## **走向**

****handleNext()** 函数需要执行以下操作:**

*   **计算沿水平轴向左转换的距离**
*   **应用过渡并显示序列中的下一张幻灯片**
*   **一旦达到幻灯片的最大数量限制，禁止进一步过渡**
*   **返回包含这些更改的状态的新版本**

```
function handleNext({numOfVisibleSlides, numOfSlides, translateValue}) { if (numOfVisibleSlides >= numOfSlides) {
    return { numOfVisibleSlides, numOfSlides, translateValue }
   } let value = numOfVisibleSides() translateValue += -(slideTrackWidth() / value); return {
   numOfVisibleSlides: numOfVisibleSlides += 1,
   numOfSlides,
   translateValue
  }
}
```

## **向后移动**

****handlePrev()** 函数需要执行以下操作:**

*   **计算沿水平轴向右过渡的距离**
*   **应用过渡并显示序列中的下一张幻灯片**
*   **一旦达到幻灯片的基本限制，防止进一步过渡**
*   **返回包含这些更改的状态的新版本**

```
function handlePrev({numOfVisibleSlides, numOfSlides, translateValue}) { if (numOfVisibleSlides <= numOfVisibleSides()) {
  return { numOfVisibleSlides, numOfSlides, translateValue }
 } let value = numOfVisibleSides() translateValue += slideTrackWidth() / value; return {
  numOfVisibleSlides: numOfVisibleSlides -= 1,
  numOfSlides,
  translateValue
 }
}
```

**两个函数都调用 **slideTrackWidth** 函数，该函数返回当前**的宽度。光滑轨道**。它的宽度将根据窗口宽度而变化。**

```
const slideTrackWidth = () => {
 return slideTrackRef.current.clientWidth;
};
```

## **结论**

**经过几次重构和一点点钩子魔术，我创建了一个可重用的响应组件。**