---
title: 理解webpack
subtitle: webpack常用配置和HMR热更新原理
link: realizeWebpack
catalog: true
date: 2023-02-18 17:52:00
tags:
  - 前端
  - 前端工程化
  - webpack
categories:
  - [笔记, 前端, 前端工程化]
---

# webpack 常用配置

```javascript
const path = require('path');
const CleanWebpackPlugin = require('clean-webpack-plugin');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const CopyWebpackPlugin = require('copy-webpack-plugin');
const HappyPack = require('happypack');
const BundleAnalyzerPlugin =
  require('webpack-bundle-analyzer').BundleAnalyzerPlugin;
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const { VueLoaderPlugin } = require('vue-loader/dist/index');
const { DefinePlugin } = require('webpack');

module.exports = {
  // 表示当前是什么环境
  mode: 'development',
  // 用于设置入口文件路径的相对值，也就是说入口文件的路径不是相对于自身的，二是相对于context的
  // 默认值是webpack的启动路径
  context: path.resolve(__dirname, './'),
  // 打包的时候生成一个source-map文件方便调试
  devtool: 'source-map',
  // 入口文件
  entry: './src/main.js',
  // 出口
  output: {
    // 打包后的文件名
    filename: 'bundle.js',
    // 打包后的文件的存储路径
    path: path.resolve(__dirname, '/dist'),
    // 该配置和CleanWbpeckPlugin效果一样，配置了就不需要再配置插件了
    // clear: true
  },
  module: {
    rules: [
      {
        test: /\.m?js$/,
        // 因为loder在处理文件的时候是单线程的，所以交给happypack来开启多线程处理，详细请看插件部分的happypack
        loader: 'happypack/loader?id=happybabel',
        // 只处理src文件夹下面的js文件
        include: [path.resolve('src')],
        // 排除exclude文件
        exclude: /node_modules/,
      },
      {
        test: /\.vue$/,
        use: [{ loader: 'vue-loader' }],
      },
      {
        test: /\.css$/,
        use: [{ loader: 'style-loader' }, { loader: 'css-loader' }],
      },
      {
        test: /\.less$/,
        use: [
          // 将css样式插入到DOM中
          { loader: 'style-loader' },
          // 解析css
          { loader: 'css-loader' },
          // 解析less，将less转换为css
          { loader: 'less-loader' },
        ],
      },
      {
        test: /\.(png|jpe?g|gif|svg)$/i,
        // webpack5以前的方法

        // use: [
        //   {
        //     loader: 'file-loader',
        //     options: {
        //       name: 'img/[name].[hash:8].[ext]',
        //     },
        //   },
        // ],

        // webpack5的方法
        type: 'asset/resource',
        genetator: {
          filename: 'img/[name].[hash:8].[ext]',
        },
      },
      {
        test: /\.(png|jpe?g|gif|svg)$/i,
        // webpack5以前的方法

        // use: [
        //   {
        //     // 对小的图片直接转换为base64形式的url，和页面一起请求，提高图片加载的速度
        //     loader: 'url-loader',
        //     options: {
        //       limit: 100 * 1024,
        //       name: '[name].[hash:8].[ext]',
        //       outputPath: 'img',
        //     },
        //   },
        // ],

        // webpack5的方法

        type: 'asset',
        genetator: {
          filename: 'img/[name].[hash:8].[ext]',
        },
        parser: {
          dataUrlConditon: {
            maxSize: 100 * 1024,
          },
        },
      },
      {
        test: /\.(woff2?|eot|ttf)$/,
        // 利用webpack5的资源模块来处理字体图标
        type: 'asset/resource',
        genetator: {
          filename: 'font/[name].[hash:8].[ext]',
        },
      },
    ],
  },
  plugins: [
    // 用来处理.vue文件
    new VueLoaderPlugin(),
    // 每次重新打包的时候都需要手动删除dist文件夹，配置这个插件，就可以自动删除dist
    new CleanWebpackPlugin(),
    // webpack打包的时候默认没有index.html文件，但是我们部署到静态服务器的时候是需要这个文件的，用这个插件就可以生成一个index.html
    new HtmlWebpackPlugin({
      title: '这里设置index.html的标题',
      // 用来配置生成index.html文件的模板
      // template: './public/index.html'
    }),
    // 用来定义环境变量
    new DefinePlugin({
      //   BASE_URL: './',
    }),
    // 可以将public文件夹里面的一些内容复制到打包的dist文件夹里面
    new CopyWebpackPlugin({
      // 进行相关配置
      patterns: {
        // 从哪里开始复制
        from: 'public',
        // 复制到哪里，默认为打包的目录下
        to: 'dist',
        // 一些额外的选项
        globOptions: {
          // 忽略哪些文件
          ignore: ['**/.DS_Store', '**/index.html'],
        },
      },
    }),
    // 开启多线程，增快webpack的速度
    new HappyPack({
      id: 'happybabel',
      loaders: [
        {
          // 使用babel将es6的代码转换成es5的代码,并将cacheDirectory设置为true，将babel编译过的文件缓存起来
          loader: 'babel-loader?cacheDirectory=true',
          options: {
            //   plugins: [
            //     // 转化const
            //     '@babel/plugins-transform-block-scoping',
            //     // 转化箭头函数
            //     '@babel/plugins-transform-arrow-functions',
            //   ],

            // 一个一个的加载插件太麻烦了，可以使用babel的预设来自动根据预设加载插件
            // 如果要使用ts，也可以用@babel/preset-typescript预设
            presets: ['@babel/preset-env'],
          },
        },
      ],
      // 开启4个线程
      threads: 4,
    }),
    // 可视化webpack输出文件的体积
    new BundleAnalyzerPlugin(),
    // 将每个js文件包含的css单独打包成一个文件，支持按需映入
    new MiniCssExtractPlugin(),
  ],
  resolve: {
    // 当模块导入的时候要是没有后缀名，会自动加上下面配置的后缀名
    extensions: ['.js', '.json', '.vue', '.jsx', '.tsx', '.ts'],
    // 配置路径别名，注意如果有ts的话，需要在tsconfig.ts中也配置路径别名
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
  devServer: {
    // 开启热更新，默认就是开启的，但是需要手动指定对哪些模块进行热更新
    hot: true,
    // 项目运行的端口号
    port: 8888,
    // 项目运行的主机
    host: '127.0.0.1',
    // 打包的时候是否自动打开浏览器
    open: true,
    // 是否对文件进行压缩
    compress: true,
    // 配置代理，解决跨域的问题
    proxy: {
      '/api': 'http://localhost:3000',
      pathRewrite: { '^/api': '' },
    },
  },
  optimization: {
    // 开启scope Hoisting 分析模块之间的依赖关系，经可能的将模块打包到一个函数里面
    concatenateModules: true,
  },
};
```

