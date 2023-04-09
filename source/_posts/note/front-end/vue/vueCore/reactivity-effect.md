---
title: Vue响应式模块-reactivity-effect
subtitle: effect, track, trigger刨析与实现
link: reactivity-effect
catalog: true
date: 2023-02-14 13:57:48
tags:
  - 前端
  - Vue
  - Vue原理刨析
  - Vue响应式模块刨析
  - effect
categories: [笔记, 前端, Vue, Vue原理实现]
---

# effect 方法剖析与实现

## 完整代码

```typescript
export type EffectFn = {
  (): any;
  deps: Array<Set<EffectFn>>;
  options?: Options;
};

type Options = {
  scheduler?: (fn: EffectFn) => void;
  lazy?: boolean;
};

let activeEffect: EffectFn;
const effectStarck: Array<EffectFn> = [];
const targetMap: WeakMap<
  object,
  Map<any, Set<EffectFn> | undefined> | undefined
> = new WeakMap();

export function effect(fn: () => any, options?: Options) {
  const effectFn: EffectFn = () => {
    clearUp(effectFn);
    activeEffect = effectFn;
    effectStarck.push(effectFn);
    const res = fn();
    effectStarck.pop();
    activeEffect = effectStarck[effectStarck.length - 1];
    return res;
  };
  effectFn.deps = [] as Array<Set<EffectFn>>;
  effectFn.options = options;
  if (effectFn.options?.lazy) {
    return effectFn;
  }
  effectFn();
}

function clearUp(fn: EffectFn) {
  fn.deps?.forEach((deps) => {
    deps.delete(fn);
  });
}
```

## effect 概述

`effect`是一个用来注册副作用的函数，它默认会将注册的副作用立即执行一次，并支持传递一个选项参数`options`，可以配置是否立即执行，将副作用函数返回以便于手动执行 and 配置调度器`scheduler`

## 部分变量，属性，方法介绍

- **`activeEffect`** : 用来存储当前的副作用函数
- **`effectStrack`** : 因为`effect` 可能会嵌套 `effect` ，所以需要用`effectStrack`来存储这些嵌套的**副作用函数**，当内层的`effect`执行完毕后，将**内层的副作用函数**弹出，将**外层的副作用函数**赋值给用来存储**当前的副作用函数**的`activeEffect`。
- **`targetMap`**: 一种`WeakMap`结构，用来存储`target`, `key`, **副作用**之间的对应关系。
  ![关系图](https://i.328888.xyz/2023/02/14/mVXrd.png)
- **`effectFn.deps`**: 类型为`Array<Set>`，用来存储保存的有该副作用的`target`对应的`key`对应的`Set`,以便于在副作用函数**再次执行**的时候调用的`clearUp`方法来将`effectFn`从这些`Set`中移除。
- **`clearUp`** : 利用`effectFn.deps`将`effectFn`从其所在的`Set`中移除。

## 详细步骤

- 定义`effectFn`增强版副作用函数, 以便于增强它,并做进一步处理
  - `effectFn`函数将做的事情
    1. 调用`clearUp`来将`effectFn`从其所在的`Set`中移除，从而避免`trigger`触发没必要的副作用函数。
    2. 将`effectFn`赋值给`activeEffect`, 并将`effectFn`放进`effectStack`。
    3. 调用`fn`副作用函数，将返回值保存进`res`。
    4. 将`effectFn`从`activeEffect`中弹出, 并将`activeEffect`的最后一个元素赋值给`activeEffect`（让`activeEffect`指向外层的副作用函数）
    5. 返回`res`
- 在`effectFn`函数上添加属性，增强它
  - 在`effectFn`上添加`deps`属性，用来存储保存有该副作用函数的`target`对应的`key`的对应的`Set`, 以提供给`clearUp`函数使用。
  - 在`effectFn`函数上添加`options`属性, 保存副作用函数的配置选项。
- 根据`effectFn`函数的配置选项`options`属性，来进行下一步的操作
  - 如果`effectFn.options.lazy`为`true`，则不立即执行`effectFn`, 而是将`effectFn`返回，手动调用。
  - 如果为`false`， 则立即执行。

# track 方法剖析与实现

## 完整代码

```typescript
export function track(target: {}, key: any) {
  if (!activeEffect) return;
  let depsMap = targetMap.get(target);
  if (!depsMap) targetMap.set(target, (depsMap = new Map()));
  let deps = depsMap.get(key);
  if (!deps) depsMap.set(key, (deps = new Set()));
  deps.add(activeEffect);
  activeEffect.deps.push(deps);
}
```

## track 概述

用来收集副作用，将副作用存储进对应的`target`对应的`key`对应的`Set`。

## 详细步骤

- 边界处理
  - 如果`activeEffect == undefined`，则直接返回
- 将副作用函数存储进对应的`Set`
  1. 调用`targetMap.get(target)`取出`target`对应的`Map`（里面存储的是`key`和对应的`Set`）并赋值给`depsMap`。如果`depsMao`为空，则调用`targetMap.set(target, (depsMap = new Map))`新建一个`Map`存进`targetMap`。
  2. 调用`depsMap.get(key)`取出`key`对应的`Set`(里面存储的是`key`对应的副作用函数)并赋值给`deps`。如果`deps`不存在，则调用`depsMap.set(key, (deps = new Set()))`新建一个`Set`存进`depsMap`。
  3. 调用`deps.add(activeEffect)`将副作用函数存储进`Set`.
- 将存有该副作用函数的`Set`存储进`activeEffect.deps`

# trigger 方法剖析与实现

## 完整代码

```typescript
export function trigger(target: {}, key: any) {
  const depsMap = targetMap.get(target);
  if (!depsMap) return;
  const effects = depsMap.get(key);
  if (!effects) return;
  const effectsToRun: Set<EffectFn> = new Set();
  effects?.forEach((effectFn) => {
    if (effectFn !== activeEffect) {
      effectsToRun.add(effectFn);
    }
  });
  effectsToRun.forEach((effectFnToRun) => {
    if (effectFnToRun.options?.scheduler) {
      effectFnToRun.options.scheduler(effectFnToRun);
    } else {
      effectFnToRun();
    }
  });
}
```

## trigger 概述

用来调用`track`收集的副作用，如果有调度器，则调用调度器。

### 详细步骤

- 取出对应的`target`的对应的`key`的对应的`Set`, 并做边界处理
  1. 调用`targetMap.get(target)`取出`target`对应的`Map`赋值给`depsMap`。如果`depsMap`不存在，则直接返回
  2. 调用`depsMap.get(key)`取出`key`对应的`Set`并复制给`effects`，如果`effects`不存在，则直接返回。
- 定义一个`effectsToRun`, 将`effects`存储的副作用函数全部添加到`effectsToRun`, 以等待遍历`effectsToRun`执行存储的所有副作用函数。（之所以这么做是因为，副作用函数在执行的时候会先调用`clearUp`将其从`Set`中删除，执行`fn`的时候又会将`effectFn`添加到`Set`, 这样就会陷入无限循环， 所以定义一个新的`Set`来避免这个问题）
- 调用`effectsToRun.forEach`来遍历它，并调用存储的副作用函数，但是在调用前会做判断，如果`effectToRun.options.scheduler`存在的话，会调用`effectToRun.options.sheduler`而不是调用`effectToRun`
