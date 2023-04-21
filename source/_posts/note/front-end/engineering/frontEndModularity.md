---
title: 前端模块化分析与对比
subtitle: CommmonJS VS ESModule
catalog: true
link: frontEndModularity
date: 2023-04-18 13:50:07
tags:
  - 前端
  - 前端工程化
  - 前端模块化
categories:
  - [笔记, 前端, 前端工程化]
---

# CommonJS

## 核心函数

- exports
  `exports`是一个 object，可以在它身上添加需要导出的属性，是默认的根本导出的对象
- module.exports
  `module.exports`是被模块直接导出的对象，它实际上默认是`exports`的引用
- require
  `require`实际上是会执行要导出的模块里的 code，然后导入被导入模块的`module.exports`

## 加载模式

commonJS 的加载模式是**运行时**的，**同步**的，它是当代码运行到`require`函数的这一行时，才会开始加载模块，并且执行模块里面的代码，模块里的代码执行完毕才会执行下一行。

## 加载模式带来的特点

- 基于这个运行时的机制，require 才可以在**任何地方**使用，并且在路径里面可以**使用变量**，因为此时变量已经有了值
- 基于这个同步的机制，当 require 的模块还在加载中时，导入的变量的值会是`undefined`(常见于循环引用中，比如 b 依赖于 a， 而 a 又没执行完毕， 那么`require A`的值则会是`undefined`)


## CommonJS对循环依赖的处理

```javascript
// index.js
let count = require("./counter.js").count;

console.log(count);
exports.message = "hello world!";

// counter.js
let message = require("./index.js").message;

exports.count = 5;
setTimeout(() => {
  console.log(message);
}, 0);
```

打印的结果为： 5 undefined

原因：

- 首先加载`index.js`，执行到第一行的时候遇到 require，开始加载`counter.js`
- `counter.js`的第一行被执行，又遇到 require，需要去加载`index.js`，但此时`index.js`还没有加载完毕， 所以此时 message 的值为 undefined
- count 变量被导出
- 注册定时器宏任务，打印 message（值为 undefined）
- 回到`index.js`
- 打印 count（值为 5）
- 导出 message（值为 ‘hello world’）， 然而已经没用了，因为`counter.js`里面的 message 的值已经被解析为了 undefined
- 执行定时器宏任务，打印 message（值为 undefined）

# ESModule

## 加载模式

ESModule 的加载模式是分为构建阶段，实例化阶段，执行阶段的， 并且所有模块都是**异步递归加载解析的**。而对依赖关系的处理是在构建阶段，也就是**解析时**进行

1. **构建阶段**
   主要是浏览器尝试下载所有需要的模块文件，并形成模块记录的过程

- 浏览器开始解析入口模块文件（是解析，不是执行，只是为了确定依赖），确定入口文件的依赖模块，从而发送请求，下载相关依赖模块。等到依赖模块文件返回后，浏览器继续解析这些依赖模块文件，从而确定依赖文件的依赖，然后发请求下载，并解析，如此循环递归，直到所有依赖文件全部被下载。
- 每个模块加载完毕后，都会创建相应的**模块记录**，同时浏览器还会维护一张**模块映射表**，它保存了**模块路径 -- 模块记录**的映射关系。

2.  **实例化阶段**
    主要是将模块里面的**export 和 import**和**内存**建立起关系， 也就是为**export**开辟内存空间。

- 实现 JS 引擎慧创建一个模块环境记录，用来管理模块中导入和导出的变量
- 首先处理**export**, 为每一个**export**在内存中开辟一个对应的空间，但是这个时候内存空间里面没有值，赋值操作是在执行阶段发生的。
- JS 引擎处理完模块所有的导出之后，才会开始处理模块的导入，对同一个模块的导入和导出指**向的是内存中的同一片内存空间**

3.  **执行阶段**

- 开始执行 js 代码，进行赋值操作，将变量的值填充到对应的内存空间

## 加载模式带来的特点

- ESModule 对模块的下载和解析是发生在构建阶段的时候的，这个时候 js 代码还没执行，变量还没值（js 引擎连内存空间都还没给它开辟），所以**import 语句不能携带变量**，**不能直接在代码块中嵌入**

# ESModule 对循环依赖的处理

下面让我们来看一个循环依赖的例子，并分析和处理。

```javascript
// index.js
import { count } from "./counter.js";

const message = "666";

console.log(count);

export { message };

// counter.js
import { message } from "./index.js";

const count = 5;

setTimeout(() => {
  console.log(message);
}, 0);

export { count };
```

打印的结果为：5 666

原因：

- 构建阶段
  - 浏览器下载`index.js`文件
  - 下载完毕后，开始解析`index.js`文件, 发现`index.js`依赖`counter.js`
  - 下载`counter.js`
  - 下载完毕后，解析`counter.js`文件，发现没有依赖
  - 构建阶段完毕
- 实例化阶段
  - 为`index.js`和`counter.js`的`export`在内存中开辟一段内存空间
  - 将`import`指向其对应的模块的`export`的内存空间
- 执行阶段
  - 执行`index.js`中的代码，将局部变量message赋值为666（此时message还没有被写入到export的那片内存）
  - 打印导入的count，但是此时import指向的那片内存，count还没有被填充，所以开始执行`counter.js`的代码。
  - 将局部变量count赋值为5（此时count还没有被写入到export的那片内存）
  - 将定时器宏任务加入消息队列
  - 执行`export`来将count填充到内存空间
  - 继续执行`index.js`，此时import指向的内存空间（就是counter.js的export的内存空间）count已经被填充，顺利打印出5.
  - 执行`export`来将message填充到内存空间
  - 执行计时器宏任务中的callback，此时`counter.js`的`import`指向的内存空间（就是index.js的export的内存空间）message已经被填充，顺利答应出666
