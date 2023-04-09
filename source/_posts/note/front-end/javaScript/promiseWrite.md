---
title: 手写Promise及其相关方法
subtitle: Promise及其相关方法实现和讲解
link: promiseWrite
catalog: true
date: 2023-02-16 16:02:20
tags:
  - 前端
  - JS高级
  - Promise
categories:
  - [笔记, 前端, JavaScript]
---

# 手写 promise

## 完整代码

```javascript
const PENDING = Symbol('pending');
const FULFILL = Symbol('resolve');
const REJECT = Symbol('reject');

class myPromise {
  constructor(executor) {
    /* 初始化 promise pending状态*/
    this.status = PENDING;
    /* 记录当前 promise 兑现值和拒因*/
    this.value = undefined;
    this.reason = undefined;
    /* 
      记录当前 promise 的fulfilled与rejected回调
      使用数组是因为一个 promise 可以接受多次then,catch回调，否则只需各存储一个函数即可
     */
    this.onFulFilledFns = [];
    this.onRejectedFns = [];

    const resolve = (value) => {
      /* 
        此处存在两次判断 promise 状态，分别是执行executor和执行微任务时
        保证 promise 状态一经确定不再改变
      */
      if (this.status === PENDING) {
        /* resolve方法处理 promise 和 thenable 特殊参数 */
        if (
          value instanceof myPromise ||
          (typeof value === 'object' && typeof value.then === 'function')
        ) {
          return value.then(resolve, reject);
        }

        /* 使用微任务API，加入到微任务栈中 */
        queueMicrotask(() => {
          if (this.status === PENDING) {
            this.status = FULFILL;
            this.value = value;
            /* 执行fulfilled回调 */
            this.onFulFilledFns.forEach((fn) => {
              fn(this.value);
            });
          }
        });
      }
    };

    const reject = (reason) => {
      if (this.status === PENDING) {
        queueMicrotask(() => {
          if (this.status === PENDING) {
            this.status = REJECT;
            this.reason = reason;
            this.onRejectedFns.forEach((fn) => {
              fn(this.reason);
            });
          }
        });
      }
    };

    /* 此处捕获executor执行中抛出的错误，若存在错误则 promise 变成拒绝状态 */
    try {
      executor(resolve, reject);
    } catch (err) {
      reject(err);
    }
  }

  then(onFulFilledFn, onRejectedFn) {
    /* 根据Promise A+标准，若回调不为函数类型则忽视，默认值回调会将状态继续传入下一个 promise 中 */
    onFulFilledFn =
      typeof onFulFilledFn === 'function' ? onFulFilledFn : (res) => res;
    onRejectedFn =
      typeof onRejectedFn === 'function'
        ? onRejectedFn
        : (err) => {
            throw err;
          };

    /* then方法返回值是一个新 promise */
    return new myPromise((resolve, reject) => {
      /* 若捕获当前 promise 已经改变状态，则直接调用回调 */
      if (this.status === FULFILL) {
        /* 捕获fulfilled回调错误，捕获到则改变新promise状态为拒绝状态 */
        try {
          let result = onFulFilledFn(this.value);
          /* 返回值传递给下一个fulfilled回调 */
          resolve(result);
        } catch (err) {
          reject(err);
        }
      }

      if (this.status === REJECT) {
        try {
          let result = onRejectedFn(this.reason);
          /* 返回值传递给下一个fulfilled回调 */
          resolve(result);
        } catch (err) {
          reject(err);
        }
      }

      /* 若当前 promise 没改变状态/改变状态的异步还未执行，先储存回调 */
      if (this.status === PENDING) {
        this.onFulFilledFns.push(() => {
          try {
            let result = onFulFilledFn(this.value);
            resolve(result);
          } catch (err) {
            reject(err);
          }
        });
        this.onRejectedFns.push(() => {
          try {
            let result = onRejectedFn(this.reason);
            resolve(result);
          } catch (err) {
            reject(err);
          }
        });
      }
    });
  }

  catch(onRejectedFn) {
    /* catch方法类似then方法的语法糖 */
    return this.then(undefined, onRejectedFn);
  }

  finally(onFinallyFn) {
    /* finally无任何影响，仅单独调用，且无任何参数 */
    return this.then(
      (res) => {
        onFinallyFn();
        return res;
      },
      (err) => {
        onFinallyFn();
        throw err;
      }
    );
  }
}
```

## 构造函数 constructor 讲解

- 初始化相关属性

  1. 初始化状态 status

     - `this.status = PENDING`将状态初始化为`PENDING`

  2. 保存 promise 成功时的要传递的值 value 和失败时的原因 reason

     - `this.value = undefined`, `this.reason = undefind`初始化`value` and `reason`用来保存成功时要传递的值，和失败时要传递的原因

  3. 设计 onFullfilledFns, onRejectedFns 两个数组

     - `this.onFullfilledFns = []`, `this,onRejectedFns = []`，初始化`onFullfilledFns`,`onRejectedFns`来存储成功时和失败时要触发的`callback`。

