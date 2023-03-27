# 用 esbuild 构建一个库(第 2 卷)

> 原文：<https://itnext.io/build-a-library-with-esbuild-vol-2-c0e3caa25150?source=collection_archive---------1----------------------->

![](img/fd91f93a28de3749a973f88adf760c13.png)

约瑟夫·格雷夫在 [Unsplash](https://unsplash.com/?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上的照片

一年前我分享了一个[帖子](https://medium.com/geekculture/build-a-library-with-esbuild-23235712f3c)，解释了如何用 [esbuild](https://esbuild.github.io/) 构建一个库。虽然它仍然是一个有效的解决方案，但自从它发布以来，我对我的工具进行了一些改进。

下面是我希望对你的项目有用的几个附件。

## 源和输出

为库定义一个以上的入口点有时会很有用——也就是说，不只是使用一个唯一的`index.ts`文件作为入口，而是使用多个源来提供逻辑上独立的代码组。esbuild 通过参数[入口点](https://esbuild.github.io/api/#entry-points)支持该选项。

例如，在我的项目中，我经常列出我的`src`文件夹中的所有 TypeScript 文件，并将它们作为单独的条目。

```
import {
  readdirSync,
  statSync
} from "fs";
import { join } from "path";

// Select all typescript files of src directory as entry points
const entryPoints = readdirSync(join(process.cwd(), "src"))
  .filter(
    (file) =>
      file.endsWith(".ts") &&
      statSync(join(process.cwd(), "src", file)).isFile()
  )
  .map((file) => `src/${file}`);
```

由于每个构建之前的输出文件夹可能已经被删除了，所以我也希望在继续之前通过创建它来确保它存在。

```
import {
  existsSync,
  mkdirSync
} from "fs";
import { join } from "path";

// Create dist before build if not exist
const dist = join(process.cwd(), "dist");

if (!existsSync(dist)) {
  mkdirSync(dist);
}

// Select entryPoints and build
```

## 未定义全局

构建 ESM 目标时，您的库可能会使用某些依赖项，从而导致构建错误*“未捕获的引用错误:未定义全局”*。根本原因是依赖关系期望一个`global`对象(如在 NodeJS 中),而浏览器需要`window`。

为了解决这个问题，esbuild 有一个 [define](https://esbuild.github.io/api/#define) 选项，可以用来用常量表达式替换全局标识符。

```
import esbuild from "esbuild";

esbuild
  .build({
    entryPoints,
    outdir: "dist/esm",
    bundle: true,
    sourcemap: true,
    minify: true,
    splitting: true,
    format: "esm",
    define: { global: "window" },
    target: ["esnext"],
  })
  .catch(() => process.exit(1));
```

## esm 和 cjs

为了发布一个既支持 CommonJS (cjs)又支持 ECMAScript 模块(esm)的库，我将包输出到发行目录的两个子文件夹中——例如`dist/cjs`和`dist/esm`。使用 esbuild，这可以通过指定选项 [outdir](https://esbuild.github.io/api/#outdir) 或 [outfile](https://esbuild.github.io/api/#outfile) 到这些相对路径来实现。

```
import esbuild from "esbuild";
import {
  existsSync,
  mkdirSync,
  readdirSync,
  statSync,
  writeFileSync,
} from "fs";
import { join } from "path";

const dist = join(process.cwd(), "dist");

if (!existsSync(dist)) {
  mkdirSync(dist);
}

const entryPoints = readdirSync(join(process.cwd(), "src"))
  .filter(
    (file) =>
      !file.endsWith(".ts") &&
      statSync(join(process.cwd(), "src", file)).isFile()
  )
  .map((file) => `src/${file}`);

// esm output bundles with code splitting
esbuild
  .build({
    entryPoints,
    outdir: "dist/esm",
    bundle: true,
    sourcemap: true,
    minify: true,
    splitting: true,
    format: "esm",
    define: { global: "window" },
    target: ["esnext"],
  })
  .catch(() => process.exit(1));

// cjs output bundle
esbuild
  .build({
    entryPoints: ["src/index.ts"],
    outfile: "dist/cjs/index.cjs.js",
    bundle: true,
    sourcemap: true,
    minify: true,
    platform: "node",
    target: ["node16"],
  })
  .catch(() => process.exit(1));

// an entry file for cjs at the root of the bundle
writeFileSync(join(dist, "index.js"), "export * from './esm/index.js';");

// an entry file for esm at the root of the bundle
writeFileSync(
  join(dist, "index.cjs.js"),
  "module.exports = require('./cjs/index.cjs.js');"
);
```

由于分发两个不同的文件夹会导致库的`dist`路径中不再有条目文件，所以我还喜欢添加两个文件来重新导出代码。在项目中导入库时，这很有用。

此外，`package.json`条目也应该相应地更新。

```
{
  "name": "mylibary",
  "version": "0.0.1",
  "main": "dist/cjs/index.cjs.js",
  "module": "dist/esm/index.js",
  "types": "dist/types/index.d.ts",
}
```

## CSS 和 SASS

你知道 esbuild 也可以捆绑 [CSS](https://esbuild.github.io/content-types/#css) 文件吗？甚至有一个 [SASS 插件](https://github.com/glromeo/esbuild-sass-plugin)使它易于构建。scss 文件😃。

```
npm i -D esbuild-sass-plugin postcss autoprefixer postcss-preset-env
```

在下面的例子中，我捆绑了两个不同的 SASS 文件— `src/index.scss`和`src/doc/index.scss`。我使用插件来转换代码——即给 CSS 添加前缀——我还使用选项[图元文件](https://esbuild.github.io/api/#metafile),它告诉 esbuild 以 JSON 格式生成一些关于构建的元数据。

使用它，我可以检索生成的 CSS 文件的路径和名称，例如，稍后将它们包含在我的 HTML 文件中。

```
import esbuild  from 'esbuild';

import {sassPlugin}  from 'esbuild-sass-plugin';
import postcss  from 'postcss';
import autoprefixer  from 'autoprefixer';
import postcssPresetEnv  from 'postcss-preset-env';

const buildCSS = async () => {
  const {metafile} = await esbuild.build({
    entryPoints: ['src/index.scss', 'src/doc/index.scss'],
    bundle: true,
    minify: true,
    format: 'esm',
    target: ['esnext'],
    outdir: 'dist/build',
    metafile: true,
    plugins: [
      sassPlugin({
        async transform(source, resolveDir) {
          const {css} = 
            await postcss([autoprefixer, 
                           postcssPresetEnv()
                          ]).process(source, {
              from: undefined
          });
          return css;
        }
      })
    ]
  });

  const {outputs} = metafile;
  return Object.keys(outputs);
};
```

## 结论

[esbuild](https://esbuild.github.io/) 还是滑头！

到无限远处
大卫

更多冒险，请在推特上关注我