---
title: 前端工作流配置——代码不仅是给机器看的，也是给人看的（持续更新中）
link: workflowConfig
catalog: true
date: 2023-06-19 19:41:24
tags:
  - 前端
  - 前端工程化
  - 工作流配置
categories:
  - [笔记, 前端, 前端工程化]
---

# eslint

### 为什么需要 eslint

因为我们写的代码有时候可能质量并不高，所以我们需要这样的一个工具来`规范我们的代码`，保障我们的`代码质量`（比如我们写了一个变量 但是这个变量我们没有使用 那么这个变量就是多余的 这时候我们的 eslint 可以直接报错 让我们知道这个多余的变量应该删除）

### 怎么在项目中集成 eslint

1. 下载 eslint
   使用你的包管理工具（pnpm，yarn，npm，npx）安装 eslint 具体命令 `pnpm install eslint`
2. 初始化 eslint（）
   具体命令 `pnpm eslint --init`, 执行该命令后会让你`回答相关问题`，从而选择适合你的规则来约束你的代码，目的只有一个提高你的代码质量
   - how would you like to use eslint?（你想使用 eslint 来做什么）
     - to check syntax only（只检查语法）
     - to check syntax and find problems（检查语法并找出错误）
     - to check syntax, find problems, and enforce code style （检查语法，找出错误，规范代码）
   - what type of modules does your project use? （你想使用什么模块化规范）
     - javascript modules (import/export)
     - commonjs (require/exports)
     - none of these
   - which framework does your project use? （你想使用什么框架）
     - React
     - Vue
     - none of these
   - Does your project use Typescript No/Yes （是否要使用 ts）
   - Where does your code run? （代码运行在什么环境，一般两个都选）
     - Browser
     - Node
   - How would you like to define a style for your project? （你想怎么来定义 eslint 的规则， 推荐直接使用流行的规范）
     - use a popular style guide （使用流行的规范）
     - answer questions about your style （通过询问问题 得到你想要的规范吧）
     - inspect your javascript files （根据配置文件 生成规范）
   - Which style guide do you want to to follow? （你想使用哪个流行的规范）
     - Standard
     - XO
   - What format do you want your config file to be in ? （配置文件的格式 建议直接选择 JavaScript）
     - javascript
     - yaml
     - json
3. vscode 安装 eslint 插件

### .eslint.cjs 配置文件的相关配置项解析

1.  parser - 解析器
    ESLint 底层默认使用 Espree 来进行 AST 解析，这个解析器目前已经基于 Acron 来实现，虽然说 Acron 目前能够解析绝大多数的 ECMAScript 规范的语法，但还是不支持 TypeScript ，因此需要引入其他的解析器完成 TS 的解析。

    社区提供了@typescript-eslint/parser 这个解决方案，专门为了 TypeScript 的解析而诞生，将 TS 代码转换为 Espree 能够识别的格式(即 Estree 格式)，然后在 Eslint 下通过 Espree 进行格式检查， 以此兼容了 TypeScript 语法。

2.  parserOptions - 解析器选项
    这个配置可以对上述的解析器进行能力定制，默认情况下 ESLint 支持 ES5 语法，你可以配置这个选项，具体内容如下:

    ecmaVersion: 这个配置和 Acron 的 ecmaVersion 是兼容的，可以配置 ES + 数字(如 ES6)或者 ES + 年份(如 ES2015)，也可以直接配置为 latest，启用最新的 ES 语法。
    sourceType: 默认为 script，如果使用 ES Module 则应设置为 module
    ecmaFeatures: 为一个对象，表示想使用的额外语言特性，如开启 jsx。

3.  rules - 具体代码规则
    rules 配置即代表在 ESLint 中手动调整哪些代码规则，比如禁止在 if 语句中使用赋值语句这条规则可以像如下的方式配置:

    ```javascript
    // .eslintrc.js
    module.exports = {
      // 其它配置省略
      rules: {
        // key 为规则名，value 配置内容
        'no-cond-assign': ['error', 'always'],
      },
    };
    ```

    在 rules 对象中，key 一般为规则名，value 为具体的配置内容，在上述的例子中我们设置为一个数组，数组第一项为规则的 ID，第二项为规则的配置。

    这里重点说一说规则的 ID，它的语法对所有规则都适用，你可以设置以下的值:

    off 或 0: 表示关闭规则。

    warn 或 1: 表示开启规则，不过违背规则后只抛出 warning，而不会导致程序退出。
    error 或 2: 表示开启规则，不过违背规则后抛出 error，程序会退出。
    具体的规则配置可能会不一样，有的是一个字符串，有的可以配置一个对象，你可以参考 ESLint 官方文档。

    当然，你也能直接将 rules 对象的 value 配置成 ID，如: "no-cond-assign": "error"。

