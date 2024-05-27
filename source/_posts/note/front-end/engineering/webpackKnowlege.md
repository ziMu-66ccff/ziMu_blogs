---
title: webpack八股文整理
link: webpackKnowlege
catalog: true
date: 2024-03-13 14:26:25
tags:
  - 前端
  - 前端工程化
  - webpack
  - 打包工具
categories:
  - [笔记, 前端, 前端工程化]
---

# 1. 常见的 loader

1. **资源加载处理**

- `file-loader`: 通常用来处理图片和字体，用来将文件输出到一个文件夹中，代码中通过相对 url 来引用输出的文件
- `url-loader`：和 file-loader 类似，不同的是可以设定一个阈值，小于这个阈值的返回它的 base-64 形式编码，大的交给 file-loader 处理
- `image-loader`：用来加载和压缩图片
- `svg-inline-loader`：用来将压缩后的 svg 内容注入到代码中
- `json-loader`：用来加载 json

2. **代码加载处理转换**

- `ts-loader`: 用来将 ts 转换成 js
- `awesome-typescript-loader`：和 ts-loader 一样，但是性能比 ts-loader 好
- `babel-loader`：用来将 es6 的代码转换成 es5 的代码
- `sass-loader`：用来将 sass 代码转换成 css 代码
- `less-loader`：用来将 less 代码转换成 css 代码
- `postcss-loader`：用来加载扩展的 css 代码，支持使用下一代 css，可以配合 autoprefixer 来自动补齐 css3 前缀
- `css-loader`：用来加载 css
- `style-loader`：用来将 css 注入到 js 中，通过 dom 的形式去操作 css

3. **代码检查**

- `tslint-loader`：用来检查 ts 代码的质量
- `eslint-loader`：用来检查 js 代码的质量

4. **测试覆盖率**

- `caverjs-loader`：用来加载测试的覆盖率

5. **支持第三方框架 vue**

- `vue-loader`：用来加载 vue 的单文件组件

6. **国际化**

- `i18-loader`：用于国际化

7. **缓存**

- `cache-loader`：可以在一些性能开销较大的 Loader 之前添加，目的是将结果缓存到磁盘里

8. **调试**

- `source-map-loader`: 生成对应的 map 文件，方便对应的断点调试

# 2. 有哪些常见的 Plugin？你用过哪些 Plugin？

- `define-plugin`：定义环境变量 (Webpack4 之后指定 mode 会自动配置)
- `ignore-plugin`：忽略部分文件
- `html-webpack-plugin`：简化 HTML 文件创建 (依赖于 html-loader)
- `web-webpack-plugin`：可方便地为单页应用输出 HTML，比 html-webpack-plugin 好用
- `uglifyjs-webpack-plugin`：不支持 ES6 压缩 (Webpack4 以前)
- `terser-webpack-plugin`: 支持压缩 ES6 (Webpack4)
- `webpack-parallel-uglify-plugin`: 多进程执行代码压缩，提升构建速度
- `mini-css-extract-plugin`: 分离样式文件，CSS 提取为独立文件，支持按需加载 (替代 extract-text-webpack-plugin)
- `serviceworker-webpack-plugin`：为网页应用增加离线缓存功能
- `clean-webpack-plugin`: 目录清理
- `speed-measure-webpack-plugin`: 可以看到每个 Loader 和 Plugin 执行耗时 (整个打包耗时、每个 Plugin 和 Loader 耗时)
- `webpack-bundle-analyzer`: 可视化 Webpack 输出文件的体积

# 3. Loader 和 Plugin 的区别？

`loader`: 因为 webpack 本身只是针对 js 模块的打包器，所以本身只认识和加载 js 模块，而 loader 就是用来加载和转移其他模块的。本质上是一个函数，对内容进行转换，返回转换后的结果
`plugin`: plugin 是插件，用来扩展 webpack 的功能，会监听 webpack 生命周期中广播出的事件，从而做一切其他事情，扩展 webpack 的功能

# 4. webpack 的构建流程

1. `初始化`：通过从配置文件和 shell 脚本中读取参数，得到最终的参数，加载插件，并实列化 compiler，调用 run 方法进行编译
2. `编译`：从 entry 入口文件开始，利用配置的 loader 递归的加载并编译模块，并得到依赖关系
3. `输出`：根据入口模块的依赖关系，组装一个个 chunk，最后打包成一个单独的文件，按照配置文件输出。

# 5. 如何优化 webpack 的构建速度

