---
title: React 和 Vue的区别
subtitle: 记录React学习过程中发现的和Vue的区别
link: differenceWithVue
catalog: true
date: 2023-02-25 16:52:19
tags:
  - 前端
  - React
  - React和Vue的区别
categories:
  - [笔记, 前端, React, React学习]
---

# 对数据管理和页面渲染的解析

- Vue
  - 用户只需要去关心数据，而完全不需要关心界面是怎么渲染怎么更新的，响应式数据一改变，vue 内部就会做数据劫持，触发收集的依赖，从而调用 render 函数完成视图的更新，这都是 vue 帮我们做好的，我们只需要关心数据。
- React
  - React 是没有数据劫持的，我们想要界面随着数据更新而更新就必须调用`setState`, 而调用`setState`实际上不仅改了数据，内部还相当于调用了`render`函数来对页面进行更新，也就是说用户不仅仅需要去关心数据的改变，还需要手动的去触发页面的渲染更新。

# DOM 的渲染

- Vue
  - template -> 编译器 -> h 函数 -> 虚拟 DOM -> render 函数 -> 真实 DOM
  - 组件的 render 函数里面的 h 函数 -> 虚拟 DOM -> render 函数 -> 真实 DOM
- React

  - JSX -> React.createElement() -> 虚拟 DOM -> ReactDom.render() -> 真实 DOM
  - React.createElement() -> 虚拟 DOM -> ReactDom.render() -> 真实 DOM
    ps: babel 会自动将 JSX 代码交给 React.createElement 来处理以生成虚拟 DOM

# 组件的创建

- Vue
  SFC 单文件组件，即.vue 文件
- React
  1. 函数组件，类组件，本质都是返回一个 JSX.
  2. 类组件有自己的状态`state`
  3. 函数组件需要借助 hooks 来拥有自己的状态

# 书写 html 的形式

- Vue
  **模板语法**

  1. 在`{{}}`里面书写 js 表达式，存在模板语法，v-bind，v-model，v-on 等

- React
  **JSX**
  1. 只能有一个根节点
  2. 在`{}`里面写 JS 表达式，但是写子文本节点的时候有以下规则
  - Number，String， Array 类型可以直接显示为子文本节点, Array 类型会转成字符串展示为文本节点
  - null， undefined，Boolean 会显示为空文本节点
  - object 类型不能作为子文本节点
  3. 注释要以`{/* */}`的形式书写

# 让组件接收 DOM 并展示在组件里的指定位置

- Vue
  `slot`插槽来实现
  - 作用域插槽，让插槽的内容能够访问到子组件的状态
    在子组件的`<slot/>`上通过属性来定义传递给父组件的信息，父组件在插槽里通过`v-slot=xxx`来获取
- React
  1. `prop.children`属性来实现，在组件里面写的内容会被添加到`prop.children`，当子节点个数为一个的时候，`children`就为这个子节点，当子节点为多个的时候，`children`为一个数组
  2. 通过给`prop`传递 JSX 来实现（更推荐这种）
  - 类似作用域插槽的实现
    父组件传递给子组件一个属性，属性值为一个函数，返回一个 JSX，JSX 也就是要让子组件展示的内容；然后子组件调用这个函数，通过传参数来给父组件传递信息

# 生命周期

