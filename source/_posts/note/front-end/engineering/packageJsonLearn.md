---
title: package.json文件详解
link: packageJsonLearn
catalog: true
date: 2023-10-28 14:56:50
tags:
  - 前端
  - 前端工程化
  - package.json文件
categories:
  - [笔记, 前端, 前端工程化]
---

# package.json 是什么

`package.json`文件会描述我们项目的所有配置信息（名称，版本，使用协议），所有 npm 包的信息（版本，是否是开发环境依赖）

# 怎么创建 package.json 文件

```bash
# npm
npm init

# pnpm
pnpm init
```

# 属性介绍

## name

包的名字， 不能以`.`, `_`, `大写字母`开头

- npm 域级包

  - 介绍及其作用
    在 npm 的包管理系统中，有一种 `scoped packages` 机制，用于将一些 npm 包以`@scope/package` 的命名形式集中在一个命名空间下面，实现域级的包管理。域级包不仅不用担心会和别人的包名重复，同时也能对功能类似的包进行统一的划分和管理；比如我们用 vue 脚手架搭建的项目，里面就有`@vue/cli-plugin-babel`、`@vue/cli-plugin-eslint` 等等域级包。相同域级范围内的包会被安装在相同的文件路径下，比如`node_modules/@username/`，可以包含任意数量的作用域包；安装域级包也需要指明其作用域范围：

    ```bash
      npm install @username/package
    ```

  - 我们在初始化项目时可以使用命令行来添加 `scope`：

    ```bash
      npm init --scope=username
    ```

## version

包的版本号，npm 包的版本号也是有规范要求的，通用的就是遵循 `semver` 语义化版本规范，版本格式为：`major.minor.patch`，每个字母代表的含义如下：

- 主版本号(major)：当你做了不兼容的 API 修改
- 次版本号(minor)：当你做了向下兼容的功能性新增
- 修订号(patch)：当你做了向下兼容的问题修正
- 先行版本号：先行版本号是加到修订号的后面，作为版本号的延伸；当要发行大版本或核心功能时，但不能保证这个版本完全正常，就要先发一个先行版本。

  - 格式
    先行版本号的格式是在修订版本号后面加上一个连接号（-），再加上一连串以点（.）分割的标识符，标识符可以由英文、数字和连接号（[0-9A-Za-z-]）组成。例如：

    ```
    1.0​​.0-alpha
    1.0.0-alpha.1
    1.0.0-0.3.7
    ```

  - 常见先行版本号

    - alpha：不稳定版本，一般而言，该版本的 Bug 较多，需要继续修改，是测试版本
    - beta：基本稳定，相对于 Alpha 版已经有了很大的进步，消除了严重错误
    - rc：和正式版基本相同，基本上不存在导致错误的 Bug
    - release：最终版本

    :::info
    当主版本号升级后，次版本号和修订号需要重置为 0，次版本号进行升级后，修订版本需要重置为 0。
    :::

## description, keywords

- description
  `String`类型，描述项目的信息， 可以显示在`npm search`命令的返回结果中
