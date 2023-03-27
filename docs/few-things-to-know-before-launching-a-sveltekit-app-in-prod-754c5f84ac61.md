# 在 prod 中启动 SvelteKit 应用程序之前需要知道的几件事

> 原文：<https://itnext.io/few-things-to-know-before-launching-a-sveltekit-app-in-prod-754c5f84ac61?source=collection_archive---------3----------------------->

![](img/81b153810f1076f8b5eae9433fa3f75b.png)

照片由[安迪·赫尔曼万](https://unsplash.com/@kolamdigital?utm_source=Papyrs&utm_medium=referral)在 [Unsplash](https://unsplash.com/?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄

上周，新版本的[NNS-dapp](https://internetcomputer.org/nns)(NNS 的 dapp，世界上最大的 daop 之一，管理着[互联网计算机](https://internetcomputer.org/))引入了一个名为“Stake Maturity”的新特性，对其模型进行了轻微的设计更新，并改变了其构建系统。

事实上，虽然前端应用程序曾经在唯一的 [Rollup](https://rollupjs.org/) bundler 的帮助下打包，但它被迁移到了同时使用 [Vite](https://vitejs.dev/) 、 [esbuild](https://esbuild.github.io/) 和 Rollup 的 [SvelteKit](https://kit.svelte.dev/) *上。

以下是我一路走来学到的三件事。我希望它们对您有所帮助，这样您也可以在生产中安全地部署您的应用程序。

**路由方面没有任何变化，但*

# ​1.CSP 破解 Firefox 中的应用

内容安全策略(CSP)是一个附加的安全层，有助于检测和缓解某些类型的攻击，包括跨站点脚本(XSS)和数据注入攻击(来源 [MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP) )。

因为我们关心安全性，所以我们当然实施了这类规则。值得注意的是策略`script-src`将`index.html`页面的脚本标签列入白名单——使用它们的`sha256`脚本散列——而`'script-dynamic'`用于加载在浏览器中运行应用程序所需的所有代码块。

虽然这在以前的 bundler 中行得通，但我们实际上惊讶地发现，SvelteKit(2022 年 10 月)并不真正支持这种政策组合(见第 [#3558](https://github.com/sveltejs/kit/issues/3558) 期)。它可以在 Chrome 和 Safari 中运行，但是在 Firefox 中会中断，我说的“中断”是指 all 应用程序根本不会被渲染。

为了解决这个问题，我们找到了以下解决方法:添加一个构建后脚本，将 SvelteKit 注入 HTML 页面的代码提取到一个单独的 JS 文件中，然后注入我们自己的脚本加载器🤪。

这可以通过以下方式实现:

​1.将一个空的`main.js`添加到`static`文件夹中(这有助于避免在本地开发时出现问题)。

​2.在 html 根页面的`<head />`中添加一个脚本加载器，即在`src/app.html`页面中。

```
<script>
    const loader = document.createElement("script");
    loader.type = "module";
    loader.src = "./main.js";
    document.head.appendChild(loader);
</script>
```

​3.创建一个后期构建脚本—例如`./scripts/build.csp.mjs`。

```
#!/usr/bin/env node

import { readFileSync, writeFileSync } from "fs";
import { join } from "path";

const publicIndexHTML = join(process.cwd(), "public", "index.html");

const buildCsp = () => {
  const indexHTMLWithoutStartScript = extractStartScript();
  writeFileSync(publicIndexHTML, indexHTMLWithoutStartScript);
};

/**
 * Using a CSP with 'strict-dynamic' with SvelteKit breaks in Firefox.
 * Issue: https://github.com/sveltejs/kit/issues/3558
 *
 * As workaround:
 * 1\. we extract the start script that is injected by SvelteKit in index.html into a separate main.js
 * 2\. we remove the script content from index.html but, let the script tag as anchor
 * 3\. we use our custom script loader to load the main.js script
 */
const extractStartScript = () => {
  const indexHtml = readFileSync(publicIndexHTML, "utf-8");

  const svelteKitStartScript =
    /(<script type=\"module\" data-sveltekit-hydrate[\s\S]*?>)([\s\S]*?)(<\/script>)/gm;

  // 1\. extract SvelteKit start script to a separate main.js file
  const [_script, _scriptStartTag, content, _scriptEndTag] =
    svelteKitStartScript.exec(indexHtml);
  const inlineScript = content.replace(/^\s*/gm, "");

  writeFileSync(
    join(process.cwd(), "public", "main.js"),
    inlineScript,
    "utf-8"
  );

  // 2\. replace SvelteKit script tag content with empty
  return indexHtml.replace(svelteKitStartScript, "$1$3");
};

buildCsp();
```

​4.在`package.json`连锁剧本。

```
{
    "scripts": {
        "build:csp": "node scripts/build.csp.mjs",
        "build": "vite build && npm run build:csp"
    }
}
```

# ​2.构建再现性

可复制构建是一个编译软件的过程，它确保生成的二进制代码可以被复制(来源[维基百科](https://en.wikipedia.org/wiki/Reproducible_builds))。我们关心确定性编译，因为我们希望允许验证在编译过程中没有引入漏洞或后门。

这一直都很管用。然而，在迁移之后，我们无法再在多台计算机上为捆绑的 wasm 计算相同的 sha。

经过一些调试，我们找到了问题的两个根本原因。

​1.如果没有特定的[版本](https://kit.svelte.dev/docs/configuration#version)提供给 SvelteKit，它将生成一个时间戳来标识当前的应用程序版本——也就是说，如果没有提供版本，SvelteKit 会在绑定的 JS 代码中注入一个时间戳。每次构建，每次都有一个新的时间戳。

为了解决这个问题，我们在`package.json`中读取版本号，并在`svelte.config.js`中将其提供给套件。通过这种方式，只要我们不改变语义数字，每个构建版本都是静态的。

```
import adapter from "@sveltejs/adapter-static";
import autoprefixer from "autoprefixer";
import { readFileSync } from "fs";
import preprocess from "svelte-preprocess";
import { fileURLToPath } from "url";

const file = fileURLToPath(new URL("package.json", import.meta.url));
const json = readFileSync(file, "utf8");
const { version } = JSON.parse(json);

const config = {
  preprocess: preprocess({
    postcss: {
      plugins: [autoprefixer],
    },
  }),

  kit: {
    adapter: adapter({
      pages: "public",
      assets: "public",
      fallback: "index.html",
      precompress: false,
    }),
    serviceWorker: {
      register: false,
    },
    version: {
      name: version, // <---- here provide version
    },
    trailingSlash: "always",
  },
};

export default config;
```

​2.SvelteKit——或 Vite——添加一个`public/vite-manifest.json`文件，该文件包含应用程序所有生成的不可变资产的列表。很遗憾，此文件当前未被排序。因此，作为一个快速解决方案，我们添加了一个 bash 脚本来完成这项工作。

```
#!/usr/bin/env bash
set -euxo pipefail
cd "$(dirname "$(realpath "$0")")/.."
# shellcheck disable=SC2094 # This reads the entire file into memory and then writes it out, so is correct.
cat <<<"$(jq --sort-keys . public/vite-manifest.json)" >public/vite-manifest.json
```

我们在`package.json`中也链接了 Bash 脚本。

```
{
    "scripts": {
        "build:csp": "node scripts/build.csp.mjs",
        "build": "vite build && npm run build:csp && ./scripts/make-reproducible"
    }
}
```

# 3.多填充缓冲剂

我一直使用 Chovy 的 [SO 解决方案](https://stackoverflow.com/a/72220289/5404186)为 IC 上的前端 dapps 填充缓冲 API，但它不再完全有效。虽然在`vite.config.js`中将`global`重新定义为`globalThis`仍然有效，但是没有为“缓冲区”应用聚合填充。

这就是为什么我们在安装了(`npm i buffer`)浏览器的[缓冲区](https://www.npmjs.com/package/buffer)模块依赖之后，在根`+layout.ts`中添加了一个“手动”polyfill。

```
import { Buffer } from "buffer";
globalThis.Buffer = Buffer;
```

然而，我们发现这在开发或生产版本中可以在本地正常工作，但是在生产中可能会成为一个问题，因为不能保证获取`+layout.js`文件的速度会比使用它的页面更快。

这就是为什么除了上面的附加组件之外，在捆绑的生产 JS 代码中注入 polyfied 缓冲区是值得的。这可以在一个汇总插件(`npm i @rollup/plugin-inject -D`)的帮助下完成。

```
import inject from "@rollup/plugin-inject";
import { sveltekit } from "@sveltejs/kit/vite";
import type { UserConfig } from "vite";

const config: UserConfig = {
  plugins: [sveltekit()],
  build: {
    target: "es2020",
    rollupOptions: {
      // Polyfill Buffer for production build. 
      // The hardware wallet needs Buffer.
      plugins: [
        inject({
          include: ["node_modules/@ledgerhq/**"],
          modules: { Buffer: ["buffer", "Buffer"] },
        }),
      ],
    },
  },
  optimizeDeps: {
    esbuildOptions: {
      // Node.js global to browser globalThis
      define: {
        global: "globalThis",
      },
    },
  },
};

export default config;
```

注意事项:

*   我们需要上述硬件钱包相关功能的 polyfill。这就是为什么当我们使用 Rollup 插件时，我们将它的范围扩大到`ledgerhq`库。
*   这个解决方案还不是最佳的，因为我们应用了两次 polyfill 也就是说，我们实际上在产品构建中加载了太多的 JavaScript 代码。我不太确定我是否同意，但这仍然可以改进。
*   web worker 代码无法使用上述解决方案进行聚合。如果你需要这样做，你可能需要进一步调查。

# 结论

这是所有的乐趣和游戏，直到你发现的问题并不存在，当你在当地发展😁。我很高兴我们解决了所有这些问题，并能够迁移。使用 ViteJS 简化开发人员的体验，并将 dapp 移植到 SvelteKit 打开了新的可能性，特别是我们关于路由的一些想法，但是，我可能会在另一篇博客文章中讲述更多😉。

到无限远处
大卫

更多冒险，请在🖖推特上关注我