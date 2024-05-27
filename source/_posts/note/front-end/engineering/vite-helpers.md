---
title: vite的两个得力助手-esbuild-rollup
subtitle: vite的两个得力助手-esbuild-rollup
link: vite的两个得力助手-esbuild-rollup
catalog: true
date: 2023-07-26 17:47:06
tags:
  - 前端
  - 前端工程化
  - 深入浅出vite
categories:
  - [笔记, 前端, 前端工程化]
---

# esbuild

### esubuild 为什么快(vite 选择 esbuild 在开发环境打包第三方依赖的原因)

- 使用 Go 开发
  Go 的代码会被直接编译成原生机器码，而不需要像 JS 一样先解析为字节码，然后再转换成字节码，大大的节省了程序运行的时间、
- 多核并行
  因为 Go 中多线程共享内存的优势，内部打包算法充分的利用了多核 CPU 的多核优势，使得所有步骤尽可能的并行
- 从零造轮子
  几乎没有使用第三方库，全部是自己编写的逻辑，保证了代码的极致性能
- 高效的内存利用
  Esbuild 中从头到尾尽可能地复用一份 AST 节点数据，而不用像 JS 打包工具中频繁地解析和传递 AST 数据（如 string -> TS -> JS -> string)，造成内存的大量浪费。

### esbuild 的不足（vite 为什么在生产环境不选择 esbuild 而选择了 rollup 来进行打包）

- 无法兼容低端浏览器
  不支持把语法降级到`es5`，不支持`const enum`等高级语法
- 无法灵活处理打包产物，无法自定义拆包
  没有提供操作打包产物的接口，没有提供子定义拆包策略的 api

### vite 用 esbuild 做了什么

- bunder
  1. 在开发环境，为了顺利的实现`no-bunder`，在**预构建**使用 esuild 来将其他的模块化规范转换成了`esm`（因为`no-bundler`本质上是利用了浏览器原生支持 esm，让浏览器来请求对应模块）
  2. 使用 esbuild 来对第三方依赖进行打包（避免出现请求瀑布流，浏览器发起过多请求，导致首屏时间过长）
- transformer
  进行单文件编译，作为 TS, JSX, TSX 的编译工具，并且在**生产环境**也是用的 esbuild 来进行编译。但是无法进行 ts 的类型检查，所以还是需要利用`tsc`来进行类型检查
- minifier
  对 css, js 文件进行压缩

# rollup

### vite 用 rollup 做了什么

1. 生产环境利用`rollup`进行打包，并对其进行扩展和优化

   - CSS 代码分割
     如果某个异步模块中引入了一些 CSS 代码，Vite 就会自动将这些 CSS 抽取出来生成单独的文件，提高线上产物的缓存复用率。
   - 自动预加载
     Vite 会自动为入口 chunk 的依赖自动生成预加载标签`<link       rel="modulepreload">`
   - 异步 Chunk 加载优化
     在异步引入的 Chunk 中，通常会有一些公用的模块，如现有两个 异 步引入的 Chunk: A 和 B，而且两者有一个公共依赖 C，一般 情况 下，Rollup 打包之后，会先请求 A，然后浏览器在加载 A 的过程 中才决定请求和加载 C，但 Vite 进行优化之后，请求 A 的同时会 自动预加载 C，通过优化 Rollup 产物依赖加载方式节省 了不必要 的网络开销。

2. 兼容了`rollup`的插件机制
   vite 插件可以直接传入 rollup 作为插件使用，但是 rollup 的插件不一定能直接传入 vite 作为 vite 插件使用
