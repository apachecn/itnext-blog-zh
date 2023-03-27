# react Hooks——设计简单的表单 API——第 1 部分

> 原文：<https://itnext.io/react-hooks-designing-a-simple-forms-api-part-1-307b04bc6007?source=collection_archive---------3----------------------->

如果您致力于创建功能组件而不是类组件，那么您需要深入钩子。您将使用大量的钩子，随着代码规模的增加，您可能需要创建自定义钩子。React 钩子提供了一种抽象可重用逻辑的新方法。开始使用它们很容易，但是我发现随着用例复杂性的增加，抽象您的定制钩子变得更具挑战性。

在这个由多个部分组成的系列中，我们将设计一个简单的 useForm 钩子，并讨论测试策略。我不建议您构建自己的自定义表单 API，因为有许多 React 表单库，既有基于组件的，也有基于 hook 的，您可以开始使用。但是我相信设计和实现表单 API 是加深对 React 钩子理解的一个好方法。

在第 1 部分中，我们将从创建一个简单的 useForm 挂钩开始，并讨论如何在表单中使用它。我们还将讨论测试钩子的方法。我们开始吧！

顺便说一下， [part 2](/react-hooks-designing-a-simple-forms-api-part-2-1fe5d12f23d9) 、 [part 3](/react-hooks-designing-a-simple-forms-api-part-3-validation-and-a-running-example-18b835a3b817) 和 [part 4](/react-hooks-designing-a-simple-forms-api-part-4-scaling-to-other-input-types-e738db0a3fc3) 在线💪。

# 步骤 1 —设计公共 API

我发现设计 hooks APIs 的最佳起点是首先理解你希望它做什么，然后你将如何使用它。所以让我们想象一个叫做`useForm`的钩子。

## 它会做什么？