- `多进程构建`： 使用 thread-loader 来实现多进程构建
- `代码压缩`
  - 多进程并行压缩 js 代码
    - webpack-paralle-uglifyjs-plugin
    - uglifyjs-webpack-plugin 开启 paraller 参数
    - terser-webpack-plugin 开启 parallel 参数
  - 通过 mini-css-extra-plugin 将 css 代码提取到单独的文件，利用 css-loader 的 minimize 选项开启 cssnano 压缩 css 代码
- `图片压缩`
  - 使用 imagemin 库来压缩图片
  - 配置 image-loader 来压缩图片
- `缩小打包域`
  - 通过 include，exclude 来缩小 loader 的范围
  - 通过 ignorePlugin 来忽略指定模块
- `利用cdn来引入一些小的基础包`
  - 利用 html-webpack-plugin 生成一个 html 文件，在这个 html 文件里将一些小的基础包通过 cdn 引入，不打包
- `提取公共第三⽅库`
  - SplitChunksPlugin 插件来进⾏公共模块抽取,利⽤浏览器缓存可以⻓期缓存这些⽆需频繁变动的公共代码
- `利用缓存提升二次构建速度`
  - babel-loader 开启缓存
  - terser-webpack-plugin 开启缓存
  - 使用 cache-loader
- `Tree shaking`
  - 可以通过在启动 webpack 时追加参数 --optimize-minimize 来实现
- `Scope hoisting`

# 6. webpack 文件监听原理

在发现源码发生变化时，自动重新构建出新的输出文件。
Webpack 开启监听模式，有两种方式：

- 启动 webpack 命令时，带上 --watch 参数
- 在配置 webpack.config.js 中设置 watch:true

缺点：每次需要手动刷新浏览器

原理：轮询判断文件的最后编辑时间是否变化，如果某个文件发生了变化，并不会立刻告诉监听者，而是先缓存起来，等 aggregateTimeout 后再执行。

```javascript
module.export = {
    // 默认false，也就是不开启
    watch: true
    watchOptions {
        // 不监听的文件或者文件夹
        ignored: /node_modules/
        // 监听到变化后会等aggregateTimeout后再去执行，默认是300ms
        aggregateTimeout: 300
        // 判断文件是否发生变化时通过不停的询问指定文件有没有变化实现的，默认每秒问1000词
        poll: 1000
    }
}
```

# 7. webpack 热更新原理

![HMR](https://i0.imgs.ovh/2024/03/12/cAqtU.webp)

- 监视阶段
  - webpack 的 watch 模式会监视文件的变化，一旦文件有变化，就会对该文件重新编译打包，然后将重新编译打包后的代码存在一个 js 对象里
  - `webpack-dev-server` 的中间件`webpack-dev-middleware`会对代码进行监控，一旦有代码发生改变，就会通知 webpack 将代码打包到内存中
  - 当`devServer.watchContentBase`设置为`true`的时候，server 就会监视这些配置文件夹里面的静态文件的变化，一旦有变化就会让浏览器刷新也就是`live reload`。
- 传递阶段
  - `webpack-dev-server`和`webpack-dev-server/client`客户端之间会维护一个 websockt 长连接，将 webpack 的信息传递给客户端，其中最主要的是传递新模块的 hash 值。
  - 因为客户端`webpack-dev-server/client`不能请求更新的代码，所以它会将信息传递给`webpack/hot/dev-server`.
  - `webpack/hot/dev-server`会将新模块的 hash 值与`devSever`配置里面允许进行热更新模块的模块的 hash 值进行比对，如果一致，则说明需要热更新，就会把 hash 值进一步传递给`HotModuleReplacementRuntime`，不过不一致则会直接刷新浏览器。
  - `HotModuleReplacementRuntime`会通过`jsonpMainTemplateRuntime`来发送 Ajax 请求，来返回一个 json，这个 json 文件包括了所有要进行热更新的模块的 hash 值，然后会再发送一个 jsonp 请求，获取到最新的模块代码。
- 比对阶段
  - `HotModulePlugin`会对新旧模块进行对比，来决定是否进行热更新，在决定热更新后，就会更新模块，以及模块的依赖

# 8. 文件指纹是什么？怎么用？

文件指纹是打包后输出的文件名的后缀。

- `Hash`：和整个项目的构建相关，只要项目文件有修改，整个项目构建的 hash 值就会更改
- `Chunkhash`：和 Webpack 打包的 chunk 有关，不同的 entry 会生出不同的 chunkhash
- `Contenthash`：根据文件内容来定义 hash，文件内容不变，则 contenthash 不变