- Vue
  ![Vue生命周期](https://i.328888.xyz/2023/02/27/e2qPy.png)
- React
  ![React生命周期](https://i.328888.xyz/2023/02/27/eH4IH.png)

# 父子组件通信

- Vue
  - 父传子时子组件对传递过来的属性做验证，设置默认值
    通过`defineProps`给其传递一个校验对象，在校验对象里面设置默认值
  - 子组件向父组件传递事件
    通过`defineEmit`定义事件，父组件监听这个事件，然后子组件通过`emit`发送事件
- React
  - 父传子时子组件对传递过来的属性做验证
    通过从`prop-types`包中导入`PropTypes`进行校验，通过给组件添加`deafultProps`属性来添加默认值
  - 子组件向父组件传递事件
    父组件给子组件传递一个函数作为`prop`, 子组件调用父组件传过来的属性里面存储的函数，来修改父组件的状态

# 爷孙组件通信

- Vue
  provide, inject
- React
  - Context（超级麻烦，一般不用）
  1.  `React.createContext()`返回一个上下文组件比如`MyContext`
  2.  在爷爷组件里面用`<MyContext.Provider></MyContext.Provider>`, 通过 value 属性来传递要传递给孙子的值， 上下文组件里面嵌套要传递给哪个孙子组件的后代。
  3.  在孙子组件里面，如果是类组件比如`sunZi`，就通过`sunZi.contextType = MyContext`来指定自己接收哪个 context，然后通过`this.context`来使用。如果是函数组件，则通过`<MyContext.consumer>(value) => {}</MyContext.consumer>`来使用，参数 value 就是传递过来的数据
      **这。。。。。。谁发明的反人类的玩意，其他具体的使用去看文档吧，实在不想写了**

# 界面的更新为异步操作，在修改状态后，使用新的状态的解决方案

- Vue
  nextick 里面传一个回调函数，这个回调函数会在页面更新后调用
- React
  setState 中给第二个参数传一个回调函数，这个回调函数会在页面更新完毕后调用

**注意：**

1. 界面的更新为异步操作的原因：
   Vue 和 React 都是一样的，都是避免状态的多次改变导致多次调用 render 函数，影响性能，都想在同步代码（包含对状态的修改）执行完毕后，再一次性的调用 render 函数，对界面进行更新，提高性能。
2. 二者的不同：
   Vue 对响应式数据的更新是同步的，所以它会把触发的多次副作用（对界面的更新）放进一个`Set`，利用自动去重的机制，相同的渲染操作只执行一次。
   React 对状态的更改也是异步的，所以它会把多次状态的更改放进一个队列，然后合并每次对状态的修改，然后拿这个最终的状态去更改状态，并调用 render 函数进行渲染，从而做到只执行一次渲染。
3. React18 之前的版本和 React18 版本中对于 setState 特殊情况的处理
   React18 之前的版本：在事件里面，setTimeout，Promise 等这些的`callback`中`setState`是同步的
   React18 中：上面这些特殊情况里面`setState`也已经是异步的了，原因同上，如果这些特殊情况还是想让`setState`为同步的话，需要从`react-dom`中引入一个`flashSync`方法，在里面传递一个`callback`，在`callback`里面调用`setState`则为同步

# React 性能优化(我认为是比 Vue 强大的一点)

1. SCU

   - 原理:
     可以在 shouldComponentUpdate 这个生命周期钩子函数里面，从参数里拿到`newState`,`newProps`和当前的`state`,`props`作比较，当有`key`对应的`value`不一样的时候才返回`true`才调用 render 函数返回 jsx，进行 diff，然后更新变化了的部分。（而 Vue 完全没有这个说法，只要父组件发生了更新，就会对子组件调用 render 函数生成虚拟 DOM，然后开始比较新旧 DOM, 这个过程是避免不了的，只不过 Vue 是在 render 函数内部对新旧 DOM 做了一个比较判断，但是虚拟 DOM 生成这一步的开销避免不了）
   - 自动化 SCU 解决方案
     类组件：extends pureComponent
     函数组件：用 memo 方法把函数组件包裹起来 然后返回一个新的组件

# 对表单元素的处理

- Vue
  利用`v-model`语法糖来实现双向绑定
- React
  React 没有双向绑定，需要给 JSX 表单元素的`value`属性手动绑定一个变量，然后监听 change 事件，当 change 事件被触发的时候拿`event.target.value`来通过`setState`修改绑定的变量

# 对 DOM 和 Component 的获取

- Vue
  通过给 DOM, Component 传递`ref`属性，然后用一个`ref`属性值同名的响应式变量来接收 DOM, Component。
- React
  1. 对于 DOM, 类组件, 在`state`里通过`createRef`来创建一个存储 DOM, 类组件的变量，然后把这个变量赋值给 DOM, 类组件的`ref`属性。这个变量保存的就是 DOM，变量的`current`属性保存的就是 Component
  2. 对于函数组件，通过`forwardRef`高阶函数来对函数组件进行一个包裹，此时函数组件将会有两个参数，`props`,`ref`,再把`ref`这个参数用 ref 属性绑定到函数组件返回的 JSX 中的 DOM 上

# CSS 的书写

- Vue
  直接在 SFC 里面的`<style></style>`标签里面写，通过`lang`属性来设置自己想用的预处理器，非常方便
- React
  一般是使用`css in js`方案，需要用到`styled-components`库,然后默认导出一个`styled`方法 下面是一个使用案列
  ![6Vqnd.png](https://i.328888.xyz/2023/03/01/6Vqnd.png)

# 动态添加类名

- Vue
  利用`v-bind`绑定 class
- React
  利用一个`classnames`的库，然后默认导出一个`classNames`方法，下面是一个使用案列
  ![6flyy.png](https://i.328888.xyz/2023/03/01/6flyy.png)