*   `useForm`将是主要的 API 入口点
*   `useForm`将向消费者提供输入状态，即名称和值。出于表单初始化的目的，输入状态必须能够传递给 useForm 钩子
*   `useForm`将提供表单的整体 UI 状态，如“表单是否提交”、“表单是否提交”、“表单是否验证”
*   `useForm`将尝试为表单提供合理的 HTML 属性
*   `useForm`提供可访问性道具，如果消费者希望支持可访问性标准，如 [WCAG AA](https://medium.com/@shanplourde/strategies-for-building-accessible-websites-f0d256decae6) ，他们可以选择使用这些道具
*   消费者将通过`useForm`钩子定义表单、输入和验证器
*   `useForm`将支持异步验证和提交
*   `useForm`原料药应进行测试
*   API 将完全是非可视化的，这样消费者就可以完全控制他们表单的外观和感觉

出于第 1 部分的目的，我们将创建一个简单的钩子初始版本，它包括定制的 onSubmit 函数支持和测试策略。

## 我们将如何使用它？

我设想用法如下:

```
const sleep = ms => new Promise(resolve => setTimeout(resolve, ms));function SampleForm(props) {
  const { settings } = props;**const { getFormProps, formState, formSummary } = useForm("settingsForm", {
    ...settings
  });**const onSubmit = async ({ evt, formState } => {
    await sleep(2000);
  }; return (
    <div>
      <h2>Sample form</h2>
      {**formSummary.isSubmitting** && <div>Submitting</div>}
      <form **{...getFormProps({ onSubmit })}**>
      /* form stuff here */
      </form>
    </div>
  );}
```

我们的表单 API 将通过`useForm`钩子可用。在 useForm 钩子的第 1 部分中，`formState`将返回当前的表单状态(此时将是一个空对象)。`getFormProps`将被消费者用来设置一个表单的道具，比如`onSubmit`，因为`useForm`钩子将实现一个`onSubmit`函数来包装消费者的`onSubmit`回调，以便向消费者提供一些高级 UI 状态。

自定义的`onSubmit`函数只是一个休眠 2 秒的存根。这对于异步测试表单很有用。

`formSummary`将表示表单的 UI 摘要。在第一个 UI 示例中，我们正在查看`isSubmitting`道具，如果为真，我们将向消费者显示一条提交消息。这种属性可以通过其他方式实现。也许您的应用程序中有一个全局 Redux store 来跟踪端点何时被请求和完成，如果是这样，这可能是跟踪表单提交状态的另一种方法。但是为了保持一个全面的 API，我们将包括它，但是如果用户不使用它，允许他们忽略它。

# 首次实施

让我们来看看`useForm`钩子的初始版本:

```
import { useState } from "react";export const defaultFormProps = {
  autoComplete: "on"
};
export const useForm = (name, initialState = {}, props) => {
  const [formState, setFormState] = useState({
    values: initialState
  });
  const [formSummary, setFormSummary] = useState({
    isSubmitting: false
  });const getFormProps = (props = {}) => ({
    ...defaultFormProps,
    ...props,
    onSubmit: async evt => {
      evt.preventDefault();
      try {
        setFormSummary({ isSubmitting: true });
        props.onSubmit && (await props.onSubmit({ evt, formState }));
      } finally {
        setFormSummary({ isSubmitting: false });
      }
    }
  });return {
    getFormProps,
    formState: formState,
    formSummary
  };
};
```

钩子的初始版本支持 defaultFormProps 的 API，默认情况下[表单自动完成](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/form)属性为 on。`getFormProps`返回调用者可以扩展到表单上的表单属性。`onSubmit`包装调用者`onSubmit`，以便能够跟踪`isSubmitting`。

# 测试

首先，我们将测试 hooks API。当我们开始创建 UI 测试用例时，我们将保存 UI 测试以备后用。为了开始测试钩子，我们将使用 [jest](https://jestjs.io/) 和[react-hooks-testing-library](https://www.npmjs.com/package/react-hooks-testing-library)。建议使用 react-hooks-testing-library 测试钩子。

因为我们将使用 setTimeout 异步测试，所以我们将使用`jest.useFakeTimers`，因为 jest 全局覆盖 setTimeout 以避免阻塞单元测试。

此时，测试的目标是测试包含在`useForm`钩子中的所有特性。这些测试验证以下内容:

*   传入`useForm`的表单状态被正确返回
*   自定义`onSubmit`回调被调用
*   `useForm`表单提交前后正确设置整体界面状态(`isSubmitting` ) —适用于异步提交
*   由于端点故障而导致的承诺拒绝等错误得到了妥善处理

还有其他可以也应该做的测试，比如验证 onSubmit 回调的参数。随着 API 的进一步构建，这些将被添加到将来的部分中

```
import { renderHook, cleanup, act } from "react-hooks-testing-library";
import { useForm } from "./use-form";const noop = () => {};jest.useFakeTimers();describe("useForm tests", () => {
  afterEach(cleanup);it("should return empty form state", () => {
    const { result } = renderHook(() => useForm());
    const { getFormProps, formState } = result.current;
    expect(getFormProps).toBeDefined();
    expect(formState.values).toEqual({});
  });it("should return an initial formSummary", () => {
    const { result } = renderHook(() => useForm());
    const { getFormProps, formSummary } = result.current;
    expect(getFormProps).toBeDefined();
    expect(formSummary).toEqual({
      isSubmitting: false
    });
  });it("should support custom form props", () => {
    const { result } = renderHook(() => useForm());
    const { getFormProps } = result.current;
    const formProps = getFormProps({ foo: "bar" });
    expect(formProps.foo).toEqual("bar");
  });it("should support custom onSubmit", async () => {
    const { result } = renderHook(() => useForm());
    const { getFormProps, formSummary } = result.current;const onSubmit = jest.fn();const formProps = getFormProps({ onSubmit });
    expect(formProps.onSubmit).toBeDefined();// Could be some weirdness right now due to
    // [https://github.com/facebook/react/issues/14769](https://github.com/facebook/react/issues/14769)
    act(() => {
      formProps.onSubmit({ preventDefault: noop });
    });
    expect(formSummary).toEqual({
      isSubmitting: false
    });
    expect(onSubmit).toHaveBeenCalledTimes(1);
  });it("should support async onSubmit", async () => {
    const { waitForNextUpdate, result } = renderHook(() => useForm());
    const { getFormProps, formSummary } = result.current;const onSubmit = evt =>
      new Promise(r => {
        setTimeout(() => {
          r();
        }, 1000);
      });const formProps = getFormProps({ onSubmit });
    expect(formProps.onSubmit).toBeDefined();// Could be some weirdness right now due to
    // [https://github.com/facebook/react/issues/14769](https://github.com/facebook/react/issues/14769)
    act(() => {
      formProps.onSubmit({ preventDefault: noop });
    });
    jest.runAllTimers();
    expect(result.current.formSummary).toEqual({
      isSubmitting: true
    });
    await waitForNextUpdate();
    expect(formSummary).toEqual({
      isSubmitting: false
    });
  });it("should gracefully handle onSubmit errors", async () => {
    const { result } = renderHook(() => useForm());
    const { getFormProps, formSummary } = result.current;const onSubmit = evt => new Error();const formProps = getFormProps({ onSubmit });
    expect(formProps.onSubmit).toBeDefined();// Could be some weirdness right now due to
    // [https://github.com/facebook/react/issues/14769](https://github.com/facebook/react/issues/14769)
    act(() => {
      formProps.onSubmit({ preventDefault: noop });
    });
    expect(formSummary).toEqual({
      isSubmitting: false
    });
  });it("should gracefully handle async onSubmit errors", async () => {
    const { waitForNextUpdate, result } = renderHook(() => useForm());
    const { getFormProps, formSummary } = result.current;const onSubmit = evt =>
      new Promise((resolve, reject) => {
        setTimeout(() => {
          reject();
        }, 1000);
      });const formProps = getFormProps({ onSubmit });
    expect(formProps.onSubmit).toBeDefined();// Could be some weirdness right now due to
    // [https://github.com/facebook/react/issues/14769](https://github.com/facebook/react/issues/14769)
    act(() => {
      formProps.onSubmit({ preventDefault: noop });
    });
    jest.runAllTimers();
    expect(result.current.formSummary).toEqual({
      isSubmitting: true
    });
    await waitForNextUpdate();
    expect(formSummary).toEqual({
      isSubmitting: false
    });
  });
});
```

# 摘要

在第一篇文章中，我们设计了高级的`useForm`钩子，设想了如何使用它，并创建了一些初始测试用例来验证`useForm`钩子。

如果您有任何问题、反馈或建议，请告诉我。

[本系列的第 2 部分可以在这里找到](https://medium.com/@shanplourde/react-hooks-designing-a-simple-forms-api-part-2-1fe5d12f23d9)。