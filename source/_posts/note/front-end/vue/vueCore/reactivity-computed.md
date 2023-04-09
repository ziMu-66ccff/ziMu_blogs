---
title: Vue响应式模块-reactivity-computed
subtitle: computed刨析与实现
link: reactivity-computed
catalog: true
date: 2023-02-14 10:21:11
tags:
  - 前端
  - Vue
  - Vue原理刨析
  - Vue响应式模块刨析
  - computed
categories:
  - [笔记, 前端, Vue, Vue原理实现]
---

# 文档描述

接受一个 getter 函数，返回一个只读的响应式 ref 对象。该 ref 通过 .value 暴露 getter 函数的返回值。它也可以接受一个带有 get 和 set 函数的对象来创建一个可写的 ref 对象。
具有缓存机制，只有当依赖的响应式数据发生改变的时候才会重新计算，否则会直接读取缓存的值

# 实现思路

1. computed 返回的是一个`ref`，所以我们需要一个类来进行`get value`,`set value`的操作
2. 当依赖的响应式数据发生改变时会重新计算，所以需要引入`effect`函数来注册依赖副作用，并进行依赖的收集和触发操作，收集的依赖就是`getter`
3. computed 具有缓存机制，当依赖的响应式数据发生改变的时候才需要用`getter`重新计算来更新缓存的 value 值，没改变的时候直接用缓存的 value，所以需要一个`isDirty`标志变量，来标志是否需要重新计算
4. 当依赖的响应式数据改变时，修改`isDirty`标志变量，这里需要利用`scheduler`调度器，当响应式数据发生改变触发依赖的时候，不是直接执行收集的`getter`而是去修改`isDirty`,这样 computed 的 value 被取值的时候就知道需要重新计算更新缓存的 value 值。

# 完整代码

```typescript
import { isFunction } from '../utils/index';
import { effect, track, trigger } from './effect';
import type { EffectFn } from './effect';

type computedOptions =
  | { getter: () => any; setter: (newValue?: any) => void }
  | (() => any);

export function computed(computedOptions: computedOptions) {
  let getter: () => any;
  let setter: (newValue?: any) => void;

  if (isFunction(computedOptions)) {
    getter = computedOptions;
    setter = () => {
      console.warn('Write operation failed: computed value is readonly');
    };
  } else {
    getter = computedOptions.getter;
    setter = computedOptions.setter;
  }

  return new computedRefImpl(getter, setter);
}

class computedRefImpl {
  _value: any;
  _isDirty: boolean;
  _setter: (newValue?: any) => void;
  _effectFn: EffectFn;

  constructor(getter: () => any, setter: (newValue?: any) => void) {
    this._value = undefined;
    this._isDirty = true;
    this._setter = setter;
    this._effectFn = effect(getter, {
      lazy: true,
      scheduler: (fn) => {
        if (!this._isDirty) {
          this._isDirty = true;
          trigger(this, 'value');
        }
      },
    }) as EffectFn;
  }

  get value() {
    if (this._isDirty) {
      this._value = this._effectFn();
      this._isDirty = false;
      track(this, 'value');
    }
    return this._value;
  }

  set value(newValue) {
    this._setter(newValue);
  }
}
```

# 代码执行过程梳理

1. 判断传递进来的参数是不是 function，是的话则说明只传递了 getter，就把传递的函数赋值给 getter 变量，setter 变量则被赋值为一个警告，不能修改 computed 计算属性的 value 值。如果不是 function，则说明传递了一个配置对象，里面有 getter，也有 setter，就把 getter 赋值给 getter 变量，setter 赋值给 setter 变量，然后调用`computedRefimpl`实列化来返回`Ref`,实例化的过程中在 constructor 构造函数里，会利用 effect 来收集依赖`getter`
2. 当我们第一次的去读取计算属性的 value 值的时候，会执行`get value`方法，此时标志变量`isDirty = true`所以会调用`getter`函数来进行计算，计算完毕后则将标志变量`isDirty`再修改为`false`。然后将计算得来的值保存在 value 中进行缓存。计算的过程中会读取依赖的响应式变量，依赖 getter 也就顺利的被响应式变量收集。并且此时 computed 还会手动调用`track`函数进行依赖的收集，这个时候收集的依赖其实是用到了计算属性的地方
3. 当我们去修改计算属性依赖的响应式数据的时候，响应式数据收集的依赖`getter`函数就会触发，但是因为有`scheduler`调度器，所以执行的不是`getter`而是`scheduler`，将标志变量`isDirty`修改为`true`，并手动调用`trigger`函数来触发 computed 计算属性收集的依赖，也就是用到 computed 计算属性地方。
4. 在用到计算属性的地方就会去读取 computed 的 value 值，computed 就会发现`isDirty`为`true`,就会重新计算。