- 设计 resolve 函数

  1. 判断当前 promise 的状态

     - `if (this.status === PENDING)` 判断当前 promise 的状态是否为`PENDING`
     - 是的话，则进行接下来的操做
     - 不是的话，则什么都不做（因为 promise 的状态一旦确定就不能再更改。）

  2. 判断 value 是否为 promise or thenabale

     - `if (value instanceof myPromise || typeof value.then === 'function')`判断 value 是否为 promise or thenable。
     - 是的话，`return value.then(resolve, reject)`则调用 value 的`then`方法并将`resolve`和`reject`作为参数传递， 并结束 resolve 函数的调用，不再进行接下来的操作。这样当 value 的状态发生改变时就会触发`resolve` or `reject`来修改现在这个 promise 的状态。（因为当`resolve`的`value`为 promise or thenable 的时候，当前 promise 的状态将随 value 的状态改变而改变，并保持一致）。
     - 不是的话，则进行接下来的操作

  3. 将 promise 状态的更改，以及 promise 状态更改时要触发的响应的回调函数的触发操作，放进微任务里面

     - 调用`queueMicrotask()`将接下来的操作注册为微任务放进微任务队列
     - `this.status === PENDING`判断当前 promise 的状态是否为 PENDING，是的话进行接下来的操作，不是的话，则不做接下来的操作（这样做也是为了保证 promise 的状态一旦确定就不再改变）
     - `this.status = FULLFILL`, `this.value = value`, 将 promise 的状态修改为成功，并将成功的值 value 赋值给 promise.value。
     - `this.onFullfilledFns.forEach((fn) => fn(this.value))`遍历`onFullfilledFns`，将其中存储的成功时的`callback`全部调用，并将`this.value`作为参数赋值

- 设计 rejeted 函数

  1. 判断当前 promise 的状态

     - `if (this.status === PENDING)` 判断当前 promise 的状态是否为`PENDING`
     - 是的话，则进行接下来的操作
     - 不是的话，则什么都不做（因为 promise 的状态一旦确定就不能再更改。）

  2. 将 promise 状态的更改，以及 promise 状态更改时要触发的响应的回调函数的触发操作，放进微任务里面

     - 调用`queueMicrotask()`将接下来的操作注册为微任务放进微任务队列
     - `this.status === PENDING`判断当前 promise 的状态是否为 PENDING，是的话进行接下来的操作，不是的话，则不做接下来的操作（这样做也是为了保证 promise 的状态一旦确定就不再改变）
     - `this.status = REJECT`, `this.reason = reason`, 将 promise 的状态修改为失败，并将成功的值 reason 赋值给 promise.reason。
     - `this.onRejectedFns.forEach((fn) => fn(this.reason))`遍历`onRejectedFns`，将其中存储的成功时的`callback`全部调用，并将`this.reason`作为参数赋值

- 以 try-catch 的形式调用传递来的 executor 函数, 并捕获错误
  1. 在 try 里面调用`executor`，并将`resolve`和`reject`作为参数传递
  2. 在 catch 里面捕获错误，一点捕获到错误，立刻调用`reject`(这样做是因为 executor 执行器函数在执行的时候，一旦发生错误，promise 的状态将马上变成失败)

## then 方法讲解

1. 判断 onFullfilledFn, onRejectedFn 类型

- `onFullfilledFn = typeof onFullfilled === 'function ? onFullfilled : (res) => res`判断`onFullfilledFn`是否为一个函数，当不为函数时，则默认将其赋值为(res) => res(这样做时因为根据 Promise A+标准，若回调不为函数类型则忽视，默认值回调会将状态继续传入下一个 promise 中)。
- `onRejectedFn`的处理同上

2. new 一个 promise 返回，并在 executor 中做如下操作

3. 判断调用 then 方法的 promise 状态

- 状态为`FULLFILL`时

  1. 将接下来的操作放到 try-catch 中
  2. 在 try 中调用`onFullfilledFn`并将调用`then`方法的 promise 的`value`作为参数传递，将返回值赋值给`result`， 调用`resolve`并将`result`作为参数传递，来修改要返回的 promise 的状态。
  3. 在 catch 中捕获错误，一旦捕获到错误，马上调用`reject`将要返回的 promise 修改为失败（之所以这样做是因为 onFullfilledFn, onRejectedFn 执行中发生了错误，则会立即返回一个错误的 promise）

