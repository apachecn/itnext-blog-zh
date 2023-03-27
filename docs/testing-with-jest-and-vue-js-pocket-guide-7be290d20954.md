# 用 Jest 和 Vue.js 测试:袖珍指南

> 原文：<https://itnext.io/testing-with-jest-and-vue-js-pocket-guide-7be290d20954?source=collection_archive---------2----------------------->

![](img/89163d259d990e0bb905583db07b2af3.png)

在测试应用程序时，我从过去与 **Vue.js** 的合作中收集了一些例子，并意识到我可以写一篇文章来帮助我的同事和(希望)你。

Vue.js 是一个非常流行的开源 javascript 框架，用于构建用户界面和单页面应用程序，易于理解，许多公司开始采用。

为了测试 **Vue.js** 应用程序，我使用 [**Vue 测试工具**](https://github.com/vuejs/vue-test-utils) 作为 javascript 测试工具。它有很多好的方法来帮助你安装你的组件和测试你需要测试的案例。比如单元测试你的组件功能或者测试交互。

写这篇文章的原因是为了测试应用程序的某些部分，你总是要做同样的事情。你有办法测试函数，模拟请求，模拟函数返回，等等。

那么，为什么不创建一个袖珍指南供以后使用呢？
**我们开始吧？**

# 组件状态

你需要小心的一件事是改变状态。在 Vue 中，如果你不声明状态属性，你将不能改变它们，因为它们不是反应性的！

请确保从您需要的所有属性开始，不要这样做，因为这样做是行不通的:

```
beforeEach(() => {
  state = {};
}it('should do someting', () => {
 state.foo = 'bar';
});
```

您需要:

```
beforeEach(() => {
  state = {
    foo: 'hello',
  }
}it('should do someting', () => {
 state.foo = 'bar';
});
```

注意，我使用了`beforeEach`方法在每次测试前重置状态。

# 元件安装

首先要做的是安装组件。为此，我们将使用 [**Vue 测试工具**](https://github.com/vuejs/vue-test-utils) 来帮助我们。

`shallowMount`是我们将要使用的第一种方法。此方法允许您挂载组件，并将使用存根子组件进行呈现。

像这样，你也不需要担心使用或安装它们。这个方法将为您挂载和渲染所有内容。

## Vue 实例

为了在添加组件、混音、道具等时创建一个 Vue 类`wrapper`而不污染全局 Vue 类，我们也将使用 **Vue 测试工具**中的一个方法，即`createLocalVue`方法。

```
import Vue from 'vue';
import { shallowMount, createLocalVue } from '@vue/test-utils';const localVue = createLocalVue();describe('caption.vue', () => {
  let wrapper;beforeEach(() => {
    wrapper = shallowMount(ComponentName, {
      localVue,
      propsData: {},
    });
  });
});
```

# 模拟功能

为了模仿函数，我们将得到 **Jest** 和的帮助，这是一个特殊的方法`fn`，它返回一个模仿函数，在这里你可以访问一些有用的方法。下面我会给你看更多！

要模拟 Vue 实例中的方法，可以使用 Vue 类的`vm`变量进行访问。

`wrapper.vm.didBlur = jest.fn();`

## 函数的模拟返回值

如果您需要模拟模块中某个方法的返回值，您将需要`require`整个模块，即使您已经使用了命名导出。

```
const captionsHelper = require('@/js/helpers/captions');
captionsHelper.didClickOutside.mockReturnValueOnce(false);
```

## 期待 n 次函数调用

如果您希望某个函数被调用`n`次，那么在另一个`it`语句中调用该函数时要小心。

对`expect n`函数的调用:
`expect(wrapper.vm.didBlur.mock.calls.length).toBe(1);`

# 预期是否发出了事件

## 通过组件发出的事件

*   您需要调用将发出事件的函数
*   检查是否发出了事件
*   检查传递了哪些参数

```
describe('#didBlur', () => {
  it('should emit setActiveCaptionState event', () => {
    wrapper.vm.didBlur();

    expect(wrapper.emitted('setActiveCaptionState')).toBeTruthy();
    expect(wrapper.emitted('setActiveCaptionState')).toEqual([
      [{ caption: wrapper.vm.subtitle, isActive: false }],
    ]);
  });
});
```

## 通过全局$eventHub 的事件

*   用 jest 创建一个存根函数
*   添加存根作为事件的回调
*   调用将发出事件的方法
*   检查是否发出了事件或传递了参数

```
it('emit handleCaptionClicked event', () => {
  const stub = jest.fn();
  wrapper.vm.$eventHub.$on('handleCaptionClicked', stub);
  wrapper.vm.handleClick(); expect(stub).toBeCalled();
  expect(stub).toBeCalledWith(wrapper.vm.subtitle.start_time);
});
```

# 检查是否调用了组件中的方法

*   调用将发出事件的方法
*   检查是否发出了事件或传递了参数

```
it('emit changeLineBackgroundWarning event', () => {
  wrapper.vm.changeLineBackgroundWarning  = jest.fn();
  wrapper.vm.handleInput(event);

  expect(wrapper.vm.changeLineBackgroundWarning).toHaveBeenCalled();
});
```

# 测试观察者

*   设置你正在观看的道具或状态
*   改变你正在观看的道具或状态
*   预期

```
it('should change the state prop isRead to true', () => {
  wrapper = shallowMount(Caption, {
    localVue,
    propsData: {
      subtitle: {
        start_time: 0.5,
      },
      currentTime: 0,
    },
  });

  wrapper.setData({ isRead: false });
  wrapper.setProps({
    currentTime: 1,
  }); expect(wrapper.vm.isRead).toBeTruthy();
});
```

# 独立测试 Vuex

不要忘记您需要导入您想要测试的`real`突变/动作/getter，因为您稍后会调用它。

## 测试操作

*   你应该从商店打电话给行动
*   提交、分派等应该是 jest 函数，这样你就可以检查它们是否被调用

如果您想要检查某个操作是否调度了其他操作或提交了任何变异，您可以执行以下操作:

```
// Check if a specific action has been called
it('should initiate the smartcheck plugin', () => {
  state.showSmartcheck = true;
  actions.setTaskState(
    { dispatch, commit, getters, state },
    { taskFromServer: task, localStorageTaskType: 'captioning' },
  ); expect(dispatch).toHaveBeenCalledWith('initSmartcheckPlugin');
});
```

## 测试吸气剂

对于 getter，您唯一需要做的就是检查 getter 是否返回了您所期望的结果。根据您的当前状态，它可以是不同的数据结构，如对象或不同的数据。

```
describe('taskInfo', () => {
  const state = {
    id: 'cenas',
    type: 'cenas',
    status: 'cenas',
  }; it('should return the right properties', () => {
    expect(getters.taskInfo(state)).toEqual({
      id: state.id,
      type: state.type,
    });
  });
});
```

## 测试突变

要测试突变，唯一需要测试的是状态是否已经改变。为此，我们:

*   创建状态对象
*   将状态作为变异的参数传递
*   检查状态是否是你所期望的

```
import mutations from '@/js/store/modules/foo/mutations';beforeEach(() => {
  state = {
    foo: 'bar'
  };
});it('should change foo state to john', () => {
  mutations.CHANGE_TO_JOHN(state);
  expect(state).toEqual({
    foo: 'john',
  });
});
```

暂时就这样吧！我可能会用更多的案例来更新这篇文章。这将是我未来参考的袖珍指南！