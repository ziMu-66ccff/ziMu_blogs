---
title: JS执行上下文
subtitle: 深入理解JS执行上下文
link: executionContext
catalog: true
date: 2023-02-16 23:04:30
tags:
  - 前端
  - JS高级
  - 执行上下文
categories:
  - [笔记, 前端, JavaScript]
---

# 什么是执行上下文

简单地说，执行上下文是评估和执行 Javascript 代码的环境的一个抽象概念。任何代码在 JavaScript 中运行时，都在执行上下文中运行。

# 执行上下文的类型

- 全局执行上下文
  默认的，基础的执行上下文，任何不在函数内部的代码都在全局执行上下文中，全局执行上下文只能有一个。
- 函数执行上下文
  函数被调用时创建的执行上下文，程序中可以有多个函数执行上下文
- Eval 函数执行上下文
  Eval 函数内部的代码执行时也会有自己的执行上下文，但是我们一般不使用 Eval 函数，所以这里我们不讨论。

# 执行栈

- 在别的编程语言中也被称为**调用栈**, 是一种先入后出的栈形结构, 用来存储程序执行过程中创建的所有执行上下文。
- 当 js 代码开始执行的时候就会创建一个全局执行上下文，然后将其加入执行栈中，当遇到函数调用的时候就将创建一个函数执行上下文， 将其加入执行栈中，当函数调用完毕再将其移出执行栈。

下面看一个例子。

```javascript
let a = 'Hello World!';
function first() {
  console.log('Inside first function');
  second();
  console.log('Again inside first function');
}
function second() {
  console.log('Inside second function');
}
first();
console.log('Inside Global Execution Context');
```

![执行栈](https://i.328888.xyz/2023/02/16/p0Z35.png)
上图为执行栈

当浏览器加载上述代码时，Javascript 引擎会创建一个全局执行上下文，并将其推入当前执行栈。当遇到对`first()`的调用时，Javascript 引擎会为该函数创建一个新的执行上下文（函数执行上下文），并将其推到当前执行堆栈的顶部。
当在 `first()`函数中调用 `second()`函数时，Javascript 引擎会为该函数创建一个新的执行上下文，并将其推到当前执行栈的顶部。当 `second()`函数结束时，它的执行上下文从当前栈中弹出，控件到达它下面的执行上下文，也就是 `first()`函数的执行上下文。

# 词法环境（Lexical Environment）

词法环境由三部分组成

- 环境记录器， 环境记录器有两类
  - 声明型环境记录: 存储着变量与函数的声明。
  - 变量型环境记录 除了存储着变量与函数的声明以外，还存储着一个全局对象，浏览器环境为`window`
  - _ps_：对于函数执行上下文，环境记录还应该包括一个参数对象（arrguments），它保存着参数索引与参数之间的映射，以及参数数组的长度
- 指向外部环境的引用
  - 保存着外部环境的引用，因为它才有作用域链查找
- this 的绑定
  - 全局执行上下文`this`绑定为 window， 函数上下文 this 绑定为函数的调用者

# 变量环境（Variable Environment）

变量环境本身就是一个词法环境，所以上文的东西它都有。

# 词法环境与变量环境的区别

在**ES6**中，`let` 和 `const`声明的变量，他们的变量声明式存储在词法环境的。而`var`声明的变量是存储在变量环境的。

# 执行上下文创建与执行过程发生了什么

- 创建过程
  - 创建词法环境
    - 环境记录器开始保存`let`和`const`声明的变量的声明，以及函数的声明， 如果是函数上下文的话，还会保存一个`arrguments`参数对象。
    - 外部环境的引用开始保存外部环境，以便于进行作用域链查找
    - 开始进行 this 的绑定，如果是全局执行上下文，则让 this 绑定 window 对象，如果是函数执行上下文，则让 this 绑定函数的调用者。
  - 创建变量环境
    - 环境记录器开始保存`var`声明的变量
    - 外部环境的引用开始保存外部环境，以便进行作用域链查找
    - 同上
- 执行过程
  完成变量的赋值

下面是一个例子

```javascript
let a = 20;
const b = 30;
var c;
function multiply(e, f) {
  var g = 20;
  return e * f * g;
}
c = multiply(20, 30);
```

创建阶段，全局执行上下文中词法环境与变量环境的情况

```javascript
GlobalExectionContext = {
  LexicalEnvironment: {
    EnvironmentRecord: {
      Type: "Object",
      // Identifier bindings go here
      a: < uninitialized > ,
      b: < uninitialized > ,
      multiply: < func >
    }
    outer: < null > ,
    ThisBinding: < Global Object >
  },
  VariableEnvironment: {
    EnvironmentRecord: {
      Type: "Object",
      // Identifier bindings go here
      c: undefined,
    }
    outer: < null > ,
    ThisBinding: < Global Object >
  }
}
```

执行阶段，全局执行上下文中词法环境与变量环境的情况

```javascript
GlobalExectionContext = {
  LexicalEnvironment: {
      EnvironmentRecord: {
        Type: "Object",
        // Identifier bindings go here
        a: 20,
        b: 30,
        multiply: < func >
      }
      outer: <null>,
      ThisBinding: <Global Object>
    },
  VariableEnvironment: {
      EnvironmentRecord: {
        Type: "Object",
        // Identifier bindings go here
        c: undefined,
      }
      outer: <null>,
      ThisBinding: <Global Object>
    }
  }
```

当调用` multiply`时，将会创建一个函数执行上下文

创建阶段，函数执行上下文中词法环境和变量环境的情况

```javascript
FunctionExectionContext = {
LexicalEnvironment: {
    EnvironmentRecord: {
      Type: "Declarative",
      // Identifier bindings go here
      Arguments: {0: 20, 1: 30, length: 2},
    },
    outer: <GlobalLexicalEnvironment>,
    ThisBinding: <Global Object or undefined>,
  },
VariableEnvironment: {
    EnvironmentRecord: {
      Type: "Declarative",
      // Identifier bindings go here
      g: undefined
    },
    outer: <GlobalLexicalEnvironment>,
    ThisBinding: <Global Object or undefined>
  }
}
```

执行阶段，函数执行上下文中词法环境和变量环境的情况

```javascript
FunctionExectionContext = {
LexicalEnvironment: {
    EnvironmentRecord: {
      Type: "Declarative",
      // Identifier bindings go here
      Arguments: {0: 20, 1: 30, length: 2},
    },
    outer: <GlobalLexicalEnvironment>,
    ThisBinding: <Global Object or undefined>,
  },
VariableEnvironment: {
    EnvironmentRecord: {
      Type: "Declarative",
      // Identifier bindings go here
      g: 20
    },
    outer: <GlobalLexicalEnvironment>,
    ThisBinding: <Global Object or undefined>
  }
}
```