4.  plugins
    上面提到过 ESLint 的 parser 基于 Acorn 实现，不能直接解析 TypeScript，需要我们指定 parser 选项为@typescript-eslint/parser 才能兼容 TS 的解析。同理，ESLint 本身也没有内置 TypeScript 的代码规则，这个时候 ESLint 的插件系统就派上用场了。我们需要通过添加 ESLint 插件来增加一些特定的规则，比如添加@typescript-eslint/eslint-plugin 来拓展一些关于 TS 代码的规则，如下代码所示:
    ```javascript
    // .eslintrc.js
    module.exports = {
      // 添加 TS 规则，可省略`eslint-plugin`
      plugins: ['@typescript-eslint'],
    };
    ```
    值得注意的是，添加插件后只是拓展了 ESLint 本身的规则集，但 ESLint 默认并没有 开启这些规则的校验！如果要开启或者调整这些规则，你需要在 rules 中进行配置，如:
    ```javascript
    // .eslintrc.js
    module.exports = {
      // 开启一些 TS 规则
      rules: {
        '@typescript-eslint/ban-ts-comment': 'error',
        '@typescript-eslint/no-explicit-any': 'warn',
      },
    };
    ```
5.  extends - 继承配置
    extends 相当于继承另外一份 ESLint 配置，可以配置为一个字符串，也可以配置成一个 字符串数组。主要分如下 3 种情况:

    从 ESLint 本身继承；
    从类似 eslint-config-xxx 的 npm 包继承；
    从 ESLint 插件继承。

    ```javascript
    // .eslintrc.js
    module.exports = {
    "extends": [
    // 第 1 种情况
    "eslint:recommended",
    // 第 2 种情况，一般配置的时候可以省略 `eslint-config`
    "standard"
    // 第 3 种情况，可以省略包名中的 `eslint-plugin`
    // 格式一般为: `plugin:${pluginName}/${configName}`
    "plugin:react/recommended"
    "plugin:@typescript-eslint/recommended",
    ]
    }
    ```

    有了 extends 的配置，对于之前所说的 ESLint 插件中的繁多配置，我们就不需要手动 一一开启了，通过 extends 字段即可自动开启插件中的推荐规则:

    ```javascript
    extends: ["plugin:@typescript-eslint/recommended"]
    ```

6.  env 和 globals
    这两个配置分别表示运行环境和全局变量，在指定的运行环境中会预设一些全局变量，比如:

    ```javascript
    // .eslint.js
    module.export = {
      env: {
        browser: 'true',
        node: 'true',
      },
    };
    ```

    指定上述的 env 配置后便会启用浏览器和 Node.js 环境，这两个环境中的一些全局变量 (如 window、global 等)会同时启用。

    有些全局变量是业务代码引入的第三方库所声明，这里就需要在 globals 配置中声明全局变 量了。每个全局变量的配置值有 3 种情况:

    "writable"或者 true，表示变量可重写；
    "readonly"或者 false，表示变量不可重写；
    "off"，表示禁用该全局变量。

# prettier

### 为啥需要 prettier

因为我们写的代码有时候需要可以的控制格式 比如：换行 缩进之类的 当代码量多起来 这就显得非常麻烦 所以我们需要一个工具 来自动帮我们格式化代码（让代码格式变正确）

### 怎么在项目中集成 prettier

1. 下载 prettier 命令： `pnpm install prettier`
2. 和 eslint 适配

   - 下载 eslint-config-prettier(用于用 prettier 的规则来覆盖部分 eslint 的规则) eslint-plugin-prettier（用来让 prettier 来接管 eslint 修复代码的功能）
   - 在.eslintrc.cjs 文件里面的`extend`属性里面添加`"prettier"`来解决 eslint 和 prettier 的冲突，让 prerrier 适配 eslint

   ```javascript
   // .eslintrc.cjs
   module.export = {
     extends: ['prettier'],
   };
   ```

3. 添加.prettierrc 文件来配置 prettier 相关规则

   ```javascript
   // .prettierrc.js
   module.exports = {
     printWidth: 80, //一行的字符数，如果超过会进行换行，默认为80
     tabWidth: 2, // 一个 tab 代表几个空格数，默认为 2 个
     useTabs: false, //是否使用 tab 进行缩进，默认为false，表示用空格进行缩减
     singleQuote: true, // 字符串是否使用单引号，默认为 false，使用双引号
     semi: false, // 行尾是否使用分号，默认为true
     trailingComma: 'none', // 是否使用尾逗号
     bracketSpacing: true, // 对象大括号直接是否有空格，默认为 true，效果：{     a: 1 }
   };
   ```

4. vscode 下载 prettier 插件
   vscode 下载了 prettier 插件后 在 vscode 的设置里面将 Format on Save 开启。这样当你 ctrl + s 保存的时候代码就会自动被 prettier 格式化
   ![Vj92oU.png](https://i.imgloc.com/2023/06/19/Vj92oU.png)
