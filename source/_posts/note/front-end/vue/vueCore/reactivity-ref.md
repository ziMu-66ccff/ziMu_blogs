---
title: Vue响应式模块-reactivity-ref
subtitle: ref刨析与实现
link: reactivity-ref
catalog: true
date: 2023-02-14 10:21:11
tags:
  - 前端
  - Vue
  - Vue原理刨析
  - Vue响应式模块刨析
  - ref
categories:
  - [笔记, 前端, Vue, Vue原理实现]
---

# 文档介绍

接受一个内部值，返回一个响应式的、可更改的 ref 对象，此对象只有一个指向其内部值的属性 .value。

# 实现原理

1. 因为返回的是一个 ref 对象，所以需要一个`RefImpl`类，在类里面进行`get value`和`set value`操作
2. `get value`里进行依赖的收集，`set value`里进行依赖的触发

# 完整代码

```typescript
import { hasChanged, isObject } from '../utils/index';
import { track, trigger } from './effect';
import { reactive } from './reactive';

export function ref(value: any) {
  if (isRef(value)) return value;
  return new RefImpl(value);
}

export function isRef(value: any) {
  return !!(value && value.__isRef);
}

class RefImpl {
  _value: any;
  __isRef: boolean;

  constructor(value: any) {
    this._value = convert(value);
    this.__isRef = true;
  }

  get value() {
    track(this, 'value');
    return this._value;
  }
  set value(newValue) {
    if (hasChanged(this._value, newValue)) {
      this._value = newValue;
      trigger(this, 'value');
    }
  }
}

function convert(value: any) {
  return isObject(value) ? reactive(value) : value;
}
```

# 代码执行过程梳理

1. 判断传递过来的是不是一个`ref`，是的话则直接返回，不是的话，则调用`RefImpl`来实列化，返回一个 ref 对象
2. 实列化的过程中，constructor 构造函数会调用`convert`对 value 进行处理，判断是不是对象，是对象则调用`reactive`将对象变成响应式的，不是则直接返回，将标志变量`isRef`设置为`true`。
3. 读取 ref 对象的 value 时则触发`get value`，进行依赖的收集。
4. 修改 ref 对象的 value 时则触发`set value`, 进行依赖的触发。
