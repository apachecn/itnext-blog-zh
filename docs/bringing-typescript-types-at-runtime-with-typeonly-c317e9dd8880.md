# 使用 TypeOnly 在运行时引入 TypeScript 类型

> 原文：<https://itnext.io/bringing-typescript-types-at-runtime-with-typeonly-c317e9dd8880?source=collection_archive---------1----------------------->

![](img/0a4a1252ed2aa803a8038af17ffff00b.png)

感谢[好的免费照片](https://www.goodfreephotos.com/south-africa/cape-town/sunrise-above-the-clouds-on-the-mountain-in-cape-town-south-africa.jpg.php)。

我的团队中有一个人，我们最近发布了一个附带项目的初步版本，可能很有前途。我写这篇文章是为了介绍我们的工作。我从这个问题开始:为什么我们决定创建 TypeOnly？

# 因为用 TypeScript 编写枯燥的代码并不总是可能的…

TypeScript 类型定义在运行时不可用。有时这迫使我们重复自己，如下面的例子:

```
type ColorName = "red" | "green" | "blue"function isColorName(name: string): name is ColorName {
  return ["red", "green", "blue"].includes(name)
}
```

这种代码并不理想。Github 上有一个与此主题相关的[公开讨论](https://github.com/microsoft/TypeScript/issues/3628)，TypeScript 团队还不准备提供解决方案。

# 一种新语言？

TypeOnly 是一种新语言，但不是新语法。TypeOnly 旨在成为并保持 TypeScript 的严格子集:任何使用 TypeOnly 编译的代码也将使用 TypeScript 编译。它是 TypeScript 的“纯类型化”部分:只有`interface`和`type`定义。要了解更多，请阅读 TypeOnly 语言的详细描述。

TypeOnly 解析器是从头开始实现的，不需要 TypeScript 作为依赖项。它可以在 TypeScript 项目之外使用，比如在 JavaScript 项目中，或者使用命令行工具来验证 JSON 数据。

# 如何用 TypeOnly 验证 JSON 数据(命令行版本)

用以下代码创建一个文件 *"drawing.d.ts"* :

```
// drawing.d.tsexport interface Drawing {
  color: ColorName
  dashed?: boolean
  shape: Rectangle | Circle
}export type ColorName = "red" | "green" | "blue"export interface Rectangle {
  kind: "rectangle",
  x: number
  y: number
  width: number
  height: number
}export interface Circle {
  kind: "circle",
  x: number
  y: number
  radius: number
}
```

然后，创建一个 JSON 文件 *"drawing.json"* :

```
{
  "color": "green",
  "shape": {
    "kind": "circle",
    "x": 100,
    "y": 100,
    "radius": "wrong value"
  }
}
```

我们准备验证 JSON 文件:

```
$ npx @typeonly/validator-cli -s drawing.d.ts -t "Drawing" drawing.json
In property 'radius', value '"wrong value"' is not conform to number.
```

在 JSON 文件中检测到一个错误。通过用一个有效的数字替换属性`"radius"`的值来修复它。比如:`"radius": 50`。并再次运行该命令:

```
$ npx @typeonly/validator-cli -s drawing.d.ts -t "Drawing" drawing.json
The JSON file is conform.
```

很好。验证器不再抱怨。

# 仅类型工具套件

在上一节中，包含类型的文件是动态解析的。对于命令行验证程序来说，这是一个非常有用的特性，但是如果您需要验证 Node.js 程序中的几个 JSONs，那么您需要解析一次输入，然后多次重用解析过的内容。或者，甚至更好:您希望在编译时解析类型定义，保存解析结果，然后在运行时不再需要解析器。这是使用 TypeOnly 的默认方式。因此，使用类型化元数据是一个快速和轻量级的过程。

TypeOnly 目前附带 4 个 npm 包:

*   [typeonly](https://www.npmjs.com/package/typeonly) :使用 [ANTLR](https://www.antlr.org/) 实现的解析器，性能很好，而且很小。它获取`.d.ts`文件并生成 RTO 文件(RTO 代表 Raw TypeOnly，文件扩展名为`.rto.json`)。
*   [@typeonly/loader](https://www.npmjs.com/package/@typeonly/loader) :帮助加载 RTO 文件的轻量级 API。
*   [@typeonly/validator](https://www.npmjs.com/package/@typeonly/validator) :验证 JSON 或 JavaScript 数据的轻量级 API。
*   @typeonly/validator-cli :一个动态使用解析器和验证器的 cli。

# 教程，第一部分:解析仅类型代码

在本教程中，我们将了解如何在 JavaScript 或 TypeScript 项目中使用 TypeOnly。在一个新目录中，安装`typeonly`作为一个依赖项:

```
npm init
npm install typeonly --save-dev
```

创建一个子目录`src/`，并将上面例子中给出的`drawing.d.ts`文件复制到其中。然后，编辑文件`package.json`并在`"scripts"`部分添加一个条目:

```
"scripts": {
  "typeonly": "typeonly -o dist-rto/ -s src/"
},
```

现在我们可以通过我们的脚本执行 TypeOnly 解析器:

```
npm run typeonly
```

该命令在新目录`dist-rto/`中创建一个文件`drawing.rto.json`。RTO ( `.rto.json`)文件包含从`.d.ts` 打字文件中提取的有用元数据。在接下来的部分中，我们将看到使用这个生成的 RTO 文件的两种方法。

# 教程，第二部分:在运行时使用类型化元数据

为了在运行时浏览 RTO ( `.rto.json`)文件，我们需要`@typeonly/loader`包:

```
npm install @typeonly/loader
```

创建一个包含以下内容的文件`src/main.js`:

```
// src/main.js
const { loadModules, literals } = require("@typeonly/loader")async function main() {
  const modules = await loadModules({
    modulePaths: ["./drawing"],
    baseDir: `${__dirname}/../dist-rto`
  }) const { ColorName } = modules["./drawing"].namedTypes
  console.log("Color names:", literals(ColorName, "string"))
}main().catch(console.error)
```

如果您在 TypeScript 源文件中编写这段代码，只需用标准的`import`替换`require` 语法。

我们可以执行我们的程序:

```
$ node src/main.js
Color names: [ 'red', 'green', 'blue' ]
```

是的，看起来很简单:颜色名称列表现在在运行时可用。

注意，在运行时，我们的代码不依赖于 TypeOnly 解析器。我们使用`@typeonly/loader`，它是`.rto.json`文件的轻量级包装器。

# 教程，第三部分:如何用 TypeOnly 验证 JSON 数据(API 版本)

包`@typeonly/validator`是使用`@typeonly/loader`构建的。我们需要它:

```
npm install @typeonly/validator
```

创建一个`src/validate-main.js`文件，内容如下:

```
// src/validate-main.js
const { createValidator } = require("@typeonly/validator")const data = {
  "color": "green",
  "shape": {
    "kind": "circle",
    "x": 100,
    "y": 100,
    "radius": 50
  }
}async function main() {
  const validator = await createValidator({
    loadModules: {
      modulePaths: ["./drawing"],
      baseDir: `${__dirname}/../dist-rto`
    }
  })
  const result = validator.validate("./drawing", "Drawing", data)
  console.log(result)
}main().catch(console.error)
```

执行这个新程序:

```
$ node src/validate-main.js
{ valid: true }
```

如前一节所述，我们的代码在运行时不依赖于 TypeOnly 解析器。验证器使用加载器，它只加载`.rto.json`文件。

# 结论

## 这篇文章没有涉及到什么？

这里描述的是仅类型语言[。特别是，它允许导入和导出，这意味着您可以将您的输入拆分到几个源文件中。](https://github.com/tomko-team/typeonly/blob/master/typeonly-language.md)

包`typeonly`、`@typeonly/loader`和`@typeonly/validator`的 API 还没有很好的文档化，但是它们类型化得很好。您将需要 TypeScript IDE 的帮助来查看所有选项和提供的数据结构。

## 下一步是什么？

非常感谢贡献者的帮助。对我们来说，TypeOnly 只是一个附带项目，我们没有太多的时间。但是，我们计划在不久的将来研究几个主题:

1.  泛型还没有完全实现，我们希望在这方面努力。
2.  我们必须改进解析器，以便检测更棘手的情况，我们目前允许通过，但在 TypeScript 中无效。
3.  验证器可以通过`/** doc comments */`中的注释来增强，以改进检查，比如字符串等日期格式。
4.  我正在考虑使用 TypeOnly 和 Express 来替代 Node.js 的 GraphQL。我不喜欢 GraphQL 打字和 TypeScript 打字的糟糕集成，我认为可以做一些更令人愉快的事情。