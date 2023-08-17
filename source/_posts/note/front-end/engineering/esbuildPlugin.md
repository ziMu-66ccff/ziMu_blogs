---
title: 怎么写一个esbuild插件
subtitle: esbuild提供的API，及esbuild插件
link: esbuildPlugin
catalog: true
date: 2023-07-27 16:07:33
tags:
  - 前端
  - 前端工程化
  - 深入浅出vite
categories: [笔记, 前端, 前端工程化]
---

# esbuild API

### 项目打包 build API

- 普通 api

  1. build API

     ```typescript
     import * as esbuild from 'esbuild';

     async function runBuild() {
       // 异步方法，返回一个 Promise
       const result = await esbuild.build({
         // ---- 如下是一些常见的配置 ---
         // 当前项目根目录
         absWorkingDir: process.cwd(),
         // 入口文件列表，为一个数组
         entryPoints: ['./src/index.jsx'],
         // 打包产物目录
         outdir: 'dist',
         // 是否需要打包，一般设为 true
         bundle: true,
         // 模块格式，包括`esm`、`commonjs`和`iife`
         format: 'esm',
         // 需要排除打包的依赖列表
         external: [],
         // 是否开启自动拆包
         splitting: true,
         // 是否生成 SourceMap 文件
         sourcemap: true,
         // 是否生成打包的元信息文件
         metafile: true,
         // 是否进行代码压缩
         minify: false,
         // 是否将产物写入磁盘
         write: true,
         // Esbuild 内置了一系列的 loader，包括 base64、binary、  css、dataurl、file、js(x)、ts(x)、text、json
         // 针对一些特殊的文件，调用不同的 loader 进行加载
         loader: {
           '.png': 'base64',
         },
       });
       console.log(result);
     }

     runBuild();
     ```

     `result`为 esbuild 打包的原信息
     [![esbuild打包原信息](https://s1.ax1x.com/2023/07/31/pP938ud.md.png)](https://imgse.com/i/pP938ud)

- 高级 api（依赖于构建上下文）

  1. watch API
     开启`watch mode`，一旦有文件发生改变，直接自动重新打包

     ```typescript
     let ctx = await esbuild.context({
       entryPoints: ['app.ts'],
       bundle: true,
       outdir: 'dist',
     });

     await ctx.watch();
     ```

  2. serve API
     开启`serve mode`，启动本地开发服务器 ，不是将打包后的代码写进磁盘，而是通过请求来获取，当每次请求到来时，都会`rebuild`，返回最新的产物

     ```typescript
     // build.js
     const { build, buildSync, serve } = require('esbuild');

     function runBuild() {
       serve(
         {
           port: 8000,
           // 静态资源目录
           servedir: './dist',
         },
         {
           absWorkingDir: process.cwd(),
           entryPoints: ['./src/index.jsx'],
           bundle: true,
           format: 'esm',
           splitting: true,
           sourcemap: true,
           ignoreAnnotations: true,
           metafile: true,
         }
       ).then((server) => {
         console.log('HTTP Server starts at port', server.port);
       });
     }

     runBuild();
     ```

### 单文件转译 transform API

- transform Api

```typescript
// transform.js
const { transform, transformSync } = require('esbuild');

async function runTransform() {
  // 第一个参数是代码字符串，第二个参数为编译配置
  const content = await transform('const isNull = (str: string): boolean => str.length > 0;', {
    sourcemap: true,
    loader: 'tsx',
  });
  console.log(content);
}

runTransform();
```

# esbuild 插件开发

### 插件基本结构

`esbuild`插件的结构为一个有`name`, `setup`属性的 obj。
`name`属性为插件的名称
`setup`属性为一个函数，有一个参数为`build`对象，这个`build`对象身上有我们需要的一些钩子函数

```typescript
let envPlugin = {
  name: 'env',
  setup(build) {
    build.onResolve({ filter: /^env$/ }, (args) => ({
      path: args.path,
      namespace: 'env-ns',
    }));

    build.onLoad({ filter: /.*/, namespace: 'env-ns' }, () => ({
      contents: JSON.stringify(process.env),
      loader: 'json',
    }));
  },
};

require('esbuild')
  .build({
    entryPoints: ['src/index.jsx'],
    bundle: true,
    outfile: 'out.js',
    // 应用插件
    plugins: [envPlugin],
  })
  .catch(() => process.exit(1));
```

### 插件钩子函数

1. `onStart`
   执行时机是每次 build 的时候，包括`watch mode`和`serve mode`模式下的重新构建
2. `onResolve`
   用来控制路径解析, 具有两个参数`options`, `callback`

   - `options`的类型

     ```typescript
     interface Options {
       filter: RegExp;
       namespace?: string;
     }
     ```

     `filter`为必传参数，为一个正则表达式，用来过滤出需要处理的文件
     `namespace`为选填参数， 为我们过滤出来的需要处理的文件进行一个标记，在`callback`里面返回，从而在`onLoad`钩子函数中找到并处理

   - `callback`常见参数及其返回值

     ```typescript
      build.onResolve({ filter: /^env$/ }, (args: onResolveArgs): onResolveResult => {
        // 模块路径
        console.log(args.path)
        // 父模块路径
        console.log(args.importer)
        // namespace 标识
        console.log(args.namespace)
        // 基准路径
        console.log(args.resolveDir)
        // 导入方式，如 import、require
        console.log(args.kind)
        // 额外绑定的插件数据
        console.log(args.pluginData)

        return {
            // 错误信息
            errors: [],
            // 是否需要 external
            external: false;
            // namespace 标识
            namespace: 'env-ns';
            // 模块路径
            path: args.path,
            // 额外绑定的插件数据
            pluginData: null,
            // 插件名称
            pluginName: 'xxx',
            // 设置为 false，如果模块没有被用到，模块代码将会在产物     中   会删除。否则不会这么做
            sideEffects: false,
            // 添加一些路径后缀，如`?xxx`
            suffix: '?xxx',
            // 警告信息
            warnings: [],
            // 仅仅在 Esbuild 开启 watch 模式下生效
            // 告诉 Esbuild 需要额外监听哪些文件/目录的变化
            watchDirs: [],
            watchFiles: []
        }
      }
     ```

3. `onLoad`
   控制模块内容加载的过程， 具有两个参数`options`, `callback`

   - `options`的类型

     ```typescript
     interface Options {
       filter: RegExp;
       namespace?: string;
     }
     ```

     `filter`为必传参数，为一个正则表达式，用来过滤出需要处理的文件
     `namespace`为选填参数， 通过`namespace`来将需要处理的模块的过滤出来

   - `callback`常见参数及其返回值

     ```typescript
     build.onLoad({ filter: /.*/, namespace: 'env-ns' }, (args: OnLoadArgs): OnLoadResult => {
       // 模块路径
       console.log(args.path);
       // namespace 标识
       console.log(args.namespace);
       // 后缀信息
       console.log(args.suffix);
       // 额外的插件数据
       console.log(args.pluginData);

       return {
         // 模块具体内容
         contents: '省略内容',
         // 错误信息
         errors: [],
         // 指定 loader，如`js`、`ts`、`jsx`、`tsx`、`json`等         等
         loader: 'json',
         // 额外的插件数据
         pluginData: null,
         // 插件名称
         pluginName: 'xxx',
         // 基准路径
         resolveDir: './dir',
         // 警告信息
         warnings: [],
         // 同上
         watchDirs: [],
         watchFiles: [],
       };
     });
     ```

4. `onEnd`
   在打包结束时做一些事情，比如拿到打包的原信息`matafile`来做一些事情，但是想要拿到`metafile`，必须将`metafile`属性设置为`true`

```typescript
let examplePlugin = {
  name: 'example',
  setup(build) {
    build.onStart(() => {
      console.log('build started');
    });
    build.onEnd((buildResult) => {
      if (buildResult.errors.length) {
        return;
      }
      // 构建元信息
      // 获取元信息后做一些自定义的事情，比如生成 HTML
      console.log(buildResult.metafile);
    });
  },
};
```

# esbuild 插件示例

### html-plugin.js

```typescript
const fs = require('fs/promises');
const path = require('path');
const { createScript, createLink, generateHTML } = require('./util');

module.exports = () => {
  return {
    name: 'esbuild:html',
    setup(build) {
      build.onEnd(async (buildResult) => {
        if (buildResult.errors.length) {
          return;
        }
        const { metafile } = buildResult;
        // 1. 拿到 metafile 后获取所有的 js 和 css 产物路径
        const scripts = [];
        const links = [];
        if (metafile) {
          const { outputs } = metafile;
          const assets = Object.keys(outputs);

          assets.forEach((asset) => {
            if (asset.endsWith('.js')) {
              scripts.push(createScript(asset));
            } else if (asset.endsWith('.css')) {
              links.push(createLink(asset));
            }
          });
        }
        // 2. 拼接 HTML 内容
        const templateContent = generateHTML(scripts, links);
        // 3. HTML 写入磁盘
        const templatePath = path.join(process.cwd(), 'index.html');
        await fs.writeFile(templatePath, templateContent);
      });
    },
  };
};

// util.js
// 一些工具函数的实现
const createScript = (src) => `<script type="module" src="${src}"></script>`;
const createLink = (src) => `<link rel="stylesheet" href="${src}"></link>`;
const generateHTML = (scripts, links) => `
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Esbuild App</title>
  ${links.join('\n')}
</head>

<body>
  <div id="root"></div>
  ${scripts.join('\n')}
</body>

</html>
`;

module.exports = { createLink, createScript, generateHTML };
```
