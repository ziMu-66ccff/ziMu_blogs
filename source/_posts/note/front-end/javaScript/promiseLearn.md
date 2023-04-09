---
title: 理解为什么需要Promise
subtitle: 彻底搞懂Promise的前世今生
link: promiseLearn
catalog: true
date: 2023-02-12 18:23:50
tags:
  - 前端
  - JS高级
  - Promise
categories:
  - [笔记, 前端, JavaScript]
---

# 为什么需要 Promise？🤔

## 明白同步任务 and 异步任务

首先，前端 er 们需要明确一点，在 js 这种单线程的事件循环模型中，一件事情没做完是不能做下一件事情的。这时候睿智的友友们就会说这又咋了呢，会有什么问题吗？哎嘿，还真有问题，问题还大了，要是中间有一件事情耗时太长，导致后面耗时很短的事情做不了怎么办呢？你看，这不就堵车了，还会堵很久。但是我们聪明的前端 er 们马上就有了解决方法，我们先不管那个耗时很长的事情，先做后面的事情，最后再管这个耗时很长的事情不就完事了吗 😁。
对的，就是这种解决方案，敲黑板 😸，js 中代码分为同步任务和异步任务，异步任务通常为那种可能会耗时很长 or 需要触发条件的，也就是前面说的可能会堵车，还堵很久的那种。同步任务当然就是除了异步任务以外的任务啦。在 js 的执行机制中，会先执行同步任务，等同步任务执行完毕再执行异步任务。

## 早期异步操作的种种问题

早期 js 中只支持定义回调函数来表明异步操作的完成，接下来我们会以 setTimeout 这个异步操作为例子进行讲解。
试想这样的一种场景，我们需要在 setTimeout 中传递一个回调函数，在几秒后执行，这显然是一个异步操作，它有明显的触发条件，时间。这个回调函数中我们会进行时间到了后我们希望进行的操作，那么问题来了，如果我们在这个回调函数希望对外返回一个数据，以便于进一步的处理怎么办？🤔 这时候你就会发现问题，我们无法直接返回数据。好的，这是一个问题。其实还有更严重的问题，如果我们进一步的操作也是异步操作，比如也是在几秒后执行怎么办，你会想到，再使用一次 setTimeout，再传递一个回调函数。那么现在回调函数里面又嵌套了一个回调函数，如果还要进一步操作呢，还是异步操作呢，再嵌套，嵌套，嵌套，嵌套，最后你一个回调函数里面嵌套了 n 个函数，最后的场景是什么样子呢。回调地狱！！！请看下图。
![回调地狱](https://i.328888.xyz/2023/02/12/cwAbF.png)

相信前端 er 看到类似这种代码，血压是马上就会上升的，所以可不要写这种代码奥。

实际上异步操作的问题可不止这些，比如如果异步操作失败了，怎么进行后续的处理，怎么拿到失败的原因，这都是需要解决的。
那么为了解决，如何拿到返回值； 异步操作失败后如何拿到失败的原因，并进一步处理；如何避免回调地狱；Promise 应运而生 😍

# Promise 是什么？🤔

Promise 是一种异步编程机制，是对回调地狱的一种解决方案。
Promise 实列则是一种状态机，它有三种基本状态，待定（pending），兑现（fulfilled)， 拒绝(rejected)，最初的状态为 pending，状态一旦改变，就定了，就不会再发生任何改变了。

# Promise 是怎么解决早期异步操作的疑难杂症的 🤔

## 解决如何拿到返回值；当异步操作失败后如何拿到失败的原因，并进一步处理的问题

构造函数 Promise（）中可以传递一个回调函数，我们一般称这个回调函数为执行器函数，执行器函数主要有两项职责：1.初始化期约的异步行为（在执行器函数中进行我们需要执行的异步操作）和控制状态的最终转换（将 Promise 实列的状态由最初的 pending 转换为 fulfilled or rejected）。那么它是怎么来控制状态的最终转换的呢？🤔 很简单，这个执行器函数是自带两个参数的，这两个参数都为函数，我们一般称为 resolve， reject。你可以在 resolve 中传递参数，来返回你需要的数据；可以在 reject 中传递参数，来返回异步操作失败的原因。当你在执行器函数中调用 resolve，Promise 实列的状态就由 pending 变为了 fulfilled（注：这里不是十分准确，需要格外注意一种特别情况，如果返回的也是一个 Promise，那么 Promise 实列的状态将由返回的这个 Promise 决定，为这个 Promise 的状态，不一定为 fulfilled）；同理，当你调用了 reject，Promise 实列的状态就由 pending 变为了 rejected。
通过执行器函数的两个参数函数，resolve，reject 我们成功的解决了如何拿到返回值，异步操作结束后拿到失败的原因的问题。
那么我们接下来，解决怎么进一步处理的问题。这里我们就需要知道 Promise 实列上面其实是由一个 then 方法的，不过其实这个方法是定义在 Promise 实列的构造函数的原型上面的。这个 then 方法一样有两个参数函数，一般称为 onResolved, onRejected。当 promise 实列的状态为 fulfilled 的时候，就会执行第一个参数函数，即 onResolved；当 promise 实列的状态为 rejected 的时候，就会执行第二个参数函数，即 onRejected。我们可以在这两个参数函数中写入对应的进一步处理的逻辑。
至此，我们就解决了如何拿到返回值，拿到异步操作失败的时候失败的原因，并进一步处理的问题ヽ(✿ ﾟ ▽ ﾟ)ノ 😁

## 解决回调地狱的问题

这里我们需要知道 promise 实列身上的 then 方法是会返回一个 promise 实列的，返回的这个新的实列是基于 onResolved 函数 or onRejected 函数的返回值构建的，是将返回值传递给 Promise.resolve 函数来包装生成新的 promise 实列。

- 1.当触发的 onResolved or onRejected 函数没有返回值时，会默认返回 undefined 来传递给 Promise.resolve 函数来生成新的成功的 promise 实列。
- 2.当有返回值的时候会将返回值传递给 Promise.resolve 函数来生成新的成功的 promise 实列。
- 3.当返回值为一个 promise 实列的时候会将该 promise 实列传递 Promise.resolve 函数来返回该 promise 实列。
- 4.当在触发的 onResolved or onRejected 函数中抛出了一个错误时，会将该错误传递给 promise.resolve 来返回一个新的失败的 promise 实列。
- 5.当压根没有需要触发的 onResolved or onRejected 函数时，会将调用 then 方法的 Promise 实列原样返回。

由于 then 方法固定会返回 promise 实列，这就为链式调用提供了可能，而链式调用便可给嵌套解套，避免嵌套，解决回调地狱。
至此，也解决了回调地狱的问题。请看下图 😁
![解决回调地狱](https://i.328888.xyz/2023/02/12/cwD6H.png)

当然，Promise 还有很多的实列方法，这些方法就不一一讲解，读者们下去了可以自行了解。