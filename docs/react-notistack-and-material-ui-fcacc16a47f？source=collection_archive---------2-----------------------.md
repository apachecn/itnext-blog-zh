# 反应、通知堆栈和材质-用户界面

> 原文：<https://itnext.io/react-notistack-and-material-ui-fcacc16a47f?source=collection_archive---------2----------------------->

[Notistack](https://github.com/iamhosseindhv/notistack) 是处理堆叠通知的一个很好的队列。它有许多优秀的配置功能，在整个应用程序中一直是我们的一个很好的工具。

![](img/5533160b7e470bc77bfb121266d0733f.png)

我与同事分享过几次的一件事是，我们如何设置它，以便可以从任何地方发出通知，而不必使用大量的导入或机械来实现它。

如果您使用 React 和 Material，这就是您所需要的:

App.tsx:

```
...
  return (
    <SnackbarProvider
      anchorOrigin={{
        vertical: 'bottom',
        horizontal: 'left'
      }}
      autoHideDuration={5000}
    >
  <>
    <SnackbarUtilsConfigurator />
    <Suspense fallback={<div id={'app'} />}>
      <Routes />
...
```

从文档来看，这里没有太大的区别。将 Snackbar 提供程序添加到 App.tsx 中。但是，如果我们必须在 notistack 的上下文中到处创建依赖关系，这有什么好处呢？如果我们想用一个不同的库来改变它，并保持相同或相似的 API 来发出消息，我们该如何封装它呢？

重要的是，它在悬念和路线组件之外，因为我们不想在发出路线更改或通知时进行重新渲染。如果你发出一个信息，注意到一个路线变化，或者反复的路线变化，这就是为什么！

注意 SnackbarUtilsConfigurator 组件吗？这就是我们为 Notistack 创建 Snack.tsx 的地方:

```
import { useSnackbar, OptionsObject, WithSnackbarProps } from 'notistack';
import React from 'react';
import { LinearProgressEstimator } from 'custom-components';

export enum VariantType {
  *default* = 'default',
  *error* = 'error',
  *success* = 'success',
  *warning* = 'warning',
  *info* = 'info'
}

interface SnackProps {
  setUseSnackbarRef: (showSnackbar: WithSnackbarProps) => void;
}

const InnerSnackbarUtilsConfigurator: React.FC<SnackProps> = (props) => {
  props.setUseSnackbarRef(useSnackbar());
  return null;
};

let useSnackbarRef: WithSnackbarProps;
const setUseSnackbarRef = (useSnackbarRefProp: WithSnackbarProps) => {
  useSnackbarRef = useSnackbarRefProp;
};

export const SnackbarUtilsConfigurator = (props: {
  children?: React.ReactNode;
}) => {
  return (
    <InnerSnackbarUtilsConfigurator setUseSnackbarRef={setUseSnackbarRef}>
      {props.children}
    </InnerSnackbarUtilsConfigurator>
  );
};

//sets a default length for all Snack messages
const defaultSnackMessageLength = 1000;

const trimMessage = (
  msg: string,
  length: number = defaultSnackMessageLength
) => {
  return msg.substring(0, length);
};

export default {
  success(msg: string, options: OptionsObject = {}) {
    this.toast(trimMessage(msg), { ...options, variant: VariantType.*success* });
  },
  warning(msg: string, options: OptionsObject = {}) {
    this.toast(trimMessage(msg), { ...options, variant: VariantType.*warning* });
  },
  info(msg: string, options: OptionsObject = {}) {
    this.toast(trimMessage(msg), { ...options, variant: VariantType.*info* });
  },
  error(msg: string, options: OptionsObject = {}) {
    this.toast(trimMessage(msg), { ...options, variant: VariantType.*error* });
  },
  toast(msg: string, options: OptionsObject = {}) {
    useSnackbarRef.enqueueSnackbar(msg, options);
  },
  loading(msg: string, options: OptionsObject) {
    this.toast(trimMessage(msg), {
      ...options,
      variant: 'info',
      persist: true,
      content: (id: number, _message: string) => {
        return (
          <div key={id} style={{ width: '100%' }}>
            {/* Should take about 4 min to get to 100% */}
            <LinearProgressEstimator velocity={0.4} />
          </div>
        );
      }
    });
  }
};
```

有了这些，我们就可以导入组件并在应用程序中的任何地方调用它。组件只是简单地导出我们需要使用的方法，而`Snack.loading`组件调用展示了定制内容仍然是可能的。它还可以接受覆盖作为第二个参数，这些选项与我们使用的 API 相同。总的来说，我们的目标是将这些参数用作边缘情况参数。

感谢您的阅读，并祝您好运堆叠这些通知！