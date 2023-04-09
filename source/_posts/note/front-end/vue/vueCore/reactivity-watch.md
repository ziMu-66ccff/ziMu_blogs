---
title: Vue响应式模块-reactivity-watch(watchEffect)
subtitle: watch刨析与实现
link: reactivity-watch
catalog: true
date: 2023-02-14 10:21:11
tags:
  - 前端
  - Vue
  - Vue原理刨析
  - Vue响应式模块刨析
  - watch
  - watchEffect
categories:
  - [笔记, 前端, Vue, Vue原理实现]
---

# 文档描述

侦听一个或多个响应式数据源，并在数据源变化时调用所给的回调函数。
watch() 默认是懒侦听的，即仅在侦听源发生变化时才执行回调函数。
watch 的第一个参数可以是不同形式的“数据源”：它可以是一个 ref (包括计算属性)、一个响应式对象、一个 getter 函数、或多个数据源组成的数组：

# 实现思路

1. 因为监听的响应式对象 or getter 函数返回的响应式数据发生改变的时候，就要执行回调函数`callback`，所以需要引入`effect`函数来进行副作用函数依赖的注册，收集，触发，收集的依赖也就是`getter`函数
2. 当监听的对象 or getter 函数返回的响应式数据发送改变的时候，触发依赖，这里需要用到`scheduler`,让它执行`scheduler`，从而调用上次回调函数传递过来的`Clearup`副作用清理函数，并调用`getter`获取新值值，然后调用回调函数`callback`,并将旧值，新值，注册副作用清理函数的函数`onInvilidate`作为参数传递，并将新值赋值给保存旧值的变量，作为下一次 callback 执行时的旧值。

# 完整代码

```typescript
import { isFunction } from '../utils/index';
import { effect } from './effect';
import type { EffectFn } from './effect';

export type WatchOptions = {
  immediate?: boolean;
  flush?: 'flush';
};

export function watch(
  source: {} | (() => any),
  cb: (
    oldValue?: any,
    newValue?: any,
    onInvalidate?: (fn: () => void) => void
  ) => void,
  options?: WatchOptions
) {
  let getter: () => any;
  let oldValue: any;
  let newValue: any;
  let clearUp: (() => any) | undefined = undefined;

  if (isFunction(source)) {
    getter = source;
  } else {
    getter = () => traverse(source);
  }

  function onInvalidate(fn: () => void) {
    clearUp = fn;
  }

  const effectFn = effect(() => getter(), {
    lazy: true,
    scheduler() {
      if (options?.flush) {
        // To do
      } else {
        job();
      }
    },
  }) as EffectFn;

  const job = () => {
    newValue = effectFn();
    if (clearUp) clearUp();
    cb(oldValue, newValue, onInvalidate);
    oldValue = newValue;
  };

  if (options?.immediate) {
    job();
  } else {
    oldValue = effectFn();
  }
}

function traverse(value: any, seen: Set<any> = new Set()) {
  if (typeof value !== 'object' || value === null || seen.has(value)) return;
  seen.add(value);
  for (let k in value) {
    traverse(value[k], seen);
  }
  return value;
}
```

# 代码执行过程梳理

1. 对监视源做处理，如果是`getter`则直接传递给`getter`变量，如果是响应式对象，则调用`traverse`函数完成对响应式对象所有属性的读取，并将其包装为`getter`再赋值给`getter`变量
2. 用`effect`注册`getter`副作用函数依赖，以进行依赖的收集，触发。
3. 手动调用`getter`，调用的过程中会读取监视源的响应式数据，以完成对依赖的收集，并将获取的值赋值给旧值`oldValue`。
4. 当监视源的响应式数据被修改时，收集的依赖被触发，因为配置的有`scheduler`调度器，就会执行调度器，从而执行上一次回调函数`callback`执行时注册的副作用清理函数`Clearup`，并手动调用`getter`来获取新值`newValue`, 执行`callback`回调函数，并将旧值`oldValue`，新值`newValue`, 注册副作用清理函数的函数`onInvalidate`作为参数传递。最后将新值赋值给`oldValue`作为下一次 callback 执行时的旧值

# watchEffect

# 文档介绍

立即运行一个函数，同时响应式地追踪其依赖，并在依赖更改时重新执行。

# 实现原理（和 watch 基本一样）

1. 同样，因为追踪的响应式依赖被修改时，也要执行 callback，所以也需要引入 effect, 来进行副作用函数依赖的注册，收集，触发，这里收集的依赖也就是`callback`
2. 先手动执行一波`callback`，执行的时候会读取响应式数据，也就完成了依赖的收集。
3. 当响应式数据被修改的时候，收集的依赖就会被触发，这里我们也要配置`scheduler`，在执行的时候先看有没有上次`callback`执行的时候传递给我们的副作用清理函数，有便执行，然后执行`callback`。

# 完整代码

```typescript
import { effect } from './effect';
import type { EffectFn } from './effect';
import type { WatchOptions } from './watch';

export function watchEffect(
  cb: (onInvalidate?: (fn: () => void) => void) => void,
  options: WatchOptions
) {
  let clearUp: (() => void) | undefined = undefined;

  const job = () => {
    if (clearUp) clearUp();
    cb(onInvalidate);
  };

  function onInvalidate(fn: () => void) {
    clearUp = fn;
  }

  const effectFn = effect(() => cb(), {
    lazy: true,
    scheduler() {
      job();
    },
  }) as EffectFn;

  effectFn();
}
```