- 状态为`REJECT`时

  1. 将接下来的操作放到 try-catch 中
  2. 在 try 中调用`onRejectedFn`并将调用`then`方法的 promise 的`reason`作为参数传递，将返回值赋值给`result`， 调用`resolve`并将`result`作为参数传递，来修改要返回的 promise 的状态。
  3. 在 catch 中捕获错误，一旦捕获到错误，马上调用`reject`将要返回的 promise 修改为失败（之所以这样做是因为 onFullfilledFn, onRejectedFn 执行中发生了错误，则会立即返回一个错误的 promise）

- 状态为`PENDING`时
  将`FULLFILL`, `REJECT`时要进行的操作，分别封装到两个箭头函数里面（必须用箭头函数，这样`this`才能指向调用`then`方法的 promise），在分别添加到调用`then`方法的 promise 的`onFullfilledFns` and `onRejectedFns`中。

# 手写 Promise.all 方法

## 完整代码

```javascript
function PromiseAll(promises) {
  return new Promise((resolve, reject) => {
    // 没有传入参数 or 传入的参数内部没有迭代器的时候会返回一个错误的promise
    if (promises == null || typeof promises[Symbol.iterator] !== 'function') {
      throw new Error('argument is not iterator');
    }

    // 将迭代器对象转化为数组
    promises = [...promises];
    // 处理空数组的情况
    if (promises.length === 0) resolve([]);

    let resolvedCount = 0;
    let promiseNum = promises.length;
    const values = [];

    promises.forEach((promise, index) => {
      // 判断数组元素是否为promise
      if (!promise instanceof Promise) {
        promise = Promise.resolve(promise);
      }
      promise
        .then((res) => {
          resolvedCount++;
          values[index] = res;
          if (resolvedCount === promiseNum) resolve(values);
        })
        .catch((err) => {
          reject(err);
        });
    });
  });
}
```

## Promise.all 方法实现讲解

- 返回一个 promise, 在 executor 里进行接下来的操作
- 判断传递的参数`promises`是否可迭代, 参数是否为空
  - `if (promises == null || typeof promises[Symbol.iterator] !== 'function')`进行判断
  - 如果参数为空 or 参数不能迭代，则调用抛出一个错误，让返回的 promise 状态变为失败
  - 如果有参数并且可以迭代，则进行接下来的操作
- 将可迭代的参数转换为数组
  - `promises = [...promises]`
- 判断转换为数组后的参数是否为空,并进行相关操作
  - 如果数组为空，则调用`resolve`将一个空数组作为参数传递，让返回的 promise 状态变为成功
  - 如果数组不为空，则进行接下来的接下来的操作
- 初始化相关变量
  - `promisesNum = promises.length`保存参数数组元素的数量
  - `resolvedCount = 0` 保存已经处理过的 promise 的数量
  - `values = []` 保存处理过的 promise 的`value`
- 遍历参数数组, 并进行相关处理
  - 判断数组元素是否为 promise
    - 不是 promise 则调用`Promise.resolve()`将其转换为 promise
    - 是 promise，则直接进行接下来的操作
  - 对已经是 promise 的数组元素调用`then`方法
    - 调用`then`方法，传递 promise 成功时的`callback`，在`callback`里面将参数`res`存储进`values`，并且`resolvedCount++`计数器加一，`resolvedCount === promiseNum`判断已经处理过的数组元素的数量和数组的元素的数量，是否相等，相等时则直接调用`resolve(values)`，将返回的 promise 的状态改为成功
  - 对已经时 promise 的数组元素调用`catch`方法捕获错误
    - 一旦捕获到错误，直接调用`reject(err)`，将返回的 promise 的状态改为失败

# 手写 Promise.any 方法

## 完整代码

```javascript
function PromiseAny(promises) {
  return new Promise((resolve, reject) => {
    if (promises == null || typeof promises[Symbol.iterator] !== 'function') {
      throw new Error('arrgument is not iterator');
    }

    promises = [...promises];
    if (promises.length === 0) resolve([]);

    let rejectedCount = 0;
    let promiseNum = promises.length;
    let errs = [];

    promises.forEach((promise, index) => {
      if (!promise instanceof Promise) {
        promise = Promise.resolve(promise);
      }
      promise
        .then((res) => {
          resolve(res);
        })
        .catch((err) => {
          errs[index] = err;
          rejectedCount++;
          if (rejectedCount === promiseNum) reject(errs);
        });
    });
  });
}
```

# 手写 Promise.race 方法

## 完整代码

```javascript
function promiseRace(promises) {
  new Promise((resolve, reject) => {
    if (promises == null || typeof promises[Symbol.iterator] !== 'function') {
      throw new Error('arrgument is not iterator');
    }
    promises = [...promises];
    if (promises.length === 0) resolve([]);
    promises.forEach((promise) => {
      if (!promise instanceof Promise) {
        promise = Promise.resolve(promise);
      }
      promise
        .then((res) => {
          resolve(res);
        })
        .catch((err) => {
          reject(err);
        });
    });
  });
}
```