- keywords
  Array`类型， 描述项目的信息， 可以显示在`npm search`命令的返回结果中

## homepage

用来指定项目的主页 or 部署网站的根目录。

- 开发环境的作用

  - 避免路径问题

    一些前端框架和构建工具在路由和资源加载时依赖于 `homepage` 属性。如果你不在开发环境中设置 `homepage`，可能会在构建应用时遇到问题，尤其是当你使用前端路由（如 `React Router`）时。

  - 测试相对路径:

    开发环境中的开发服务器通常使用相对路径来加载资源，而不需要指定完整的 URL。设置 `homepage` 属性可以帮助你测试应用在不同路径上的行为，以确保它在生产环境中正常工作。

- 生产环境的作用
  指定应用程序的根 URL，确保所有资源（例如 CSS、JavaScript 文件等）的加载路径正确。这是非常关键的，因为在生产环境中，你的应用可能托管在不同的域名、子目录或路径上，而 `homepage` 可以确保资源正确加载。

## author, contributors, maintainers

- author
  作者，`string | {name: string, url?: sting, email?: string}`类型
- contributors
  贡献者列表， `Array<string | {name: string, url?: sting, email?: string}>`类型
- maintainers
  维护者列表, `Array<string | {name: string, url?: sting, email?: string}>`类型

## bugs

提供地址来让用户反馈存在的问题, `{url: string, email: string}`类型

## license, license

- license
  开源协议名称，`string`类型
- licenses
  多个包的开源协议名称, `Array<{type: string, url: string}>`类型
  ![license.png](https://i0.imgs.ovh/2023/10/28/FDPvK.png)

## main, module, browser

- main
  指定加载时的入口文件，`cmd`模块规范导入的时候就会加载这个文件，默认为根目录下的`index.js`文件， `browser`和`node`环境下均可使用, `string`类型
- module
  指定`esm`模块规范时的入口文件, `browser`和`node`环境下均可使用，`string`类型
- browser
  指定`browser`环境下的入口文件

## dependencies, devDependencies

- dependencies
  项目`运行`所需的依赖，`开发`和`生产`环境都需要的依赖，命令如下:
  ```bash
  # 不缩写
  npm install xxx --save
  # 缩写
  npm install xxx -S
  ```
- devDependencies
  项目`开发`所需的依赖，只会在`开发`环境被安装, 命令如下：

  ```bash
  # 不缩写
  npm install xxx --save-dev
  # 缩写
  npm install xxx -D
  ```

  - 版本号规则：

    - 没有任何符号：完全百分百匹配，必须使用当前版本号
    - 对比符号类的：>(大于) >=(大于等于) <(小于) <=(小于等于)
    - 波浪符号~：固定主版本号和次版本号，修订号可以随意更改，例如~2.0.0，可以使 用 2.0.0、2.0.2 、2.0.9 的版本。
    - 插入符号^：固定主版本号，次版本号和修订号可以随意更改，例如^2.0.0，可以使 用 2.0.1、2.2.2 、2.9.9 的版本。
    - 任意版本\*：对版本没有限制，一般不用
    - 或符号：||可以用来设置多个版本号限制规则，例如 >= 3.0.0 || <= 1.0.0

## peerDependencies, bundledDependencies, optionalDependencies

- peerDependencies
  用来告诉别人当使用这个依赖的时候，需要使用那些特定版本的依赖，比如`依赖 A` 的 `package.json` 中声明了 `peerDependencies`, `xxx6.0` 吗，那么如果安装了`依赖 A` 就也应该安装 `xxx6.0`

  :::info
  但是请注意，`peerDependencies`下的依赖并不会被强行安装，它只是告诉你，应该安装 对应版本的这些依赖，因为该依赖依赖于这些依赖，如果不安装，可能会出现问题
  :::

- bundledDependencies

  依赖默认时不会被打包的，但是`bundledDependencies`下的依赖也会被一起打包

- optionalDependencies

  指定一些可选的依赖，这些依赖也会被`npm install`安装，但是安装失败了，不会报错，不会导致整个`npm install`失败

## files

用来指定 npm 发包时应该包括哪些目录和文件，`string | Array<string>`类型

## bin

用来指定命令和对应的可执行文件，例如：

```json
{
  "name": "my-cli-tool",
  "version": "1.0.0",
  "bin": {
    "my-command": "bin/my-command.js"
  }
}
```

`bin` 中的键（例如 `"my-command"`）是用户在命令行中执行的命令名称。`bin` 中的值是指向模块中实际可执行文件的相对路径。当全局安装`my-cli-tool`的时候，系统会创建一个符号链接，将 `bin` 中指定的命令名称与相应的可执行文件关联, 然后这个符号连接会被添加到系统的`PATH`变量中。当我们执行`my-command`命令的时候，系统就会通过`PATH`的符号连接找到对应的`bin/my-command`文件并执行其中的代码。如果我们的包以`依赖`的方式被安装时，如果有`bin`，就会在`node_modules/.bin/`生成对应的文件，然后建立符号链接，所有`node_modules/.bin/`目录下的命令都可以使用`npm run [命令]`的格式下运行。

## directories

展示项目的目录结构信息，用来告诉用户每个目录在什么位置， 比如：

```json
{
  "name": "my-module",
  "version": "1.0.0",
  "directories": {
    "bin": "bin",
    "lib": "src",
    "doc": "docs"
  }
}
```

用来告诉用户 or 其他开发者 可执行文件请放到 `bin` 目录，库的核心代码请放到 `lib` 目录，文档代码请放到 `doc` 目录。

## repository

用来告诉想要加入我们，对我们的代码做贡献的人，我们的代码仓库在哪里，`{type: string, url: string, directory: string}`类型。比如：

```json
"repository": {
  "type": "git",
  "url": "https://github.com/DomeSy",
  "directory": "描述话语"
}

```

## script

指定对应命令的脚本（缩写）, 比如：

```json
"scripts": {
   "start": "cross-env UMI_ENV=dev umi dev",
}

```

当执行`npm run script`的时候就相当于执行了`cross-env UMI_ENV=dev umi dev`

## config

用来添加命令行的环境变量， 比如：

```json
{
  "name": "domesy",
  "config": {
    "port": "8088"
  },
  "script": {
    "start": "node server.js"
  }
}
```

当我们在 node 环境中打印`process.npm_package_config_port`的时候就会打印出 8080

## engines

用来指定本库的运行条件， 比如：

```json
{
  "engines": {
    "node": ">=0.10.3 <0.12"
  }
}
```

这里就限制了，如果想要运行本库，node 的版本必须大于等于 0.10.3 小于 0.12

## os

用来指定可以在哪些操作系统上运行，比如：

```json
{
  "os": ["linux", "win64"]
}
```

就说明可以在`linux`和`64位的windows`操作系统上运行

## cpu

用来指定可以在哪些架构下的`cpu`上运行， 比如:

```json
{
  "cpu": ["x64", "ia32"]
}
```

## private

用来指定这是否是一个私人的库，如果是，那么就不能在 npm 上面发布公开， `boolean`类型

## publishConfig

用来指定发包时候的一些配置，比如发到哪个包管理服务器上面，是发布哪个版本，访问级别（哪些用户可以访问）比如：

```json
{
  "publishConfig": {
    "tag": "1.0.0", // 发布1.0.0版本
    "registry": "https://www.npmjs.com/package/domesy-cli", // 发布到这个包管理服务器上面
    "access": "public" // 所有用户都可以访问
  }
}
```

## preferGlobal

用来指定当用户不全局下载该库的时候是否发出警告， `boolean`类型

## browserslist

指定该库支持的浏览器类型， 比如

```json
{
  "browserslist": {
    "production": [">0.2%", "not dead", "not op_mini all"],
    "development": ["last 1 chrome version", "last 1 firefox version", "last 1 safari version"]
  }
}
```