# 不同环境怎么切换不同的 webpack

- 通常会在 config 文件夹里面写`webpack.prod.config.js`, `webpack.dev.config.js`, `webpack.common.js`这样的三份文件
- 在**生产环境** and **开发环境**对应的配置文件中，利用`webpack-merge`这个包提供的`merge`函数，来对**通用**的配置文件进行合并，下面是一个列子

```javascript
const { merge } = require('webpack-merge');
const commonConfig = require('./webpack.comman.config');

module.exports = merge(commonConfig, {
  mode: 'development',
  devServer: {
    // 开启热更新，默认就是开启的，但是需要手动指定对哪些模块进行热更新
    hot: true,
    // 项目运行的端口号
    port: 8888,
    // 项目运行的主机
    host: '127.0.0.1',
    // 打包的时候是否自动打开浏览器
    open: true,
    // 是否对文件进行压缩
    compress: true,
  },
});
```

- 再通过配置`package.json`里面的`script`属性，来执行不同的 webpack 配置文件

```json
"scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "build": "webpack --config ./config/webpack.prod.config.js",
    "serve": "webpack serve --config ./config/webpack.dev.config.js --progress"
  },
```

# 怎么利用 webpack 来优化前端性能

- 压缩代码体积
  利用 webpack 的`UglifyJsPlugin`和`ParallelUglifyPlugin`来压缩 js 代码（在 webpack 里面`mode`为`production`的时候会自动开启），通过`mini-css-extract`来 压缩 css，将每个 js 文件包含的 css 单独打包成一个 css 文件，支持按需引入。
- 利用 CDN 加速
  将引用的一些资源的路径修改成对应的 CDN 上面的路径
- 利用 tree Shaking
  将永远不会执行的代码片段删除
- 利用 Scope Hoisting 将可能的将打包出来的模块合并到一个函数里面

# 怎么提高 webpack 的打包速度

- 优化 Loader
  - 利用`include`, `exclude`来优化 loader 的搜索范围，比如只对`src`文件下的 js 代码进行编译，忽略`node_modules`文件。
  - 利用缓存，将 loader 处理过的文件缓存起来。
- 利用 HappyPack
  受限于 node，webpack 在打包过程中也是单线程的，但是利用 HappyPack 就可以将 loader 的同步执行变成多线程的。
- 利用 DllPlugin
  Dllolugin 可以提前将指定的类库打包然后引入，这样可以减少打包的次数，只有类库版本更新的时候才需要打包。
- 代码压缩
  压缩的一些操作同上

# 怎么减小 webpack 打包的体积

- tree shaking
- Scope Hoisting
- 代码压缩

# webpack 构建流程

- 初始化参数
  从配置文件和 shell 语句中读取参数，合并参数
- 开始编译
  利用上一步得到的参数初始化`compiler`对象，并加载所有需要的插件，调用`compiler`对象的`run`方法，开始编译
- 确定入口
  从`entry`中找到所有的入口文件
- 编译模块
  利用`loader`开始编译文件，在从文件中找到相关的依赖模块，然后递归此步骤，直到所有的文件都被编译
- 完成模块编译
  此时所有的文件都已经被编译，并且也已经得到了文件之间的依赖关系
- 整理输出的资源
  将所有的编译完成的文件，按照他们之间的依赖关系，组成多个 chunk， 再把 chunk 合并为一个单独的文件，作为输出的文件内容
- 输出文件
  按照配置文件里面确定的路径和文件名写入文件内容

# webpack HMR 热更新原理

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

![HMR](https://i.328888.xyz/2023/02/18/BuGsQ.png)
