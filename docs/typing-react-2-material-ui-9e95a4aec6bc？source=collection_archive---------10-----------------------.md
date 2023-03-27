# 键入 React(2)-Material-UI

> 原文：<https://itnext.io/typing-react-2-material-ui-9e95a4aec6bc?source=collection_archive---------10----------------------->

![](img/dc23fa3be495db7d192bdca3fd5e4627.png)

本系列主要讨论如何在 React 应用程序中正确使用 TypeScript。上一篇文章解释了键入 React 本身的一些细节，因此本文将重点讨论 Material-UI，这是最流行的 UI 库之一。

以前的文章:

*   [打字反应(1) —基础](/typing-react-1-basic-488f661149f6)

添加素材 UI 就像`npm install --save @material-ui/core`一样简单。一般来说，Material UI 的类型很好，所以在 TypeScript 中使用它应该没有问题。唯一的挑战是 CSS-in-JS。如果你正在使用 Material UI 3.x，可能你必须使用更高阶的组件方式来将样式集成到你的组件中:

```
// NOTE: This is for Material UI 3.x
import React from 'react';
import { Theme, createStyles, WithStyles, withStyles } from '@material-ui/core';const styles = (theme: Theme) =>
  createStyles({
    root: {},
  });export interface IProps extends WithStyles<typeof styles> {}class Todo extends React.Component<IProps> {
  render() {
    const { classes } = props;
    return <div className={classes.root} />;
  }
};export default withStyles(styles)(Todo);
```

这对于类风格的组件来说是完美的，但是对于功能组件来说可能有点奇怪。幸运的是 [Material UI 团队刚刚发布了 v4.0.0](https://medium.com/material-ui/material-ui-v4-is-out-4b7587d1e701) ，它提供了一种使用 React 钩子注入样式的新方法。让我们看看它是如何工作的:

```
// makeStyles only supported in Material UI 4+
import React from 'react';
import { Theme, makeStyles } from '@material-ui/core';const useStyles = makeStyles((theme: Theme) => ({
  root: {
    border: `1px solid ${theme.palette.divider}`,
    shadow: theme.shadows[1],
  }
}));const Todo: React.FC = () => {
  const classes = useStyles();
  return <div className={classes.root} />;
};export default Todo;
```

这肯定比特别的方法更容易和干净，而且最重要的是，默认的导出仍然是一个类型，这样就可以用样式定义通用组件。

*原载于 2019 年 5 月 27 日*[*https://charlee . Li*](https://charlee.li/typing-react-2-material-ui/)*。*