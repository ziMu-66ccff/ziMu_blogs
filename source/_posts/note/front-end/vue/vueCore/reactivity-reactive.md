---
title: Vue响应式模块-reactivity-reactive
subtitle: reactive刨析与实现
link: reactivity-reactive
catalog: true
date: 2023-02-14 10:21:11
tags:
  - 前端
  - Vue
  - Vue原理刨析
  - Vue响应式模块刨析
  - reactive
categories:
  - [笔记, 前端, Vue, Vue原理实现]
---

# ractive 方法剖析与实现

## 完整代码实现

```typescript
import { isArray, isObject, hasChanged } from '../utils/index';
import { effect, track, trigger } from './effect';

export function reactive(target: any) {
  if (!isObject(target)) return target;
  if (isReactive(target)) return target;
  if (reactiveMap.has(target)) return reactiveMap.get(target);

  const reactiveProxy: Record<string | number | symbol, any> = new Proxy(
    target,
    {
      get(target, key) {
        if (key === '_isReactive') return true;
        track(target, key);
        const res = Reflect.get(target, key);
        return isObject(res) ? reactive(res) : res;
      },
      set(target, key, newValue) {
        const oldValue = Reflect.get(target, key);
        const oldLength = Reflect.get(target, 'length');
        const res = Reflect.set(target, key, newValue);
        if (hasChanged(oldValue, newValue)) {
          trigger(target, key);
          if (
            isArray(target) &&
            oldLength != Reflect.get(target, 'length') &&
            key !== 'length'
          ) {
            trigger(target, 'length');
          }
        }
        return res;
      },
    }
  );

  reactiveMap.set(target, reactiveProxy);
  return reactiveProxy;
}

export function isReactive(target: any) {
  return !!(target && target.__isReactive);
}
```

## 边界处理（对**特殊情况**的处理）

- `target`的类型不为 `Object`时：
  - 利用`isObject`方法判断`target`的类型是否为`Object`
  - `isObject`方法的返回值为`false`时，即`target`的类型不为`Object`, 则直接返回`target`, 不做接下来的响应式处理
- `target`**本身**已经为一个**响应式代理**时:
  - 利用`isRective`方法判断`target`是否为**响应式代理**
  - `isReactive`方法的返回值为`true`时， 即`target`已经为一个**响应式代理**，则直接返回`target`，不再做接下来的处理
- `target`已经**被做过响应式代理**时:
  - 利用`reactiveMap.has(target)`方法来判断`target`是否已经被做过**响应式代理**
  - `reactiveMap.has(target)`方法返回值为`true`时， 即`target`已经被做过**响应式代理**，则直接调用`reactiveMap.get(targey)`方法返回**它的响应值代理**

## 响应式处理

### 整体概述

利用`Proxy`对`target`做一个**数据代理**，拦截其`getter` and `setter`, 最后返回这个**数据代理**

### 详细步骤

- **拦截 getter**
  1. _做边界处理_
     - `if(key === '_isReactive')`判断其要获取的`key`是否为`_isReactive`, 当`if`判断为`true`时，则直接返回`false`，**不再进行接下来的处理**
  2. _收集副作用_
     - 调用`track`方法来收集**副作用函数**，将其收集到`target`对应的`key`的对应的`Set`里面，等待`setter`被触发，并修改`target[key]`的时候调用`trigger`来将这个`Set`里面存储的**副作用函数**一一触发.
     - **ps**: _[(track 方法的讲解请点击这里奥)](https://zimu-66ccff.github.io/reactivity-effect/)_
  3. _返回`target[key]`的值，做深度响应式处理_
     - 利用`Reflect.get(target, key)`取出`target[key]`的值
     - 判断`target[key]`的值的类型是否为`Object`, 当为`Object`时，则返回`reactive(target[key])`对其做深度响应式处理，当不为`Object`时，则直接返回`target[key]`
- **拦截 setter**

  1. _取出相关数据， 进行更新操作_
     - 利用`Reflect.get`方法取出没有修改前的`target[key]`即`oldValue` and 没有修改前的`target[length]`即`oldLength`, 利用`Reflect.set`进行对`target[key]`的更新
  2. _判断是否需要调用对应的`key`对应的`Set`存储的副作用函数_
     - 利用`hasChanged()`判断新旧值是否发生了改变，如果没有改变则不进行接下来的处理。如果发生了改变，则调用`trigger`方法，来调用对应的`key`对应的`Set`存储的副作用函数
     - **ps**: _[trigger 方法的讲解请点击这里奥](https://zimu-66ccff.github.io/reactivity-effect/)_
  3. _判断`target`是否为`Array`, 并且数组的长度是否发生了改变_
     - 利用`isArray`判断`target`，并利用`Reflect.get()`取得更新后数组的长度，并与之前数组的长度`oldLength`比较，看是否发生了改变，如果发生了改变，则调用`tigger`方法，来调用`length`对应的`Set`存储的副作用。_（之所以要这样做，是因为当给数组增加 or 删除元素的时候，数组的`length`就已经改变了， 当对`length`的更新的拦截的时候，就会发现新值和旧值是一样的，就不会触发`trigger`）_

- **将响应式代理存储起来，并返回响应式代理**
  1. 调用`reactiveMap.set`方法，将响应式代理和`target`关联起来并存储，以便于做上文的**边界处理**(`target`已经被做过响应式代理时)
  2. **返回响应式代理**
